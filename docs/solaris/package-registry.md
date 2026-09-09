# Solaris Package Registry and Source Lookup

This is the canonical identity map for Solaris packages. Documentation references package owners
and source-relative paths; never treat a repository folder name or absolute path as portable.

## Package identities

| Owner | Key | Composer package | PHP namespace | Blade | JS | CSS |
|---|---|---|---|---|---|---|
| Core | `core` | `solaris/solaris-laravel-core` | `Solaris\Core` | `core::` | `@core-js/*` | `@core-css/*` |
| MasterData | `master-data` | `solaris/solaris-laravel-masterdata` | `Solaris\MasterData` | `master-data::` | `@master-data-js/*` | `@master-data-css/*` |
| BaseCRM | `base-crm` | `solaris/solaris-laravel-basecrm` | `Solaris\BaseCRM` | `base-crm::` | `@base-crm-js/*` | `@base-crm-css/*` |
| Sales | `sales` | `solaris/solaris-laravel-sales` | `Solaris\Sales` | `sales::` | `@sales-js/*` | `@sales-css/*` |
| Marketing | `marketing` | `solaris/solaris-laravel-marketing` | `Solaris\Marketing` | `marketing::` | `@marketing-js/*` | `@marketing-css/*` |
| Service | `service` | `solaris/solaris-laravel-service` | `Solaris\Service` | `service::` | `@service-js/*` | `@service-css/*` |
| CRM | `crm` | `solaris/solaris-laravel-crm` | `Solaris\CRM` | `crm::` | `@crm-js/*` | `@crm-css/*` |

The key is used by Solaris-Kit, `resource.path.<key>`, asset aliases, and package selection. Do not
substitute display terms such as `MasterData` for `master-data` in config or Blade props. Consumer
Vite/jsconfig aliases are generated from installed package keys; confirm the package is installed
and allowed by dependency layering before importing it.

## Resolve a documented source reference

Documentation should identify an owner and source-relative path, for example:

```text
Owner: Core
Source-relative path: resources/js/solaris/solar/solar-page.js
```

### Package repository

1. Match root `composer.json` name to the registry.
2. If it is the requested owner, resolve from the current repository root.
3. Otherwise use the owner's registered source repository path when known.
4. If that path was not supplied and is not already known, ask; never guess an absolute folder.

Package source is editable only when the task targets that package and its repository is the active
or explicitly approved workspace.

### Sandbox

A sandbox is a Laravel app registered by Solaris-Kit and normally has `solaris.dev.json`:

- Resolve app-owned source from the sandbox root.
- Resolve installed package reference source under `vendor/<vendor>/<package>/...`.
- Treat package files in `vendor/` as read-only mirrored copies.
- Change package code only in its registered source repository; Solaris-Kit mirrors it back.

### Consumer project

Resolve app-owned source from the project root. Installed package source under
`vendor/<vendor>/<package>/...` is reference-only. Request the package source repository when a
package change is required; never edit `vendor/`.

## Mandatory lookup sequence

1. Identify environment: package, sandbox, or project.
2. Identify owner: application or canonical package key.
3. Resolve the physical root from `composer.json`, Solaris-Kit registration, or Composer install;
   never from a hardcoded machine path.
4. Resolve the documented source-relative path under that root.
5. Confirm whether it is editable or reference-only.
6. If ownership or source root remains unknown, ask before broad search or modification.

## Import and view rules

- PHP uses the owner's PHP namespace.
- Package Blade uses the owner's Blade namespace.
- JS/CSS uses the owner alias; ancestor imports use the ancestor alias.
- App-owned JS may use aliases configured by that project.
- Assets resolve through `SolarAsset` or the config-driven Page loader, never hardcoded `vendor/`.
- Dependency direction is defined by [Layering](architecture/layering.md).
