# Fresh Install System

**Version:** 2.0.0  
**Last Updated:** 17 de noviembre de 2025  
**Status:** STABLE

---

## 🎯 Overview

El sistema Fresh Install permite reinstalar completamente una extensión, útil para:
- Desarrollo y testing
- Recuperación de errores
- Actualización mayor
- Resetear datos de demo

---

## 🔄 What is Fresh Install?

**Fresh Install = Uninstall + Install**

```
Current State
     ↓
Uninstall (remove everything)
     ↓
Clean State
     ↓
Install (fresh start)
     ↓
New State
```

---

## 📋 Fresh Install Process

### Step 1: Uninstall

```bash
php artisan bithoven:extension:uninstall tasks
```

**What happens:**
1. Extension disabled
2. Permissions removed
3. Menu items removed
4. Migrations rolled back
5. Composer package removed
6. Config files cleaned

### Step 2: Install

```bash
php artisan bithoven:extension:install tasks
```

**What happens:**
1. Package downloaded
2. Migrations run
3. Seeders run
4. Permissions created
5. Menu items added
6. Extension activated

---

## 🚀 Use Cases

### Use Case 1: Development Testing

**Scenario:** Testing migration changes

```bash
# Make changes to migration
vim database/migrations/2025_01_01_000001_create_tasks_table.php

# Fresh install
cd /path/to/CPANEL
php artisan bithoven:extension:uninstall tasks
php artisan bithoven:extension:install tasks --local --path=../EXTENSIONS/bithoven-extension-tasks

# Verify
php artisan tinker
>>> Schema::hasTable('tasks')
=> true
```

### Use Case 2: Reset Demo Data

**Scenario:** Testing with clean demo data

```bash
# Uninstall
php artisan bithoven:extension:uninstall tasks

# Reinstall
php artisan bithoven:extension:install tasks

# Reseed demo data
php artisan db:seed --class="Bithoven\Tasks\Database\Seeders\TasksDemoSeeder"
```

### Use Case 3: Fix Broken State

**Scenario:** Extension in broken state

```bash
# Nuclear option: fresh install
php artisan bithoven:extension:uninstall tasks --force
php artisan bithoven:extension:install tasks
```

---

## ⚙️ Fresh Install Options

### Standard Fresh Install

```bash
# Uninstall + Install
php artisan bithoven:extension:uninstall tasks
php artisan bithoven:extension:install tasks
```

### Force Fresh Install

```bash
# Force uninstall even if errors
php artisan bithoven:extension:uninstall tasks --force
php artisan bithoven:extension:install tasks
```

### Keep User Data

**NOT BUILT-IN - Custom Implementation:**

```php
// In migration down()
public function down(): void
{
    // Instead of dropping table
    // Schema::dropIfExists('tasks');
    
    // Just truncate
    DB::table('tasks')->truncate();
}
```

---

## 🗄️ Database Handling

### Option 1: Drop Tables (Default)

**Pros:** Clean slate  
**Cons:** All data lost

```php
public function down(): void
{
    Schema::dropIfExists('tasks');
}
```

### Option 2: Truncate Tables

**Pros:** Keep structure  
**Cons:** Still lose data

```php
public function down(): void
{
    DB::table('tasks')->truncate();
}
```

### Option 3: Backup Before Drop

**Custom implementation:**

```php
public function down(): void
{
    // Backup
    $tasks = DB::table('tasks')->get();
    Storage::put('backups/tasks_' . now()->timestamp . '.json', $tasks->toJson());
    
    // Drop
    Schema::dropIfExists('tasks');
}
```

---

## 🔧 Manual Fresh Install Steps

### Step 1: Backup Data (Optional)

```bash
# Database dump
php artisan backup:run --only-db

# Or manual
mysqldump -u root bithoven_laravel tasks > tasks_backup.sql
```

### Step 2: Uninstall Extension

```bash
php artisan bithoven:extension:uninstall tasks
```

