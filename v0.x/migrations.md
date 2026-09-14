# Migrations

Migrations provide version control for your database schema. Each migration is a PHP class that represents a set of database changes.

<p class="tip">Please note that only <b>MySQL/MariaDB</b> based migrations are supported.</p>

## Creating Migrations

To create a new migration file, fire this command from console:

```terminal
php console create:migration create_table_products
```

This will create the migration file prefixed with current **datetime** in `database/migrations` folder. The migration class contains methods `up()` and `down()`.

The `up()` method contains definition for required schema changes. Any reverse operations should go inside `down()` folder.

| Method  | Called when                | Purpose                                                                 | Common contents                                                   |
|---------|----------------------------------|------------------------------------------------------------------------|-------------------------------------------------------------------|
| `up()`  | When you apply or run a migration| Build or evolve the schema—create tables, add columns/indexes/constraints, insert seed data that must exist, rename things, etc. | DDL or data-manipulating statements written in the migration DSL   |
| `down()`| When you roll back or undo a migration | Reverse whatever `up()` did so the database returns to its previous state | The inverse DDL (drop tables, remove columns, delete seed rows, etc.) |

> In short: `up()` is the **do** part of a migration; `down()` is the **undo**.</p>

## Reserved Table Names

Lightpack uses certain table names internally for framework features (jobs queue, cache, settings, etc.). These names are reserved — avoid using them for your own application tables.

| Table | Used by |
|-------|---------|
| `users` | Auth |
| `access_tokens` | Auth |
| `jobs` | Queue |
| `cache` | Cache |
| `settings` | Settings |
| `secrets` | Secrets |
| `audit_logs` | Audit |
| `cable_messages` | Cable |
| `cable_presence` | Cable |
| `roles` | RBAC |
| `permissions` | RBAC |
| `user_role` | RBAC |
| `role_permission` | RBAC |
| `tags` | Tags |
| `tag_morphs` | Tags |
| `taxonomies` | Taxonomies |
| `taxonomy_morphs` | Taxonomies |
| `uploads` | Uploads |
| `social_accounts` | Social Auth |
| `webhook_events` | Webhooks |
| `migrations` | Migrations |

If your application requires a table name that conflicts with a reserved name, use a domain-specific name instead. For example, name your job listings table `job_postings` instead of `jobs`. If you already have a conflicting table, create a rename migration to rename your domain table before generating the framework table.

## Running Migrations

To run your migration files:

```terminal
php console migrate:up
```

This command will run the `up()` method in all the migration scripts defined inside **database/migrations** folder. 

To track the files that have been migrated, it will also create a `migrations` table in the database.

## Rollback Migrations

To rollback or undo all your migrations:

```terminal
php console migrate:down
```

This will run the `down()` method in all the migration scripts inside **database/migrations** folder.

To rollback a limited number of migrations, provide the `steps` flag. For example, 
this will rollback last two **batches** of migrations if present.

```terminal
php console migrate:down --steps=2
```

To rollback all the migrations in one go, provide the `--all` flag:

```terminal
php console migrate:down --all
```

## Defining Migrations

All table operations take a closure that receives a `Table` object:

```php
$this->create('users', function(Table $table) {
    // column definitions, indexes, foreign keys...
});

$this->alter('users')->add(function(Table $table) {
    // add columns, indexes, foreign keys...
});
```

### Complete Example

```php
public function up(): void
{
    $this->create('posts', function(Table $table) {
        $table->id();
        $table->varchar('title');
        $table->varchar('slug')->unique();
        $table->text('body')->nullable();
        $table->foreignKey('user_id')->references('id')->on('users');
        $table->timestamps();
    });
}

public function down(): void
{
    $this->drop('posts');
}
```

### Create Table

```php
public function up(): void
{
    $this->create('users', function(Table $table) {
        $table->id();
        $table->varchar('name');
        $table->varchar('email');
    });
}

public function down(): void
{
    $this->drop('users');
}
```

### Rename Table

```php
public function up(): void
{
    $this->rename('users', 'customers');
}

public function down(): void
{
    $this->rename('customers', 'users');
}
```

### Truncate Table

```php
public function up(): void
{
    $this->truncate('users');
}
```

### Execute Raw SQL

You can execute raw SQL queries when needed:

```php
public function up(): void
{
    $this->execute('ALTER TABLE users ADD COLUMN custom_field VARCHAR(255)');
}
```

### Alter Table

You may alter an existing table definition as documented below.

#### Add New Columns

```php
public function up(): void
{
    $this->alter('users')->add(function(Table $table) {
        $table->varchar('password');
        $table->timestamps();
    });
}
```

#### Modify Existing Columns

```php
public function up(): void
{
    $this->alter('users')->modify(function(Table $table) {
        $table->varchar('name', 55);
    });
}
```

```php
// Modify an enum column
public function up(): void
{
    $this->alter('users')->modify(function(Table $table) {
        $table->enum('status', ['active', 'inactive', 'banned']);
    });
}
```

#### Drop Existing Columns

```php
public function up(): void
{
    // Drop single column
    $this->alter('users')->dropColumn('password');
}

public function down(): void
{
    // Drop multiple columns at once
    $this->alter('users')->dropColumn('password', 'email', 'phone');
}
```

