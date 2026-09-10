# Form

Owner: Core

Source-relative paths:

- Blade: `resources/views/components/form.blade.php`
- JS: `resources/js/solaris/form/form.js`
- Field Blade: `resources/views/components/field.blade.php`
- Field JS: `resources/js/solaris/form/field-input.js`

Form owns fields, validation, payload serialization, request lifecycle, and deferred uploads. A Field
is a visual wrapper with a child input plugin; standalone inputs are also registered. Always access an
owned component through `this.form.get(bindingKey)`, not Page `this.get()`.

## Identity and ownership

`solar-bind` is the payload/map key; otherwise the component ID is used. `id="visual:key"` is shorthand
for visual ID `visual` bound as `key`. Inputs inside Field are registered once through Field. `ignore`
excludes a component from Form ownership; `exclude(key)` temporarily omits an owned field from submit.

```js
const email = this.form.get('email')
email.set('user@example.com')
const value = this.form.getValue('email')
this.form.set('email', value, true)
```

Use `lazy` when configuration must be supplied before first initialization. Put it on Field for a
wrapped input or directly on a standalone input, then call `init(config)` on that owning component.

Read [Field](field.md) before wrapping an input. Not every standalone input type is Field-compatible.

| UI | Form-owned | Serialized normally | Deferred |
|---|---:|---:|---:|
| Field and supported standalone inputs | yes | yes | no |
| Notes, DocumentEditor, StageButtons | yes | yes | no |
| ProfilePicture or Dropzone | no | no | optional |
| Component carrying `ignore` | no | no | no |

## Modes and submit lifecycle

- New mode: `POST`; call `newMode()`.
- Edit mode: `PUT` or `PATCH`; call `editMode({ type, id })`.
- `active()`/`nonactive()` enable or suppress submit, including Enter-key submit.
- Events run in this order: `submitting`, `validating`, `invalid` or `validated`, `sending`, then
  `failed`, `error`, or `success`.
- `submit(showAlert)` resolves to a boolean. Do not infer success only from an event firing.

Lifecycle listeners are asynchronous. Throwing from a listener aborts the request, but currently enters
Form's error path; there is no documented silent-cancellation primitive. Do not standardize intentional
`throw` cancellation until Core defines one.

## Validation

Required fields are detected from component state. Add asynchronous field rules with
`addValidation(key, callback)`; return `{ success, message }`. Multiple rules for a key run in order.
`validation()` returns `{ success, errors }`; `setErrors(errors)` maps server-style keyed messages back
to owned fields. Hidden does not imply ignored or optional.

## Serialization

JSON is default; use `setToFormData()` only when the main request itself contains files. Form normalizes:

- single Select, single Lookup, radio group, and StageButtons to an ID;
- multiple Select/Lookup to ID arrays;
- checkbox groups to their option result;
- DatePicker through `toString()`;
- Notes to raw HTML;
- ordinary inputs and DocumentEditor through `get()`.

`load()` resets before populating. Per-field conversion errors are logged and skipped, so a resolved
load does not guarantee that every field accepted its value.

Use `addData()` only for payload values that are not represented by owned components. Disabled fields
are preserved across submission state changes, while excluded and ignored fields are not serialized.

## Deferred uploads

ProfilePicture and Dropzone can wait for the successful record response. Register them through
`registerProfilePicture()` or `registerDropzone()`; see [Deferred uploads](deferred-uploads.md). Their
failure reports separately and does not reverse an already-successful main record request.
