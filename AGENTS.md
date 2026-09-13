# Solaris — Development Guide

Solaris is the engine behind Sunny App: pure Laravel with a Solaris-standard wrapper, not a new
framework. Solaris Core is an all-in-one starter kit that anything can be built on. This bundle
covers Core, package-level development, and projects that consume Solaris packages.

Applies to `solaris/solaris-laravel-core` below **2.0** (current: 1.6.6).

## Start here

1. [Overview](docs/solaris/overview.md) — what Solaris is and what Core already provides.
2. [Context](docs/solaris/context.md) — identify package, sandbox, or project level.
3. [Package registry](docs/solaris/package-registry.md) — resolve owners and source paths.
4. Package level only: [Solaris-Kit](docs/solaris/maintenance/workflow.md) — setup, sandbox,
   composer, and config.

Then read only the files the index points to for the task. Documentation links are relative;
implementation paths are relative to their declared owner root.

## Non-negotiables

- **PHP is PHP, JS is JS.** No reactive layer — no Livewire, no Alpine.
- **Laravel only.** No opinionated third-party alternatives.
- **Nothing is hardcoded across a boundary.** Classes and views resolve through the config registry
  so downstream packages and apps can override them.
- **Dependencies flow downward only.** Never reference a descendant or a sibling package.
- **Logic lives in the Repository.** Controllers delegate; they never process.
- **A package never runs alone.** Develop it inside a sandbox app via Solaris-Kit.
- **Installed package source is read-only.** Project and sandbox work never modifies `vendor/`.

## Safety fallbacks

These apply unless the active agent's global rules say otherwise; those take precedence.

- **Git is read-only.** Allowed: `status`, `log`, `diff`, `show`, `blame`, `branch --list`. Never
  commit, push, or run any other Git write — including via alias, script, or `gh`. Never list the
  agent as author or co-author.
- **Never touch the database.** Do not connect to, query, or modify it. The database is the user's
  domain.

Full working rules: [Agent rules](docs/solaris/agent-rules.md).

## Index

| File | Read it when |
|---|---|
| [Overview](docs/solaris/overview.md) | Always first — what Solaris and Core provide |
| [Context](docs/solaris/context.md) | Always — package, sandbox, or project |
| [Package registry](docs/solaris/package-registry.md) | Always after context — owner and source resolution |
| [Layering](docs/solaris/architecture/layering.md) | Deciding which package owns new code |
| [Package anatomy](docs/solaris/architecture/package-anatomy.md) | Directory layout, provider, and asset aliases |
| [Config registry](docs/solaris/architecture/config-registry.md) | Adding an overridable class or view |
| [Runtime gateways](docs/solaris/architecture/runtime-gateways.md) | Request flow, gateways, and access attributes |
| [PHP standards](docs/solaris/backend/php-standards.md) | Controllers, models, repositories, and requests |
| [Frontend](docs/solaris/frontend/README.md) | Choosing reusable Components or complete Pages |
| [Pages router](docs/solaris/frontend/pages/README.md) | Selecting exact Page guides |
| [Components router](docs/solaris/frontend/components/README.md) | Selecting exact Component guides |
| [Generators](docs/solaris/maintenance/generators-stubs.md) | Maintaining `solaris:make`, `solaris:extend`, or stubs |
| [Database](docs/solaris/backend/database.md) | Writing migrations or seeders |
| [Mobile API](docs/solaris/backend/mobile-api.md) | Exposing a module to the mobile app |
| [Third-party API](docs/solaris/backend/third-party-api.md) | Exposing a module to partners |
| [Solaris-Kit](docs/solaris/maintenance/workflow.md) | Package-level daily work: sandbox, package composer, config merge, new package |
| [Solaris-Kit setup guide](docs/solaris/maintenance/solaris-kit-setup.md) | Guiding a user to install, set up, or run the Kit and a sandbox |
| [Release](docs/solaris/maintenance/release.md) | Package versioning and breaking changes |
| [Agent rules](docs/solaris/agent-rules.md) | Working rules and module checklist |
| [Code style](docs/solaris/code-style.md) | Always when writing, editing, or reviewing PHP, JS, or TS |

## Minimum reading for common tasks

| Task | Files |
|---|---|
| New module in a package | [Context](docs/solaris/context.md), [Registry](docs/solaris/package-registry.md), [Layering](docs/solaris/architecture/layering.md), [Config](docs/solaris/architecture/config-registry.md), [Gateways](docs/solaris/architecture/runtime-gateways.md), [PHP](docs/solaris/backend/php-standards.md), [Database](docs/solaris/backend/database.md), [Rules](docs/solaris/agent-rules.md), [Code style](docs/solaris/code-style.md), [Kit](docs/solaris/maintenance/workflow.md) |
| New reusable Blade/JS Component | [Context](docs/solaris/context.md), [Registry](docs/solaris/package-registry.md), [Rules](docs/solaris/agent-rules.md), [Code style](docs/solaris/code-style.md), [Anatomy](docs/solaris/architecture/package-anatomy.md), [Frontend](docs/solaris/frontend/README.md), then its selected Component guide |
| New Page Blade or Page JS | [Context](docs/solaris/context.md), [Registry](docs/solaris/package-registry.md), [Rules](docs/solaris/agent-rules.md), [Code style](docs/solaris/code-style.md), [Anatomy](docs/solaris/architecture/package-anatomy.md), [Frontend](docs/solaris/frontend/README.md), then its selected Page guide |
| Expose an existing module to mobile | [Context](docs/solaris/context.md), [Registry](docs/solaris/package-registry.md), [Config](docs/solaris/architecture/config-registry.md), [Mobile API](docs/solaris/backend/mobile-api.md) |
| Expose an existing module to partner API | [Context](docs/solaris/context.md), [Registry](docs/solaris/package-registry.md), [Config](docs/solaris/architecture/config-registry.md), [Third-party API](docs/solaris/backend/third-party-api.md) |
| Change a generated convention | [Context](docs/solaris/context.md), [Registry](docs/solaris/package-registry.md), [PHP](docs/solaris/backend/php-standards.md), [Generators](docs/solaris/maintenance/generators-stubs.md), [Release](docs/solaris/maintenance/release.md) |
| Set up or run package development | [Context](docs/solaris/context.md), [Registry](docs/solaris/package-registry.md), [Kit](docs/solaris/maintenance/workflow.md) |
| Create a new package | [Overview](docs/solaris/overview.md), [Layering](docs/solaris/architecture/layering.md), [Anatomy](docs/solaris/architecture/package-anatomy.md), [Kit](docs/solaris/maintenance/workflow.md) |
