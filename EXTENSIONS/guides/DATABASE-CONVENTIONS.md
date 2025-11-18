# Database Conventions for BITHOVEN Extensions

**Version:** 2.0.0  
**Last Updated:** 17 de noviembre de 2025  
**Status:** STABLE

---

## 🎯 Overview

Estas convenciones garantizan que las extensiones sean compatibles con:
- Sistema de instalación/desinstalación automática
- Fix Extension System
- Backups y restauración
- Múltiples extensiones coexistiendo

**⚠️ CRITICAL:** El cumplimiento de estas convenciones es OBLIGATORIO.

---

## 📋 Table Naming Conventions

### Rule: Slug-Based Prefixes

**Tabla principal:** Usa el slug de la extensión sin prefijo adicional
**Tablas relacionadas:** Usa `{slug}_` como prefijo

```php
// extension.json
{
  "slug": "tickets"
}

// Migrations:
Schema::create('tickets', function (Blueprint $table) {
    // ✅ Main table = slug
});

Schema::create('tickets_comments', function (Blueprint $table) {
    // ✅ Related table = slug_tablename
});

Schema::create('tickets_attachments', function (Blueprint $table) {
    // ✅ Related table = slug_tablename
});
```

### ❌ Common Mistakes

```php
// ❌ WRONG - No prefix at all
Schema::create('items', function (Blueprint $table) {

// ❌ WRONG - Wrong prefix
Schema::create('dummy_extension_items', function (Blueprint $table) {

// ❌ WRONG - Plural prefix
Schema::create('tickets_comment', function (Blueprint $table) {
    // Should be 'tickets_comments' (plural)
});
```

### ✅ Correct Examples

```php
// Extension slug: "tasks"
Schema::create('tasks', function (Blueprint $table) {
    // Main table
});

Schema::create('tasks_categories', function (Blueprint $table) {
    // Related table
});

Schema::create('tasks_attachments', function (Blueprint $table) {
    // Related table
});
```

---

## 🔑 Primary Keys

### Rule: Use `id` as Primary Key

**ALWAYS** use `id` (bigint unsigned auto_increment) as primary key.

```php
// ✅ CORRECT
Schema::create('tasks', function (Blueprint $table) {
    $table->id(); // Creates BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY
});
```

### ❌ Wrong Approaches

```php
// ❌ WRONG - Custom primary key name
$table->bigIncrements('task_id');

// ❌ WRONG - Composite primary key
$table->primary(['user_id', 'ticket_id']);

// ❌ WRONG - UUID as primary
$table->uuid('id')->primary();
```

**Exception:** Pivot tables (ver sección Pivot Tables)

---

## 🔗 Foreign Keys

### Rule: Standard Foreign Key Naming

```php
// Format: {referenced_table_singular}_id
$table->foreignId('user_id')->constrained()->onDelete('cascade');
$table->foreignId('category_id')->constrained('tasks_categories')->onDelete('set null');
```

### Full Foreign Key Definition

```php
Schema::create('tickets', function (Blueprint $table) {
    $table->id();
    
    // Reference to system table (users)
    $table->foreignId('user_id')
        ->constrained('users') // Explicit table name
        ->onDelete('cascade'); // Delete tickets when user deleted
    
    // Reference to extension table
    $table->foreignId('category_id')
        ->nullable()
        ->constrained('tickets_categories')
        ->onDelete('set null'); // Set NULL when category deleted
    
    // Reference to another extension table
    $table->foreignId('assigned_to')
        ->nullable()
        ->constrained('users')
        ->onDelete('set null');
});
```

### Best Practices

**Use `onDelete()` Strategy:**
- `cascade` - When parent is essential (user → tickets)
- `set null` - When parent is optional (category → tickets)
- `restrict` - Prevent deletion if children exist

**Always specify table name in `constrained()`:**
```php
// ✅ CORRECT - Explicit table name
$table->foreignId('user_id')
    ->constrained('users')
    ->onDelete('cascade');

// ❌ WRONG - Laravel guesses table name (unreliable)
$table->foreignId('user_id')
    ->constrained() // Might guess wrong table
    ->onDelete('cascade');
```

---

## 🔄 Pivot Tables

### Rule: Alphabetical Order + Singular Names

```php
// ✅ CORRECT - Alphabetical: task, user
Schema::create('task_user', function (Blueprint $table) {
    $table->foreignId('task_id')->constrained('tasks')->onDelete('cascade');
    $table->foreignId('user_id')->constrained('users')->onDelete('cascade');
    
    $table->primary(['task_id', 'user_id']); // Composite primary key
    $table->timestamps(); // Optional
});
```

