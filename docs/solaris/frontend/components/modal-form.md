# ModalForm

Owner: Core

Source-relative paths:

- Blade: `resources/views/components/modal-form.blade.php`
- JS: `resources/js/solaris/modal-form.js`
- Integration: `resources/js/solaris/solar/solar-ui.js`

ModalForm composes a Modal and Form so implementation normally provides only fields:

```blade
<x-core::modal-form id="role-modal" title="Role" model="Role">
    <x-core::field id="name" label="Name">
        <x-core::input />
    </x-core::field>
</x-core::modal-form>
```

Generated IDs are deterministic:

```text
Modal:     role-modal
Form:      role-modal_form
Save:      role-modal_save
ModalForm: role-modal_ModalForm
```

SolarUI creates the JS `ModalForm` after Modal and nested Form Components initialize. Its default
behavior is:

- `new(callback)`: Form new mode, show Modal, optionally configure Form.
- `edit(data, callback)`: enter edit mode/load data, show Modal, optionally configure Form.
- Form success: hide Modal.
- Modal hidden: reset Form.

Blade props are `id`, `title`, `size`, `fullscreen`, `ignore`, `model`, `action`, `method`, `center`,
`save`, and `cancel`. The default slot contains fields; `buttons` inserts custom buttons before the
standard Save and Cancel controls.

## Automatic Table integration

A Table with `new="@role-modal"` emits `new-action="role-modal"`. SolarUI then connects:

- Table New to `role-modal_ModalForm.new()`.
- Table Edit to `role-modal_ModalForm.edit(data.id)`.
- Form success to Table refresh.

Do not reproduce these handlers in a simple List Page. Retrieve the composition as
`this.get("role-modal_ModalForm")` only for additional behavior or another modal flow.
