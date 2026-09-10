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

## Static Lookup constants

If frontend logic needs a UUID belonging to a static, stable Lookup record, define the canonical
constant on the backend and mirror it in the owning package or application's `resources/js/const.js`:

```js
const CaseStatus = {
	New: '019b5a10-0000-7005-8000-000000000001',
	InProgress: '019b5a10-0000-7005-8000-000000000002',
	WaitingForResponse: '019b5a10-0000-7005-8000-000000000003',
	Resolved: '019b5a10-0000-7005-8000-000000000004',
	Closed: '019b5a10-0000-7005-8000-000000000005',
}

const ServiceAgreementStatus = {
	Draft: '019b5a10-0000-7001-8000-000000000001',
	Active: '019b5a10-0000-7001-8000-000000000002',
	Completed: '019b5a10-0000-7001-8000-000000000003',
	Canceled: '019b5a10-0000-7001-8000-000000000004',
}

export {
	CaseStatus,
	ServiceAgreementStatus,
}
```

Import the named constant through the owning asset alias, then use its key instead of embedding a
UUID in Page or Component logic:

```js
import { CaseStatus } from '@service-js/const'

const isResolved = statusId === CaseStatus.Resolved
```

Keep object names, keys, and UUID values synchronized with the backend constants. Use this pattern
only for static Lookup records. Obtain dynamic Lookup IDs from selected or returned data.

## Interaction rules

- Use `this.get(id)` for global Components and `this.form.get(bindingKey)` for Form-owned controls.
- Prefer public APIs and events over DOM access.
- Use named listeners when supported.
- Await asynchronous operations and restore loading/disabled state in `finally`.
- Do not add empty Page action or lifecycle overrides.
- Do not create another SolarUI instance or globally reinitialize it for one fragment.

Follow each Page guide's constructor order. Do not add a constructor merely to call `init()`.
