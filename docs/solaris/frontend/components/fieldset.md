# Fieldset

Owner: Core

Source-relative paths:

- Blade: `resources/views/components/fieldset.blade.php`

Fieldset visually separates related form sections. It is a Blade/Bootstrap composition, not a
SolarUI JavaScript Component; do not resolve it through Page `this.get()`.

## Divider mode

Without `accordion`, Fieldset renders only a labeled divider. It does not render its default slot:

```blade
<x-core::fieldset label="Billing address" />
```

Use divider mode only as a heading between sibling content. Place fields after it, not inside it:

```blade
<x-core::fieldset label="Billing address" />

<x-core::field id="billing_address" label="Address">
	<x-core::textarea />
</x-core::field>
```

## Accordion mode

Use `accordion` when Fieldset must own and render section content. Supply a unique `id` because
Bootstrap Collapse uses it as the target:

```blade
<x-core::fieldset
	id="account-address"
	label="Address"
	accordion
	:open="false">

	<x-slot:actions>
		<x-core::button id="copy-address" size="sm">Copy</x-core::button>
	</x-slot>

	<x-core::field id="address" label="Address">
		<x-core::textarea />
	</x-core::field>
</x-core::fieldset>
```

| Prop/slot | Contract |
|---|---|
| `label` | Required section heading |
| `accordion` | Enables collapsible content and renders the default slot |
| `id` | Required and document-unique in accordion mode |
| `open` | Initial collapse state; defaults to `true` |
| `actions` | Header actions; rendered only in accordion mode |
| Default slot | Section content; rendered only in accordion mode |

Normal attributes apply to the outer `.solar-fieldset` only in accordion mode. Fieldset does not alter
Form ownership: Fields nested in an accordion remain owned, serialized, and validated by the enclosing
Form.
