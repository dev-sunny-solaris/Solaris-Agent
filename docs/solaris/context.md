# Context Levels: Package, Sandbox, and Project

Solaris ships as Composer packages. There are three working contexts. Package development and
consumer development have almost inverted rules; a sandbox is a disposable consumer application
used to exercise package source at runtime.

| | **Project** | **Sandbox** | **Package** |
|---|---|---|---|
| Purpose | Production consumer app | Throwaway consumer app for package runtime development | Reusable package source |
| You edit | App-owned `app/`, `resources/`, routes, config | Sandbox-owned app files; package changes belong in source repositories | Package `src/`, `resources/`, routes, database, config |
| Solaris lives in | `vendor/solaris/*`, read-only | Mirrored `vendor/solaris/*`, read-only | Current or registered package repository |
| Scaffolding | `solaris:make` generates app files | Same consumer behavior | Maintain generators/stubs themselves |
| Testing | Run the app | Solaris-Kit runs the app and mirrors package edits | Tests plus runtime verification through sandbox |

**How to tell where you are:**

| Markers at the repository root | Context |
|---|---|
| `composer.json` named `solaris/solaris-laravel-*` and a `src/*ServiceProvider.php` | Package |
| Laravel app (`artisan`) whose `composer.json` requires `solaris/*`, with `solaris.dev.json` | Sandbox — confirm with `solaris sandbox check` |
| Laravel app (`artisan`) whose `composer.json` requires `solaris/*`, without `solaris.dev.json` | Project |

When markers conflict or are unavailable, ask rather than infer from the directory name.

Tooling repositories — Solaris-Kit, Solar-UI, this documentation bundle — are none of these. Follow
their own README; these docs do not apply.

If the request builds a feature in an application, apply project-level rules and keep changes in the
consumer app. If it changes reusable Solaris behavior, apply package-level rules and work in the
owning package source repository—not the consumer's `vendor/` copy.

After identifying context, read [Package registry](package-registry.md) to resolve package
owners, aliases, namespaces, and physical source paths safely.
