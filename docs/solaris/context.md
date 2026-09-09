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

**How to tell where you are:** if the repo root has a `composer.json` with
`"name": "solaris/solaris-laravel-*"` and a `src/*ServiceProvider.php`, you are at package level.

If the root is a Laravel app and contains `solaris.dev.json` or is registered by Solaris-Kit, it is
a sandbox. A Laravel app consuming Solaris without that development registration is project level.
When markers conflict or are unavailable, ask rather than infer from the directory name.

If the request builds a feature in an application, apply project-level rules and keep changes in the
consumer app. If it changes reusable Solaris behavior, apply package-level rules and work in the
owning package source repository—not the consumer's `vendor/` copy.

After identifying context, read [Package registry](package-registry.md) to resolve package
owners, aliases, namespaces, and physical source paths safely.
