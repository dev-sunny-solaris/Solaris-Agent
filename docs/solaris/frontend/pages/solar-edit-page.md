# SolarEditPage

Owner: Core

Source-relative paths:

- JS: `resources/js/solaris/solar/solar-edit-page.js`
- Blade: `resources/views/components/page/edit.blade.php`

Required prerequisites:

- [SolarPage](solar-page.md)
- [Form](../components/form/form.md)

Read only the Component guides used by the view. For fields, start from the
[Components router](../components/README.md) and select the relevant input family.

## Purpose and construction

`SolarEditPage` standardizes create/edit screens around one Form. It wires Save and Refresh, forwards
Form lifecycle events to overridable Page hooks, tracks unsaved field changes, manages an optional
profile panel, and can coordinate list-detail sections.

Define the reusable Page class in one module, then instantiate it from the Blade-selected entry module:

`resources/js/pages/account/page.js`:

```js
import SolarEditPage from "@core-js/solaris/solar/solar-edit-page"

export default class AccountEditPage extends SolarEditPage {
	/**
	 * @returns {void}
	 */
	init() {
		this.status = this.form.get("status_id")
	}
}
```

`resources/js/pages/account/page-index.js`:

```js
import AccountEditPage from "./page"

new AccountEditPage()
```

```blade
<x-core::page.edit
    :id="$account->id"
    title="Account"
    model="Account"
    path="account/page-index">
    <x-core::field id="name" label="Name">
        <x-core::input />
    </x-core::field>
</x-core::page.edit>
```

`export default class` only defines and exports the class; it does not execute the constructor. The
entry loaded by Blade must call `new AccountEditPage()` exactly once. Package-owned Pages should keep
the reusable class and entry separate. A project-only Page may define and instantiate in one entry file,
but it still must contain the `new` call.

The constructor deliberately calls `super(false)`, resolves standard components, installs handlers,
calls overridden `init()`, then initializes dirty tracking, details, `initDetail()`, and the profile
panel. Do not add a constructor merely to call `init()`.

Standard references are:

| Property | Contract |
|---|---|
| `this.form` | Form with fixed ID `_form` |
| `this.saveButton` | `_save`, or `null` when Blade `save=false` |
| `this.refreshButton` | `_refresh` |
| `this.id` | Initial record ID obtained from Form |
| `this.breadcrumbName` | Active breadcrumb DOM element, when present |
| `this.details` | Registered list-detail definitions |

Page-level components use `this.get(id)`. Form-owned inputs use `this.form.get(bindingKey)`.

## Blade contract

```blade
<x-core::page.edit
    :id="$account?->id"
    title="Account"
    :breadcrumbs="$breadcrumbs"
    model="account"
    path="account/edit-index">

    <x-core::field id="name" label="Name" required>
        <x-core::input />
    </x-core::field>
</x-core::page.edit>
```

| Prop | Contract |
|---|---|
| `id` | `null` creates a POST Form; non-null creates a PUT Form and supplies its record ID |
| `title`, `breadcrumbs` | Passed to BasePage |
| `package`, `path` | Resolve the Page entry through BasePage |
| `model` | Form model shorthand used by Core's generic Form endpoint |
| `action` | Explicit Form endpoint; overrides generic model routing |
| `save` | Controls creation of `_save`; Cancel and Refresh remain present |
| `mainCard` | Wraps the default slot in Core Card when true |
| `profilePosition` | `left` or `right` placement for the optional profile slot |
| `profileClass`, `mainClass` | Grid classes used when a profile slot exists |

The generated Form always has ID `_form`. A hidden native `_id` input is rendered, but because it has
no `solar-ui`, Form ownership and record mode come from the Form component rather than this input.

Slots:

| Slot | Position |
|---|---|
| `preButtons` | Before Save |
| `buttons` | After Save, Cancel, and Refresh |
| `header` | Inside Form, before the profile/main row |
| Default | Main panel, optionally inside Card |
| `footer` | Main panel after the default content/Card |
| `profile` | Collapsible side panel |

Button IDs and Form ID are framework contracts. Do not reuse them for unrelated components on the
same Page.

## Organizing the Form with Tabs and Fieldsets

Put Tab in SolarEditPage's default slot so every pane remains inside the generated `_form`. Tab changes
visibility only; Fields in inactive panes remain Form-owned and participate in normal serialization and
validation.

Each TabNav ID must exactly match one TabContent ID, and exactly one initial pair must be active. Use an
accordion Fieldset when it wraps fields; a non-accordion Fieldset is only a labeled divider and discards
its slot. Read [Tab](../components/tab.md) and [Fieldset](../components/fieldset.md) for the complete
contracts.

```blade
<x-core::page.edit
	:id="$account?->id"
	title="Account"
	model="account"
	path="account/page-index">

	<x-core::tab id="account-tabs" variant="tab-style-7">
		<x-slot:nav>
			<x-core::tab-nav id="account-general" active>General</x-core::tab-nav>
			<x-core::tab-nav id="account-address">Address</x-core::tab-nav>
		</x-slot>

		<x-slot:content>
			<x-core::tab-content id="account-general" active>
				<x-core::fieldset id="account-identity" label="Identity" accordion>
					<x-core::field id="name" label="Name" required>
						<x-core::input />
					</x-core::field>
				</x-core::fieldset>
			</x-core::tab-content>

			<x-core::tab-content id="account-address">
				<x-core::fieldset id="account-location" label="Location" accordion>
					<x-core::field id="address" label="Address">
						<x-core::textarea />
					</x-core::field>
				</x-core::fieldset>
			</x-core::tab-content>
		</x-slot>
	</x-core::tab>
</x-core::page.edit>
```

