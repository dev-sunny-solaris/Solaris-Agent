# Table

Owner: Core

Source-relative paths:

- Blade: `resources/views/components/table.blade.php`
- JS: `resources/js/solaris/table/table.js`

Table is a reusable SolarUI Component. `SolarListPage` commonly orchestrates one primary Table, but
Table may be used by other Pages without adopting SolarListPage.

## Props

| Prop | Default | Contract |
|---|---:|---|
| `id` | required | Root SolarUI ID and generated child-ID basis |
| `model` | `null` | Model/controller shorthand sent in `X-Model` |
| `mode` | `simple` | `simple`, `grid`, or `card`; read DataGrid for `grid` |
| `card_wrapper` | `true` | Bootstrap card wrapper |
| `new` | `true` | Boolean, label, `@modal-id`, or `Label@modal-id` |
| `search`, `refresh`, `pagination` | `true` | Standard controls |
| `length` | `10` | Records per page |
| `lazy` | `false` | Suppress constructor auto-render |
| `hidden` | `false` | Hide wrapper without suppressing initialization |
| `ignore` | `false` | Suppress SolarUI auto-initialization for manual use |
| `filter`, `order_by` | `null` | Initial filter/order; string or array |
| `extend_columns` | `null` | Supporting response fields without visible columns |
| `extend_search` | `null` | Search fields without visible columns |
| `fixed_column` | `false` | `true` fixes one starting column; integer fixes that count |
| `vertical` | `null` | Vertical scroll height in pixels |
| `select_all` | `true` | Multiple-selection mode |
| `delete_all` | `false` | Batch delete and progress Modal |
| `import`, `export` | `false` | Boolean or custom label; standard Modal included when enabled |
| `save_all`, `discard_all` | `false` | DataGrid controls |
| `folder` | `false` | Connected Folder Component |

`new="@modal-id"` uses label New; `new="Create@modal-id"` changes it. Non-prop attributes apply to
the `<table>`; prefix wrapper attributes with `wrapper:`.

## Slots and exact placement

```text
wrapper
├── header
├── toolbar
│   ├── left: pre-left-action, New, general action, grid actions, Folder, left-action,
│   │         selection state and selected-action
│   └── right: pre-right-action, Search, Refresh, right-action
├── responsive table: column/thead and body/tbody
└── pagination
```

`action` is appended inside the general dropdown after built-in Select/Import/Export.
`selected-action` is appended after Clear All and optional Delete All. The default Table slot is not
rendered; use named slots. Slot attributes on `header`, `column`, and `body` belong to their wrapper,
`thead`, and `tbody` respectively.

When used through `<x-core::page.list>`, prefix Table slots with `table-`, for example
`table-column`, `table-pre-right-action`, and `table-selected-action`.

## Table Column

`<x-core::table-column>` produces a visual `<th>` and declarative backend request metadata.

- `data` is the backend field/relation path and displayed row property.
- `name` is the logical identity for events, rendering, and ordering.
- Omitted normal `name` defaults to `data`.
- `data="customer.name"` enables lookup and infers `name="customer_id"`.
- Names beginning with `_` are special and are not selected from the backend.

| Prop | Contract |
|---|---|
| `data`, `name` | Data path and logical identity |
| `searchable` | Boolean or explicit Filter operator |
| `orderable`, `default_order` | Server sorting and optional `asc`/`desc` default |
| `lookup` | Boolean or relation subcolumns; dot paths enable it automatically |
| `width`, `cell_class` | Body-cell layout; normal `class` applies to `<th>` |
| `type`, `format`, `prefix` | Built-in display formatting |
| `decimal_separator`, `thousand_separator` | Numeric formatting |
| `expand` | `true` uses `15rem`, or pass CSS width; click toggles truncation |
| `editable` | DataGrid editor declaration only |

Use `expand` instead of custom render/created code solely for expandable text.

## Action Column

`<x-core::table-action-column>` is special `_action` metadata. Defaults: View, Edit, Delete, and
grouped rendering enabled; Discard disabled. Table owns standard Delete. With a linked ModalForm,
SolarUI owns standard New and Edit.

Page-specific row action orchestration belongs in the selected Page guide, not here.

## Backend contract

Table posts normal columns as `{ name, data, searchable, orderable, lookup }` plus pagination,
search, filters, orders, and extensions. `DataTableController` validates `X-Model` through
`DataTableAccess`, then calls an allowed custom controller or `DataTable::get()`.

`DataTable` removes empty/forbidden fields and passes prepared fields to model `scopePrepared()`
when available. `extend_columns` adds non-visible response fields. Search, filters, and orders use
the model's corresponding scopes.
