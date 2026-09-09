# TableLookup

Owner: Core

Source-relative paths:

- Blade: `resources/views/components/table-lookup.blade.php`
- JS: `resources/js/solaris/table/table-lookup.js`

TableLookup composes an ignored internal Modal, Table, and Buttons into a table-driven picker. Blade
pushes it to `modal-content`; retrieve the outer Component by its declared ID.

Props configure model, title/size, table scrolling/filter/order/extensions, folder, search/refresh, and
Choose label. The `column` slot declares Table columns; optional `body` and `footer` extend the dialog.

`open(preFilter)` renders on first open and refreshes later; `close`, `reset`, `getTable`, and runtime
Table setters are available. Events: `open`, `close`, `cancel`, `choose`, and `choose-no-close`.
Choose handlers receive selection metadata and selected row data. Single-select mode highlights on click
and chooses on double-click; checkbox columns activate multi-selection behavior.
