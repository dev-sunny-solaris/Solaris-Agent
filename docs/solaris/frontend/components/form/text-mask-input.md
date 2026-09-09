# Text and Mask Input

Owner: Core

Source-relative paths:

- Blade: `resources/views/components/input.blade.php`
- Textarea Blade: `resources/views/components/textarea.blade.php`
- JS: `resources/js/solaris/form/text-input.js`
- Mask JS: `resources/js/solaris/form/mask-input.js`

Use `<x-core::input>` for plain and masked single-line values and `<x-core::textarea>` for multiline
text. Both support `id`/`bind`, `lazy`, sizing, and ordinary HTML attributes. Input additionally supports
`ignore`; textarea currently does not declare it.

Mask modes are mutually constrained by Blade: `number`, `numeric`, `phone`, and `credit_card`. Other
options include `uppercase`, `lowercase`, `prefix`, `blocks`, and `delimiters`. Number mode uses
`thousand_separator`, `decimal_separator`, and `decimal`; `get()` returns a number or `null`, not the
formatted display. `numeric` means digits-only and still returns a string.

```blade
<x-core::input id="amount" number :decimal="2" />
<x-core::input id="phone" phone="ID" />
<x-core::textarea id="description" />
```

Supported events: `input`, `change`, keyboard events, `focus`, and `blur`. `set(value, true)` suppresses
the synthetic change event. Use `getFormatted()` only for display concerns; payload uses `get()`.
