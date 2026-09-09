# SolarListPage

Owner: Core

Source-relative paths:

- JS: `resources/js/solaris/solar/solar-list-page.js`
- Blade: `resources/views/components/page/list.blade.php`

Required prerequisite: [SolarPage](solar-page.md).

Component details are intentionally separate:

- [Table props, slots, columns, and actions](../components/table.md)
- [ModalForm](../components/modal-form.md), only when the list uses one
- [DataGrid](../components/datagrid.md), only for `mode="grid"`

## Purpose and construction

`SolarListPage` standardizes a screen with one primary Table. It exposes Table/ModalForm references,
registers standard hooks, and prevents each Page from rebuilding Table behavior at the atomic level.

```js
constructor(id, autoRender = true)
```

The ID is the Page Blade ID:

```blade
<x-core::page.list id="role" ...>
```

```js
new RoleListPage("role")
```

Blade creates the primary Table as `<page-id>_list`. Construction resolves standard references,
calls subclass `init()`, then renders an unrendered Table when `autoRender` is true.

| Property | Value |
|---|---|
| `this.table` | Primary `<page-id>_list` Table |
| `this.modalForm` | Linked `<modal-id>_ModalForm`, or `null` |
| `this.modal` | Linked Modal, or `null` |
| `this.form` | Linked Form, or `null` |

Use `hasModalForm()` before behavior that requires the linked Modal and Form.

## Blade contract

`<x-core::page.list>` consumes these Page props:

| Prop | Contract |
|---|---|
| `id` | Page identity; generates the primary Table ID `<page-id>_list` |
| `title` | Page title passed to BasePage |
| `breadcrumbs` | Breadcrumb items passed to BasePage |
| `package`, `path` | Resolve the Page JS entry through BasePage |

All remaining attributes are forwarded to the primary Table. Read
[Table](../components/table.md) for their contracts.

Page-owned slots are:

| Slot | Position |
|---|---|
| `buttons` | BasePage top-button area |
| `header` | Immediately before the primary Table |
| Default slot | Immediately after the primary Table |

Table slots are forwarded with a `table-` prefix, such as `table-header`, `table-column`,
`table-pre-right-action`, and `table-selected-action`. Their exact placement belongs to the Table
guide.

## Rendering control

There are two related controls:

- Table `lazy` prevents the Table constructor from rendering immediately.
- `SolarListPage` constructor `autoRender` controls its final render after Page `init()`.

A custom List Page that registers `columnRender()`, `columnCreated()`, or other pre-render column
configuration must set the Blade Table/List Page `lazy` prop. This guarantees Page `init()` can
finish registration before the final render:

```blade
<x-core::page.list id="account" model="Account" lazy>
```

```js
new AccountListPage("account")
```

Pass `false` as `autoRender` only when custom code deliberately owns the final call to
`this.table.render()`. Do not disable it without supplying that render flow.

## Keep simple CRUD declarative

For Table plus ModalForm, `new="@modal-id"` is sufficient. SolarUI automatically links New and Edit,
closes/resets the ModalForm, and refreshes the Table after success. Table owns standard Delete.

Do not implement `new()`, `edit()`, or `delete()` for this simple case. A non-empty `new()` or
`edit()` replaces the automatic SolarUI handler; override only when the standard flow cannot meet the
requirement. Never add an empty action override.

View has no automatic destination. If `<x-core::table-action-column>` enables View, implement
`view(row, data)` or disable it with `:view="false"`. Do not leave an enabled action that does
nothing.

Custom Delete has a different contract. Defining non-empty `delete(data)` disables Table's default
`Model.delete()` call. Every registered custom Delete handler must resolve to boolean `true` before
Table removes the row, triggers `deleted`, and refreshes. Return `true` only after the custom server
deletion succeeds; any other result keeps the row:

```js
async delete(data) {
    await this.axios.delete(`${BASE_URL}/module/${data.id}`)
    return true
}
```

## Hooks

| Hook | Called for |
|---|---|
| `init()` | Page startup before final auto-render |
| `new()` | Custom New flow replacing automatic ModalForm New |
| `edit(row, data)` | Custom Edit flow replacing automatic ModalForm Edit |
| `view(row, data)` | View action |
| `rendering()` | Before DataTables initialization |
| `ready(settings, json)` | Initial render/data load complete |
| `draw(event, settings)` | Every draw |
| `rowCreated(row, data, dataIndex)` | Row DOM created |
| `rowClick`, `rowDoubleClick`, `rowContextMenu` | Row interactions |
| `columnsCreated(name, row, cell, cellData, rowData)` | Any cell created |
| `onActionCreated(row, cell, rowData)` | `_action` cell created |
| `deleting(row, data)`, `deleted(data)` | Around standard deletion |

