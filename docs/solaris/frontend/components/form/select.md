# Select

Owner: Core

Source-relative paths:

- Blade: `resources/views/components/select.blade.php`
- JS: `resources/js/solaris/form/select-input.js`
- Multiple JS: `resources/js/solaris/form/select-multiple-input.js`

Use native Select only for explicitly static/native behavior; otherwise use [Lookup](lookup.md).
Options accept array/object items with `value`, `name` or `text`, and optional `selected`. Props include
`placeholder`, `multiple`, `value`, `size`, `lazy`, and `ignore`. Setting `lookup` delegates entirely to
Lookup and activates its source-related props.

Single `get()` returns `{ id, name }` or `null`; Form serializes it to the ID. Single `set()` accepts an
ID string or lookup-shaped object and rejects values absent from options. Multiple `get()` returns an
array or `null`; `set()` accepts strings or `{ id, selected }` instructions. Only `change` is supported.