#### Rename Column

```php
public function up(): void
{
    $this->alter('users')->renameColumn('name', 'full_name');
}

public function down(): void
{
    $this->alter('users')->renameColumn('full_name', 'name');
}
```

#### Add/Drop Indexes

When altering a table, add or remove indexes directly:

```php
// Add indexes
$this->alter('users')->add(function(Table $table) {
    $table->index('email');
    $table->unique('phone');
});

// Drop indexes by name
$this->alter('users')->dropIndex('email_index');
$this->alter('users')->dropUnique('phone_unique');
```

#### Add Foreign Keys to Existing Tables

```php
// Add a foreign key to an existing table
public function up(): void
{
    $this->alter('products')->add(function(Table $table) {
        $table->foreignKey('category_id')->references('id')->on('categories');
    });
}

// Add a column and foreign key together
public function up(): void
{
    $this->alter('products')->add(function(Table $table) {
        $table->column('category_id')->type('bigint')->attribute('unsigned');
        $table->foreignKey('category_id')->references('id')->on('categories');
    });
}
```

## Table Columns

All column methods return a `Column` object for chaining configuration.

### Column Types

| Type | Method | Notes |
|------|--------|-------|
| Auto PK | `id(string $name = 'id')` | BIGINT UNSIGNED AUTO_INCREMENT |
| Integer | `int($name, $len=11)` | |
| | `bigint($name)` | |
| | `smallint($name)` | |
| | `tinyint($name)` | |
| | `boolean($name, $default=false)` | TINYINT(1) |
| String | `varchar($name, $len=255)` | |
| | `char($name, $len=255)` | |
| | `text($name)` | |
| | `tinytext($name)` | |
| | `mediumtext($name)` | |
| | `longtext($name)` | |
| | `enum($name, $values)` | |
| | `json($name)` | |
| Date/Time | `date($name)` | |
| | `time($name)` | |
| | `datetime($name)` | |
| | `timestamp($name)` | |
| | `year($name)` | |
| Timestamps | `createdAt()` | Default CURRENT_TIMESTAMP |
| | `updatedAt()` | Nullable, ON UPDATE CURRENT_TIMESTAMP |
| | `deletedAt()` | Nullable |
| | `timestamps()` | Adds createdAt + updatedAt |
| Special | `ipAddress($name='ip_address')` | VARCHAR(45) |
| | `macAddress($name='mac_address')` | VARCHAR(17) |
| | `morphs($name)` | Adds `{name}_id` + `{name}_type` |
| Numeric | `decimal($name, $p=10, $s=2)` | |

### Column Configuration

Chain these after any column method:

```php
$table->varchar('username', 50)->unique()->nullable();
$table->int('age')->default(0)->unsigned();
$table->decimal('balance', 12, 2)->attribute('UNSIGNED');
$table->datetime('created_at')->current();
```

| Method | Purpose |
|--------|---------|
| `type(string)` | Override SQL type |
| `length(int)` | Set length |
| `default(string\|bool)` | Set default value |
| `nullable()` | Allow NULL |
| `attribute(string)` | Add SQL attribute (UNSIGNED, etc.) |
| `unsigned()` | Shortcut for UNSIGNED |
| `current()` | Shortcut for CURRENT_TIMESTAMP default |
| `increments()` | AUTO_INCREMENT + PRIMARY KEY |
| `primary()` | Mark as primary key |
| `unique(?string $name)` | Add unique index |
| `index(?string $name)` | Add regular index |
| `fulltext(?string $name)` | Add fulltext index |

---

## Table Indexes

**Supported Index Types**

- **Primary Key**
- **Unique Index**
- **Regular Index**
- **Fulltext Index**
- **Spatial Index**

### Index Naming

You can provide a custom name, or let the framework generate one based on the column name(s) and index type.

| How defined | Default name | Example |
|-------------|-------------|---------|
| Chained on column: `->unique()` | `{column}_{type}` | `$table->varchar('email')->unique()` → `email_unique` |
| Standalone single: `->unique('email')` | `{column}_{type}` | `$table->unique('email')` → `email_unique` |
| Standalone composite: `->unique(['a','b'])` | `{col1}_{col2}_{type}` | `$table->unique(['first','last'])` → `first_last_unique` |
| Composite too long (>60 chars) | `idx_{8charhash}` | `$table->unique(['very_long_name', ...])` → `idx_a1b2c3d4` |
| Custom name provided | Your name | `$table->unique('email', 'u_email')` → `u_email` |

<p class="tip"><b>Tip:</b> When dropping an index, pass the exact name shown in <code>SHOW INDEXES</code> or generated as above.</p>

---

### primary() 

- Defines a primary key (single or composite).
- primary(string|array $columns)

```php
// single primary key
$table->primary('id');

// composite key
$table->primary(['user_id', 'post_id']);
```

### dropPrimary()

- Drops the primary key constraint (not the column).

<p class="tip"><b>Important Notes:</b></p>

