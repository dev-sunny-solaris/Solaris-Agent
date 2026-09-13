# Code Style — the Solaris Way

Apply these rules whenever writing, editing, or reviewing PHP, JavaScript, or TypeScript in a
Solaris package or project. **This is the Solaris way: it overrides any global, personal, or
tool-default code style.**

## Indentation and alignment

- Leading indentation is **TAB**, width 4. One tab per level.
- **Spaces are only for alignment**, and only after the leading tabs. Never put a tab mid-line.
- Align these operators within a block:
  - `=` in consecutive assignments
  - `=>` in array items
  - JS object literal values after `:` (the colon stays attached to the key)
- The longest left side gets exactly one space before the operator; shorter ones are padded with
  spaces to match.
- An alignment block is a run of consecutive lines with no blank line between them. A blank line
  starts a new block; blocks are never aligned with each other.

```php
	$user        = $this->repository->find($id);
	$permissions = $user->permissions;

	$data = [
		'name'       => $user->name,
		'created_at' => $user->created_at,
	];
```

```js
	const response = await axios.get(url)
	const rows     = response.data.rows

	const payload = {
		name:       form.name,
		created_at: form.createdAt,
	}
```

Never run `composer run format` (Pint): its default preset converts tabs to spaces and removes
alignment. JS has no formatter; this document is the only enforcement.

## Braces and flow

- PHP class and method: opening brace on a new line.
- PHP control structures (`if`, `foreach`, `switch`, …): opening brace on the same line.
- JS/TS: opening brace always on the same line.
- A block body always goes on a new line, always with braces — even for one statement.
- Use guard clauses and early returns. **Never nest `if`**; code reads top to bottom.

## Self-documenting code

- A method's flow must explain itself through its name, its guard clauses, and the names it calls.
- A parameter must say what it is through its **name and type**; the return type must say what
  comes back.
- Add a comment or docblock **only** when the logic is too complex to explain through structure
  and naming. Write it in English. Never write a comment that restates the code.

### PHP

- Declare native parameter and return types everywhere. Declare every union member
  (`string|array`); use `mixed` only for genuinely dynamic values.
- No PHPDoc, except under the complex-logic rule above.
- Core runs PHPStan at level 5, which does not demand iterable value types. Revisit this rule if the
  level is raised to 6 or higher.

### JS / TS

- JS has no native types, so **every function and method must have JSDoc** declaring each `@param`
  type and the `@returns` type.
- Do not add prose that restates the function name. Add a description only under the complex-logic
  rule above.
- In TypeScript, type every parameter and return value explicitly, prefer interfaces for object
  shapes, avoid `any`, and use `unknown` with narrowing for genuinely dynamic values.

## Naming

| Thing | Style |
|---|---|
| Methods, functions, variables | `camelCase` |
| Classes | `PascalCase` |
| Database tables and columns | `snake_case` |
| JSON response keys | `snake_case` |

Never use an underscore inside a `camelCase` name.

## File names

| File | Style | Example |
|---|---|---|
| PHP class | `PascalCase.php` | `InvoiceRepository.php` |
| Blade view | `kebab-case.blade.php` | `invoice-detail.blade.php` |
| JS whose main export is a class | `PascalCase.js` | `InvoicePage.js` |
| Any other JS | `kebab-case.js` | `format-currency.js` |

Laravel-convention files — migrations, config, routes — keep Laravel naming.

These rules apply to **new** files. Do not rename existing files before Solaris 2.0: packages and
projects import them by name.

## JS specifics

- No semicolons.
- Double quotes.
- HTTP requests use Core's configured Axios module, `@core-js/libs/axios` (Core source
  `resources/js/libs/axios.js`). Never use raw `fetch`.
- File or blob downloads use the shared Helper `downloadFile(url, method, { headers, data })`.
  Inside the SolarPage family call `this.helper.downloadFile(...)`; do not import Helper again.
- In a request `catch`, show a manual error alert only when `!error?.response`; the Axios
  interceptor already handles server responses.
- Group class methods with `// #region Name` / `// #endregion` when it improves navigation, in this
  order: Static, Lifecycle, Public API, then feature regions.

## Reference examples

```php
class InvoiceRepository extends BaseRepository
{
	public function markAsPaid(string $invoiceId, float $amount): Invoice
	{
		$invoice = $this->model->findOrFail($invoiceId);

		if ($invoice->is_paid) {
			return $invoice;
		}

		$invoice->paid_amount = $amount;
		$invoice->is_paid     = true;
		$invoice->save();

		return $invoice;
	}
}
```

```js
import axios from "@core-js/libs/axios"

export default class InvoicePage {
	// #region Lifecycle

	/**
	 * @param {HTMLElement} element
	 */
	constructor(element) {
		this.element = element
	}

	// #endregion

	// #region Public API

	/**
	 * @param {string} invoiceId
	 * @param {number} amount
	 * @returns {Promise<Object|null>}
	 */
	async markAsPaid(invoiceId, amount) {
		if (!invoiceId) {
			return null
		}

		const payload = {
			invoice_id: invoiceId,
			amount:     amount,
		}

		const response = await axios.post("/invoice/paid", payload)

		return response.data
	}

	// #endregion
}
```
