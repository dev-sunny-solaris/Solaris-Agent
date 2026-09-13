# Solaris-Kit Setup Guide

Use this guide to walk a user through installing and setting up Solaris-Kit, building a sandbox,
and running it. Most steps are interactive screens: give the user the command and explain what the
screen asks; do not run interactive commands yourself. Daily agent usage is in
[Solaris-Kit](workflow.md).

## 1. Requirements

On the machine that runs the Kit:

| Tool | Why |
|---|---|
| PHP 8.2+ (with the PDO driver for the chosen database, and OpenSSL) | Laravel, artisan, config merge |
| Composer | Package and sandbox dependencies |
| Node LTS + npm | Vite and frontend assets |
| Git + network access | `solaris package new` clones the Spatie skeleton |
| A database server (MySQL/MariaDB, PostgreSQL, SQL Server) or SQLite | The sandbox database |
| Redis | Cache, sessions, and queues |

Windows 10 1809+ or Linux (including WSL). Windows and WSL are separate installs with separate
configs, even when they look at the same project directory.

## 2. Install the Kit

The Kit is distributed as a release zip (`solaris_<version>_windows_amd64.zip` or
`solaris_<version>_linux_amd64.zip`) containing the binary, `README.md`, and `docs/`. No runtime is
needed; the binary is static.

**Windows (PowerShell):**

```powershell
Expand-Archive solaris_<version>_windows_amd64.zip -DestinationPath $env:LOCALAPPDATA\Programs\Solaris -Force
[Environment]::SetEnvironmentVariable(
  "Path",
  [Environment]::GetEnvironmentVariable("Path", "User") + ";$env:LOCALAPPDATA\Programs\Solaris",
  "User")
```

Open a **new** terminal — the PATH change does not reach the one that made it.

**Linux / WSL:**

```bash
unzip solaris_<version>_linux_amd64.zip -d ~/.local/solaris
chmod +x ~/.local/solaris/solaris
sudo ln -sf ~/.local/solaris/solaris /usr/local/bin/solaris
```

**From source** (Go 1.26+), inside the Solaris-Kit repository:

```bash
go build -o bin/solaris.exe ./cmd/solaris   # Windows
go build -o bin/solaris ./cmd/solaris       # Linux / WSL
```

Then put `bin/` on `PATH`.

Verify: `solaris --version`.

**Upgrading** means replacing the binary only; the config is never touched. On Windows, close every
`solaris dev` session first — a running `solaris` locks its own `.exe`.

## 3. First-time setup — `solaris setup`

The binary knows nothing about the machine until this runs. It writes the Kit registry to
`~/.solaris/solaris.json` (`%USERPROFILE%\.solaris\solaris.json` on Windows; override the location
with `SOLARIS_CONFIG`).

```bash
solaris setup
```

1. With no config yet, choose **Pakai preset bawaan** (built-in preset). It carries every Solaris
   package, its dependencies, queues, and the SunnyApp presets — everything except paths.
2. On the package list, press `a`. It scans the parent directory and matches each folder by the
   composer name in its `composer.json`. With every repository under one parent, this finishes most
   of the setup.
   `●` valid · `○` no path yet · `✕` set but wrong
3. For a package still without a path, `⏎` on the Path field opens a directory browser. It only
   accepts a directory whose composer name matches that package.
4. Save (`s` jumps to save). Saving is refused, with the reason, while validation fails: a package
   without a path, a preset naming a missing package, a dependency cycle.

Keys: arrows / `j` `k` move · `⏎` open or accept · `esc` back · `space` toggle · `ctrl+c` quit
without writing.

Other things `setup` manages:

- **SunnyApp presets** — named recipes of packages; dependencies lock themselves on.
- **Sandboxes** — add an existing sandbox (`ctrl+n`, it must have `solaris.dev.json`), rename,
  re-point a moved folder, or forget one (`ctrl+d`; the directory is left alone).

`solaris setup --reset` starts again from the built-in defaults, including when the registry file
no longer parses.

Check the result:

```bash
solaris package list            # every known package and whether it has a path
solaris package list <key>      # dependency chain, e.g. core → … → <key>
```

## 4. Prepare package repositories — `solaris package init`

Run once inside every package repository that will be developed (after cloning, or after
`solaris package new`):

```bash
cd <package-repository>
solaris package init
```

It locates every required Solaris package (a `--path <key>=<dir>` flag first, then the Kit
registry, then sibling directories), writes `composer.local.json` and `composer.dev.json`,
git-ignores them, and runs `composer install` through the manifest swap. A checklist shows one row
per dependency; `⏎` on a row picks its directory, `w` writes once every row is ticked.
`--install=false` skips the install. `composer.json` is never rewritten.

## 5. Build a sandbox — `solaris sandbox new`

```bash
solaris sandbox new
```

