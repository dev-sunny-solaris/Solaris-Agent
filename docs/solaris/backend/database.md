# Database

- Migrations live in `database/migrations/` and are auto-discovered and auto-run by the provider.
- Migration filenames are prefixed so ordering across packages is deterministic — core first, then
  each layer above it (e.g. masterdata uses `0003_01_01_NNNNNN_md_...`). Follow the prefix already
  used by the package you are in, and increment the sequence.
- A package must not alter another package's table in a way that ancestor code depends on. Additive
  columns on an ancestor's table are acceptable; changing semantics is not.
- Solaris is database-agnostic across MySQL/MariaDB, PostgreSQL and SQL Server. Do not use
  vendor-specific SQL, types or functions in migrations or query builders. If a raw expression is
  unavoidable, branch on the driver.
- Seeders ship menu entries, permissions and demo data. A new module needs its menu and permission
  seeding, or it is invisible and inaccessible in the app.
- **Never connect to or query a database directly.** Express schema changes as migrations and data
  changes as seeders.
- Seeder execution order across packages is handled by `solaris seed` (core first) — see [Workflow](../maintenance/workflow.md).
