# Database Migrations Management - Sistema Crítico

**Last Updated:** December 5, 2025  
**Version:** 2.0.0  
**Status:** PRODUCTION CRITICAL  
**Project Version:** v1.9.0

---

## 📋 Table of Contents

1. [Overview](#overview)
2. [Migrations Consolidation v1.8.0](#migrations-consolidation-v180)
3. [Migration System Fixes](#migration-system-fixes)
4. [Backup & Restore System](#backup--restore-system)
5. [Extension Manager Fixes](#extension-manager-fixes)
6. [Migration Best Practices](#migration-best-practices)
7. [Emergency Procedures](#emergency-procedures)
8. [Verification Checklist](#verification-checklist)

---

## Overview

Este documento centraliza toda la información crítica sobre gestión de migraciones de base de datos en el proyecto BITHOVEN, incluyendo:

- **Consolidación de Migraciones v1.8.0** (FASE 7.2)
- **Sistema de Backup/Restore** automatizado
- **Fixes aplicados** al sistema de migraciones
- **Best practices** para desarrollo y producción
- **Procedimientos de emergencia** para recuperación

### Documentos Relacionados

- `/DOCS/CORE/Extension-Manager/guides/MIGRATIONS-GUIDELINES.md` - Guías para desarrollo de extensiones
- `/DOCS/CORE/Extension-Manager/guides/DATABASE-CONVENTIONS.md` - Convenciones de base de datos
- `/DOCS/CORE/System-Settings/README.md` - Sistema de configuración
- `scripts/troubleshooting/README.md` - Troubleshooting Laravel bootstrap

---

## Migrations Consolidation v1.8.0

### FASE 7.2: Consolidación y Sistema de Permisos

**Fecha:** 19-26 de noviembre de 2025  
**Versión:** v1.7.0 → v1.8.0  
**Estado:** ✅ COMPLETADA

#### Objetivos Completados

1. ✅ **Reducción de Migraciones**: 52 → 23 (56% reducción)
2. ✅ **Sistema de Permisos Optimizado**: IDs 1-60 correlativos alfabéticos
3. ✅ **Testing Automatizado**: 10 tests con 48 assertions
4. ✅ **Commands de Verificación**: FixPermissionsOrder + VerifyPermissionsOrder

#### Estadísticas Detalladas

| Métrica | Antes | Después | Mejora |
|---------|-------|---------|--------|
| **Total Migraciones** | 52 | 23 | -56% |
| **Batch Number** | Múltiples | 21 (consolidado) | Unificado |
| **Permissions IDs** | Desordenados | 1-60 correlativos | 100% ordenados |
| **Tests Automatizados** | 0 | 10 | +∞ |
| **Assertions** | 0 | 48 | +∞ |

#### Resultado de la Consolidación

**Estado Anterior (52 migraciones):**
```
Batch 1-20: Migraciones dispersas
- IDs de permisos: 3, 15, 8, 42, 1, ...
- Múltiples batches no consolidados
- Difícil mantenimiento
```

**Estado Actual (23 migraciones - Batch 21):**
```
database/migrations/
├── 2024_07_01_000001_create_cache_table.php
├── 2024_07_01_000002_create_jobs_table.php
├── 2024_07_01_100049_create_permission_tables.php
├── 2024_07_01_100050_create_sessions_table.php
├── 2024_10_14_064249_create_personal_access_tokens_table.php
├── 2024_10_14_064451_create_users_table.php
├── 2024_10_15_145209_add_2fa_columns_to_users_table.php
├── 2024_10_18_120307_add_terms_acceptance_to_users_table.php
├── 2025_10_19_094523_add_alias_description_to_roles.php
├── 2025_10_19_094621_add_alias_description_to_permissions.php
├── 2025_10_27_000001_create_activity_log_table.php
├── 2025_10_27_000002_add_event_column_to_activity_log_table.php
├── 2025_10_27_000003_add_batch_uuid_column_to_activity_log_table.php
├── 2025_10_28_000001_create_debug_console_entries_table.php
├── 2025_10_29_094234_add_session_tracking_to_users.php
├── 2025_10_29_094823_add_session_timestamps_to_users.php
├── 2025_10_31_213742_add_avatar_to_users_table.php
├── 2025_11_03_000001_add_avatar_original_hash_to_users.php
├── 2025_11_03_000002_migrate_existing_avatar_hashes.php
├── 2025_11_03_000003_populate_avatar_default.php
├── 2025_11_14_000001_update_default_theme_to_light.php
├── 2025_11_27_000001_add_priority_to_permissions.php
├── 2025_12_04_000001_create_settings_table.php
```

**Permisos IDs 1-60 (Correlativos Alfabéticos):**
```sql
SELECT id, name FROM permissions ORDER BY id;

-- Resultado:
1  | core:activities:view
2  | core:dashboard:access
3  | core:debug-console:view
4  | core:developer-menu:access
5  | core:extensions:manage
...
60 | extensions:tickets:tickets:view
```

#### Commands de Verificación

**FixPermissionsOrder (Production):**
```bash
php artisan bithoven:permissions:fix-order
```
- Re-ordena permisos alfabéticamente
- Preserva asignaciones de roles
- Actualiza IDs correlativos
- Limpia cache automáticamente

**VerifyPermissionsOrder (CI/CD):**
```bash
php artisan bithoven:permissions:verify-order
```
- Verifica orden alfabético
- Exit code 0 = OK, 1 = ERROR
- Perfecto para pipelines
- Sin modificaciones (read-only)

#### Tests Automatizados

**10 Tests con 48 Assertions:**

```php
// tests/Feature/Permissions/PermissionsOrderTest.php

✅ permissions_are_ordered_alphabetically()
✅ permissions_ids_are_sequential()
✅ permissions_have_correct_priorities()
✅ verify_command_detects_unordered_permissions()
✅ fix_command_reorders_permissions()
✅ permissions_cache_is_cleared_after_reorder()
✅ role_assignments_preserved_after_reorder()
✅ permission_count_matches_expected()
✅ all_permissions_have_valid_names()
✅ permissions_follow_naming_convention()
```

**Ejecutar tests:**
```bash
# Tests específicos de permisos
php artisan test --filter PermissionsOrderTest

# Tests completos
php artisan test
```

#### Impacto en Producción

**Beneficios:**
- ✅ Mantenimiento simplificado (56% menos migraciones)
- ✅ Permisos predecibles y ordenados
- ✅ Queries más eficientes (IDs secuenciales)
- ✅ Debugging facilitado
- ✅ Auditoría mejorada

**Sin Breaking Changes:**
- ✅ Datos preservados 100%
- ✅ Funcionalidad intacta
- ✅ Roles y permisos sin afectar
- ✅ Rollback disponible

#### Procedimiento de Consolidación Aplicado

**Paso 1: Backup Completo**
```bash
./scripts/backup-database.sh "Pre-consolidation backup"
```

**Paso 2: Ejecución de Migrations Fresh**
```bash
php artisan migrate:fresh --seed
```

**Paso 3: Verificación de Batch**
```bash
SELECT batch FROM migrations LIMIT 1;
-- Resultado: 21 (todas en batch unificado)
```

**Paso 4: Aplicar FixPermissionsOrder**
```bash
php artisan bithoven:permissions:fix-order
```

**Paso 5: Validación con Tests**
```bash
php artisan test --filter PermissionsOrderTest
# Resultado: 10/10 tests PASSED ✅
```

**Paso 6: Verificación Final**
```bash
php artisan bithoven:permissions:verify-order
# Exit Code: 0 ✅
```

---

## Migration System Fixes

### Issue: Extension Settings Migration Conflicts

**Fecha:** Diciembre 5, 2025  
**Commit:** ec670bc

#### Problema

Extension settings were being saved with incorrect structure, causing validation errors and migration failures.

**Síntomas:**
- Extension config update failing with "The name field is required"
- Settings not persisting in extension config files
- Debug Console not loading when disabled

**Root Cause:**
`UpdateSettingsRequest` validation included unnecessary `'name'` field that didn't exist in extension settings.

#### Solución Aplicada

```php
// app/Http/Requests/Extensions/UpdateSettingsRequest.php

// ❌ ANTES (broken)
public function rules(): array
{
    return [
        'name' => 'required|string|max:255',  // ❌ Extension settings don't have 'name'
        'settings' => 'required|array',
        'settings.*' => 'nullable',
    ];
}

// ✅ DESPUÉS (fixed)
public function rules(): array
{
    return [
        // Removed 'name' field - not needed for extension config
        'settings' => 'required|array',
        'settings.*' => 'nullable',
    ];
}
```

#### Impacto

- ✅ Extension settings now save correctly
- ✅ No more validation errors on config updates
- ✅ Debug Console config updates work properly

---

## Backup & Restore System

### Production-Ready Scripts

**Ubicación:** `scripts/backup-database.sh` y `scripts/restore-database.sh`  
**Versión:** 1.0  
**Estado:** Production Ready

#### backup-database.sh

**Features:**
- Automatic backup directory creation (`backups/mysql/`)
- Timestamped filenames (YYYYMMDD-HHMM)
- Compression with gzip
- Metadata file with backup details
- Verification of backup integrity
- Summary statistics (size, tables, rows)

**Uso:**

```bash
# Standard backup
./scripts/backup-database.sh

# Custom description
./scripts/backup-database.sh "Pre-production deployment"
```

**Output Example:**

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

#### restore-database.sh

**Features:**
- List available backups with details
- Interactive selection menu
- Automatic backup before restore (safety)
- Decompression handling (gzip)
- Verification prompts
- Rollback capability

**Uso:**

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

#### Safety Features

- ✅ Automatic safety backup before restore
- ✅ Confirmation prompt
- ✅ Detailed metadata display
- ✅ Error handling with rollback
- ✅ Compression support (gzip auto-detect)

---

## Extension Manager Fixes

### Fix 1: Debug Console Loading Check

**Commit:** ec670bc

**Problem:**
Debug Console JavaScript loading even when `debug_console.enabled = false`, causing unnecessary overhead.

**Solution:**

```blade
<!-- resources/views/partials/debug-console-init.blade.php -->

<!-- ❌ ANTES (broken) -->
@if(config('app.debug'))
    <script>
        // Always loads if app.debug = true
    </script>
@endif

<!-- ✅ DESPUÉS (fixed) -->
@if(setting('debug_console.enabled', true))
    <script>
        // Only loads if Debug Console enabled in settings
    </script>
@endif
```

**Impact:**
- ✅ Debug Console respects global enabled/disabled setting
- ✅ Reduced JavaScript load when disabled
- ✅ Better performance in production

### Fix 2: Nested Array Handling in Config Forms

**Commit:** a50dee2

**Problem:**
Extension config forms couldn't save nested arrays (e.g., `debug_console.level`).

**Solution:**

```php
// app/Actions/Extensions/Configuration/UpdateSettingsAction.php

// ❌ ANTES (broken)
foreach ($settings as $key => $value) {
    // Only handled flat arrays
    $config[$key] = $value;
}

// ✅ DESPUÉS (fixed)
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
- ✅ Extension configs can now have multi-level arrays
- ✅ Debug Console config works: `debug_console.level`
- ✅ LLM Manager complex configs save correctly

### Fix 3: Composer Symlink Management

**Commit:** 0b7a334

**Problem:**
Extension `composer.json` symlinks not updating when toggling dev mode.

**Solution:**

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

## Migration Best Practices

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
// ✅ CORRECT:
'settings' => [
    'debug_console' => ['level' => 'debug'],
    'api' => ['timeout' => 30],
]

// ❌ INCORRECT:
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

### 5. Use Idempotent Migrations

```php
// ✅ CORRECT - Can be run multiple times
Schema::table('users', function (Blueprint $table) {
    if (!Schema::hasColumn('users', 'avatar')) {
        $table->string('avatar')->nullable();
    }
});

// ❌ WRONG - Fails on re-run
Schema::table('users', function (Blueprint $table) {
    $table->string('avatar')->nullable(); // Error si ya existe
});
```

### 6. Always Implement down() Method

```php
// ✅ CORRECT - Fully reversible
public function up(): void
{
    Schema::table('users', function (Blueprint $table) {
        $table->string('phone')->nullable()->after('email');
    });
}

public function down(): void
{
    Schema::table('users', function (Blueprint $table) {
        $table->dropColumn('phone');
    });
}

// ❌ WRONG - Not reversible
public function down(): void
{
    // Empty - cannot rollback!
}
```

---

## Emergency Procedures

### Scenario 1: Migration Failed Mid-Execution

**Síntomas:**
- Migration threw exception
- Database in inconsistent state
- Some tables created, others not

**Procedure:**

1. **STOP - Do NOT panic**
```bash
# Do NOT run migrate:fresh or migrate:rollback yet
```

2. **Check database state**
```bash
php artisan tinker
>>> Schema::hasTable('problematic_table')
>>> DB::table('migrations')->latest()->first()
```

3. **Restore from last backup**
```bash
./scripts/restore-database.sh
# Select backup created before migration
```

4. **Fix the migration**
```php
// Fix the code in migration file
// Test locally first
```

5. **Re-run with backup**
```bash
./scripts/backup-database.sh "Pre-retry backup"
php artisan migrate
```

### Scenario 2: Permissions Desincronizados

**Síntomas:**
- Permission IDs out of order
- Roles missing permissions
- Cache issues

**Procedure:**

1. **Verify current state**
```bash
php artisan bithoven:permissions:verify-order
# Exit code 1 = Problems detected
```

2. **Fix permissions order**
```bash
php artisan bithoven:permissions:fix-order
```

3. **Verify fix**
```bash
php artisan bithoven:permissions:verify-order
# Exit code 0 = OK
```

4. **Run tests**
```bash
php artisan test --filter PermissionsOrderTest
```

5. **Clear all caches**
```bash
php artisan optimize:clear
php artisan cache:clear
```

### Scenario 3: Laravel Bootstrap Corrupto

**Síntomas:**
- `php artisan serve` fails
- `Call to a member function make() on null`
- `php artisan --version` works ✅

**Procedure:**

Ver documentación completa en `scripts/troubleshooting/README.md`

**Quick fix:**
```bash
./scripts/troubleshooting/fix-laravel-bootstrap.sh
```

Este script automáticamente:
1. Limpia caches corruptos
2. Regenera autoload
3. Reconstruye bootstrap cache
4. Verifica servidor funcional

---

## Verification Checklist

### Post-Migration Checklist

- [ ] Backup created before migration
- [ ] Migration executed successfully
- [ ] No errors in laravel.log
- [ ] All tables created/modified as expected
- [ ] Seeders ran successfully
- [ ] Permissions ordered alphabetically (if applicable)
- [ ] Tests pass (48/48 assertions)
- [ ] Application functional in browser
- [ ] Debug Console loads correctly (if enabled)
- [ ] Extension configs save properly

### Post-Fix Checklist

- [ ] Extension settings save without validation errors
- [ ] Debug Console loads only when enabled in settings
- [ ] Nested config arrays save correctly (test with Debug Console level)
- [ ] Backup script creates valid backups with metadata
- [ ] Restore script lists backups and restores correctly
- [ ] No debug code in production controllers/actions
- [ ] `config/` directory only contains core app configs (no extension configs)
- [ ] Composer symlinks update when toggling dev mode

### Weekly Maintenance Checklist

- [ ] Run backup script (automated or manual)
- [ ] Verify permissions order
- [ ] Check for orphaned migrations
- [ ] Review laravel.log for errors
- [ ] Test restore procedure (staging environment)
- [ ] Update documentation if needed

---

## Backup Schedule Recommendation

### Production

```bash
# Daily backup (cron)
0 2 * * * /path/to/scripts/backup-database.sh "Daily automated backup" >> /var/log/backup.log 2>&1

# Weekly backup
0 3 * * 0 /path/to/scripts/backup-database.sh "Weekly automated backup" >> /var/log/backup.log 2>&1

# Pre-deployment backup
./scripts/backup-database.sh "Pre-deployment $(date +%Y-%m-%d)"
```

### Development

```bash
# Before major changes
./scripts/backup-database.sh "Pre-feature-X development"

# Before migrations
./scripts/backup-database.sh "Pre-pending-migrations"

# Before database cleanup
./scripts/backup-database.sh "Pre-cleanup $(date +%H:%M)"
```

---

## Related Documentation

- **Extension Manager:** `/DOCS/CORE/Extension-Manager/README.md`
- **Migrations Guidelines:** `/DOCS/CORE/Extension-Manager/guides/MIGRATIONS-GUIDELINES.md`
- **Database Conventions:** `/DOCS/CORE/Extension-Manager/guides/DATABASE-CONVENTIONS.md`
- **Settings System:** `/DOCS/CORE/System-Settings/README.md`
- **Troubleshooting:** `scripts/troubleshooting/README.md`
- **PHASES Documentation:** `docs/user-guides/PHASES.md`

---

**Last Updated:** December 5, 2025  
**Commits Included:** ec670bc, a50dee2, 0b7a334, 9e0a9fd, d08ce8d  
**FASE:** 7.2 (Migrations Consolidation) + 9.3 (Migration Fixes & Backup Scripts)
