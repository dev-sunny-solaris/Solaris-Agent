# Package Layering

Core is the root of every Solaris package graph. Every other package sits on top of it, and its
parents are exactly the `solaris/*` packages listed under `require` in its `composer.json`.

```
solaris-laravel-core
└── solaris-laravel-<a>             require: core
    ├── solaris-laravel-<b>         require: <a>
    └── solaris-laravel-<c>         require: <a>
```

Resolve the real graph from the packages themselves: each `composer.json`, or
`solaris package list <key>` for the full chain in dependency order.

## Dependency rule

Dependency flows **downward only**. A package may reference only its transitive `require` set —
Core and the packages it requires, directly or indirectly. It must never reference:

- a descendant (a package that requires it),
- a sibling (a package sharing a parent but not required by it),
- any unrelated package.

In the diagram, `<b>` may use `<a>` and Core, but never `<c>`. Core references nothing above it.

## Where does new code go?

1. Generic infrastructure with no business meaning (a UI component, a trait, a gateway, a helper)
   → **Core**.
2. A business entity → the lowest package that owns that domain.
3. Would placing it in package X force X to know about a package above it? → move it up, or invert
   with an extension point (config indirection, event, attribute).

Never add a dependency to a package's `composer.json` just to reach one class. Move the class down
instead, or expose a hook. See [Package registry](../package-registry.md) for identities.
