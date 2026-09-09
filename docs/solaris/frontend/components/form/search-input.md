# SearchInput

Owner: Core

Source-relative paths:

- Blade: `resources/views/components/search-input.blade.php`
- JS: `resources/js/solaris/form/search-input.js`

SearchInput extends TextInput and adds a `search` event. It emits after 450 ms of input debounce, on the
search icon, and after reset; reset emits a `null`/empty value. `show()` and `hidden()` affect the wrapper,
not only the input.

```blade
<x-core::search-input id="search" ignore />
```

```js
this.form.get("search").on("search", value => this.reload(value))
```

Use `ignore` when search filters UI and must not enter a submit payload.
