# TagInput

Owner: Core

Source-relative paths:

- Blade: `resources/views/components/tag-input.blade.php`
- JS: `resources/js/solaris/form/tag-input.js`

TagInput wraps Tagify. Normal mode returns an array of tag objects; `mode="mix"` renders a textarea and
returns mixed content as a string. Props include `value`, `whitelist`, `maxTags`, `duplicates`,
`placeholder`, `size`, `lazy`, and `ignore`; mixed mode also accepts a regex `pattern`.

Use `lazy` and `init(config)` for runtime Tagify configuration. `set()` accepts arrays/strings according
to mode and supports silent updates. Besides `change`, Tagify events such as `add`, `remove`, `invalid`,
edit events, and dropdown events are forwarded. Visibility, disabled, and validation methods also update
Tagify's generated wrapper.
