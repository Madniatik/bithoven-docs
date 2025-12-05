# Migration Fixes & Database Management

**Date:** December 5, 2025  
**Version:** 1.8.1

---

## Overview

This document covers critical fixes applied to the migration system and database management tools, including backup/restore scripts and Extension Manager validation fixes.

---

## Migration System Fixes

### Issue: Extension Settings Migration Conflicts

**Problem:**
Extension settings were being saved with incorrect structure, causing validation errors and migration failures.

**Symptoms:**
- Extension config update failing with "The name field is required"
- Settings not persisting in extension config files
- Debug Console not loading when disabled

**Root Cause:**
`UpdateSettingsRequest` validation included unnecessary `'name'` field that didn't exist in extension settings.

**Fix Applied (Commit: ec670bc):**

```php
// app/Http/Requests/Extensions/UpdateSettingsRequest.php

// BEFORE (broken)
public function rules(): array
{
    return [
        'name' => 'required|string|max:255',  // ❌ Extension settings don't have 'name'
        'settings' => 'required|array',
        'settings.*' => 'nullable',
    ];
}

// AFTER (fixed)
public function rules(): array
{
    return [
        // Removed 'name' field - not needed for extension config
        'settings' => 'required|array',
        'settings.*' => 'nullable',
    ];
}
```

**Impact:**
- ✅ Extension settings now save correctly
- ✅ No more validation errors on config updates
- ✅ Debug Console config updates work properly

---

## Database Backup & Restore Scripts

### New Scripts Added (Commit: ec670bc)

Two production-ready scripts were added to manage database backups:

#### 1. Backup Script: `scripts/backup-database.sh`

**Features:**
- Automatic backup directory creation (`backups/mysql/`)
- Timestamped filenames (YYYYMMDD-HHMM)
- Compression with gzip
- Metadata file with backup details
- Verification of backup integrity
- Summary statistics (size, tables, rows)

**Usage:**
```bash
# Standard backup
./scripts/backup-database.sh

# Custom description
./scripts/backup-database.sh "Pre-production deployment"
```

**Output:**
```
📦 Bithoven Laravel - Database Backup Script
============================================

📋 Configuration:
   Database: bithoven_laravel
   User: root
   Backup Dir: /path/to/backups/mysql

🔄 Creating backup...
   Timestamp: 20251205-0544
   Description: Pre-pending-migrations backup
   
✅ Backup created successfully!
   File: /path/to/backups/mysql/bithoven_laravel_20251205-0544.sql.gz
   Size: 200 KB
   Tables: 47
   Total rows: ~1,250
   
📄 Metadata saved: bithoven_laravel_20251205-0544.meta.json
```

**Metadata File Structure:**
```json
{
    "database": "bithoven_laravel",
    "timestamp": "20251205-0544",
    "datetime": "2025-12-05 05:44:12",
    "description": "Pre-pending-migrations backup",
    "file": "bithoven_laravel_20251205-0544.sql.gz",
    "size_bytes": 204800,
    "size_human": "200 KB",
    "tables": 47,
    "estimated_rows": 1250,
    "hostname": "localhost",
    "mysql_version": "8.0.33",
    "created_by": "backup-database.sh v1.0"
}
```

#### 2. Restore Script: `scripts/restore-database.sh`

**Features:**
- List available backups with details
- Interactive selection menu
- Automatic backup before restore
- Decompression handling (gzip)
- Verification prompts
- Rollback capability

**Usage:**
```bash
# List and select backup
./scripts/restore-database.sh

# Restore specific backup
./scripts/restore-database.sh bithoven_laravel_20251205-0544.sql.gz
```

**Interactive Flow:**
```
📦 Bithoven Laravel - Database Restore Script
============================================

📋 Available Backups:

   1. bithoven_laravel_20251205-0544.sql.gz
      Date: 2025-12-05 05:44:12
      Size: 200 KB
      Description: Pre-pending-migrations backup
      Tables: 47 | Rows: ~1,250

   2. bithoven_laravel_20251204-1530.sql.gz
      Date: 2025-12-04 15:30:00
      Size: 195 KB
      Description: Daily backup
      Tables: 47 | Rows: ~1,200

Select backup number (or 'q' to quit): 1

⚠️  Warning: This will replace ALL data in 'bithoven_laravel'!
   Current database will be backed up first as safety measure.

Proceed with restore? (yes/no): yes

🔄 Creating safety backup of current database...
✅ Safety backup created: bithoven_laravel_20251205-0600_pre-restore.sql.gz

🔄 Restoring database from backup...
✅ Database restored successfully!

📊 Restore Summary:
   Restored from: bithoven_laravel_20251205-0544.sql.gz
   Description: Pre-pending-migrations backup
   Tables restored: 47
   Safety backup: bithoven_laravel_20251205-0600_pre-restore.sql.gz
```

**Safety Features:**
- Automatic safety backup before restore
- Confirmation prompt
- Detailed metadata display
- Error handling with rollback

---

## Extension Manager Fixes

### Fix 1: Debug Console Loading Check

**Problem:**
Debug Console JavaScript loading even when `debug_console.enabled = false`, causing unnecessary overhead.

**Fix (Commit: ec670bc):**

```blade
<!-- resources/views/partials/debug-console-init.blade.php -->

<!-- BEFORE (broken) -->
@if(config('app.debug'))
    <script>
        // Always loads if app.debug = true
    </script>
@endif

<!-- AFTER (fixed) -->
@if(setting('debug_console.enabled', true))
    <script>
        // Only loads if Debug Console enabled in settings
    </script>
@endif
```

**Impact:**
- Debug Console respects global enabled/disabled setting
- Reduced JavaScript load when disabled
- Better performance in production

