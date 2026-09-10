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
- Every foreign key must declare its deletion behavior explicitly:
  - Use `cascadeOnDelete()` when the row is a detail/child that has no meaning without its parent.
  - Use `nullOnDelete()` for a normal Lookup/reference. The foreign-key column must also be nullable.
- Create an explicit compatibility index for every foreign-key column with
  `MigrationHelper::createIndexIfNeeds($table, ['column_id'])`. Call it once per foreign key after
  declaring the columns. This helper accepts `array|string`, but array form is the standard usage.
- `MigrationHelper::createIndexIfNeeds()` is only for foreign keys. For a normal non-FK index, use
  Laravel's standard `$table->index(...)` API instead.
- Seeders ship menu entries, permissions and demo data. A new module needs its menu and permission
  seeding, or it is invisible and inaccessible in the app.
- **Never connect to or query a database directly.** Express schema changes as migrations and data
  changes as seeders.
- Seeder execution order across packages is handled by `solaris seed` (core first) — see [Workflow](../maintenance/workflow.md).

## Foreign-key example

```php
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;
use Solaris\Core\Helpers\Migration as MigrationHelper;

Schema::create('order_items', function (Blueprint $table) {
	MigrationHelper::baseTable($table);

	$table->foreignUuid('order_id')->constrained('orders')->cascadeOnDelete();
	$table->foreignUuid('status_id')->nullable()->constrained('order_statuses')->nullOnDelete();

	MigrationHelper::createIndexIfNeeds($table, ['order_id']);
	MigrationHelper::createIndexIfNeeds($table, ['status_id']);
});
```
