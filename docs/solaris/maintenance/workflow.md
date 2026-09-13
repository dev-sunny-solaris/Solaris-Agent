# Solaris-Kit

**A package never runs alone.** It is developed and tested inside a host Laravel *sandbox* that has
the packages installed. Solaris-Kit is the toolchain for that: a single binary (`solaris`) that
builds and runs the sandbox, mirrors package edits into it, merges configs, and guards composer. It
drives PHP, composer, node, and npm — it does not replace them.

## Prerequisite

`solaris --version` must succeed before any package-level work. If it does not, stop and tell the
user; agents never install or build the Kit themselves. When the user asks how to install, set up,
or run the Kit, guide them with [Solaris-Kit setup guide](solaris-kit-setup.md).

## Agent vs human

Interactive screens need a real terminal: `solaris` (the shell), `solaris setup`, `solaris dev`, and
every wizard or picker (a command run without its required arguments). **Ask the user to run these.**

An agent may run only fully flagged, non-interactive forms, after confirming with the user. Without
a terminal, a command that still needs an answer fails instead of prompting.

## Primary commands

### `solaris sandbox new` — build a sandbox

Creates a Laravel application with the selected Solaris packages installed and wired, writes its
`solaris.dev.json`, and registers it with the Kit.

```bash
solaris sandbox new <Directory> --app=<preset-or-package-key> [--db=pgsql] [--migrate --seed --npm]
```

`--app` takes a SunnyApp preset or a single package key. Database and Redis defaults come from the
Kit registry. `--resume <Directory>` continues a build that stopped part way.

### `solaris dev` — run the sandbox

```bash
solaris dev                 # inside the sandbox
solaris dev <SandboxName>   # from anywhere
```

One panel supervises every sandbox process (web, Vite, queues, Reverb, …) and mirrors package source
into the sandbox `vendor/` as files are saved. **Edit in the package repository, verify in the
sandbox. Never edit the sandbox `vendor/`.** This is a human-run command.

### `solaris sandbox config` — merge config

```bash
solaris sandbox config      # inside the sandbox
```

Merges every package's `config/solaris.php` into the sandbox's, in dependency order, so later
packages override earlier ones. Formatting and comments are preserved.

**Run it after every change to a package `config/solaris.php`.** A new or changed key does not reach
the sandbox until this runs, and a missing merge looks like a functional bug, not a config problem.

### `solaris package composer` — composer inside a package

```bash
solaris package composer require vendor/name:^1.0
solaris package composer update
solaris package composer --recover
```

**Never run bare `composer install`, `update`, `require`, or `remove` inside a package.** Script
runs such as `composer run test` and `composer run analyse` are fine — they do not resolve
dependencies. A package in
development carries three manifests:

| File | Contents | Git |
|---|---|---|
| `composer.json` | The real manifest, without `repositories` | Committed |
| `composer.local.json` | Only path repositories pointing at local Solaris package source | Ignored |
| `composer.dev.json` | `composer.json` plus those repositories | Ignored |

The path repositories make a change in one package immediately visible to the packages above it.
They are absolute, machine-specific paths, so git must never see them. Bare `composer` either misses
them — resolving Solaris dependencies from a remote instead of local source — or, if they were added
by hand, commits them.

`solaris package composer` swaps the dev manifest in, runs composer, copies any new requirement back
to `composer.dev.json`, strips `repositories`, and restores a clean `composer.json`. On failure it
restores the backup and exits non-zero. If a run was interrupted, the next one refuses until
`--recover` is run. A package without `composer.dev.json` runs composer directly.

## Flows

### First-time setup

Full walkthrough: [Solaris-Kit setup guide](solaris-kit-setup.md).

1. Install the Kit (see Prerequisite).
2. The user runs `solaris setup`, chooses the built-in preset (every Solaris package, its
   dependencies, queues, and SunnyApp presets), presses `a` to scan package paths, then saves.
3. The result is the Kit registry at `~/.solaris/solaris.json` (override with `SOLARIS_CONFIG`). It
   holds each package's key, composer name, namespace, **source path**, dependencies, and queues,
   plus presets and known sandboxes.

Resolve package source paths and dependency order from the registry, never by guessing:

```bash
solaris package list            # every known package and its path status
solaris package list <key>      # dependency chain, e.g. core → … → <key>
```

### Create a new package

The Kit scaffolds a package from the Spatie package skeleton and applies the Solaris layer. Requires
`git` and network access.

```bash
solaris package new <Name> --deps=core [--parent=<directory>]
cd Solaris-Laravel-<Name>
solaris package init
```

One name drives every derived identity:

| | `Inventory` |
|---|---|
| Directory | `Solaris-Laravel-Inventory` |
| Composer | `solaris/solaris-laravel-inventory` |
| Namespace | `Solaris\Inventory` |
| Provider | `SolarisInventoryServiceProvider` |
| Registry key | `inventory` |
| JS alias | `@inventory-js/*` |

The namespace and class name are different words: `Solaris\Inventory` holds `SolarisInventory`.
The new package is registered in the Kit automatically (`--register=false` skips it).

### Set up a cloned package

1. The user clones the package repository.
2. The user registers its path through `solaris setup` if the Kit does not know it yet.
3. Inside the package: `solaris package init`. It locates every required Solaris package, writes
   `composer.local.json` and `composer.dev.json`, git-ignores them, and installs through the swap.
   Use `--path <key>=<directory>` when a dependency cannot be found automatically.

`init` never rewrites `composer.json`.

### Run composer

Inside a package, always `solaris package composer <args>`. See above for why.

### Change package config

After editing a package `config/solaris.php`, run `solaris sandbox config` in the sandbox.

## Other commands

| Command | Purpose |
|---|---|
| `solaris sandbox setup [--dry-run]` | Generate `solaris.dev.json` for an existing sandbox and register it |
| `solaris sandbox check` | Validate `solaris.dev.json` and print what would run; starts nothing |
| `solaris sandbox seed [--package <key>] [--class <Seeder>] [--dry-run]` | Run seeders in dependency order, or specific ones |
| `solaris sandbox build` | Bundle a sandbox for distribution |
| `solaris package pack [--package <keys>] [--out <dir>]` | Zip packages |

`solaris <command> --help` lists every flag.

## Quality gates

Run from the package repository:

```bash
composer run test        # Pest, on Orchestra Testbench
composer run analyse     # PHPStan
```

Never run `composer run format`; see [Code style](../code-style.md).

## Stack

PHP 8.2+, Laravel 12, Bootstrap 5.3.8, Vite, Node LTS, Redis for cache and queue, Laravel Reverb
for WebSocket. Databases: MySQL/MariaDB, PostgreSQL, SQL Server.