**Verify:**
```bash
# Check tables dropped
php artisan tinker
>>> Schema::hasTable('tasks')
=> false

# Check permissions removed
>>> Permission::where('name', 'like', '%tasks%')->count()
=> 0
```

### Step 3: Clean Vendor (Optional)

```bash
rm -rf vendor/bithoven/tasks
composer dump-autoload
```

### Step 4: Reinstall

```bash
php artisan bithoven:extension:install tasks
```

### Step 5: Verify

```bash
# Check installation
php artisan bithoven:extension:list

# Check tables
php artisan tinker
>>> Schema::hasTable('tasks')
=> true

# Check routes
php artisan route:list | grep tasks
```

### Step 6: Restore Data (Optional)

```bash
# If you backed up
mysql -u root bithoven_laravel < tasks_backup.sql
```

---

## 🧪 Testing Fresh Install

### Test 1: Basic Flow

```bash
# Install
php artisan bithoven:extension:install tasks --local --path=../EXTENSIONS/bithoven-extension-tasks

# Verify working
curl http://localhost:8000/tasks

# Uninstall
php artisan bithoven:extension:uninstall tasks

# Verify removed
curl http://localhost:8000/tasks # Should 404

# Reinstall
php artisan bithoven:extension:install tasks --local --path=../EXTENSIONS/bithoven-extension-tasks

# Verify working again
curl http://localhost:8000/tasks
```

### Test 2: Data Persistence

```bash
# Install & seed
php artisan bithoven:extension:install tasks
php artisan db:seed --class="Bithoven\Tasks\Database\Seeders\TasksDemoSeeder"

# Check data
php artisan tinker
>>> Task::count()
=> 50

# Uninstall
php artisan bithoven:extension:uninstall tasks

# Check data removed
php artisan tinker
>>> Task::count() # Error: class not found (expected)

# Reinstall
php artisan bithoven:extension:install tasks

# Check data empty
php artisan tinker
>>> Task::count()
=> 0
```

---

## ⚠️ Common Issues

### Issue: Tables Not Dropped

**Cause:** Foreign key constraints

**Solution:**
```php
// In migration down()
public function down(): void
{
    Schema::dropIfExists('task_user'); // Drop FK tables first
    Schema::dropIfExists('tasks');      // Then main table
}
```

### Issue: Permissions Remain

**Cause:** Soft delete or cache

**Solution:**
```bash
# Clear cache
php artisan cache:clear

# Manually remove
php artisan tinker
>>> Permission::where('name', 'like', '%tasks%')->forceDelete();
```

### Issue: Routes Still Cached

**Cause:** Route cache not cleared

**Solution:**
```bash
php artisan route:clear
php artisan optimize:clear
```

---

## 🔄 Fresh Install vs Update

### Fresh Install
- Uninstall + Install
- All data lost
- Clean slate
- Use for: development, major changes

### Update
- Code update only
- Data preserved
- Migrations run incrementally
- Use for: production, minor updates

**Update process:**
```bash
# Pull latest code
cd ../EXTENSIONS/bithoven-extension-tasks
git pull

# Update in CPANEL (if local mode)
cd /path/to/CPANEL
composer update bithoven/tasks

# Run new migrations
php artisan migrate
```

---

## 📋 Checklist: Before Fresh Install

- [ ] Backup database if needed
- [ ] Export critical data
- [ ] Notify users (if production)
- [ ] Check no active processes using tables
- [ ] Verify extension is in clean state
- [ ] Have rollback plan ready

---

## 🔗 Related Documentation

- **[Installation Flow](../../CPANEL/docs/extensions/EXTENSION-INSTALLATION-FLOW.md)** - Detailed install process
- **[Backup & Recovery](./BACKUP-RECOVERY.md)** - Backup strategies
- **[Development Workflow](./DEVELOPMENT-WORKFLOW.md)** - Dev practices
- **[Migrations Guidelines](./MIGRATIONS-GUIDELINES.md)** - Migration best practices

---

**Remember:** Fresh install is powerful but destructive. Always backup production data first!
