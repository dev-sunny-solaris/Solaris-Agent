# Solaris Overview

## What Solaris is

Solaris is the platform/engine behind Sunny App. It ships as Laravel packages built on the Spatie
package skeleton. It is **not a new framework** — it is pure Laravel with a wrapper on top that
defines the Solaris standard: backend architecture, frontend UI system, and development flow.

## Solaris Core

Core is a ready-to-use, all-in-one starter kit. Everything common is already built in and
extendable, so work focuses on the business process only:

- Auth (login, register, password reset, layouts) with Google and Entra ID SSO
- Users, roles, permissions, and object-level and record-level access rights
- Approval flow, menu management, and dynamic lookups via `LookupModel`
- SolarUI: Bootstrap-based Blade components with auto-wired JS, DataTables with dynamic filters
- Code generator (document/transaction numbering), kanban, Excel export/import, metric dashboards
- Attachments with chunked upload; in-app, web push, and email notifications
- WhatsApp integration and WhatsApp AI assistant (`AiToolRegistry`)
- `solaris:make` / `solaris:extend` generators

**Check Core before building anything. Extend it; never duplicate it.**

## Scope

Core is a general-purpose base engine — anything can be built on it. Domain packages (for example
the CRM family) are consumers of Core like any other package. This bundle covers only:

- **Core** itself
- **Package level** — developing any package built on Core
- **Project level** — any Laravel application that consumes Solaris packages

A domain package documents its own domain; do not assume one from this bundle.

Package-level development runs through Solaris-Kit — see [Solaris-Kit](maintenance/workflow.md).