The wizard asks, in order:

| Screen | What it asks |
|---|---|
| Name | The sandbox name |
| Location | A directory picker; an existing folder is refused |
| SunnyApp | A preset, or `Custom` to tick packages |
| Database | Driver, host, port, name, user, password — credentials are tested against the server |
| Redis | Host, port, password — tested — then the index for `REDIS_DB` and `REDIS_CACHE_DB`, read from the server so free indexes are visible |
| Options | `npm install`, `migrate`, `seed` — all off by default |
| Review | Everything once; `⏎` builds |

The same build without the wizard:

```bash
solaris sandbox new <Directory> --app=<preset-or-key> --db=pgsql --db-password=<secret> --redis-db=3 --npm --migrate --seed
```

Other flags: `--db-host`, `--db-port`, `--db-name` (defaults to the sandbox name), `--db-username`,
`--redis-host`, `--redis-port`, `--redis-password`, `--redis-cache-db`, `--custom`,
`--profile` (WhatsApp profile, default `sunny`).

What the build does: `composer create-project laravel/laravel` → path repositories and
`composer require` of the packages → `composer update -w` → `.env` / `.env.example` → create the
database if missing → wire the app (User model, routes, Vite, `jsconfig.json`, `package.json`,
`bootstrap/app.php`, config, `solaris.dev.json`) → Reverb and web-push keys → register the sandbox →
optional npm / migrate / seed.

**A failed build is resumed, not repeated:**

```bash
solaris sandbox new --resume <Directory>
```

Early steps (download, require, update, `.env`, wiring) stop the build on failure. Later steps
(database, broadcasting, registration, npm, migrate, seed) are reported as skipped and can be run
by hand in the sandbox.

**Existing sandbox without `solaris.dev.json`** (built by hand or by the old DevKit):

```bash
cd <sandbox>
solaris sandbox setup --dry-run   # print the generated config, write nothing
solaris sandbox setup             # write solaris.dev.json and register the sandbox
```

An existing `solaris.dev.json` is only overwritten with `--force`.

## 6. Run the sandbox — `solaris dev`

```bash
cd <sandbox>
solaris dev
# or from anywhere
solaris dev <SandboxName>
```

One panel: a process sidebar (web, Vite, workers, queues, WhatsApp, external services) and one log
pane. The `watcher` → `sync` row mirrors package source into the sandbox `vendor/`; it starts with
everything else.

| Key | Action |
|---|---|
| `S` / `X` / `R` | Start / stop / restart everything |
| `s` / `x` / `r` | Start / stop / restart the selection (a process or a whole group) |
| `↑` `↓`, `enter` | Move, fold a group |
| `/` | Filter the log |
| `n` / `N` | Next / previous stderr line |
| `e` | Execute a one-shot command: artisan chores, migrate, seed, `npm run build`, per-package composer and pack |
| `?` | Key list |
| `q` | Quit, stopping every process |

Flags: `--profile=<name>`, `--only=<group,group>`, `--start` (bring everything up on open),
`--no-watch`.

Status glyphs: `●` running · `○` stopped · `✕` crashed (with exit code) · `⚠n` stderr lines ·
`↑n` files mirrored.

Processes come from `solaris.dev.json` (committed). Personal overrides — ports, extra processes, a
disabled group — go in `solaris.dev.local.json` (git-ignored); it replaces whole groups, not single
processes. After editing either file:

```bash
solaris sandbox check    # validate and print what would run; starts nothing
```

## 7. Sandbox chores

```bash
solaris sandbox config                                     # merge package config/solaris.php into the sandbox
solaris sandbox seed                                       # every package seeder in dependency order, then the sandbox's
solaris sandbox seed --package <key> --class <Seeder>      # specific seeders
solaris sandbox seed --dry-run                             # list what would run
solaris sandbox build [--format=zip] [--filename=<name>]   # production bundle
solaris package pack [--package <keys> | --all] [--out <dir>]
```

## Troubleshooting

| Symptom | Fix |
|---|---|
| `solaris` not found after install | Open a new terminal; check `PATH` |
| Registry will not parse | Fix `~/.solaris/solaris.json` by hand, or `solaris setup --reset` |
| `no sandbox called …` | `solaris setup` → Sandboxes, or `solaris sandbox setup` inside it |
| Sandbox shown as `missing` | Re-point its path in `solaris setup` → Sandboxes |
| `an interrupted composer swap is pending` | `solaris package composer --recover` inside the package |
| New config key has no effect in the sandbox | `solaris sandbox config` |
| Package edit not visible in the sandbox | Is `solaris dev` running with the `sync` row started? Edit the package repository, not `vendor/` |
| Sandbox build stopped | `solaris sandbox new --resume <Directory>` |
| Upgrade copy fails on Windows | Close `solaris dev`; rename the old `.exe` aside, then copy |
