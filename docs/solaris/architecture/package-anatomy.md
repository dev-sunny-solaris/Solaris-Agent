# Anatomy of a Solaris Package

```
composer.json                 name, version, PSR-4 autoload, provider under extra.laravel
config/solaris.php            package defaults, merged into the `solaris` config key
database/migrations/          auto-discovered, auto-run
database/seeders/             menu, permissions, demo data
resources/views/              Blade — pages/ and components/
resources/js/                 page JS and (core only) the SolarUI runtime
resources/css/                stylesheets (only when the package ships its own)
routes/web.php                web routes
routes/api.php                third-party API routes
routes/mobile.php             mobile client routes
src/                          PHP source, PSR-4 root
src/<Name>ServiceProvider.php Spatie PackageServiceProvider
stubs/                        (core only) generator templates for solaris:make
tests/                        Pest + Orchestra Testbench
jsconfig.json                 JS path aliases for the editor
```

## Service provider

Every package uses `spatie/laravel-package-tools`:

```php
class SolarisMasterDataServiceProvider extends PackageServiceProvider
{
    public function configurePackage(Package $package): void
    {
        $package
            ->name('master-data')      // view namespace: master-data::
            ->hasConfigFile(['solaris'])
            ->hasViews('master-data')
            ->discoversMigrations()
            ->runsMigrations();
    }

    public function packageBooted()
    {
        Route::middleware('web')->group(__DIR__.'/../routes/web.php');

        Route::middleware('api')->prefix('api')->group([
            __DIR__.'/../routes/api.php',
            __DIR__.'/../routes/mobile.php',
        ]);
    }
}
```

Register commands with `->hasCommands([...])`. Register class aliases, singletons, macros and
runtime config defaults in `registeringPackage()`. Keep the provider declarative — no business
logic in it.

## Asset aliases

Each package exposes its own JS/CSS alias, resolved in the consumer app's `jsconfig.json` and Vite
config:

| Package | View | JS | CSS |
|---|---|---|---|
| core | `core::` | `@core-js/*` | `@core-css/*` |
| masterdata | `master-data::` | `@master-data-js/*` | `@master-data-css/*` |
| basecrm | `base-crm::` | `@base-crm-js/*` | `@base-crm-css/*` |
| sales | `sales::` | `@sales-js/*` | `@sales-css/*` |
| marketing | `marketing::` | `@marketing-js/*` | `@marketing-css/*` |
| service | `service::` | `@service-js/*` | `@service-css/*` |
| crm | `crm::` | `@crm-js/*` | `@crm-css/*` |

See [Package registry](../package-registry.md) for Composer names, PHP namespaces, owner
terms, and environment-specific source lookup.

**Inside a package, its own files import via its own alias, not a relative path**, and imports from
an ancestor package use that ancestor's alias:

```js
// inside core
import LayoutMenu from '@core-js/layouts/layout-menu'

// inside masterdata, pulling from core
import SolarListPage from '@core-js/solaris/solar/solar-list-page'
```

Blade and PHP resolve physical asset paths through the config-driven helper, never a hardcoded
`vendor/...` string:

```php
SolarAsset::js('pages/account/list.js', 'master-data');
SolarAsset::css('app.css');           // defaults to core
SolarAsset::image('logo.png');
```

The base path comes from `resource.path.<package>` in the package's `config/solaris.php`.