- Dropping the primary key does not remove the column from the table but only the primary key constraint.
- Remember that there can be only one auto column and it must be defined as a key. So if the primary key is defined as auto, you will need to remove the auto attribute first.
- Do not forget to drop or alter foreign keys referencing the primary key.

```php
$table->dropPrimary();
```

### unique()

- Adds a unique index to one or more columns.
- `unique(string|array $columns, ?string $indexName = null)`
- Supports custom unique index name.

<p class="tip"><b>Note:</b> You should remove duplicate values from the columns before adding unique index otherwise it may result in "mysql error 1062".</p>

```php
$table->unique('email');
$table->unique(['first', 'last'], 'name_unique');
```

### dropUnique()

- Drops a unique index by name.
- `dropUnique(string $indexName)`

```php
$this->alter('users')->dropUnique('email_unique');
```

### index()
- Adds a regular (non-unique) index.
- `index(string|array $columns, ?string $indexName = null)`
- Supports custom index name.

```php
$table->index('created_at');
$table->index(['user_id', 'status'], 'user_status_idx');
```

### dropIndex()

- Drops a regular index by name.
- `dropIndex(string $indexName)`

```php
$this->alter('users')->dropIndex('user_status_idx');
```

### fulltext()
- Adds a FULLTEXT index for text search.
- `fulltext(string|array $columns, ?string $indexName = null)`
- Supports custom full text index name.

```php
$table->fulltext('body');
$table->fulltext(['title', 'body'], 'post_fulltext');
```

### dropFulltext() 
- Drops one or more FULLTEXT indexes by name.
- `dropFulltext(string ...$indexName)`

```php
// Drop single fulltext index
$this->alter('posts')->dropFulltext('post_fulltext');

// Drop multiple fulltext indexes
$this->alter('posts')->dropFulltext('title_fulltext', 'body_fulltext');
```

### spatial()
- Adds a SPATIAL index for GIS data.
- `spatial(string|array $columns, ?string $indexName = null)`
- Support passing custom spatial index name.

```php
$table->spatial('location', 'loc_idx');
```

### dropSpatial() 
- Drops a SPATIAL index by name.
- `dropSpatial(string $indexName)`

```php
$this->alter('locations')->dropSpatial('loc_idx');
```

---

## Table Configuration

You can configure table engine, charset, and collation:

```php
public function up(): void
{
    $this->create('users', function(Table $table) {
        $table->id();
        $table->varchar('name');
        
        // Configure table settings
        $table->engine('InnoDB');  // Default: InnoDB
        $table->charset('utf8mb4'); // Default: utf8mb4
        $table->collation('utf8mb4_unicode_ci'); // Default: utf8mb4_unicode_ci
    });
}
```

---

## Foreign Keys
**Foreign keys** enforce referential integrity between tables, ensuring that a column (or set of columns) in one table matches the primary key or unique key in another table. Foreign keys can be defined during table creation or added/removed during schema alteration. The following document details the support for working with foreign keys.


### Defining Foreign Keys

To define a foreign key constraint, use the `foreignKey()` method. The constraint name is **auto-generated** as `fk_{table}_{column}` — always predictable. Use `->name()` to override.

```php
$table->foreignKey('author_id')
    ->references('id')
    ->on('users')
    ->cascadeOnDelete();
// Generates: CONSTRAINT `fk_posts_author_id` FOREIGN KEY (`author_id`) REFERENCES `users`(`id`) ON DELETE CASCADE ON UPDATE RESTRICT

// Custom constraint name:
$table->foreignKey('author_id')->name('my_fk_name')->cascadeOnDelete();
```

### Foreign Key Actions

Available actions for `ON UPDATE` and `ON DELETE`:

- **CASCADE**: Automatically update/delete related rows
- **RESTRICT**: Prevent update/delete if related rows exist (default)
- **SET NULL**: Set foreign key column to NULL

**Available Methods:**
- `cascadeOnDelete()`: Set ON DELETE CASCADE
- `cascadeOnUpdate()`: Set ON UPDATE CASCADE
- `restrictOnDelete()`: Set ON DELETE RESTRICT (default)
- `restrictOnUpdate()`: Set ON UPDATE RESTRICT (default)
- `nullOnDelete()`: Set ON DELETE SET NULL
- `nullOnUpdate()`: Set ON UPDATE SET NULL

<p class="tip"><b>Note:</b> Default behavior is RESTRICT for both ON UPDATE and ON DELETE.</p>

**Example:**
```php
$table->foreignKey('category_id')
    ->references('id')
    ->on('categories')
    ->nullOnDelete()      // Set to NULL when parent is deleted
    ->restrictOnUpdate(); // Prevent parent updates if children exist
```

### Dropping Foreign Keys

Constraint names follow the pattern `fk_{table}_{column}`, so they are always predictable:

```php
public function up(): void
{
    // Drop single foreign key
    $this->alter('posts')->dropForeign('fk_posts_author_id');

    // Drop multiple foreign keys
    $this->alter('posts')->dropForeign('fk_posts_author_id', 'fk_posts_category_id');
}
```

<p class="tip"><b>Note:</b> If you used <code>->name()</code> to set a custom constraint name, pass that name to <code>dropForeign()</code> instead.</p>

---