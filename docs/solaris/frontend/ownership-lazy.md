# Ownership and Lazy Semantics

## Ownership

Global SolarUI owns Components outside Form and Field boundaries. Form scans supported nested controls.
Field owns one child plugin. Composite Components mark internal children `ignore` to prevent duplicate
initialization.

`ignore` means automatic ownership/initialization skips the element. It does not create a manually
accessible instance. Manual ownership must explicitly construct and destroy its instance.

## Lazy semantics

| Component | Meaning |
|---|---|
| Field and most BaseInput children | Constructor defers plugin `init()` |
| Lookup, DatePicker, TagInput | Runtime config may be supplied before `init(config)` |
| Table | Defers data render, not Component construction |
| Dropzone and Notes | Defers internal initialization |
| DocumentEditor | Blade emits it, but current JS initializes immediately |
| StageButtons | Blade emits it, but current JS does not use it as a guard |

Only use lazy when the selected guide identifies who performs later initialization. A deferred Component
left without that call remains unusable.
