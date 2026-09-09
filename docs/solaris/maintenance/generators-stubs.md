# Generators and Stubs (Core)

Core owns the scaffolding used by consumer apps:

```
php artisan solaris:make {action} {name?} [options]
    action   model | controller | repository | request | migration | view | module | approve
    options  --default --type= --table= --force --repo=none|default|custom
             --request / --no-request        use or skip a FormRequest
             --no-migration                  module only
             --model=                        controller: model to bind the repository to
             --parent= --parent-column=       model: detail, stage-history
             --stage-model= --stage-column=   model: stage-history

php artisan solaris:extend {type} {name?} [--force]
    type     controller | model | repository | request | view | module

php artisan solaris:menu | :roles | :permissions | :user | :object-permission | :code-generator
```

Naming produced by the generators: module/folder `snake_case`, classes `StudlyCase`, Blade at
`resources/views/pages/{module}/{type}.blade.php`, JS at `resources/js/pages/{module}/{type}.js`.

At package level you do not run these commands to create package source. You maintain them: the
templates live in core's `stubs/` directory (`controllers/`, `models/`, `repositories/`,
`requests/`, `views/`, `js/`, `migrations/`, `approvals/`, `ai/`).

**A change to a convention is not finished until the matching stub is updated.** A stub that no
longer matches the documented standard silently propagates the old pattern into every new project.

Core also ships the AI-instruction stubs that a consumer app publishes into its own repository
(entry files for various agent tools plus the project-level skill set). Those describe
**project-level** work — building features in an app that consumes Solaris. The root `AGENTS.md` and
`docs/solaris/` bundle are portable across package and project repositories; context routing keeps
package-maintenance rules out of normal project work. Keep published templates synchronized when a
convention changes.
