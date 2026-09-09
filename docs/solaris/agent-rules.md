# Working Rules and Checklists

## Rules for agents

1. **Gather requirements first.** Ask before implementing if anything is unclear: new or existing
   table, columns and types, relations, sub-resources, standard CRUD or a special flow (approval,
   stage pipeline), list columns and filters, edit-page layout, export/import, mobile and API
   exposure.
2. **Confirm before touching files outside the request.** Package changes ripple downstream.
3. **Read before writing.** Match the surrounding package's existing patterns; do not import a
   convention from another framework.
4. **Respect the layering.** Never introduce an upward or sideways dependency.
5. **Register everything.** New class → config registry entry. New module → routes, menu seeder,
   permissions.
6. **Avoid redundant code comments.** Keep required JSDoc and explain only non-obvious rationale;
   broader design decisions belong in documentation.
7. **Do not over-engineer UI.** Check for an existing component or a plain Bootstrap pattern first.
8. **Do not hardcode class names, vendor paths, or config values** the registry already resolves.
9. **Stop after two failed attempts** at the same problem and report what you tried.
10. **Never connect to or query a database directly.** Schema changes are migrations; data changes
    are seeders.
11. **Follow the active agent/environment Git policy.** Solaris documentation does not grant Git
    permissions; never commit, push, or rewrite history unless the active policy and user allow it.
12. **Never edit a sandbox's `vendor/`.** Edit the package; the Kit mirrors it.

## Checklist — adding a module to a package

- [ ] Package chosen by domain ownership; no new upward/sideways dependency
- [ ] Migration added with the package's filename prefix, driver-agnostic
- [ ] Model extends the right base; `$table`, `$displayValue`, `$prepareColumns`, `$searchColumns`,
      `$orderBy`, `$direction` set; access attributes declared and accurate; scopes exist
- [ ] Repository holds all logic; only the needed hooks implemented; no extra transaction wrapper
- [ ] Controller has `#[FormAccess]`, delegates only, uses `cleanData()`
- [ ] A simple web-only FormRequest defines `rules()` only; API-enabled requests follow the split
      rule contract in [PHP standards](backend/php-standards.md)
- [ ] Blade pages under `pages/<module>/`; JS under `pages/<module>/`, imported via package aliases
- [ ] Web routes registered — list, read, edit, plus custom endpoints only; controller resolved via
      `Helper::controller()`
- [ ] Mobile controller + resource + routes if the module is exposed to the app
      ([Mobile API](backend/mobile-api.md))
- [ ] API controller + routes + permission seeder if the module is exposed to partners
      ([Third-party API](backend/third-party-api.md))
- [ ] `config/solaris.php` updated: model, controller, repository, form request, views,
      resource path, `mobile.controllers`, `mobile.resources`, `api.controllers`
- [ ] Menu seeder and permission seeder updated
- [ ] Generator stub updated if a convention changed
- [ ] `solaris config` run so the sandbox picks up the new config keys
- [ ] `composer run test`, `analyse`, `format` all pass
- [ ] Version bumped, `CHANGELOG.md` updated, breaking changes flagged
