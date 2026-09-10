# Field

Owner: Core

Source-relative paths:

- Blade: `resources/views/components/field.blade.php`
- JS: `resources/js/solaris/form/field-input.js`

Field composes label, one child input plugin, validation feedback, required state, visibility, and
disabled state. Its `id`/`bind` owns the Form key; a child input may omit its ID because Field assigns it.

```blade
<x-core::field id="account-page-name:name" label="Name" required>
    <x-core::input :value="$account->name" />
</x-core::field>
```

Props: `id`, `bind`, `label`, `required`, `disabled`, `hidden`, `message`, `error`, `valid`, `horizontal`,
`lazy`, and `ignore`. Prefix nested attributes with `field:`, `label:`, or `message:`.

Field owns the lifecycle props of a wrapped input. Put `lazy` and `ignore` on Field:

```blade
{{-- Correct: Field owns each lifecycle prop. --}}
<x-core::field id="account_id" label="Account" lazy>
    <x-core::lookup />
</x-core::field>

<x-core::field id="status" label="Status" ignore>
    <x-core::select />
</x-core::field>

{{-- Incorrect: lifecycle props bypass their owning Fields. --}}
<x-core::field id="account_id" label="Account">
    <x-core::lookup lazy />
</x-core::field>

<x-core::field id="status" label="Status">
    <x-core::select ignore />
</x-core::field>
```

Without a Field wrapper, place supported lifecycle props directly on the standalone input:

```blade
<x-core::lookup id="account_id" lazy />
<x-core::select id="status" ignore />
```

Supported child types are plain/masked input, textarea, password, date/datetime/time, Select,
SelectMultiple, Lookup, checkbox/radio, option groups, and card option controls. SearchInput, TagInput,
OTP, Notes, DocumentEditor, and StageButtons are not Field plugins; use them standalone inside Form when
Form supports them.

Retrieve Field through `form.get(bindingKey)`. Its `get`, `set`, events, reset, validation, required,
disabled, visibility, label, and message operations delegate to or coordinate the child plugin. For a
lazy child, call `field.init(config)`, not a separately resolved child.