### With Pivot Data

```php
Schema::create('ticket_tag', function (Blueprint $table) {
    $table->foreignId('ticket_id')->constrained('tickets')->onDelete('cascade');
    $table->foreignId('tag_id')->constrained('tickets_tags')->onDelete('cascade');
    
    // Pivot-specific data
    $table->integer('order')->default(0);
    $table->boolean('is_primary')->default(false);
    
    $table->primary(['ticket_id', 'tag_id']);
    $table->timestamps();
});
```

### ❌ Wrong Pivot Names

```php
// ❌ WRONG - Plural
Schema::create('tasks_users', function (Blueprint $table) {

// ❌ WRONG - Not alphabetical
Schema::create('user_task', function (Blueprint $table) {
    // Should be 'task_user' (alphabetical order)
});

// ❌ WRONG - With prefixes
Schema::create('tickets_ticket_tag', function (Blueprint $table) {
```

---

## 📊 Indexes

### Rule: Index Foreign Keys and Search Columns

```php
Schema::create('tickets', function (Blueprint $table) {
    $table->id();
    $table->foreignId('user_id')->constrained()->onDelete('cascade');
    $table->string('title');
    $table->text('description');
    $table->enum('status', ['open', 'closed'])->default('open');
    $table->enum('priority', ['low', 'medium', 'high'])->default('medium');
    $table->timestamps();
    
    // ✅ Single column indexes
    $table->index('status');
    $table->index('priority');
    $table->index('created_at');
    
    // ✅ Composite indexes (order matters!)
    $table->index(['user_id', 'status']); // Queries filtering by user + status
    $table->index(['status', 'priority']); // Queries filtering by status + priority
});
```

### Index Naming Convention

Laravel auto-generates names, but you can specify:

```php
// Auto-generated (recommended)
$table->index('email'); // tickets_email_index

// Custom name
$table->index('email', 'idx_tickets_email');

// Composite index with custom name
$table->index(['user_id', 'status'], 'idx_user_status');
```

### When to Index

**✅ Index these:**
- Foreign keys (often auto-indexed, but verify)
- Columns in WHERE clauses
- Columns in ORDER BY
- Columns in JOIN conditions
- Enum columns frequently filtered

**❌ Don't index these:**
- Columns rarely queried
- Text/BLOB columns (use full-text search instead)
- Columns with low cardinality (few unique values) unless frequently filtered

---

## 🔍 Unique Constraints

### Rule: Use `unique()` for Business Logic Uniqueness

```php
Schema::create('tickets_tags', function (Blueprint $table) {
    $table->id();
    $table->string('name');
    $table->string('slug');
    $table->timestamps();
    
    // ✅ Unique constraint
    $table->unique('slug'); // One tag per slug
    
    // ✅ Composite unique
    $table->unique(['category_id', 'name']); // Unique name per category
});
```

### User-Related Unique Constraints

```php
Schema::create('ticket_notification_preferences', function (Blueprint $table) {
    $table->id();
    $table->foreignId('user_id')->constrained()->onDelete('cascade');
    $table->boolean('email_on_assigned')->default(true);
    $table->boolean('email_on_comment')->default(true);
    $table->timestamps();
    
    // ✅ One preference record per user
    $table->unique('user_id');
});
```

---

## 🕐 Timestamps

### Rule: ALWAYS Include Timestamps

```php
// ✅ CORRECT - All tables should have timestamps
Schema::create('tasks', function (Blueprint $table) {
    $table->id();
    // ... columns
    $table->timestamps(); // created_at, updated_at
});
```

### Additional Timestamp Columns

```php
Schema::create('tickets', function (Blueprint $table) {
    $table->id();
    // ... columns
    
    // Standard Laravel timestamps
    $table->timestamps(); // created_at, updated_at
    
    // Additional business timestamps
    $table->timestamp('closed_at')->nullable();
    $table->timestamp('due_at')->nullable();
    $table->timestamp('resolved_at')->nullable();
});
```

### Soft Deletes

```php
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

Schema::create('tickets', function (Blueprint $table) {
    $table->id();
    // ... columns
    $table->timestamps();
    $table->softDeletes(); // deleted_at
});
```

**Model:**
```php
use Illuminate\Database\Eloquent\SoftDeletes;

class Ticket extends Model
{
    use SoftDeletes;
    
    protected $dates = ['deleted_at'];
}
```

