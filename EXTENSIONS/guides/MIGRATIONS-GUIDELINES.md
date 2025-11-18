# Migration Guidelines for BITHOVEN Extensions

**Version:** 2.0.0  
**Last Updated:** 17 de noviembre de 2025  
**Status:** STABLE

---

## 🎯 Overview

Las migraciones son el control de versiones de tu base de datos. Estas guías aseguran que sean:
- Reversibles
- Idempotentes
- Compatibles con el sistema de extensiones BITHOVEN

---

## 📋 Migration Naming Convention

### Format: `YYYY_MM_DD_HHMMSS_action_tablename.php`

**Examples:**
```
2025_01_01_000001_create_tasks_table.php
2025_01_01_000002_add_priority_to_tasks.php
2025_01_01_000003_create_tasks_categories_table.php
```

### Action Verbs

- `create_{table}` - Nueva tabla
- `add_{column}_to_{table}` - Agregar columna
- `remove_{column}_from_{table}` - Eliminar columna
- `modify_{column}_in_{table}` - Modificar columna
- `rename_{old}_to_{new}_in_{table}` - Renombrar columna
- `create_{table1}_{table2}_pivot` - Tabla pivot

---

## 📊 Migration Ordering

**⚠️Regla:** Dependencias primero, alteraciones después.

### Example:

```
database/migrations/
├── 2025_01_01_000001_create_tasks_table.php           # 1. Tablas principales
├── 2025_01_01_000002_create_tasks_categories_table.php # 2. Tablas sin FK
├── 2025_01_01_000003_create_tasks_tags_table.php      # 3. Tablas relacionadas
├── 2025_01_01_000004_create_task_tag_pivot_table.php  # 4. Pivots (last)
├── 2025_01_01_000005_add_category_to_tasks.php        # 5. Alteraciones
```

**Why?** Para que rollback funcione en orden inverso sin romper FKs.

---

## ✅ Best Practices

### 1. Always Reversible

```php
// ✅ CORRECT - Fully reversible
public function up(): void
{
    Schema::table('tasks', function (Blueprint $table) {
        $table->string('priority', 20)->default('medium')->after('status');
    });
}

public function down(): void
{
    Schema::table('tasks', function (Blueprint $table) {
        $table->dropColumn('priority');
    });
}

// ❌ WRONG - Not reversible
public function down(): void
{
    // Empty - cannot rollback!
}
```

### 2. Use Explicit Table Names

```php
// ✅ CORRECT
$table->foreignId('user_id')
    ->constrained('users')
    ->onDelete('cascade');

// ❌ WRONG - Laravel guesses
$table->foreignId('user_id')
    ->constrained(); // Might fail
```

### 3. Set Default Values

```php
// ✅ CORRECT
$table->boolean('is_active')->default(true);
$table->enum('status', ['open', 'closed'])->default('open');
$table->integer('order')->default(0);

// ❌ WRONG - No default
$table->boolean('is_active'); // Can be NULL
```

### 4. Index Foreign Keys

```php
Schema::create('tasks', function (Blueprint $table) {
    $table->id();
    $table->foreignId('user_id')->constrained()->onDelete('cascade');
    $table->string('title');
    $table->enum('status', ['open', 'closed'])->default('open');
    
    // ✅ Index foreign keys and search columns
    $table->index('status');
    $table->index(['user_id', 'status']); // Composite
});
```

### 5. Use Soft Deletes When Appropriate

```php
Schema::create('tasks', function (Blueprint $table) {
    $table->id();
    // ... columns
    $table->timestamps();
    $table->softDeletes(); // deleted_at
});
```

**Model:**
```php
use Illuminate\Database\Eloquent\SoftDeletes;

class Task extends Model
{
    use SoftDeletes;
}
```

---

## 🔧 Common Migration Patterns

### Create Table

```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::create('tasks', function (Blueprint $table) {
            $table->id();
            $table->foreignId('user_id')->constrained()->onDelete('cascade');
            $table->string('title');
            $table->text('description')->nullable();
            $table->enum('status', ['pending', 'completed'])->default('pending');
            $table->timestamp('completed_at')->nullable();
            $table->timestamps();
            
            $table->index('status');
            $table->index(['user_id', 'status']);
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('tasks');
    }
};
```

### Add Column

```php
public function up(): void
{
    Schema::table('tasks', function (Blueprint $table) {
        $table->string('priority', 20)->default('medium')->after('status');
        $table->index('priority');
    });
}

public function down(): void
{
    Schema::table('tasks', function (Blueprint $table) {
        $table->dropIndex(['priority']); // Drop index first
        $table->dropColumn('priority');
    });
}
```

### Add Foreign Key

