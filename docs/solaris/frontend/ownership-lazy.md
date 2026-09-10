# Ownership and Lazy Semantics

## Ownership

Global SolarUI owns Components outside Form and Field boundaries. Form scans supported nested controls.
Field owns one child plugin. Composite Components mark internal children `ignore` to prevent duplicate
initialization.

`ignore` means automatic ownership/initialization skips the element. It does not create a manually
accessible instance. Manual ownership must explicitly construct and destroy its instance.

## Prop placement at Field boundaries

When a supported input is wrapped by Field, Field owns that input's lifecycle. Put `lazy` and
`ignore` on `<x-core::field>`, not on its child input Component. Field controls deferred construction,
passes runtime config when it creates the child plugin, and remains the unit discovered by Form.

When an input is used standalone, there is no Field boundary. Put `lazy` and `ignore` directly on
the standalone input Component when its guide supports those props.

## Lazy semantics

| Component | Meaning |
|---|---|
| Field and most BaseInput children | Constructor defers plugin `init()` |
| Lookup, DatePicker, TagInput | Runtime config may be supplied before `init(config)` |
| Table | Defers data render, not Component construction |
| Dropzone and Notes | Defers internal initialization |
| DocumentEditor | Blade emits it, but current JS initializes immediately |
| StageButtons | Blade emits it, but current JS does not use it as a guard |

Lazy defers the first initialization so JavaScript can supply custom config before a plugin builds. It
avoids auto-initializing with defaults and then rebuilding through re-init. Only use lazy when the
selected guide identifies who performs the deferred `init(config)` call. A deferred Component left
without that call remains unusable.
