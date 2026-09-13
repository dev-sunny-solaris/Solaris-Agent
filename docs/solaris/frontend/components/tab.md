# Tab

Owner: Core

Source-relative paths:

- Blade: `resources/views/components/tab.blade.php`
- Nav Blade: `resources/views/components/tab-nav.blade.php`
- Content Blade: `resources/views/components/tab-content.blade.php`
- JS: `resources/js/solaris/tab.js`

Tab pairs `nav` and `content` slots. Use nested Tabs for grouped sections instead of manually toggling
panes.

## Blade structure

`<x-core::tab>` requires a unique `id` and two named slots. Put every `<x-core::tab-nav>` in `nav`
and every matching `<x-core::tab-content>` in `content`:

```blade
<x-core::tab id="account-tabs" variant="tab-style-7" class="p-0 mb-4">
	<x-slot:nav>
		<x-core::tab-nav id="account-general" active>General</x-core::tab-nav>
		<x-core::tab-nav id="account-address">Address</x-core::tab-nav>
		<x-core::tab-nav id="account-audit" disabled>Audit</x-core::tab-nav>
	</x-slot>

	<x-slot:content>
		<x-core::tab-content id="account-general" active>
			General content
		</x-core::tab-content>

		<x-core::tab-content id="account-address">
			Address content
		</x-core::tab-content>

		<x-core::tab-content id="account-audit">
			Audit content
		</x-core::tab-content>
	</x-slot>
</x-core::tab>
```

The TabNav `id` becomes its button target and must exactly equal the corresponding TabContent `id`.
IDs are document-global; prefix them with the Page/module identity. Mark exactly one initial pair as
`active`, applying it to both TabNav and TabContent. An active state on only one side produces
inconsistent navigation and pane visibility.

| Component | Prop | Contract |
|---|---|---|
| Tab | `id` | Root SolarUI identity; also generates `<id>_nav` and `<id>_content` |
| Tab | `variant` | Tab style class; defaults to `tab-style-6` |
| TabNav | `id` | Target pane ID; button ID becomes `<id>_button` |
| TabNav | `active` | Marks the initial active navigation item |
| TabNav | `disabled` | Prevents user activation |
| TabNav | `size` | `sm` applies compact navigation styling |
| TabContent | `id` | Must match one TabNav ID |
| TabContent | `active` | Adds the initial `show active` pane state |

Unknown attributes on Tab apply to the navigation `<ul>`. Attributes on TabNav apply to its `<li>`,
and attributes on TabContent apply to the pane `<div>`.

## JavaScript control

Resolve Tab through the owning Page using the root Tab ID:

```js
import SolarEditPage from "@core-js/solaris/solar/solar-edit-page"

export default class AccountEditPage extends SolarEditPage {
	/**
	 * @returns {void}
	 */
	init() {
		this.tabs = this.get("account-tabs")
		this.tabs.on("change", this.onTabChanged.bind(this), "AccountEditPage")
	}

	/**
	 * @param {string} targetId
	 * @param {Event} event
	 * @returns {void}
	 */
	onTabChanged(targetId, event) {
		this.activeTabId = targetId
	}
}
```

The `change` event fires after Bootstrap activates a tab and receives the target TabContent ID without
`#`, followed by the Bootstrap event.

`getActive()` returns the active target ID. `setActive(id)`, `show(idOrIds)`, `hide(idOrIds)`,
`enable(idOrIds)`, and `disable(idOrIds)` coordinate nav and pane state. Targets may include or omit
`#`; array arguments are supported by show/hide/enable/disable.

`show()` and `hide()` change visibility of both nav and pane but do not select another active tab.
Likewise, disabling or hiding the active tab does not choose a replacement; activate a valid target
explicitly.
