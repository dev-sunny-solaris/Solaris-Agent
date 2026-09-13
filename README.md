# Solaris Agent Documentation

This directory is a distributable agent-instruction bundle for Solaris. It applies to
`solaris/solaris-laravel-core` below 2.0 (current: 1.6.6).

## Install in a project

Copy the **contents** of this directory to the project root:

```text
agents/AGENTS.md          → <project>/AGENTS.md
agents/CLAUDE.md          → <project>/CLAUDE.md
agents/docs/solaris/      → <project>/docs/solaris/
```

The resulting project structure is:

```text
project/
├── AGENTS.md
├── CLAUDE.md
└── docs/
    └── solaris/
```

Do not copy the outer `agents/` directory itself into the project. `AGENTS.md` must be at project root
so compatible coding agents discover it automatically. Keep `docs/solaris/` beside it because all links
are portable relative links.

`CLAUDE.md` only imports `AGENTS.md` for Claude Code. If the project already has a `CLAUDE.md`, do not
overwrite it; add the line `@AGENTS.md` to it instead.

## Usage

Agents start from `AGENTS.md`, identify package/project/sandbox context, then read only the linked guides
required by the task. Humans can use the same file as the documentation index.

`AGENTS.md` is the single source of truth. Never add rules to `CLAUDE.md`.

When Solaris Laravel Core is upgraded, review affected contracts, update the version line in
`AGENTS.md` and this file, and validate every local Markdown link before redistributing this bundle.
