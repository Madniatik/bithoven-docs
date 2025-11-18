# Backup & Recovery for BITHOVEN Extensions

**Version:** 2.0.0  
**Last Updated:** 17 de noviembre de 2025  
**Status:** STABLE

---

## 🎯 Overview

Sistema de backup y recuperación para extensiones, permitiendo:
- Backup automático antes de operaciones críticas
- Restauración rápida en caso de errores
- Testing sin riesgo
- Migración entre entornos

---

## 💾 What to Backup

### 1. Database

**Extension tables:**
- Main tables (`tasks`, `tickets`, etc.)
- Related tables (`tasks_categories`, etc.)
- Pivot tables (`task_user`, etc.)
- System table data (permissions, menu items)

### 2. Configuration

- `extension-settings.json`
- `config/bithoven-extensions.php`
- `composer.json` (repositories + requires)

### 3. User Files (if applicable)

- Uploaded attachments
- Generated reports
- Cached data

---

## 📋 Backup Strategies

### Strategy 1: Manual Backup

```bash
# Full database backup
php artisan backup:run --only-db

# Or mysqldump
mysqldump -u root bithoven_laravel > backup_$(date +%Y%m%d_%H%M%S).sql
```

### Strategy 2: Automatic Backup

**Before critical operations:**

```php
// In ExtensionManager before uninstall
if (!$this->backupExists($slug)) {
    $this->createBackup($slug);
}
```

### Strategy 3: Selective Backup

**Only extension tables:**

```bash
# Backup specific tables
mysqldump -u root bithoven_laravel tasks tasks_categories task_user > tasks_backup.sql
```

---

## 🔧 Backup Implementation

### PHP Implementation

```php
<?php

namespace App\Services;

use Illuminate\Support\Facades\DB;
use Illuminate\Support\Facades\Storage;

class ExtensionBackupService
{
    /**
     * Create backup of extension data
     */
    public function backup(string $slug): string
    {
        $timestamp = now()->format('Y-m-d_H-i-s');
        $filename = "extension_{$slug}_{$timestamp}.sql";
        
        // Get extension tables
        $tables = $this->getExtensionTables($slug);
        
        // Create SQL dump
        $sql = $this->generateSqlDump($tables);
        
        // Save to storage
        Storage::put("backups/{$filename}", $sql);
        
        return $filename;
    }
    
    /**
     * Restore extension data from backup
     */
    public function restore(string $filename): bool
    {
        if (!Storage::exists("backups/{$filename}")) {
            return false;
        }
        
        $sql = Storage::get("backups/{$filename}");
        
        DB::unprepared($sql);
        
        return true;
    }
    
    /**
     * Get tables belonging to extension
     */
    protected function getExtensionTables(string $slug): array
    {
        $tables = [];
        
        // Main table
        $tables[] = $slug;
        
        // Related tables (prefix pattern)
        $allTables = DB::select('SHOW TABLES');
        $dbName = env('DB_DATABASE');
        
        foreach ($allTables as $table) {
            $tableName = $table->{"Tables_in_{$dbName}"};
            if (str_starts_with($tableName, "{$slug}_")) {
                $tables[] = $tableName;
            }
        }
        
        return $tables;
    }
    
    /**
     * Generate SQL dump for tables
     */
    protected function generateSqlDump(array $tables): string
    {
        $sql = "-- Extension Backup " . now() . "\n\n";
        
        foreach ($tables as $table) {
            // Disable FK checks
            $sql .= "SET FOREIGN_KEY_CHECKS=0;\n";
            
            // Drop table
            $sql .= "DROP TABLE IF EXISTS `{$table}`;\n\n";
            
            // Create table
            $createTable = DB::select("SHOW CREATE TABLE `{$table}`");
            $sql .= $createTable[0]->{'Create Table'} . ";\n\n";
            
            // Insert data
            $rows = DB::table($table)->get();
            if ($rows->isNotEmpty()) {
                $sql .= "INSERT INTO `{$table}` VALUES\n";
                $values = [];
                foreach ($rows as $row) {
                    $rowArray = (array) $row;
                    $escaped = array_map(function($value) {
                        return is_null($value) ? 'NULL' : "'" . addslashes($value) . "'";
                    }, $rowArray);
                    $values[] = '(' . implode(', ', $escaped) . ')';
                }
                $sql .= implode(",\n", $values) . ";\n\n";
            }
            
            // Re-enable FK checks
            $sql .= "SET FOREIGN_KEY_CHECKS=1;\n\n";
        }
        
        return $sql;
    }
}
```

### Usage

```php
use App\Services\ExtensionBackupService;

$backupService = new ExtensionBackupService();

// Create backup
$filename = $backupService->backup('tasks');
echo "Backup created: {$filename}";

// Restore backup
$backupService->restore('extension_tasks_2025-01-01_12-00-00.sql');
echo "Backup restored successfully";
```

---

## 🔄 Recovery Scenarios

### Scenario 1: Failed Uninstall

**Problem:** Uninstall left orphaned data

**Solution:**
```bash
# Restore from backup
php artisan backup:restore extension_tasks_latest.sql

# Or manually
mysql -u root bithoven_laravel < backups/extension_tasks_2025-01-01.sql
```

### Scenario 2: Broken Migration

**Problem:** Migration ran but broke data

