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

## Project code style

These rules are the project-local copy of the global `code-style` contract. Apply them whenever
writing, editing, or reviewing PHP, JavaScript, or TypeScript.

### General comments

- Do not add comments that restate the code.
- Add an English explanatory comment only when the logic is genuinely too complex to be made
  self-documenting through structure and naming.
- Broader rationale and design decisions belong in documentation, not inline comments.

### PHP

- Indent with tabs at width 4; do not use spaces for indentation.
- Put class and method opening braces on a new line.
- Put control-structure opening braces on the same line.
- Always put a block body on a new line and use braces, including for single-line blocks.
- Name methods and variables in `camelCase`. An underscore separator is allowed for long, specific
  names. Database tables and columns use `snake_case`.
- Add native parameter and return types wherever possible. Declare every valid union member and use
  `mixed` only for genuinely dynamic values.
- Prefer guard clauses and early returns. Do not nest `if` branches when the flow can read from top
  to bottom.
- Never add PHPDoc to methods. Native parameter and return types must describe the contract.
- Add a normal English comment only for genuinely complex logic that cannot explain itself.

### JavaScript and TypeScript

- Indent with tabs at width 4; do not use spaces for indentation.
- Put opening braces on the same line. Always put a block body on a new line and use braces,
  including for single-line blocks.
- Name functions and variables in `camelCase`. An underscore separator is allowed for long,
  specific names. JSON response keys use `snake_case`.
- Prefer guard clauses and early returns; avoid nested `if` branches.
- Every method and function must have English JSDoc with `@param` entries that declare parameter
  types and an `@returns` entry that declares the return type. Do not add prose that merely restates
  the method name or implementation.
- In TypeScript, explicitly type every parameter and return value. Prefer interfaces for object
  shapes, avoid `any`, and use `unknown` with narrowing for genuinely dynamic values.
- Use Solaris' configured Axios module for HTTP requests; never use raw `fetch`.
- Use the shared Helper `downloadFile(url, method, { headers, data })` API for file or blob downloads.
  Inside the SolarPage family, access it through `this.helper.downloadFile(...)`; do not import Helper
  again. Outside a Page, use the shared Helper according to that module's ownership contract.
- In a request `catch`, show a manual error alert only when `!error?.response`; the Axios interceptor
  already handles server responses.
- Group related class methods with `#region` and `#endregion` when grouping improves navigation. Use
  this order when applicable: Static, Lifecycle, Public API, then feature-specific regions.

## Checklist — adding a module to a package

- [ ] Package chosen by domain ownership; no new upward/sideways dependency
- [ ] Migration added with the package's filename prefix and remains driver-agnostic
- [ ] Every foreign key declares `cascadeOnDelete()` for owned details or nullable
      `nullOnDelete()` for normal references, then calls `MigrationHelper::createIndexIfNeeds()`;
      normal non-FK indexes use Laravel's standard index API
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
