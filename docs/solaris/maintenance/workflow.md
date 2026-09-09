# Development Workflow

**A package cannot run on its own.** Development and testing happen in a host Laravel *sandbox*
with the packages linked into it — never in the package repository directly, and never by editing
a sandbox's `vendor/` tree.

The toolchain is **Solaris-Kit**, a single Go binary on `PATH` (`solaris`). It supervises the
sandbox's processes, mirrors package edits into it, and handles the composer, scaffolding and
packaging chores around it. It drives PHP, composer, node and npm — it does not replace them.

Two ways to run it:

```
solaris                    a shell with suggestions
solaris <command>          a single command
```

## Commands

| Command | Purpose |
|---|---|
| `setup` | Teach the Kit about your packages and presets |
| `sandbox` | Build a new Laravel sandbox (wizard: name, location, package preset, DB, Redis, options) |
| `dev` | Interactive supervisor — the daily loop |
| `dev init` | Wire an existing sandbox up and edit what it runs |
| `check` | Validate `solaris.dev.json` and print what would run |
| `package new` | Scaffold a new Solaris package from the spatie skeleton |
| `package init` | Set up a package's dev composer manifests |
| `package list` | List packages, or resolve one with its dependencies |
| `composer` | Run composer against the dev manifest, keeping `composer.json` clean |
| `seed` | Run every package seeder in dependency order |
| `config` | Merge package configs into the sandbox's `config/solaris.php` |
| `pack` | Zip a package |
| `build` | Bundle a sandbox for distribution |

`solaris <command> --help` for flags.

## The daily loop

```bash
cd <sandbox>
solaris dev
```

or from anywhere, by name — `solaris sandbox` registers what it builds:

```bash
solaris dev <SandboxName>
```

One window: every process in a sidebar, one log pane, selective restart, and `:` for a command tab
running in a real pseudo-terminal, so an artisan prompt can be answered without leaving the panel.
`S` starts everything, `X` stops, `R` restarts; lowercase acts on the selection.

Package edits are **mirrored** into the sandbox `vendor/` as you save (not symlinked — Vite watches
that tree), and a `.blade.php` change refreshes the browser. So: **edit in the package repo, watch
it in the sandbox.**

## The composer manifest swap

A Solaris package carries three manifests:

| File | Contents | Git |
|---|---|---|
| `composer.json` | the real manifest, **without** `repositories` | committed |
| `composer.local.json` | only the `repositories` block — path repos pointing at sibling packages | ignored |
| `composer.dev.json` | `composer.json` plus those repositories | ignored |

Development needs the path repositories so a change in core is immediately visible to the packages
above it. Git must not see them: those paths are absolute and machine-specific. `symlink: true` is
correct *here* — the no-symlink rule applies to a sandbox's `vendor/`, not to package-to-package
development links.

- `solaris package init` — run inside a package: locates its Solaris dependencies on disk, writes
  both dev manifests, gitignores them, and installs. It never rewrites `composer.json`.
- `solaris composer <args...>` — swaps the dev manifest in, runs composer, copies any new
  requirement back to `composer.dev.json`, strips `repositories`, and restores a clean
  `composer.json`. On failure it restores the backup and exits non-zero.
- If a run is interrupted mid-swap, the next invocation refuses and tells you to run
  `solaris composer --recover` first.

**Never run bare `composer require` inside a package during development** — it will either miss the
path repositories or commit them.

## Scaffolding a new package

`solaris package new` clones the spatie package skeleton and adjusts it to Solaris standards
(requires `git` and network access). One input drives every derived name:

| | `Inventory` | `master-data` |
|---|---|---|
| Directory | `Solaris-Laravel-Inventory` | `Solaris-Laravel-MasterData` |
| Composer | `solaris/solaris-laravel-inventory` | `solaris/solaris-laravel-masterdata` |
| Namespace | `Solaris\Inventory` | `Solaris\MasterData` |
| Provider | `SolarisInventoryServiceProvider` | `SolarisMasterDataServiceProvider` |
| Registry key | `inventory` | `master-data` |
| JS alias | `@inventory-js/*` | `@master-data-js/*` |

Note the namespace and class name are different words: `Solaris\Inventory` holds
`SolarisInventory`.

## Config and seeding

- `solaris config` merges each package's `config/solaris.php` into the sandbox's, in dependency
  order so later packages override earlier ones. Formatting and comments are preserved. **A config
  key you add to a package only reaches the sandbox after this runs** — a missing merge looks like
  a functional bug, not a config problem.
- `solaris seed` runs every package seeder in dependency order, core first.

## Quality gates

Run from the package repository:

```bash
composer run test        # Pest, on Orchestra Testbench
composer run analyse     # PHPStan
composer run format      # Laravel Pint
```

## Stack

PHP 8.2+, Laravel 12, Bootstrap 5.3.8, Vite, Node LTS, Redis for cache and queue, Laravel Reverb
for WebSocket. Databases: MySQL/MariaDB, PostgreSQL, SQL Server.
