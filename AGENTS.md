# Solaris — Development Guide

Instructions for AI coding agents working in Solaris package repositories and Laravel applications
that consume Solaris. Rules differ by context; identify it before selecting files or conventions.

Solaris is a Laravel package starter kit: a backend architecture plus a frontend UI system, shipped
as layered Composer packages. It is not a new framework — it runs on Laravel and follows Laravel
conventions.

**Read [Context](docs/solaris/context.md) first**, then
[Package registry](docs/solaris/package-registry.md), to identify context and resolve package owners.
Then read only the files this index points to for the work at hand. Documentation links are relative;
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

## Index

| File | Read it when |
|---|---|
| [Context](docs/solaris/context.md) | Always first — package, sandbox, or project |
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
| [Workflow](docs/solaris/maintenance/workflow.md) | Running, testing, or installing through Solaris-Kit |
| [Release](docs/solaris/maintenance/release.md) | Package versioning and breaking changes |
| [Agent rules](docs/solaris/agent-rules.md) | Working rules and module checklist |

## Minimum reading for common tasks

| Task | Files |
|---|---|
| New module in a package | [Context](docs/solaris/context.md), [Registry](docs/solaris/package-registry.md), [Layering](docs/solaris/architecture/layering.md), [Config](docs/solaris/architecture/config-registry.md), [Gateways](docs/solaris/architecture/runtime-gateways.md), [PHP](docs/solaris/backend/php-standards.md), [Database](docs/solaris/backend/database.md), [Rules](docs/solaris/agent-rules.md) |
| New reusable Blade/JS Component | [Context](docs/solaris/context.md), [Registry](docs/solaris/package-registry.md), [Anatomy](docs/solaris/architecture/package-anatomy.md), [Frontend](docs/solaris/frontend/README.md), then its selected Component guide |
| New Page Blade or Page JS | [Context](docs/solaris/context.md), [Registry](docs/solaris/package-registry.md), [Anatomy](docs/solaris/architecture/package-anatomy.md), [Frontend](docs/solaris/frontend/README.md), then its selected Page guide |
| Expose an existing module to mobile | [Context](docs/solaris/context.md), [Registry](docs/solaris/package-registry.md), [Config](docs/solaris/architecture/config-registry.md), [Mobile API](docs/solaris/backend/mobile-api.md) |
| Expose an existing module to partner API | [Context](docs/solaris/context.md), [Registry](docs/solaris/package-registry.md), [Config](docs/solaris/architecture/config-registry.md), [Third-party API](docs/solaris/backend/third-party-api.md) |
| Change a generated convention | [Context](docs/solaris/context.md), [Registry](docs/solaris/package-registry.md), [PHP](docs/solaris/backend/php-standards.md), [Generators](docs/solaris/maintenance/generators-stubs.md), [Release](docs/solaris/maintenance/release.md) |
| Set up or run package development | [Context](docs/solaris/context.md), [Registry](docs/solaris/package-registry.md), [Workflow](docs/solaris/maintenance/workflow.md) |