---

## 📝 Column Definitions

### String Columns

```php
// ✅ Use appropriate lengths
$table->string('name', 255); // Default 255
$table->string('slug', 100); // Shorter for slugs
$table->string('email', 255); // Emails

// ✅ Use text for longer content
$table->text('description'); // Short text
$table->longText('content'); // Long text (e.g., ticket body)
```

### Numeric Columns

```php
// Integers
$table->tinyInteger('priority')->default(1); // 0-255
$table->integer('order')->default(0); // -2B to 2B
$table->bigInteger('external_id')->nullable(); // Large numbers

// Decimals
$table->decimal('amount', 10, 2); // 10 digits, 2 decimals (e.g., money)
$table->float('rating', 3, 2); // 0.00 to 9.99
```

### Boolean Columns

```php
// ✅ CORRECT - Boolean with default
$table->boolean('is_active')->default(true);
$table->boolean('is_resolved')->default(false);

// ❌ WRONG - No default
$table->boolean('is_active'); // Can be NULL (confusing)
```

### Enum Columns

```php
// ✅ CORRECT - Enum with default
$table->enum('status', ['open', 'in_progress', 'closed'])->default('open');
$table->enum('priority', ['low', 'medium', 'high'])->default('medium');

// ⚠️ WARNING - Enum changes require migration
// Adding new value requires:
DB::statement("ALTER TABLE tickets MODIFY status ENUM('open', 'in_progress', 'on_hold', 'closed')");
```

**Alternative: Use String + Validation**
```php
// More flexible but less DB-enforced
$table->string('status', 20)->default('open');

// Validation in Model
public static $statuses = ['open', 'in_progress', 'closed'];

public function setStatusAttribute($value)
{
    if (!in_array($value, self::$statuses)) {
        throw new \InvalidArgumentException("Invalid status: {$value}");
    }
    $this->attributes['status'] = $value;
}
```

### JSON Columns

```php
// ✅ For structured data
$table->json('metadata')->nullable();
$table->json('settings')->nullable();

// Model casting
protected $casts = [
    'metadata' => 'array',
    'settings' => 'array',
];
```

### Nullable Columns

```php
// ✅ Explicitly mark optional fields as nullable
$table->string('middle_name')->nullable();
$table->timestamp('resolved_at')->nullable();
$table->foreignId('assigned_to')->nullable()->constrained('users');

// ❌ WRONG - Required fields without default
$table->string('title'); // OK if always provided
$table->integer('order'); // Should have default or be nullable
```

---

## 🗃️ System Tables Reference

### Declare System Tables in extension.json

```json
{
  "slug": "tickets",
  "system_tables": [
    "users",
    "permissions",
    "roles"
  ]
}
```

**Purpose:** Tells the system which tables the extension USES but does NOT create.

### Common System Tables

| Table | Description | Usage |
|-------|-------------|-------|
| `users` | User accounts | Foreign key for ownership |
| `permissions` | Permissions | Extension permissions |
| `roles` | User roles | Role-based access |
| `password_reset_tokens` | Password resets | Usually not referenced |
| `sessions` | User sessions | Usually not referenced |
| `cache` | Cache storage | Usually not referenced |

**Example Foreign Keys to System Tables:**
```php
Schema::create('tickets', function (Blueprint $table) {
    $table->id();
    
    // Reference system tables
    $table->foreignId('user_id')
        ->constrained('users')
        ->onDelete('cascade');
    
    $table->foreignId('assigned_to')
        ->nullable()
        ->constrained('users')
        ->onDelete('set null');
    
    // ... rest of table
});
```

---

## 🔄 Migration Best Practices

### File Naming

```
database/migrations/
├── 2025_01_01_000001_create_tasks_table.php           ← Main table first
├── 2025_01_01_000002_create_tasks_categories_table.php
├── 2025_01_01_000003_create_tasks_tags_table.php
├── 2025_01_01_000004_create_task_tag_pivot_table.php ← Pivot last
├── 2025_01_01_000005_add_category_to_tasks.php       ← Alterations after creation
```

**Ordering:**
1. Main tables (no foreign keys)
2. Related tables (with foreign keys to main)
3. Pivot tables (reference multiple tables)
4. Alterations to existing tables

### Migration Structure

