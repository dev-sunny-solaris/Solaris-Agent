# Blade Conventions

Solaris UI is Blade-first. Declare structure and standard behavior in Blade before adding Page JS.

## Components and props

Use the owning package namespace. Literal strings use plain attributes; variables, arrays, objects,
booleans, and numbers use Blade binding:

```blade
<x-core::page.edit
    :id="$record->id"
    title="Account"
    :breadcrumbs="$breadcrumbs"
    model="Account"
    path="account/page-index">
</x-core::page.edit>
```

Boolean shorthand means true. Use `:prop="false"` when false must be explicit. Unknown attributes are
forwarded according to the composition contract. Prefixes commonly route attributes to nested elements:
`wrapper:`, `wrap:`, `field:`, `label:`, `message:`, `body:`, `header:`, `dialog:`, and `table-*`.

## Identity and binding

- `id` identifies DOM and usually supplies SolarUI identity.
- `solar-id` is explicit global Component identity.
- `solar-bind` is the Form map and payload key.
- `id="visual-id:payload_key"` separates visual and binding identities.
- IDs must be unique across the document, including pushed modal content.

## Slots, partials, and stacks

Use named slots rather than recreating Component internals. Compositions may forward slots with a
prefix, such as `table-column` through Page List. Large tabs and detail groups may use partials or
`sections/`; the parent Page still owns layout and Page entry selection.

Components such as TableLookup and nested detail modals may push markup to `modal-content`. Page assets
are selected through `package` and `path`; do not add a second ad-hoc Vite entry for the same Page.

## Initial values

| Control | Initial shape |
|---|---|
| Input/Textarea | Scalar value |
| Checkbox/Radio | `checked` plus option `value` |
| Select | Selected option/value |
| Lookup | JSON item or item array containing display `name` |
| DatePicker | Date/time-compatible string |
| Notes/DocumentEditor | Slot HTML |
| StageButtons | Current `value` or fallback `initial` |

Remote Lookup edit state needs display data, not only a foreign-key scalar. Render unescaped rich text
only when stored content is trusted or sanitized.
