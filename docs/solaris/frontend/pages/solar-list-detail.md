# SolarListDetail

Owner: Core

Source-relative paths:

- JS: `resources/js/solaris/solar/solar-list-detail.js`

Required prerequisites:

- [SolarPage](solar-page.md)
- [SolarListPage](solar-list-page.md)
- [SolarEditPage](solar-edit-page.md) when hosted by the standard parent flow
- [Table](../components/table.md)
- [ModalForm](../components/modal-form.md) only when the detail has New/Edit

## Purpose

`SolarListDetail` extends `SolarListPage` for a child collection owned by a parent record. It preserves
normal List Page behavior, injects the parent key into a linked ModalForm, forwards detail lifecycle
events to the parent Page, and opens Edit on row double-click.

Normally, define and export the detail class, then register it in `SolarEditPage`. Do not instantiate
each detail from a separate entry; the parent constructs every registered detail after its `init()`.

## Required structure and IDs

The registration name is also the List Page ID. Its Table must use `<detail-name>_list`:

```blade
<x-core::table
    id="account-contact_list"
    model="AccountContact"
    new="@account-contact-modal"
    filter="account_id = {{ $id }}"
    lazy>

    <x-slot:column>
        <x-core::table-action-column :view="false" />
        <x-core::table-column data="contact.name">
            Contact
        </x-core::table-column>
    </x-slot:column>
</x-core::table>

@push('modal-content')
    <x-core::modal-form
        id="account-contact-modal"
        title="Contact"
        model="AccountContact">
        {{-- Form fields --}}
    </x-core::modal-form>
@endpush
```

Table `new="@account-contact-modal"` links the ModalForm through SolarListPage. The Table `filter`
limits displayed children; `parentColumn` separately guarantees that New/Edit payloads carry the parent
key. Use both. `lazy` lets detail `init()` register renderers and handlers before final Table render.

## Define the detail class

```js
// account/contact-detail.js
import SolarListDetail from '@core-js/solaris/solar/solar-list-detail'

export default class AccountContactDetail extends SolarListDetail {
	/**
	 * @returns {void}
	 */
	init() {
		this.columnRender('contact_id', (value, rowData) => {
			return rowData.contact?.name ?? '-'
		})
	}
}
```

Do not add an empty `init()` or call `super.init()` only; the inherited implementation is empty. Add an
override only for real detail initialization. All SolarListPage hooks and helpers remain available.

Do not call `new AccountContactDetail(...)` here. Export the class for parent registration.

## Register and construct from SolarEditPage

```js
// account/page.js
import SolarEditPage from '@core-js/solaris/solar/solar-edit-page'
import AccountContactDetail from './contact-detail'

export default class AccountEditPage extends SolarEditPage {
	/**
	 * @returns {void}
	 */
	init() {
		this.registerDetail('account-contact', AccountContactDetail, 'account_id')
	}

	/**
	 * @returns {void}
	 */
	initDetail() {
		this.accountContact
			.on('saved', () => this.refreshForm())
			.on('deleted', () => this.refreshForm())
	}
}
```

```js
// account/page-index.js
import AccountEditPage from './page'

new AccountEditPage()
```

Construction order:

1. `SolarEditPage` resolves Form and calls parent `init()`.
2. `registerDetail()` records class, name, parent column, and optional parent value.
3. `_initDetails()` calls `new DetailClass(name, parentColumn, parentValue)`.
4. Detail resolves `<name>_list`, links ModalForm, runs detail `init()`, then renders Table.
5. Parent `initDetail()` runs after all detail instances exist.

The registration name becomes a camelCase parent property: `account-contact` becomes
`this.accountContact`. A collision with an existing property throws.

## Constructor contract

```js
constructor(id, parentColumn, parentValue)
```

| Argument | Contract |
|---|---|
| `id` | Registration name and prefix for `<id>_list` |
| `parentColumn` | Child Form binding/payload key referencing the parent |
| `parentValue` | Parent lookup object or scalar value |

Normally do not override the constructor. `registerDetail()` supplies all arguments. Without an
explicit `parentValue`, SolarEditPage builds `{ id: parentPage.id, name: displayValue }`.

## Parent-key injection

When the linked Modal opens, SolarListDetail resolves `this.form.get(parentColumn)`:

- Existing field: silently set `parentValue`, then disable the field.
- Missing field: call `form.addData(parentColumn, value)`.
- In the fallback, a lookup-shaped parent value is reduced to its `id`.

Include a disabled parent Lookup when the relationship should remain visible. Omit it when hidden
relationship injection is sufficient. If included, its binding key must equal `parentColumn`; use
`visual-id:binding-key` to namespace the visual ID.

## Events

| Event | Arguments | Timing |
|---|---|---|
| `ready` | `(settings, json)` | Detail Table becomes ready |
| `saving` | none | Linked Form starts submission |
| `saved` | `(response)` | Linked Form succeeds |
| `deleting` | `(row, data)` | Table starts deletion |
| `deleted` | `(data)` | Table deletion succeeds |

Current `SolarListDetail.on()` accepts only `(type, callback)`. A third listener-name argument is
ignored, there is no public `off()`, and callbacks execute in registration order. Unsupported events
and non-function callbacks throw.

SolarEditPage automatically adds a `saving` handler that saves a dirty parent first and aborts detail
save if the parent save fails.

## Standard behavior

- New/Edit/Delete remain owned by Table and ModalForm.
- Row double-click calls `modalForm.edit(data.id)` when ModalForm exists.
- Never add empty `new()`, `edit()`, or `delete()` overrides.
- Custom Delete follows SolarListPage's boolean contract.
- `detail.refresh()` reloads its Table.
- Parent `refreshDetail(name)` accepts registration or generated property name.

Keep Table lazy/unrendered until detail construction when initial forwarded Table events are required.

## Parent create flow

A detail needs a valid parent ID. On create Pages, disable or hide child creation until parent save,
unless an explicit valid `parentValue` exists. After parent success, SolarEditPage updates
`parentValue` only for registrations without an explicit value.

The parent must synchronize `this.id` with the created record response when its endpoint does not update
Page/Form identity. Never submit a child with a null parent ID.

## Checklist

- [ ] Detail extends `SolarListDetail` and is exported, not directly instantiated.
- [ ] Parent entry calls `new ParentEditPage()` exactly once.
- [ ] Parent registers details in `init()`.
- [ ] Table ID is `<registration-name>_list`.
- [ ] Table filter and `parentColumn` target the same relationship.
- [ ] ModalForm is linked through `new="@modal-id"` when CRUD is required.
- [ ] Table is `lazy` when detail initialization must precede render.
- [ ] Parent event wiring occurs in `initDetail()`.
- [ ] Empty lifecycle/action overrides are removed.
- [ ] Child creation cannot proceed without a valid parent ID.
