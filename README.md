# Solaris Agent Documentation

This directory is a distributable agent-instruction bundle validated against
`solaris/solaris-laravel-core` version `1.6.5`.

## Install in a project

Copy the **contents** of this directory to the project root:

```text
agents/AGENTS.md          → <project>/AGENTS.md
agents/docs/solaris/      → <project>/docs/solaris/
```

The resulting project structure is:

```text
project/
├── AGENTS.md
└── docs/
    └── solaris/
```

Do not copy the outer `agents/` directory itself into the project. `AGENTS.md` must be at project root
so compatible coding agents discover it automatically. Keep `docs/solaris/` beside it because all links
are portable relative links.

## Usage

Agents start from `AGENTS.md`, identify package/project/sandbox context, then read only the linked guides
required by the task. Humans can use the same file as the documentation index.

When Solaris Laravel Core is upgraded, review affected contracts, update the compatibility metadata in
`AGENTS.md`, and validate every local Markdown link before redistributing this bundle.
