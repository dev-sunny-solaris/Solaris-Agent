# Working Rules and Checklists

## Safety fallbacks

These apply unless the active agent's global rules say otherwise; those take precedence.

- **Git is read-only.** Allowed: `status`, `log`, `diff`, `show`, `blame`, `branch --list`. Never
  commit, push, or run any other Git write — including via alias, script, or `gh`. Never list the
  agent as author or co-author.
- **Never touch the database.** Do not connect to, query, or modify it. The database is the user's
  domain. Schema changes are migrations; data changes are seeders.

## Working rules

1. **Deliver exactly what was asked.** If it would break something else, warn before doing it.
2. **Stay in scope.** Never change, refactor, or "improve" anything the request did not name.
3. **Plan multi-file changes.** A change to more than one file MUST start with a plan — files to
   change and approach — and wait for approval, even when the design was agreed in chat.
4. **Gather requirements before building a module.** Ask when unclear: new or existing table,
   columns and types, relations, sub-resources, standard CRUD or a special flow (approval, stage
   pipeline), list columns and filters, edit-page layout, export/import, mobile and API exposure.
5. **Look before asking.** Search the paths these docs map first, with targeted search. Read file
   slices, not whole files. Never re-read a file already read. Ask the user only when it is not
   found there.
6. **Read before writing.** Match the surrounding package's existing patterns; never import a
   convention from another framework.
7. **Ask for feedback when stuck.** When attempts repeat without a clear direction, stop and ask
   the user. Report what was tried, the exact error, the suspected cause, and the options.
8. **Respect the layering.** Never introduce an upward or sideways dependency.
9. **Register everything.** New class → config registry entry. New module → routes, menu seeder,
   permissions.
10. **Never hardcode class names, vendor paths, or config values** the registry already resolves.
11. **Do not over-engineer UI.** Use an existing component or a plain Bootstrap pattern first.
12. **Never edit `vendor/`.** Edit the package source; Solaris-Kit mirrors it into the sandbox.
13. **Be factual and concise.** No flattery; correct the user with facts when needed.
14. **Write code, comments, and documentation in English.**

## Code style

Follow [Code style](code-style.md) for all PHP, JavaScript, and TypeScript. It is the Solaris way
and overrides any global, personal, or tool-default style.

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
- [ ] `solaris sandbox config` run so the sandbox picks up the new config keys
- [ ] `composer run test` and `composer run analyse` pass
- [ ] Version bumped, `CHANGELOG.md` updated, breaking changes flagged
