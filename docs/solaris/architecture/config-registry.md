# The Config Registry

This is the single most important package-level concept. **Solaris never hardcodes a class
reference across a boundary.** Controllers, models, repositories, form requests, views, AI agents
and AI tools are all resolved through config, so any downstream package or consumer app can swap an
implementation without touching source.

Resolution order: the app's `config/sunny.php` wins over the merged `config/solaris.php`. Scalars
take the first value found; arrays are merged.

```php
Helper::config('some.key', $default);   // always use this, never Laravel's config()
Helper::model('account');               // → config solaris.model.models.account
Helper::controller('account');          // → config solaris.controller.controllers.account
Helper::repository('account');
Helper::view('account');                // → ['list' => 'master-data::pages.account.list', ...]
Helper::aiAgent($slug);
Helper::aiTool($slug);
```

Full resolver set:

| Helper | Config key |
|---|---|
| `Helper::config($key, $default)` | any key, `sunny` then `solaris` |
| `Helper::model($key)` | `model.models.*` |
| `Helper::controller($key)` | `controller.controllers.*` |
| `Helper::repository($key)` | `repositories.*` |
| `Helper::formRequest($key)` | `form_requests.*` |
| `Helper::view($key)` | `resource.views.*` |
| `Helper::mobileController($key)` | `mobile.controllers.*` |
| `Helper::mobileResouces($key)` | `mobile.resources.*` |
| `Helper::apiController($key)` | `api.controllers.*` |
| `Helper::aiAgent($slug)` / `Helper::aiTool($slug)` | `ai.agents.*` / `ai.tools.*` |

`mobileResouces` is spelled that way in the source. Use it as-is; do not "fix" it in a call site.

`SolarHelper` is a global class alias of `Helper` for use in Blade (`SolarAsset` and `SolarNumber`
are aliased the same way).

## Rules

- **Never** call Laravel's `config('solaris.x')` directly for a Solaris key — it bypasses the
  `sunny.php` override layer. Always `Helper::config()` / `SolarHelper::config()`.
- Routes reference controllers through the registry, not by class name:

  ```php
  Route::prefix('account')->group(function () {
      $controller = Helper::controller('account');

      Route::get('/', [$controller, 'index'])->name('master-data.account');
      Route::get('/{id}', [$controller, 'show'])->whereUuid('id')->name('master-data.account.read');
      Route::get('/edit/{id}', [$controller, 'edit'])->whereUuid('id')->name('master-data.account.edit');
  });
  ```
- Every new controller, model, repository, form request, view and AI agent/tool a package adds
  **must** be registered in that package's `config/solaris.php`. An unregistered class is not
  overridable and is therefore a defect.
- Config keys are part of the public API. Renaming or removing one is a breaking change.
- Do not re-derive a mapping in PHP (an attribute, a naming convention, a scan) once it has been
  made explicit in config or in the database. That is hardcoding it again.
