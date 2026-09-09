# SolarPage

Owner: Core

Source-relative paths:

- JS: `resources/js/solaris/solar/solar-page.js`

`SolarPage` is the base for every SolarUI Page and the direct base for bespoke screens unrelated to
the standard list or edit flows.

## Services and initialization

The constructor exposes:

| Property | Contract |
|---|---|
| `this.solarUI` | Shared `SolarUI` from `SolarSingleton` |
| `this.axios` | Solaris' configured Axios instance |
| `this.helper` | Shared Core `Helper` |

```js
constructor(autoInit = true)
```

Construction calls the overridden `init()` immediately by default. Put Page startup logic there:

```js
import SolarPage from "@core-js/solaris/solar/solar-page"

export default class ExamplePage extends SolarPage {
    init() {
        this.form = this.get("example-form")
    }
}
```

Do not add a constructor only to call `init()`. If subclass state must exist before initialization,
disable auto-init explicitly:

```js
constructor(options) {
    super(false)
    this.options = options
    this.init()
}
```

With auto-init enabled, fields assigned after `super()` do not exist when `init()` runs.

## Accessing Components

```js
const component = this.get("solar-id")
```

`get()` delegates to shared `SolarUI.get()` and returns the initialized Component class or `null`.
SolarFactory uses `solar-id`, falling back to HTML `id`. The element must also match a registered
selector, pass its guard, not carry `ignore`, and have initialized successfully. Do not use optional
chaining to conceal a required Component that failed to initialize.

Core `app.js` creates and initializes the one shared SolarUI before Page entries run. Never construct
a second SolarUI from a Page.

## BasePage Blade

Use Core `resources/views/components/page/base.blade.php`:

```blade
<x-core::page.base
    title="Example"
    :breadcrumbs="$breadcrumbs"
    package="core"
    path="example/index">

    <x-slot:buttons>
        {{-- Page-level actions --}}
    </x-slot>

    {{-- Default slot: main Page content --}}
</x-core::page.base>
```

| Prop | Contract |
|---|---|
| `title` | Sets layout and visible Page titles |
| `breadcrumbs` | Ordered breadcrumb items; omitted when empty |
| `package` | Selects `resource.path.<package>` as asset base |
| `path` | JS entry relative to `js/pages`, without `.js` |
| `model`, `action` | Unused legacy BasePage props; do not introduce in new usage |

Asset resolution is:

```text
valid package supplied: <resource.path.package>/js/pages/<path>.js
package omitted/not found: resources/js/pages/<path>.js
```

At package level, pass `package` for package-owned JS. At sandbox/project level, omit it for
app-owned JS; pass it only when deliberately loading an installed package entry. Never point into or
edit `vendor/`.

Slots:

- `buttons`: layout top-button section.
- Default: main Page content inside the inner Page component.

## Page entry convention

Package development must separate reusable logic and construction:

```js
// page.js or list.js
export default class ExamplePage extends SolarPage {}
```

```js
// page-index.js or list-index.js
import ExamplePage from "./page"

new ExamplePage()
```

Blade points to the `*-index` entry. This keeps the logic class importable and extendable by other
packages and projects.

At project level, a Page known to be application-specific may use one file that defines and
instantiates/exports the Page. Do not create a separate initializer mechanically.

## Breadcrumbs

Controllers pass:

```php
$breadcrumbs = [
    ['label' => 'Admin'],
    ['label' => 'Role'],
];
```

Each item supports `label` and optional `url`. The final item is always active text. Earlier items
are links; supply real URLs for navigable ancestors.

## Checklist

- [ ] The selected Page base matches the flow.
- [ ] Context is identified before choosing `package` and `path`.
- [ ] `path` resolves to an existing entry.
- [ ] Package logic and initializer are separate; project-only code may be direct.
- [ ] Startup logic is in `init()` and the Page is instantiated once.
- [ ] `autoInit` is disabled only when pre-initialization state is required.
- [ ] Components are obtained with `this.get()` rather than reconstructed.
- [ ] Shared Axios and Helper services are reused.
