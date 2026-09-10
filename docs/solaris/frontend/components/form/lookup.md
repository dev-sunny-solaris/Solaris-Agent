# Lookup

Owner: Core

Source-relative paths:

- Blade: `resources/views/components/lookup.blade.php`
- JS: `resources/js/solaris/form/lookup-input.js`
- Backend model base: `src/Models/LookupModel.php`
- Backend service: `src/Services/Lookup.php`
- Backend controller: `src/Http/Controllers/LookupController.php`


Lookup is Solaris' default dropdown/reference input. Prefer Lookup over Select for selectable values.
Use Select only when the requirement explicitly needs a native/static Select behavior.

Inside a Form, access Lookup through `form.get(bindingKey)`; outside a Form, access its global
SolarUI instance through the owning Page.

## Data sources

### Static model options

Models implementing `SolarisLookup`—including `LookupModel` descendants—can provide options directly:

```blade
<x-core::lookup
    id="status_id"
    :options="SolarHelper::model('status')::toSelect()" />
```

Static items normalize to `{ id, name, selected? }` and are searched in the browser by `name`.
Static Lookup `set()` may receive an existing option ID or the complete item object.
Use `toSelect()` when a `LookupModel` supplies the static `options` prop. Do not use `toLookup()`
for this purpose.

When frontend logic needs a stable UUID from a static Lookup, import its named constant from the
owning package or application's `resources/js/const.js`. That file mirrors the canonical backend
constant. Never repeat the UUID literal inside Page or Component logic. This convention applies only
to static, stable Lookup records; dynamic Lookup values must come from the selected or returned item.

### Remote model/controller source

```blade
<x-core::lookup id="customer_id" source="account" pagination />
```

Lookup posts to `/core/lookup` with `X-Model: <source>`. `source` is registry shorthand, not a
requirement that the model extend `LookupModel`. Any resolved model implementing the Solaris lookup
contract and allowed by `LookupAccess` may serve it; an allowed custom controller source may handle
special responses.

Remote items must provide `id` and a display value normalized to `name`; additional properties are
preserved. Set a remote Lookup with a complete item object, not a bare ID, because the UI needs its
display data.

## Backend flow

`LookupController` validates the source through `Lookup::getConfig()`, calls an allowed custom
controller when selected, or delegates to `Lookup::get()`. The service prepares `id`, the model's
display-value column, and `extend_columns`; it uses model Prepared/Search/Filter/Sort scopes when
available. `LookupModel` supplies standard lookup access, caching, limit, display `name`, prepared
columns, search, and position ordering, but it is only one valid implementation.

Non-paginated, unfiltered `LookupModel` results may use model lookup cache. Dynamic request params
disable the shared JS response cache.

## Props

| Prop | Contract |
|---|---|
| `id`, `bind` | Visual ID and optional Form payload key; `id="visual:field_id"` is shorthand |
| `options` | Static items; use `LookupModel::toSelect()` for `LookupModel` descendants |
| `source` | Remote model/controller shorthand |
| `multiple` | Return one item versus an array |
| `value` | JSON-encoded initial item or item array |
| `pagination` | Remote infinite-scroll pagination; ignored without `source` |
| `extend_columns` | Additional item properties needed by logic/templates |
| `extend_search` | Additional backend search fields |
| `order_by` | `column` or `column:direction` |
| `placeholder`, `size` | Display configuration |
| `link_url`, `link_target` | Turn selected display into an entity link; `{id}`/nested placeholders supported |
| `copy` | Show selected-value copy action |
| `lazy` | Defer initialization so JS can provide config first |
| `table_lookup` | Use TableLookup for complex selection |

Use `table_lookup_columns` to declare TableLookup columns and `table_lookup_column` for its simple
single display column.

## Dynamic configuration

Use `lazy` when initialization requires callbacks. For a Lookup wrapped by Field, call `init()` on
the returned Field; Field forwards the config to its Lookup plugin:

```js
const account = this.form.get("account_id")
const contact = this.form.get("contact_id")

contact.init({
    param: (body) => {
        const accountId = account.get()?.id
        body.filter = accountId
            ? new Filter().and("account_id", accountId).get()
            : []
        return body
    },
})
```

`param(body)` runs for every remote request and may add filters or other custom request values.

Custom rendering is configured separately:

```js
lookup.init({
    optionsTemplate: (item) => `<strong>${item.name}</strong>`,
    selectionTemplate: (item) => `<span>${item.name} · ${item.code ?? ""}</span>`,
})
```

- `optionsTemplate(item)` renders dropdown rows.
- `selectionTemplate(item)` renders the selected value.
- Request every referenced property with `extend_columns`.
- Use `link` in JS or `link_url` in Blade instead of adding a manual click handler to selection.

For TableLookup, `columnRender` maps logical column names to `(cellData, rowData) => html` callbacks.

## Values and Form serialization

`get()` returns an item, item array, or `null`. `set(value, isSilent)` updates selection; pass
`true` when dependent initialization must not emit `change`. Form serializes single Lookup to its
ID and multiple Lookup to an ID array. Do not duplicate those IDs with `form.addData()`.

When Form loads a field keyed `customer_id`, it uses response relation `customer` to populate the
Lookup object. Edit responses must therefore include display data, not only the foreign key.

## Events

Lookup supports `change`, `search`, `open`, and `close`:

```js
this.form.get("customer_id").on("change", (value) => {
    this.form.set("contact_id", null, true)
}, "EditPage")
```

Use named handlers. Use Component events instead of native DOM listeners when the event is exposed.
