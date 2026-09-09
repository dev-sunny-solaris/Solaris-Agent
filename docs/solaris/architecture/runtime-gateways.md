# Runtime Architecture and Gateways

## Request flow

```
HTTP Request
  → Controller    receive, validate via FormRequest, return response — no logic
  → Repository    mini-service bound to one Model; all domain logic and CRUD live here
  → Model         table, relations, casts, attributes
  → Service       optional; only when logic spans multiple repositories or needs orchestration
```

Middleware: web routes are wrapped in `auth` + `menu.access`. `menu.access` decides both route
access and sidebar visibility, scoped to routes registered in the Menu table.

Core registers these middleware aliases: `role`, `permission`, `menu.access`, `verified`,
`auth.mobile`, `auth.api_key`, `api.log`, `api.idempotency`, `2fa.pending`.

## Gateway controllers (core)

Core provides global gateways so packages do not declare repetitive CRUD routes. Each gateway
identifies its target through a request header and validates a PHP attribute on the target class as
a security flag before dispatching by reflection. Every gateway route is rate-limited by its own
named limiter.

| Gateway | Route | Header | Attribute on target |
|---|---|---|---|
| FormController | `GET\|POST\|PUT\|DELETE /core/form/{id?}` | `X-Model` | `#[FormAccess]` |
| DataTableController | `POST /core/datatable` | `X-Model` | `#[DataTableAccess]` |
| LookupController | `POST /core/lookup` | `X-Model` | `#[LookupAccess]` |
| KanbanController | `POST\|PUT /core/kanban/{id?}` | `X-Model` | `#[FormAccess]` |
| MetricDashboardController | `POST /core/dashboard/metric` | `X-Model` | `#[MetricAccess]` |
| FolderController | via `/core/form`, plus `GET /core/folder/all-columns` | `X-Folder-Model` | `#[FolderAccess]` |
| ExcelController | `/core/excel/export/*`, `/core/excel/import/*` | `X-Export-Model`, `X-Import-Model` | `#[ExportAccess]`, `#[ImportAccess]` |

DataTableController, LookupController, KanbanController and FormController are invokable
single-action controllers.

### FormController

The central CRUD gateway for Blade forms.

- The header is parsed by `ModelBinding`, which resolves both a controller and a model. **At least
  one of the two must carry `#[FormAccess]`** — the request is rejected only when neither does.
- Method → action: `GET`→`show`, `POST`→`store`, `PUT`→`update`, `DELETE`→`destroy`, and
  `DELETE` with `batch_delete` in the payload →`batchDelete`. The action must exist on the resolved
  controller.
- `GET`/`PUT`/`DELETE` require an `{id}` unless the request is a batch delete.
- When the model resolves, `ObjectRuleService` checks the object permission for the mapped
  operation (Read / Create / Update / Delete) against the model's table.
- Arguments are resolved by reflection: a parameter typed as a `Request` subclass is built through
  the container, so **a FormRequest is injected and validated automatically** — the controller
  method just declares it.

### DataTable and Lookup

Both read the model directly through their service (`DataTable` / `Lookup`), so no controller
method is needed. Their attribute params are `scopes` (query constraints) and `columnExceptions`
(hidden columns).

### Excel

- Export is asynchronous: `POST /core/excel/export` queues the job and notifies the user; the
  companion routes serve the column list, the log, the log list and the download.
- Import is a multi-step flow: `POST /core/excel/import/validation` → `/upload` → `/columns`, with
  log and error-log-download routes alongside.

### Attribute inheritance

All access attributes support `extend: true` to inherit and merge the parent class's config — this
is how a downstream package extends an ancestor's model without restating its attributes.

## What a package still declares

Only what the gateways cannot serve: the list page, the edit page, the read redirect, and
genuinely custom endpoints. Everything else goes through `/core/*`.
