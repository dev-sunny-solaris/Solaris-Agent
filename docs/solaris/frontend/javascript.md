# JavaScript Conventions

## Class and entry modules

Package Pages separate reusable logic from execution:

```js
// page.js
import SolarEditPage from '@core-js/solaris/solar/solar-edit-page'

export default class AccountPage extends SolarEditPage {
	/**
	 * @returns {void}
	 */
	init() {
		this.name = this.form.get('name')
	}
}
```

```js
// page-index.js
import AccountPage from './page'

new AccountPage()
```

Use aliases across package boundaries and for reusable package modules. Relative imports are acceptable
between tightly coupled sibling Page files.

## Interaction rules

- Use `this.get(id)` for global Components and `this.form.get(bindingKey)` for Form-owned controls.
- Prefer public APIs and events over DOM access.
- Use named listeners when supported.
- Await asynchronous operations and restore loading/disabled state in `finally`.
- Do not add empty Page action or lifecycle overrides.
- Do not create another SolarUI instance or globally reinitialize it for one fragment.

Follow each Page guide's constructor order. Do not add a constructor merely to call `init()`.