```php
public function up(): void
{
    Schema::table('tasks', function (Blueprint $table) {
        $table->foreignId('category_id')
            ->nullable()
            ->after('user_id')
            ->constrained('tasks_categories')
            ->onDelete('set null');
    });
}

public function down(): void
{
    Schema::table('tasks', function (Blueprint $table) {
        $table->dropForeign(['category_id']);
        $table->dropColumn('category_id');
    });
}
```

### Modify Column

**Requires:** `composer require doctrine/dbal`

```php
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

public function up(): void
{
    Schema::table('tasks', function (Blueprint $table) {
        $table->string('title', 500)->change(); // Increase length
    });
}

public function down(): void
{
    Schema::table('tasks', function (Blueprint $table) {
        $table->string('title', 255)->change(); // Restore original
    });
}
```

### Rename Column

```php
public function up(): void
{
    Schema::table('tasks', function (Blueprint $table) {
        $table->renameColumn('description', 'content');
    });
}

public function down(): void
{
    Schema::table('tasks', function (Blueprint $table) {
        $table->renameColumn('content', 'description');
    });
}
```

### Create Pivot Table

```php
public function up(): void
{
    Schema::create('task_user', function (Blueprint $table) {
        $table->foreignId('task_id')->constrained('tasks')->onDelete('cascade');
        $table->foreignId('user_id')->constrained('users')->onDelete('cascade');
        
        $table->primary(['task_id', 'user_id']);
        $table->timestamps();
    });
}

public function down(): void
{
    Schema::dropIfExists('task_user');
}
```

---

## 🗂️ List in extension.json

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

**⚠️ Important:**
- `count` must match actual number of migration files
- `details` array lists all migration names (without .php)
- Order matches execution order

---

## 🧪 Testing Migrations

### Test Creation

```bash
php artisan migrate

# Verify
php artisan tinker
>>> Schema::hasTable('tasks')
=> true
```

### Test Rollback

```bash
php artisan migrate:rollback

# Verify
php artisan tinker
>>> Schema::hasTable('tasks')
=> false
```

### Test Fresh Install

```bash
php artisan migrate:fresh

# Verify all tables created
php artisan tinker
>>> Schema::hasTable('tasks')
=> true
```

### Test Specific Migration

```bash
# Run single migration
php artisan migrate --path=database/migrations/2025_01_01_000001_create_tasks_table.php

# Rollback single migration
php artisan migrate:rollback --step=1
```

---

## ⚠️ Common Mistakes

### ❌ Wrong: No Foreign Key Constraint

```php
// ❌ WRONG - No FK constraint
$table->unsignedBigInteger('user_id');

// ✅ CORRECT
$table->foreignId('user_id')->constrained()->onDelete('cascade');
```

### ❌ Wrong: No onDelete Strategy

```php
// ❌ WRONG - No delete strategy
$table->foreignId('user_id')->constrained();

// ✅ CORRECT
$table->foreignId('user_id')
    ->constrained()
    ->onDelete('cascade'); // or set null, restrict
```

### ❌ Wrong: Empty down() Method

```php
// ❌ WRONG
public function down(): void
{
    // Empty - not reversible!
}

// ✅ CORRECT
public function down(): void
{
    Schema::dropIfExists('tasks');
}
```

### ❌ Wrong: Dropping Column with FK First

```php
// ❌ WRONG - FK still exists
public function down(): void
{
    Schema::table('tasks', function (Blueprint $table) {
        $table->dropColumn('category_id'); // Error: FK exists
    });
}

// ✅ CORRECT - Drop FK first
public function down(): void
{
    Schema::table('tasks', function (Blueprint $table) {
        $table->dropForeign(['category_id']);
        $table->dropColumn('category_id');
    });
}
```

---

## 📚 Related Documentation

- **[Database Conventions](./DATABASE-CONVENTIONS.md)** - Table naming, indexes, etc.
- **[Extension Structure](./EXTENSION-STRUCTURE.md)** - Directory organization
- **[Developer Guide](../DEVELOPER-GUIDE.md)** - Complete workflow
- **[Laravel Migrations](https://laravel.com/docs/11.x/migrations)** - Official docs

---

## ✅ Checklist: Before Committing Migration

- [ ] File name follows convention (`YYYY_MM_DD_HHMMSS_action_table.php`)
- [ ] Table name uses correct prefix (see DATABASE-CONVENTIONS.md)
- [ ] Primary key is `id` (bigint unsigned auto_increment)
- [ ] Foreign keys use `constrained()` with explicit table
- [ ] All foreign keys have `onDelete()` strategy
- [ ] Indexes added for foreign keys and search columns
- [ ] Timestamps included (`$table->timestamps()`)
- [ ] Default values for required columns
- [ ] Nullable columns marked explicitly
- [ ] `down()` method properly reverses `up()`
- [ ] Listed in extension.json `migrations.details`
- [ ] Migration count matches extension.json
- [ ] Tested: `migrate`, `migrate:rollback`, `migrate:fresh`

---

**Remember:** Well-written migrations are the foundation of a stable extension!
