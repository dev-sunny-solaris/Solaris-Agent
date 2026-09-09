# Frontend Layer (SolarUI)

SolarUI is a Blade-first component system on Bootstrap 5. Components render HTML with declarative DOM
attributes; JS auto-initializes from those attributes. **There is no reactive server round-trip —
it is not Livewire, and Solaris deliberately has no Livewire/Alpine layer. PHP is PHP, JS is JS.**

Resolve terms such as Core, MasterData, and Sales—and every source-relative path—through
[Package registry](../package-registry.md). Never infer an absolute package repository path
or edit an installed/mirrored `vendor/` copy.

## Choose the correct frontend path

SolarUI frontend work has two distinct paths. Determine which one the request targets before
reading or writing implementation code:

| Path | Use it for | Required reading |
|---|---|---|
| **Component** | Reusable atomic UI behavior, or a reusable composition of atomic UI elements | This file and [Components router](components/README.md) |
| **Page** | A complete screen coordinating components and page-specific behavior | This file and [Pages router](pages/README.md) |

A Page may use many Components. Do not move page-specific orchestration into a reusable Component,
and do not reimplement an existing Component inside a Page.

## Solaris way

Choose the least-custom layer that satisfies the interaction:

```text
existing Blade prop or slot
→ existing Component public API
→ existing Component event
→ Page hook or orchestration
→ plain Bootstrap behavior
→ reusable custom Component
→ direct DOM manipulation as the last resort
```

Blade owns structure, identity, initial state, and declarative configuration. Components own reusable
behavior. Pages coordinate Components. Keep standard Table, ModalForm, and Form behavior unless the task
requires a real replacement; empty overrides disable useful defaults.

Required conventions: [Blade](blade.md), [JavaScript](javascript.md), [Events](events.md), and
[Ownership and lazy semantics](ownership-lazy.md).

## Component runtime

Core's `app.js` constructs one `SolarUI`, publishes it through `SolarSingleton`, and calls
`init()`. `SolarUI` holds a **static** registry of `{ selector, component, guard, factory }`
entries; `init()` queries each selector and instantiates the matching class through `SolarFactory`,
destroying the previous instances first on a re-init.

ModalForm compositions are created at the end of `SolarUI.init()`, after their nested Modal and Form
instances exist. Page code may resolve `<modal-id>_ModalForm` only after this initialization.

```js
SolarUI.register(selector, ComponentClass, guard = null, factory = null)
```

`guard(el) → boolean` filters elements; `factory(el, ComponentClass)` builds the instance when
construction is not trivial.

Because the registry is static and read at `init()`, **registration must happen before `init()`
runs**. Inside Core, import and register a new global Component in `_registerDefaultComponents()` in
`solar-ui.js`. Outside Core, registration belongs in the consumer's global/bootstrap entry that runs
before Core initialization—not in a normal Page entry loaded afterward.

Base classes and runtime services live in core: `SolarComponent`, `SolarPage`, `SolarListPage`,
`SolarEditPage`, `SolarListDetail`, `SolarModalPage`, `SolarSingleton`, `SolarFactory`.

When adding an auto-initialized DOM component:

1. Extend `SolarComponent`.
2. Register it — in core, add it to `_registerDefaultComponents()`; elsewhere, call
   `SolarUI.register()` from an entry file that loads before `init()`.
3. Ship matching Blade that renders the selector and declarative attributes it reads.

Page JS (`resources/js/pages/<module>/…`) is loaded per screen and is not registered as a Component.
The reusable Page class is exported from its module; the Blade-selected entry imports it and executes
`new PageClass()` exactly once. Bespoke screens extend `SolarPage`; standard list/edit screens use their
specialized bases. `SolarListDetail` is different: export it, register it through its parent
`SolarEditPage`, and let the parent construct it. Read the [Pages router](pages/README.md).

## DOM identity and ownership

| Attribute | Contract |
|---|---|
| `solar-ui` | Selects the Component/factory contract |
| `solar-id` | Global SolarUI identity; HTML `id` is the fallback |
| `solar-bind` | Form ownership and payload key; defaults to component ID |
| `ignore` | Excludes the element from the relevant automatic ownership/initialization flow |
| `lazy` | Defers initialization only when that Component explicitly implements lazy behavior |

Use Page `this.get(id)` for globally registered Components. Use `this.form.get(bindingKey)` for Field
and input Components owned by a Form.

## Initialization order

```text
Blade renders DOM contracts
→ global/bootstrap entry registers custom Components
→ Core app initializes SolarUI Components
→ Blade-selected Page entry executes new PageClass()
→ Page resolves Components and coordinates interactions
→ SolarEditPage constructs registered SolarListDetail classes
```

`ModalForm` is a reusable Component composition. `SolarModalPage` is Page-level orchestration hosted in
a modal flow; read its dedicated Page guide before using it.

## Re-initialization and dynamic DOM

Do not call `solarUI.init()` merely to initialize one inserted fragment. A second call destroys and
recreates every registered instance, making Page references and Page-installed listeners stale. Prefer
Blade-rendered Components, a Component's own render/add API, or explicitly owned manual instances.

Never construct a second `SolarUI`. Its registry is static, so another constructor registers the
default selectors again.

Before creating a new JS component, check whether an existing Solaris component or a plain
Bootstrap behavior already covers it. Do not write a bespoke component for a small piece of UI.

## Views

Package Blade lives in `resources/views/pages/<module>/` and `resources/views/components/`. Common Page
entries are `list.blade.php`, `page.blade.php`, and `modal.blade.php`; modules may also organize
`sections/`, nested detail folders, tab content, and partials. Page views are exposed through
`resource.views.<module>` in `config/solaris.php` so consumers can override them.
