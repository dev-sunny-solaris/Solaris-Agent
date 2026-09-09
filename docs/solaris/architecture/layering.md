# Package Map and Layering

```
solaris-laravel-core                     namespace Solaris\Core          view core::
└── solaris-laravel-masterdata           namespace Solaris\MasterData    view master-data::
    ├── solaris-laravel-basecrm          namespace Solaris\BaseCRM       view base-crm::
    │   ├── solaris-laravel-sales        namespace Solaris\Sales         view sales::
    │   └── solaris-laravel-marketing    namespace Solaris\Marketing     view marketing::
    └── solaris-laravel-service          namespace Solaris\Service       view service::

solaris-laravel-crm depends on sales + marketing + service and composes the complete CRM stack.
```

Dependency flows **downward only**. A package may depend on any ancestor, never on a descendant
or a sibling. Sales must not reference Marketing, and Core must not reference anything above it.

## Domain ownership

| Package | Owns |
|---|---|
| core | Auth, users, roles, permissions, menu, gateways, UI system, approval, excel, AI, notifications, WhatsApp |
| masterdata | Accounts, Contacts, Products, Activities, Knowledge Base, Documents |
| basecrm | Leads (shared by Sales and Marketing); extends Account / Contact / Activity |
| sales | Opportunities, Orders, Invoices, Contracts |
| marketing | Email Blast, WhatsApp Blast, Events, Email Template, WA Template |

Route prefixes: masterdata `/master-data`, sales `/sales`, marketing `/marketing`. basecrm serves
Lead under `/crm`; Sales and Marketing each expose their own scoped Lead routes under their own
prefix on top of it, which is what "shared between Sales and Marketing" means in practice.

## Where does new code go?

1. Generic infrastructure with no business meaning (a UI component, a trait, a gateway, a helper)
   → **core**.
2. A business entity → the lowest package that owns that domain.
3. Would placing it in package X force X to know about a package below it? → move it up, or invert
   with an extension point (config indirection, event, attribute).

Never add a dependency to a package's `composer.json` just to reach one class. Move the class down
instead, or expose a hook. See [Package registry](../package-registry.md) for canonical owner
terms and identities.