### Fix 2: Nested Array Handling in Config Forms

**Problem:**
Extension config forms couldn't save nested arrays (e.g., `debug_console.level`).

**Fix (Commit: a50dee2):**

```php
// app/Actions/Extensions/Configuration/UpdateSettingsAction.php

// BEFORE (broken)
foreach ($settings as $key => $value) {
    // Only handled flat arrays
    $config[$key] = $value;
}

// AFTER (fixed)
foreach ($settings as $key => $value) {
    if (is_array($value)) {
        // Handle nested arrays recursively
        $config[$key] = $this->processNestedArray($value);
    } else {
        $config[$key] = $value;
    }
}

private function processNestedArray(array $data): array
{
    $result = [];
    foreach ($data as $key => $value) {
        $result[$key] = is_array($value) 
            ? $this->processNestedArray($value) 
            : $value;
    }
    return $result;
}
```

**Impact:**
- Extension configs can now have multi-level arrays
- Debug Console config works: `debug_console.level`
- LLM Manager complex configs save correctly

### Fix 3: Composer Symlink Management

**Problem:**
Extension `composer.json` symlinks not updating when toggling dev mode.

**Fix (Commit: 0b7a334):**

```php
// app/Services/ExtensionManager.php

public function toggleDevMode(string $name): bool
{
    $extension = $this->getExtension($name);
    
    // Update symlink
    $composerPath = base_path("vendor/bithoven/{$name}/composer.json");
    $realPath = $extension->path . '/composer.json';
    
    if (file_exists($composerPath)) {
        unlink($composerPath);
    }
    
    if ($extension->dev_mode) {
        symlink($realPath, $composerPath);
    }
    
    return true;
}
```

**Impact:**
- ✅ `composer.json` always points to correct source
- ✅ Dev mode changes reflected immediately
- ✅ No manual symlink recreation needed

---

## Database Cleanup

### Removed Incorrect Config File

**Issue:**
`config/tickets.php` was incorrectly placed in main app config directory instead of extension directory.

**Fix (Commit: ec670bc):**

```bash
# Removed file
config/tickets.php

# Correct location (already existed)
vendor/bithoven/bithoven-extension-tickets/config/tickets.php
```

**Why This Matters:**
- Extension configs should ONLY be in extension directories
- Prevents conflicts between extensions
- Maintains clean separation of concerns
- Follows Extension Manager architecture

---

## Debug Code Cleanup

### Removed Debug Statements (Commit: ec670bc)

**Files Cleaned:**
- `app/Actions/Extensions/Configuration/ResetSettingsAction.php`
- `app/Actions/Extensions/Configuration/UpdateSettingsAction.php`
- `app/Http/Controllers/Apps/Extensions/ExtensionConfigController.php`

**Removed:**
```php
// ❌ Debug statements removed
\Log::debug('Settings update', ['data' => $settings]);
dd($settings); // Debugging
var_dump($config); // Temporary debug
```

**Impact:**
- Cleaner codebase
- No accidental debug output in production
- Better performance (no unnecessary logging)

---

## Migration Best Practices

Based on fixes applied:

### 1. Always Backup Before Migrations

```bash
# Create backup before running migrations
./scripts/backup-database.sh "Pre-migration backup"

# Run migrations
php artisan migrate

# If something breaks, restore
./scripts/restore-database.sh
```

### 2. Validate Extension Configs

```php
// Extension config should NOT have 'name' field
// CORRECT:
'settings' => [
    'debug_console' => ['level' => 'debug'],
    'api' => ['timeout' => 30],
]

// INCORRECT:
'name' => 'My Extension', // ❌ Not needed
'settings' => [...]
```

### 3. Handle Nested Arrays Properly

```php
// When processing config arrays
if (is_array($value)) {
    // Recursively process nested arrays
    $config[$key] = $this->processNestedArray($value);
} else {
    $config[$key] = $value;
}
```

### 4. Check Global Settings Before Loading Features

```blade
<!-- Always check if feature enabled -->
@if(setting('feature.enabled', false))
    <!-- Load feature -->
@endif
```

---

## Backup Schedule Recommendation

**Production:**
```bash
# Daily backup (cron)
0 2 * * * /path/to/scripts/backup-database.sh "Daily automated backup" >> /var/log/backup.log 2>&1

# Weekly backup
0 3 * * 0 /path/to/scripts/backup-database.sh "Weekly automated backup" >> /var/log/backup.log 2>&1

# Pre-deployment backup
./scripts/backup-database.sh "Pre-deployment $(date +%Y-%m-%d)"
```

**Development:**
```bash
# Before major changes
./scripts/backup-database.sh "Pre-feature-X development"

# Before migrations
./scripts/backup-database.sh "Pre-pending-migrations"
```

---

## Verification Checklist

After applying fixes:

- [ ] Extension settings save without validation errors
- [ ] Debug Console loads only when enabled in settings
- [ ] Nested config arrays save correctly (test with Debug Console level)
- [ ] Backup script creates valid backups with metadata
- [ ] Restore script lists backups and restores correctly
- [ ] No debug code in production controllers/actions
- [ ] `config/` directory only contains core app configs (no extension configs)
- [ ] Composer symlinks update when toggling dev mode

---

## Related Documentation

- **Settings System:** `/DOCS/CORE/System-Settings/README.md`
- **Extension Manager:** `/DOCS/CORE/Extension-Manager/README.md`
- **Troubleshooting:** `scripts/troubleshooting/README.md`

---

**Last Updated:** December 5, 2025  
**Commits Included:** ec670bc, a50dee2, 0b7a334, 9e0a9fd
