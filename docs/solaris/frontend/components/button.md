# Button

Owner: Core

Source-relative paths:

- Blade: `resources/views/components/button.blade.php`
- JS: `resources/js/solaris/button.js`

Button supplies Bootstrap/Solaris variants, size, color, rounded/square/icon styling, and one `click`
event. Use `show`, `hidden`, `enabled`, and `disabled` for state. `load()` replaces content with a loader
and disables the button; `unload()` restores its previous nodes and enables it.

Use `type="submit"` only when native/Form submission is intended. Prefer a named Component listener over
a native click listener when Page logic owns the action.
