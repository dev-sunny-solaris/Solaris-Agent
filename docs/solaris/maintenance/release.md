# Releasing a Change

Every change lands in someone else's `vendor/`, so:

- Bump the `version` in `composer.json` and record the change in `CHANGELOG.md`.
- If a downstream package must move in lockstep, raise its `require` constraint on this package in
  the same release.
- Use `solaris pack` to zip a package and `solaris build` to bundle a sandbox for distribution.

## What counts as a breaking change

Call these out explicitly in the changelog:

- renaming or removing a config key (the registry is public API)
- changing a helper, repository, base controller or base resource signature
- changing a Blade component's props, or a JS component's data attributes
- renaming a view namespace or a JS/CSS alias path
- changing a gateway header or a required access attribute
- changing what a generator stub emits
- changing an API route name, payload shape, or field visibility in `columns()`
- removing a permission a seeder previously created

## Never

- Edit anything under `vendor/` to make something work. Fix the source package and let the mirror
  or a reinstall carry it over.
- Commit `composer.local.json` or `composer.dev.json`.
- Ship a new module without its menu entry, permissions, and config registry entries — it will be
  invisible or unreachable in every consuming app.
