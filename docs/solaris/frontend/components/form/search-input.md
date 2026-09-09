# SearchInput

Owner: Core

Source-relative paths:

- Blade: `resources/views/components/search-input.blade.php`
- JS: `resources/js/solaris/form/search-input.js`

SearchInput extends TextInput and adds a `search` event. It emits after 450 ms of input debounce, on the
search icon, and after reset; reset emits a `null`/empty value. `show()` and `hidden()` affect the wrapper,
not only the input.

```blade
<x-core::search-input id="search" />
```

```js
this.form.get('search').on('search', value => this.reload(value), 'SearchFilter')
```

Use `form.exclude('search')` when a Form-owned search must stay out of submit data. If `ignore` is
rendered, Form and global SolarUI do not register it; do not retrieve it through `this.form.get()`.
