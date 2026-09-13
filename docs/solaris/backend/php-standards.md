# PHP Layer Standards

## Controllers

Plain classes — they do **not** extend a base controller. They are dispatched by the gateways.

Types: `full` (index/show/edit/store/update/destroy/batchDelete — a top-level module),
`detail` (store/show/update/destroy/batchDelete — a sub-resource rendered inside a parent's edit
page; `show` always returns JSON, no index/edit), `blank` (custom, non-CRUD).

- Declare `#[FormAccess()]` on the class.
- Inject the repository in the constructor.
- Use `$request->cleanData()` in `store`/`update` — never `$request->all()`.
- `edit()` returns the detail view with data from `findComplete()`.
- `show()` returns `findComplete()` as JSON for JSON requests, otherwise redirects to the edit route.
- No domain logic. Delegate everything to the repository.

## Models

Extend `BaseModel` or a specialized base. All configuration is declared at class level.

Types: `base`, `detail` (pre-wires a `BelongsTo` to its parent), `lookup` (extends `LookupModel`,
already carries `LookupAccess` / `FolderAccess` / `ExportAccess` / `ImportAccess`), `lookup-stage`
(lookup driving a pipeline), `stage-history` (extends `StageHistoryModel`; must define
`parentModel()`, `parentColumn()`, `stageModel()`, `stageColumn()`), `none` (plain Laravel model).

Key properties: `$table`, `$displayValue` (field shown in dropdowns), `$prepareColumns` (columns
exposed by `prepared()`), `$searchColumns`, `$orderBy`, `$direction`.

- Remove any attribute that does not apply — do not leave inert declarations.
- When `ImportAccess` is declared, implement `importRules(array $row): array`.
- Every scope named in `DataTableAccess(scopes: [...])` or `LookupAccess(scopes: [...])` must exist.

### Static Lookup UUID constants

When application logic identifies stable, seeded Lookup rows by UUID, declare those identifiers as
public constants in a dedicated `<LookupName>Const` class under the owning package's `src/Const/`.
Do not repeat UUID literals in models, repositories, controllers, seeders, or other server-side logic:

```php
namespace Solaris\Inventory\Const;

class ReceiptStatusConst
{
	public const New                = '019b5a10-0000-7005-8000-000000000001';
	public const InProgress         = '019b5a10-0000-7005-8000-000000000002';
	public const WaitingForApproval = '019b5a10-0000-7005-8000-000000000003';
	public const Received           = '019b5a10-0000-7005-8000-000000000004';
	public const Closed             = '019b5a10-0000-7005-8000-000000000005';
}
```

Seeders and backend logic reference `ReceiptStatusConst::New` and the other named constants. If frontend
logic needs the same identifiers, mirror the keys and UUID values in `resources/js/const.js` as
`ReceiptStatus`; see [JavaScript conventions](../frontend/javascript.md). The backend Const class remains
canonical. This pattern is only for stable static Lookup records.

## Repositories

A repository is a mini-service bound to one model. All domain logic, transformation and CRUD live
here. Nothing queries a model directly from a controller.

```php
class ProductRepository extends BaseRepository
{
	public function __construct(Product $model)
	{
		parent::__construct($model);
	}

	public function findComplete(string $id, array $columns = [])
	{
		return parent::findComplete($id, $columns);
	}
}
```

Lifecycle hooks — all `protected`; implement only the ones you need, delete the rest:

| Hook | Purpose | Call `parent::`? |
|---|---|---|
| `creating(array $data): array` | transform before insert | yes |
| `created(Model $record, array $data, array $rawData)` | side effects after insert | yes |
| `updating(Model $record, array $data): array` | transform before update | yes |
| `updated(Model $record, array $data, array $rawData)` | side effects after update | yes |
| `deleting(Model $record)` / `deleted(Model $record)` | delete side effects | no |
| `batchDeleting(array $ids)` / `batchDeleted(array $ids)` | batch delete side effects | no |

Public surface: `all`, `list`, `find`, `findComplete`, `create`, `update`, `delete`,
`batchDelete`, `requestBatchDelete`, `getModel`.

```php
public function list(
	?array $columns = null,
	string|array|null $search = null,
	array|Filter|null $filter = null,
	string|array|null $sort = null,
	int $page = 1,
	int $length = 10
): LengthAwarePaginator
```

- `create()` and `update()` also accept `?Closure $before` and `?Closure $after` — use these for
  one-off orchestration at the call site instead of adding a hook that only one caller needs.
- **All CRUD is already wrapped in a DB transaction** (the constructor's second argument turns that
  off). Never add another transaction wrapper around a repository call unless you are coordinating
  DB work that happens outside the hooks.

## Form requests

Extend `BaseFormRequest`.

A simple, web-only request defines `rules()` and nothing else. A module that is also exposed to the
third-party API splits the rule set instead, and `rules()` becomes a dispatcher:

```php
class SupplierRequest extends BaseFormRequest
{
	public function rules(): array
	{
		if ($this->routeIs('inventory.supplier.update.avatar')) {
			return ['profile_picture' => $this->pictureValidation()];
		}

		return match ($this->method()) {
			'POST'  => $this->is('api/v1/*') ? static::apiCreateRules() : static::createRules(),
			'PUT'   => static::updateRules(),
			'PATCH' => static::patchRules(),
		};
	}

	public static function createRules(): array { /* ... */ }

	public static function updateRules(): array
	{
		return array_merge(static::createRules(), [/* fields only on the edit page */]);
	}
}
```

- `createRules()` mirrors the **web create modal**, which is deliberately short — the remaining
  fields appear only on the edit page. `updateRules()` merges onto it with the rest.
- **Do not widen `createRules()` to make an API or AI case work.** AI tools use it as their
  parameter schema, so every field added there lands in the model prompt. That is exactly why the
  API variants exist separately.

Derived rule sets provided by the base — override only when the note says so:

| Method | What it returns |
|---|---|
| `apiCreateRules()` | every writable field at once (the API has no two-step create), minus `apiExcluded()`; requiredness stays as `createRules()` declared it |
| `apiUpdateRules()` | `updateRules()` minus `apiExcluded()` |
| `patchRules()` | `updateRules()` relaxed — `required` dropped, `sometimes` prepended, minus `apiExcluded()`. A rule spanning two fields cannot survive that relaxation; such a request class overrides this and states the condition itself |
| `apiExcluded()` | fields in the web rule set but outside the API contract — empty by default, an entity opts out explicitly |

Other rules:

- Do not override `authorize()` — the base always returns `true`.
- Do not override `failedValidation()` — the base returns the JSON error shape.
- `cleanData()` returns the input filtered to the keys present in `rules()` for the current request.
- For a detail resource, require the parent foreign key on `POST` only.
- `pictureValidation(bool $required = false, int $min = 1, int $max = 1024)` is a `protected`
  helper for building the image rule inside `rules()`.
- Register the class in `form_requests` so `Helper::formRequest($key)` can resolve it — the mobile
  controllers instantiate it that way.