For a large Form, keep the Tab shell in the Page and include one section partial inside each
TabContent. Partials must contain only their section content; they do not create another Page, Form, or
JavaScript entry.

## Create and edit modes

Blade selects POST when `id` is null and PUT otherwise. Form builds its generic endpoint from `model`,
or uses `action` when supplied. `this.id` is the initial Form record ID.

Use Form's mode methods if one Page instance deliberately switches records/modes:

```js
this.form.newMode()
this.form.editMode({ type: "PUT", id })
this.id = id
```

Keep Page `this.id` synchronized when changing Form ID at runtime; SolarEditPage does not automatically
mirror later `form.setId()` calls.

## Lifecycle hooks

Override only the hooks required by the screen:

| Hook | Called when |
|---|---|
| `init()` | Standard references exist; register field behavior and detail definitions here |
| `submitting()` | Submission begins, before fields are disabled |
| `validating(fields)` | Immediately before Form client validation |
| `invalid(result)` | Client validation fails |
| `validated()` | Client validation passes |
| `sending(config)` | Axios request config has been built, before request |
| `failed(errors, message)` | Server validation failure |
| `error(error)` | Non-validation request/lifecycle error |
| `success(response)` | Main request and Form-managed deferred upload phase succeed |
| `onRefreshed(data)` | Form reload completes |
| `initDetail()` | Registered detail instances have been constructed |

All Form hooks are async-compatible. Prefer `form.addValidation()` for field validation rather than
aborting a Page hook. Throwing aborts submission but enters Form's error path; Core has no standard
silent-cancellation result yet.

`save(showAlert)` delegates to `form.submit()` and returns its boolean result. The dirty state resets
only on successful save or refresh.

## Field initialization

Configure Form fields in `init()`:

```js
/**
 * @returns {void}
 */
init() {
	const account = this.form.get("account_id")
	const contact = this.form.get("contact_id")

	contact.init({
		param: body => {
			body.account_id = account.get()?.id ?? null

			return body
		},
	})

	this.form.addValidation("contact_id", async value => ({
		success: value !== null,
		message: "Contact is required",
	}))
}
```

When JavaScript supplies initialization config, defer the first initialization with `lazy`. Put `lazy`
on `<x-core::field>` for a wrapped input and call `init(config)` on the Field returned by Form. For a
standalone input, put `lazy` on that input and call `init(config)` on the standalone instance.

## Dirty state and navigation guard

After `init()`, SolarEditPage attaches `change` listeners to owned fields whose component supports that
event and `isChanged()`. Changes trigger the browser unload warning and a confirmation before normal
anchor navigation.

Consequences:

- Initialization values should be set silently: `field.set(value, true)`.
- Components without a supported `change` event are not tracked automatically.
- Programmatic mutations that do not emit `change` do not mark the Page dirty.
- Fields added after dirty tracking initializes are not tracked automatically.
- `discard()` is an empty extension point; overriding it does not automatically connect it to a
  standard button. Implement both invocation and reset behavior when needed.

## Refresh

`refreshForm()` reloads Form data using `this.id`. `refresh(detail=true)` reloads the Form, calls
`onRefreshed(data)`, clears dirty state, then requests refresh of registered details. Pass `false` to
skip detail refresh.

Use `refreshDetail(name)` for a specific registered detail by registration name or generated property
name. Refreshing all details waits for every detail refresh to finish.

When `save=false`, no `_save` Button exists; another control must call `save()`. The standard Refresh
Button is temporarily disabled and re-enabled after two seconds. Cancel behavior belongs to the Cancel
Button/navigation contract and is separate from the empty `discard()` hook.

## Profile panel

Supplying the `profile` slot creates a side panel and ribbon toggle. Collapse state is persisted in
`localStorage` under `solar-profile-collapsed` and therefore applies across Edit Pages. Public methods
are `collapseProfile()`, `expandProfile()`, and `toggleProfile()`; all safely no-op without a profile.

Use profile content for secondary identity/summary controls. Keep primary editable fields in the main
slot unless the side-panel interaction is intentional.

## List-detail integration

Required detail contract: [SolarListDetail](solar-list-detail.md).

Register detail definitions during `init()` so construction can occur immediately afterward:

```js
/**
 * @returns {void}
 */
init() {
	this.registerDetail("contacts", ContactDetail, "account_id")
}

/**
 * @returns {void}
 */
initDetail() {
	this.contacts.on("saved", () => this.onContactSaved())
}
```

`detailClass` must extend `SolarListDetail`. The detail name becomes a camel-case Page property. Supply
an explicit `parentValue` or SolarEditPage derives `{ id: this.id, name }`, where `name` comes from the
Form field identified by `displayValueName` (default `name`). Override `displayValueName` when the
record's display field uses another binding.

Before a detail saves, SolarEditPage saves a dirty parent without alerts. A failed parent save aborts
the detail save. After the parent succeeds, details lacking an explicit parent receive the current ID
and display value. Details on a create Page therefore depend on the Page/Form ID being updated by the
successful save flow.

Use `unregisterDetail(nameOrNames)` only before the removed detail is needed; it also deletes the
generated Page property.

## Checklist

- [ ] Read SolarPage, Form, and only the Component guides actually used.
- [ ] `id=null` is intentional for create; a record ID is supplied for edit.
- [ ] `model` or explicit `action` resolves to the intended endpoint.
- [ ] Page components use `this.get()`; fields use `this.form.get()`.
- [ ] Lazy fields are initialized in `init()`.
- [ ] Field validation uses `addValidation()` and returns `{ success, message }`.
- [ ] Programmatic initialization uses silent setters.
- [ ] `success()` updates any Page ID/state required after create.
- [ ] Detail definitions are registered in `init()` and customized in `initDetail()`.
- [ ] Profile collapse persistence across Edit Pages is acceptable.