```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    /**
     * Run the migrations.
     */
    public function up(): void
    {
        Schema::create('tasks', function (Blueprint $table) {
            // Primary key
            $table->id();
            
            // Foreign keys
            $table->foreignId('user_id')
                ->constrained('users')
                ->onDelete('cascade');
            
            // Data columns
            $table->string('title');
            $table->text('description')->nullable();
            $table->enum('status', ['pending', 'completed'])->default('pending');
            
            // Timestamps
            $table->timestamps();
            
            // Indexes (at the end)
            $table->index('status');
            $table->index(['user_id', 'status']);
        });
    }

    /**
     * Reverse the migrations.
     */
    public function down(): void
    {
        Schema::dropIfExists('tasks');
    }
};
```

### Altering Tables

```php
// Add column
Schema::table('tasks', function (Blueprint $table) {
    $table->string('priority', 20)->default('medium')->after('status');
});

// Modify column (requires doctrine/dbal)
Schema::table('tasks', function (Blueprint $table) {
    $table->string('title', 500)->change(); // Increase length
});

// Drop column
Schema::table('tasks', function (Blueprint $table) {
    $table->dropColumn('old_column');
});

// Drop foreign key before dropping column
Schema::table('tasks', function (Blueprint $table) {
    $table->dropForeign(['category_id']); // Drop FK first
    $table->dropColumn('category_id');
});
```

### Rollback Safety

```php
// ✅ CORRECT - Reversible migration
public function up(): void
{
    Schema::table('tasks', function (Blueprint $table) {
        $table->string('priority')->default('medium')->after('status');
    });
}

public function down(): void
{
    Schema::table('tasks', function (Blueprint $table) {
        $table->dropColumn('priority');
    });
}

// ❌ WRONG - Non-reversible
public function down(): void
{
    // Empty - cannot rollback!
}
```

---

## 📦 List Migrations in extension.json

```json
{
  "migrations": {
    "required": true,
    "count": 4,
    "can_skip": false,
    "details": [
      "2025_01_01_000001_create_tasks_table",
      "2025_01_01_000002_create_tasks_categories_table",
      "2025_01_01_000003_create_tasks_tags_table",
      "2025_01_01_000004_create_task_tag_pivot_table"
    ],
    "description": "Creates tasks system with categories, tags, and pivot tables"
  }
}
```

**Rules:**
- `count` must match number of migration files
- `details` array must list all migration file names (without .php)
- `can_skip`: true if migrations are optional (rare)
- `required`: false only if extension has NO database component

---

## 🧪 Testing Migrations

### Test Creation

```bash
# Run migrations
php artisan migrate

# Verify tables created
php artisan tinker
>>> Schema::hasTable('tasks')
=> true
>>> Schema::hasTable('tasks_categories')
=> true
```

### Test Rollback

```bash
# Rollback last batch
php artisan migrate:rollback

# Verify tables dropped
php artisan tinker
>>> Schema::hasTable('tasks')
=> false
```

### Test Fresh Install

```bash
# Drop all tables and re-migrate
php artisan migrate:fresh

# Verify
php artisan tinker
>>> Schema::hasTable('tasks')
=> true
```

---

## 🔗 Related Documentation

- **[Extension JSON Schema](../../CPANEL/docs/extensions/EXTENSION-JSON-SCHEMA.md)** - migrations field reference
- **[Seeders Best Practices](./SEEDERS-BEST-PRACTICES.md)** - How to populate database
- **[Developer Guide](../DEVELOPER-GUIDE.md)** - Complete development workflow
- **[Laravel Migrations](https://laravel.com/docs/11.x/migrations)** - Official Laravel docs

---

## ✅ Checklist: Before Pushing Migration

- [ ] Table name uses correct prefix (`slug_tablename` or just `slug` for main)
- [ ] Primary key is `id` (bigint unsigned auto_increment)
- [ ] Foreign keys use `constrained()` with explicit table name
- [ ] All foreign keys have `onDelete()` strategy
- [ ] Timestamps included (`$table->timestamps()`)
- [ ] Indexes added for foreign keys and search columns
- [ ] Unique constraints where needed
- [ ] Column lengths appropriate
- [ ] Nullable columns marked explicitly
- [ ] Default values for required columns
- [ ] `down()` method properly reverses `up()`
- [ ] Listed in extension.json `migrations.details`
- [ ] Migration count in extension.json matches actual files
- [ ] Tested: `migrate`, `migrate:rollback`, `migrate:fresh`

---

**Remember:** Database conventions ensure your extension is compatible with the BITHOVEN ecosystem. Follow them strictly!