**Solution:**
```bash
# Rollback migration
php artisan migrate:rollback --step=1

# Restore data backup
mysql -u root bithoven_laravel < backups/before_migration.sql

# Fix migration
vim database/migrations/...

# Re-run
php artisan migrate
```

### Scenario 3: Corrupted Data

**Problem:** User data corrupted

**Solution:**
```bash
# Restore specific tables
mysql -u root bithoven_laravel -e "DROP TABLE tasks"
mysql -u root bithoven_laravel < backups/tasks_only.sql
```

---

## 🧪 Testing with Backups

### Test Workflow

```bash
# 1. Create baseline backup
mysqldump -u root bithoven_laravel > baseline.sql

# 2. Install extension
php artisan bithoven:extension:install tasks

# 3. Test features
# ... manual testing ...

# 4. If broken, restore baseline
mysql -u root bithoven_laravel < baseline.sql

# 5. Fix and retry
```

---

## 📊 Backup Best Practices

### 1. Backup Before Critical Operations

```php
// Before uninstall
$this->backup($slug);
$this->uninstall($slug);

// Before major migration
$this->backup($slug);
$this->runMigration($migration);
```

### 2. Name Backups Descriptively

```
Format: {type}_{slug}_{timestamp}.sql

Examples:
- extension_tasks_2025-01-01_12-00-00.sql
- before_migration_tasks_2025-01-01.sql
- before_uninstall_tickets_2025-01-01.sql
```

### 3. Keep Multiple Versions

```bash
backups/
├── extension_tasks_2025-01-01_12-00-00.sql
├── extension_tasks_2025-01-02_14-30-00.sql
├── extension_tasks_2025-01-03_09-15-00.sql
└── extension_tasks_latest.sql (symlink)
```

### 4. Clean Old Backups

```bash
# Keep last 10 backups
ls -t backups/extension_tasks_*.sql | tail -n +11 | xargs rm
```

---

## 🗄️ Storage Options

### Option 1: Local Storage

**Pros:** Fast, simple  
**Cons:** Lost if server fails

```php
Storage::disk('local')->put("backups/{$filename}", $sql);
```

### Option 2: S3 Storage

**Pros:** Durable, offsite  
**Cons:** Slower, costs money

```php
Storage::disk('s3')->put("backups/{$filename}", $sql);
```

### Option 3: Both

**Best of both worlds:**

```php
// Save locally
Storage::disk('local')->put("backups/{$filename}", $sql);

// Also save to S3
Storage::disk('s3')->put("backups/{$filename}", $sql);
```

---

## 🔐 Backup Security

### Encrypt Sensitive Backups

```php
use Illuminate\Support\Facades\Crypt;

// Encrypt before saving
$encrypted = Crypt::encrypt($sql);
Storage::put("backups/{$filename}.enc", $encrypted);

// Decrypt when restoring
$encrypted = Storage::get("backups/{$filename}.enc");
$sql = Crypt::decrypt($encrypted);
```

### Restrict Access

```php
// .htaccess in storage/app/backups/
Order Deny,Allow
Deny from all
```

---

## 🔧 CLI Commands

### Create Backup Command

```php
<?php

namespace App\Console\Commands;

use Illuminate\Console\Command;
use App\Services\ExtensionBackupService;

class BackupExtension extends Command
{
    protected $signature = 'extension:backup {slug}';
    protected $description = 'Create backup of extension data';
    
    public function handle(ExtensionBackupService $backup)
    {
        $slug = $this->argument('slug');
        
        $this->info("Creating backup for {$slug}...");
        
        $filename = $backup->backup($slug);
        
        $this->info("✅ Backup created: {$filename}");
    }
}
```

### Restore Backup Command

```php
class RestoreExtension extends Command
{
    protected $signature = 'extension:restore {filename}';
    protected $description = 'Restore extension from backup';
    
    public function handle(ExtensionBackupService $backup)
    {
        $filename = $this->argument('filename');
        
        if (!$this->confirm("This will overwrite current data. Continue?")) {
            return;
        }
        
        $this->info("Restoring {$filename}...");
        
        $backup->restore($filename);
        
        $this->info("✅ Backup restored successfully");
    }
}
```

### Usage

```bash
# Create backup
php artisan extension:backup tasks

# Restore backup
php artisan extension:restore extension_tasks_2025-01-01_12-00-00.sql
```

---

## 📋 Backup Checklist

**Before Uninstall:**
- [ ] Create full backup
- [ ] Verify backup file created
- [ ] Test backup is restorable
- [ ] Note backup filename

**Before Major Migration:**
- [ ] Backup current state
- [ ] Test migration on copy
- [ ] Keep rollback script ready
- [ ] Document changes

**Regular Maintenance:**
- [ ] Weekly full backups
- [ ] Test restore monthly
- [ ] Clean old backups
- [ ] Verify backup integrity

---

## 🔗 Related Documentation

- **[Fresh Install System](./FRESH-INSTALL-SYSTEM.md)** - Reinstallation process
- **[Development Workflow](./DEVELOPMENT-WORKFLOW.md)** - Dev best practices
- **[Migrations Guidelines](./MIGRATIONS-GUIDELINES.md)** - Migration safety

---

**Remember:** Backups are insurance - you hope to never need them, but you're glad they're there!
