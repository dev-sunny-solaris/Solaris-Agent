# Checkbox, Radio, Switch, and Groups

Owner: Core

Source-relative paths:

- Checkbox Blade: `resources/views/components/checkbox.blade.php`
- Radio Blade: `resources/views/components/radio.blade.php`
- Group Blades: `resources/views/components/checkbox-group.blade.php`, `resources/views/components/radio-group.blade.php`
- JS: `resources/js/solaris/form/checkbox-input.js`, `resources/js/solaris/form/radio-input.js`
- Group JS: `resources/js/solaris/form/option-group-input.js`

Checkbox returns boolean. Radio returns `{ id, name }` only when checked. `switch` is a Checkbox display
mode, not a separate component. Both support visual variants, `checked`, `hidden`, `lazy`, `ignore`, and
card rendering; radio choices sharing a group need the same `name`.

Use group components when one binding owns several options. A radio group returns one `{ id, name }`;
a checkbox group returns all option objects with `{ id, name, checked }`. Form serializes radio groups
to the selected ID and preserves checkbox-group results. Group `set()` accepts IDs or objects; checkbox
groups also accept `{ id, checked }` instructions. Only `change` is supported.

Do not register group children independently: their generated children are ignored and the wrapper owns
the Form binding.

Card variants use dedicated radio/checkbox and group plugins but preserve the same conceptual value
shapes. Use card rendering only when the choice itself is the visual card; do not recreate card selection
with click handlers.
