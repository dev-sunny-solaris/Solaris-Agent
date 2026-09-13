# Mobile API

The mobile API is a first-party client surface: a mobile app talking to the same repositories the
web UI uses, over JSON. It is **not** the third-party integration API — that is
[Third-party API](third-party-api.md), a different stack with different auth.

Per module the boilerplate is: **controller → resource → config registration → routes**. Every
sub-module / detail section of a module gets its own controller and resource.

Location per package:

```
src/Http/Controllers/Mobile/{Model}Controller.php
src/Http/Resources/Mobile/{Model}Resource.php
routes/mobile.php
```

Routes are mounted by the provider under the `api` middleware with the `api` prefix, so the final
path is `/api/mobile/...`.

## Workflow before writing code

1. **Trace** — read every Blade view of the module, not just the main detail section; check all
   sections and modals.
2. **Identify** — list the columns per section: list columns (table) vs detail columns (full page).
3. **Check** — view-only or full CRUD? `new="false"` on the Blade component means view-only.
4. **Reference** — read the web controller before writing the mobile one.
5. **Implement** — controller → resource → config → routes.
6. **Review** — for a new or complex module, confirm the tracing result with the maintainer before
   implementing.

## Controller

```php
namespace Solaris\{Package}\Http\Controllers\Mobile;

use Illuminate\Http\JsonResponse;
use Illuminate\Http\Request;
use Solaris\Core\Helpers\Helper;
use Solaris\Core\Helpers\SolarResponse;
use Solaris\Core\Repositories\BaseRepository;

class {Model}Controller
{
	protected BaseRepository $repo;   // or the custom repository's type

	public function __construct()
	{
		$this->repo = new (Helper::repository('{key}'))(new (Helper::model('{key}')));
	}

	public function index(Request $request): JsonResponse
	{
		$search = $request->input('search');
		$data   = $this->repo->list(
			columns : $this->listColumn(),
			search  : !empty($this->searchColumn())
				? ['search' => $search, 'columns' => $this->searchColumn()]
				: $search,
			filter  : $request->input('filter'),
			sort    : $request->input('sort'),
			page    : $request->input('page'),
			length  : $request->input('length'),
		);

		return SolarResponse::list(Helper::mobileResouces('{key}'), $data);
	}

	public function show(string $id): {Model}Resource
	{
		$data = $this->repo->findComplete($id, $this->detailColumns());

		return new (Helper::mobileResouces('{key}'))($data);
	}

	public function store(): JsonResponse
	{
		/** @var {Model}Request */
		$request = app(Helper::formRequest('{key}'));

		return SolarResponse::success($this->repo->create($request->cleanData()));
	}

	public function update(string $id): JsonResponse
	{
		/** @var {Model}Request */
		$request = app(Helper::formRequest('{key}'));

		return SolarResponse::success($this->repo->update($id, $request->cleanData()));
	}

	public function destroy(string $id): JsonResponse
	{
		return SolarResponse::deleted($this->repo->delete($id));
	}

	public function listColumn(): array { return [/* ... */]; }
	public function detailColumns(): array { return [/* ... */]; }
	public function searchColumn(): array { return [/* optional — narrows the search */]; }
}
```

`index()` and the write actions return a `JsonResponse`; `show()` returns the resource itself.

- **View-only module:** keep `index()`, `show()`, `listColumn()`, `detailColumns()`. Drop
  `store`/`update`/`destroy`.
- **Custom repository:** if the model has one (e.g. `LeadRepository`), type the property with it
  instead of `BaseRepository`. The class still comes from `Helper::repository()`, never `new` on a
  concrete class.
- **Child package extension:** subclass the ancestor's mobile controller and widen the column set.

  ```php
  class AccountController extends \Solaris\{ParentPackage}\Http\Controllers\Mobile\AccountController
  {
  	public function detailColumns(): array
  	{
  		return array_merge(parent::detailColumns(), ['extra_column']);
  	}
  }
  ```

  Extension hierarchy follows the package's `require` chain up to Core.

## Resource

```php
namespace Solaris\{Package}\Http\Resources\Mobile;

use Illuminate\Http\Request;
use Solaris\Core\Http\Resources\BaseResource;

class {Model}Resource extends BaseResource
{
	protected function list(Request $request): array { return [/* ... */]; }

	protected function detail(Request $request): array { return [/* ... */]; }
}
```

A child package overrides `detail()` and merges onto `parent::detail($request)`.

`LookupResource` (`Solaris\Core\Helpers\LookupResource`) shapes the non-scalar fields
consistently:

| Field type | Helper |
|---|---|
| Related model (foreign key) | `LookupResource::from($this->relation)` |
| Status / stage field | `LookupResource::lookupStage($this->status)` |
| User (created/updated by) | `LookupResource::user($this->created_by)` |
| Image / avatar | `LookupResource::image($this->avatar)` |

## Config registration

In the package's `config/solaris.php`:

```php
'mobile' => [
	'controllers' => [
		'{key}' => 'Solaris\\{Package}\\Http\\Controllers\\Mobile\\{Model}Controller',
	],
	'resources' => [
		'{key}' => 'Solaris\\{Package}\\Http\\Resources\\Mobile\\{Model}Resource',
	],
],

// only if not already registered
'repositories' => [
	'{key}' => 'Solaris\\Core\\Repositories\\BaseRepository',
],
```

Resolved with `Helper::mobileController('{key}')` and `Helper::mobileResouces('{key}')`.

## Routes

Auth: the public endpoints (login, 2FA verify/resend, refresh, forgot password) sit outside the
guard; everything else is inside `Route::middleware('auth.mobile')`. Token TTLs come from
`auth.mobile.access_token_ttl` / `refresh_token_ttl` in config.

Route file structure: a `mobile` prefix, then `Route::middleware('auth.mobile')`, then the scope
prefix (the package key, e.g. `inventory`), then one group per resource. Route name pattern:
`mobile.{scope}.{resource}.{action}` — e.g. `mobile.inventory.supplier.store`. The list route
carries the bare `mobile.{scope}.{resource}` name, with no action suffix.

```php
Route::prefix('{slug}')->group(function () {
	$controller = Helper::mobileController('{key}');

	Route::post('/list', [$controller, 'index'])
		->name('mobile.{scope}.{resource}');

	Route::get('/{id}', [$controller, 'show'])
		->whereUuid('id')
		->name('mobile.{scope}.{resource}.read');

	Route::post('/', [$controller, 'store'])
		->name('mobile.{scope}.{resource}.store');

	Route::put('/{id}', [$controller, 'update'])
		->name('mobile.{scope}.{resource}.update');

	Route::delete('/{id}', [$controller, 'destroy'])
		->whereUuid('id')
		->name('mobile.{scope}.{resource}.destroy');
});
```

View-only modules keep only the `list` and `read` routes. Sub-modules nest inside the parent
prefix (`account/bank`, `account/status-history`).

## Key rules

- The list endpoint is **`POST /list`**, not `GET /` — the filter payload goes in the body.
- The `/** @var {Model}Request */` docblock above `app(Helper::formRequest(...))` is required;
  without it static analysis cannot see the request type.
- `BaseRepository` for simple sub-models; a custom repository for complex ones.
- Status/stage history sub-modules (auto-generated by a trait) are always view-only.
- Models that must never be exposed through the generic mobile lookup go in core's
  `lookup_management.mobile_exceptions` list.
