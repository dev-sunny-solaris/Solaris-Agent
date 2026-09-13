# DataGrid

Owner: Core

Source-relative paths:

- JS: `resources/js/solaris/table/datagrid.js`

Required prerequisite: [Table](table.md).

DataGrid extends Table with inline editing. Select it with `mode="grid"`; all normal Table contracts
still apply. Editor declarations belong in the Blade Table Column `editable` prop. Do not replace
cells with hand-built inputs from Page code.

## Lifecycle and state

DataGrid parses editable metadata, renders editor HTML, then constructs the editor Component in each
cell. Editor families are text/input, number, textarea, lookup, date, datetime, and time.

Fetched rows receive `_index`, `_state: "clean"`, and `_oldValue`. Edited rows become `dirty`; added
rows are `new`. DataGrid provides row Save/Discard, optional Save All/Discard All, validation state,
and unsaved-change protection across refresh, pagination, unload, and link navigation.

Standard save uses:

```text
new row:      POST /core/form
existing row: PUT /core/form/<id>
header:       X-Model: <Table model shorthand>
```

Use `saving` only to customize the request config. Do not duplicate the standard request.

## Interaction between columns

There are two targets.

### Update column data/rendering

```js
this.table.updateCell(row, "total", value)
```

This changes row data by target column `name` and invalidates the cell for rerendering. Use it for a
calculated or non-editable display column.

### Control a column editor

```js
const editor = this.table.getEditor(row, "price")
editor?.set(value)
```

This returns the editor Component stored in that row's target cell. Available APIs depend on editor
type and may include `get`, `set`, `reset`, `disabled`, `enabled`, `error`, and
`resetValidation`. Do not manipulate its input DOM directly.

Listen to changes with `column change:<name>`:

```js
this.table.on("column change:quantity", (row, quantity, rowData) => {
	this.table.updateCell(row, "total", quantity * rowData.price)
}, "OrderListPage")
```

Use `column config:<name>` to merge dynamic configuration before an editor initializes, especially
for Lookup parameters that depend on another value in the same row.
