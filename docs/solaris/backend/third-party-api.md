# Third-Party Integration API

A separate surface from the [Mobile API](mobile-api.md): a versioned,
API-key authenticated, self-documenting REST API for external partners. Core provides the base
controller, the middleware stack and the docs generator; each package declares its own entities.

Location per package:

```
src/Http/Controllers/Api/{Model}Controller.php
routes/api.php
database/seeders/ApiPermissionSeeder.php
```

Core has no third-party entity of its own — it only ships the machinery.

## Middleware stack

```php
Route::prefix('v1')
    ->middleware(['api.log', 'auth.api_key', 'throttle:api_key'])
    ->group(function () { /* entities */ });
```

**Order matters.** `api.log` wraps auth so rejected calls are still audited, and throttling only
means something once the key is known. Rate-limit tiers (`low` / `standard` / `high`, requests per
minute) live under `auth.api.rate_limit` in config; an API key row stores the tier name, not the
number.

Per-route: a `permission:api.{entity}.{action}` middleware on every route, plus `api.idempotency`
on `POST` so a retried create cannot duplicate a record (client sends an `Idempotency-Key` header).

## Controller

Extend `BaseApiController`. The controller is a **declaration of the public contract**, not a set
of handlers — `index`, `search`, `show`, `store`, `update`, `replace` and `destroy` are implemented
by the base.

```php
namespace Solaris\{Package}\Http\Controllers\Api;

use Solaris\Core\Docs\Api\ApiEntity;
use Solaris\Core\Docs\Api\ApiField;
use Solaris\Core\Docs\Api\ApiOperation;
use Solaris\Core\Http\Controllers\Api\BaseApiController;

class AccountController extends BaseApiController
{
    public function key(): string { return 'account'; }

    public function columns(): array { return ['id', 'name', /* ... */]; }

    public function relations(): array { return ['type', 'industry']; }

    public function searchColumns(): array { return ['name', 'primary_phone', 'email']; }

    public function entityDocs(): ApiEntity
    {
        return ApiEntity::make('Account')
            ->description('Companies and organisations you do business with.');
    }

    public function fieldDocs(): array
    {
        return [
            'id'   => ApiField::uuid('Unique identifier.')->readOnly(),
            'name' => ApiField::string('Registered company name.')->example('PT Sinar Jaya Abadi'),
            'type' => ApiField::lookup('Account type, e.g. customer or prospect.'),
        ];
    }

    public function endpointDocs(): array
    {
        return [
            'index'   => ApiOperation::make('List accounts'),
            'search'  => ApiOperation::make('Search accounts')->description('...'),
            'show'    => ApiOperation::make('Get one account'),
            'store'   => ApiOperation::make('Create an account'),
            'update'  => ApiOperation::make('Update an account'),
            'replace' => ApiOperation::make('Update an account (full)'),
            'destroy' => ApiOperation::make('Delete an account'),
        ];
    }
}
```

Overridable surface: `key()`, `columns()`, `guarded()`, `relations()`, `searchColumns()`,
`includes()`, `actionRules()`, plus the three docs methods.

**`columns()` is deliberately explicit, never derived from the table.** Internal columns must not
leak, and a column added later must be an opt-in decision rather than something a partner discovers
on its own.

Semantics the base implements, which the docs must not contradict:

- `PATCH` (`update`) is a partial update — omitted fields are untouched, an explicit `null` clears.
  Its rules come from the FormRequest's `patchRules()`.
- `PUT` (`replace`) requires every required field to be present (`apiUpdateRules()`), but still
  does not blank the fields you omit.
- `POST` (`store`) uses `apiCreateRules()` — every writable field at once, since the API has no
  two-step create modal.
- `POST /search` takes the full Filter DSL in the body; `GET /` takes the query-string form.
- Lookup fields accept either an id (`industry_id`) or a name (`industry`); a name matching more
  than one record is rejected rather than guessed.

## Routes

```php
Route::prefix('accounts')->group(function () {
    $controller = Helper::apiController('account');

    Route::get('/', [$controller, 'index'])
        ->middleware('permission:api.account.view')
        ->name('api.v1.account.index');

    Route::post('/search', [$controller, 'search'])
        ->middleware('permission:api.account.view')
        ->name('api.v1.account.search');

    Route::post('/', [$controller, 'store'])
        ->middleware(['permission:api.account.create', 'api.idempotency'])
        ->name('api.v1.account.store');

    Route::get('/{id}', [$controller, 'show'])->whereUuid('id')
        ->middleware('permission:api.account.view')
        ->name('api.v1.account.show');

    Route::patch('/{id}', [$controller, 'update'])->whereUuid('id')
        ->middleware('permission:api.account.update')
        ->name('api.v1.account.update');

    Route::put('/{id}', [$controller, 'replace'])->whereUuid('id')
        ->middleware('permission:api.account.update')
        ->name('api.v1.account.replace');

    Route::delete('/{id}', [$controller, 'destroy'])->whereUuid('id')
        ->middleware('permission:api.account.delete')
        ->name('api.v1.account.destroy');
});
```

Naming: `api.v1.{entity}.{action}`. Plural, dashed URL segments.

## Config registration

```php
'api' => [
    'controllers' => [
        'account' => 'Solaris\\{Package}\\Http\\Controllers\\Api\\AccountController',
    ],
    'resources' => [],
],
```

Resolved with `Helper::apiController('{key}')`.

The docs page is generated from these declarations by `OpenApiGenerator`, which pulls the request
schema straight from the entity's FormRequest (`apiCreateRules()` for `store`, `patchRules()` for
the rest) — see [PHP standards](php-standards.md). A rule and its documented field cannot
drift apart, so do not restate validation in `fieldDocs()`.

`php artisan solaris:api-docs` builds the OpenAPI document (written to local storage) that the
`api.docs.route` endpoint serves through Swagger UI. It scans `api.v1.*` routes, so an entity
whose routes are missing simply does not appear. It also prints a **drift report** — documentation
gaps such as a field with no description. Drift is a warning, never fatal; run the command after
adding or changing an entity and clear what it reports.

Core also owns the `api.docs` block — enable flag, route, title, description, servers and the
Swagger UI source. `servers` is intentionally empty so the docs follow whichever host is being
browsed and no production URL is baked into a "Try it out" button.

## Permissions

**Without permission rows the middleware rejects every call and the API is unreachable.** Each
package ships an `ApiPermissionSeeder` that creates one role per entity (`API - Account`) holding
that entity's four permissions:

```
api.{entity}.view  api.{entity}.create  api.{entity}.update  api.{entity}.delete
```

A partner integration is granted the role, not individual permissions. Permission group:
`integration`.

## Checklist for a new API entity

- [ ] Controller extends `BaseApiController`; `key()`, `columns()`, `relations()`,
      `searchColumns()` declared; internal columns excluded
- [ ] `entityDocs()`, `fieldDocs()` (every column, with `readOnly()` and `example()` where useful),
      `endpointDocs()` filled — the docs page is generated from these
- [ ] Routes under `v1` with the standard middleware stack, `permission:` per route,
      `api.idempotency` on `POST`
- [ ] Registered in `api.controllers`
- [ ] `ApiPermissionSeeder` extended with the entity's role and four permissions