Helpers include `columnRender`, `columnCreated`, `loading`, `refresh`, `hasModalForm`, and the
`action*` family. Column handlers must be registered in `init()` before Table render.

`columnsCreated(name, row, cell, cellData, rowData)` is the global hook called for every created
column cell. `columnCreated(name, callback)` registers a callback for only one logical column.
Likewise, `columnRender(name, callback)` targets one logical column; there is no global render hook
on SolarListPage.

## Custom column rendering

Use `columnRender(name, callback)` when a cell must display custom HTML or combine multiple response
fields. `name` is the logical Table Column name—not necessarily its `data` path:

```js
init() {
    this.columnRender("customer_id", (data, rowData) => {
        return `<strong>${data ?? "-"}</strong><small>${rowData.customer?.code ?? ""}</small>`
    })
}
```

The callback receives `(cellData, rowData)`. For `data="customer.name"`, the inferred logical name
is `customer_id`; register `columnRender("customer_id", ...)`. Request supporting values that have
no visible column through Table's `extend_columns` rather than adding fake hidden headers.

Register renderers during `init()`. `columnRender()` throws after DataTables has rendered because
column definitions can no longer be changed safely.

Use `columnCreated(name, callback)` only when behavior requires the completed cell DOM:

```js
init() {
    this.columnCreated("status_id", (row, cell, cellData, rowData) => {
        cell.querySelector("button")?.addEventListener("click", () => this.openStatus(rowData))
    })
}
```

Its callback receives `(row, cell, cellData, rowData)`. Keep pure display transformation in
`columnRender()` and DOM event/binding work in `columnCreated()`. Use built-in Table Column props such
as `type`, `format`, `prefix`, and `expand` before writing either custom hook.

## Custom row actions

Add actions from `onActionCreated(row, cell, rowData)`.

### Inside the standard action group

```js
onActionCreated(row, cell, rowData) {
    this.actionBefore(row, "delete-action", {
        selector: "approve-action",
        label: "Approve",
        icon: "ri-check-line",
        color: "success",
        callback: () => this.approve(rowData.id),
    })
}
```

Action keys are `selector` (class without `.`), `label`, `icon`, optional Bootstrap `color`, and
optional `callback`.

| Helper | Placement |
|---|---|
| `actionAppend` / `actionPrepend` | End/start |
| `actionAfter` / `actionBefore` | Relative to a reference class |
| `actionRemove` | Remove by class |

Built-in references include `view-action`, `edit-action`, `delete-action`, and `discard-action`.
Missing references make After fall back to append and Before to prepend. Helpers produce dropdown
items when the action column is grouped and icon buttons when it is not.

Divider Append/Prepend/After/Before helpers apply only to grouped actions.

### Independent button beside the standard group

If an action must remain outside View/Edit/Delete, create it through a Solaris/domain helper and
insert it as a sibling of `.button-action`:

```js
onActionCreated(row, cell, rowData) {
    const standardActions = cell.querySelector(".button-action")
    const customButton = this.createActivityButton(rowData)
    if (!standardActions || !customButton) {
        return
    }

    const wrapper = document.createElement("div")

    wrapper.className = "d-flex justify-content-center align-items-center gap-1"
    wrapper.append(customButton, standardActions)
    cell.replaceChildren(wrapper)
}
```

Do not use an `action*()` helper for an independent sibling, and do not inject dropdown `<li>`
manually when a helper covers the grouped requirement.

## Complex custom List Page pattern

A custom List Page may coordinate metrics, filters, alternate views, multiple ModalForms, composite
columns, and domain row actions. Preserve standard behavior wherever it still fits:

- Keep New linked to ModalForm and Delete owned by Table.
- Override Edit only when it must open a different flow, such as a full Edit Page.
- Use `columnRender()` plus `extend_columns` for composite cells.
- Add actions through `onActionCreated()` and action helpers.
- Apply filters through `table.filter()` and reuse `table.getParamData()` for related summaries.
- Synchronize alternate representations through public Component events.

When a domain-specific Blade composition owns multiple complex Components with overlapping props,
separate forwarded attributes by prefixes such as `table:*` and `<component>:*`. This is an extracted
design pattern, not a requirement to inspect another package implementation.
