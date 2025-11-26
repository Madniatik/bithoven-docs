# Fix Extension System - Complete Guide

**Version:** 1.0.0  
**Last Updated:** 15 de noviembre de 2025

---

## 🎯 What is Fix Extension?

Fix Extension is a **non-destructive** repair mode that:
- ✅ Restores edited base configuration to original values
- ✅ Preserves ALL user data (tickets, comments, custom records)
- ✅ Runs pending migrations
- ✅ Re-executes base seeders
- ✅ Creates automatic backup before operation
- ❌ Does NOT delete any data

---

## 🔧 How It Works

### Process Flow

```
1. User clicks "Fix Extension" → Modal opens
2. User confirms → Process starts
3. Create automatic backup
4. Run pending migrations (if any)
5. Execute DatabaseSeeder (base data only)
6. Success → Extension repaired
```

### What Gets Fixed?

**Base Configuration Records (restored to defaults):**
- Categories (IDs 1-8)
- Templates (IDs 1-10)
- Canned Responses (IDs 1-24)
- Automation Rules (IDs 1-6)

**User Data (preserved untouched):**
- Custom categories (IDs > 8)
- Custom templates (IDs > 10)
- Custom canned responses (IDs > 24)
- Custom automation rules (IDs > 6)
- All tickets
- All comments
- All attachments
- All user settings

---

## 🧠 The Magic: Fixed IDs

Fix Extension works because seeders use **fixed IDs**:

```php
// Base records have known, fixed IDs
$baseCategories = [
    ['id' => 1, 'name' => 'Técnico', 'slug' => 'tecnico', ...],
    ['id' => 2, 'name' => 'Facturación', 'slug' => 'facturacion', ...],
    // ... IDs 3-8
];

foreach ($baseCategories as $category) {
    TicketCategory::updateOrCreate(
        ['id' => $category['id']],  // Find by ID (never changes)
        $category                     // Update with original values
    );
}
```

**What happens:**
1. Seeder looks for record with `id = 1`
2. Finds existing record (user may have edited name to "Técnico EDITADO")
3. **Updates** existing record with original values
4. Result: Record restored to "Técnico"

**Without fixed IDs (OLD/BROKEN way):**
```php
// ❌ WRONG - causes duplicates
TicketCategory::updateOrCreate(
    ['slug' => 'tecnico'],  // If user changed slug, not found
    $category                // Creates NEW record = DUPLICATE
);
```

---

## 📋 User Flow

### From Admin UI

1. **Navigate:** `/admin/extensions`
2. **Locate extension card**
3. **Click:** Three-dot menu → "Reinstall"
4. **Modal opens:** Two tabs visible
5. **Select:** "Fix Extension" tab (default)
6. **Review:** What will happen (shown in modal)
7. **Click:** "Fix Extension" button
8. **Confirmation:** SweetAlert confirmation
9. **Processing:** Button shows "Fixing..." with spinner
10. **Complete:** Success message, page reloads

### From CLI (if implemented)

```bash
php artisan bithoven:extension:fix tickets
```

---

## 🔍 Technical Details

### Backend Method

**Location:** `app/Services/Extensions/ExtensionManager.php`

```php
/**
 * Fix extension without removing user data
 * Runs migrations and re-seeds base configuration
 *
 * @param string $extensionSlug Extension slug (e.g., 'tickets')
 * @param bool $loadDemo Whether to load demo data after fix
 * @return array Result with success status and message
 */
public function fixExtension(string $extensionSlug, bool $loadDemo = false): array
{
    try {
        Log::info("Starting Fix Extension for {$extensionSlug}");

        // Step 1: Create backup
        $backup = app(ExtensionBackupService::class)->createBackup(
            $extensionSlug,
            'update',
            ['backup_database' => true]
        );

        // Step 2: Run pending migrations
        $this->runMigrations($extensionSlug);

        // Step 3: Run base seeders (restores config)
        $this->runBaseSeeders($extensionSlug);

        // Step 4: Optionally load demo data
        if ($loadDemo) {
            $this->seed($extensionSlug);
        }

        return [
            'success' => true,
            'message' => 'Extension fixed successfully'
        ];

    } catch (\Exception $e) {
        Log::error("Fix Extension failed: " . $e->getMessage());
        
        return [
            'success' => false,
            'message' => 'Fix failed: ' . $e->getMessage()
        ];
    }
}
```

### Key Methods Called

1. **`ExtensionBackupService::createBackup()`**
   - Creates SQL backup of extension tables
   - Stores in `storage/backups/extensions/{name}/`
   - Includes timestamp in filename

2. **`runMigrations($name)`**
   - Executes: `php artisan migrate --path=vendor/bithoven/{name}/database/migrations`
   - Only runs pending migrations
   - Safe to run multiple times

3. **`runBaseSeeders($name)`**
   - Executes: `php artisan db:seed --class="Bithoven\\{Name}\\Database\\Seeders\\DatabaseSeeder"`
   - Uses `updateOrCreate(['id' => ...])` pattern
   - Restores base records, preserves custom records

---

## ✅ Testing Fix Extension

### Test Scenario 1: Edited Base Records

**Setup:**
```sql
-- Edit base category
UPDATE ticket_categories 
SET name='Técnico EDITADO', slug='tecnico-editado' 
WHERE id=1;
```

**Run Fix Extension**

**Expected Result:**
```sql
-- After Fix Extension
SELECT * FROM ticket_categories WHERE id=1;
-- name='Técnico', slug='tecnico' (restored)
```

---

### Test Scenario 2: Custom Records

**Setup:**
```sql
-- Create custom category (auto-increments to ID > 8)
INSERT INTO ticket_categories (name, slug, description, is_active, sort_order)
VALUES ('Mi Categoría', 'mi-categoria', 'Custom category', 1, 10);
-- Gets ID=9 (or higher)
```

**Run Fix Extension**

**Expected Result:**
```sql
-- After Fix Extension
SELECT * FROM ticket_categories WHERE id=9;
-- Still exists with same values (preserved)
```

---

### Test Scenario 3: Deleted Base Record

**Setup:**
```sql
-- Delete base category
DELETE FROM ticket_categories WHERE id=2;
```

**Run Fix Extension**

**Expected Result:**
```sql
-- After Fix Extension
SELECT * FROM ticket_categories WHERE id=2;
-- Record recreated with original values (restored)
```

---

## 🚨 Common Issues

### Issue: Fix Extension creates duplicates

**Cause:** Seeder using editable field as updateOrCreate key

**Example:**
```php
// ❌ WRONG
TicketCategory::updateOrCreate(['slug' => 'tecnico'], [...]);
// If user changed slug to 'tecnico-editado', creates duplicate
```

**Fix:**
```php
// ✅ CORRECT
TicketCategory::updateOrCreate(['id' => 1], [...]);
// Always finds record by ID, updates it
```

**See:** [SEEDERS-BEST-PRACTICES.md](SEEDERS-BEST-PRACTICES.md)

---

### Issue: Custom records overwritten

**Cause:** Custom record has ID within base range

**Example:**
- Base range: IDs 1-8
- Custom record created with ID=5
- Fix Extension restores ID=5 to base value
- Custom record lost

**Fix:** Expand base ID range to avoid conflicts
```php
/**
 * ID Ranges:
 * - IDs 1-15: Base (increased from 1-8)
 * - IDs > 15: Custom (safe)
 */
```

---

### Issue: Backup fails silently

**Cause:** Storage permissions or disk space

**Check:**
```bash
# Verify storage is writable
ls -la storage/backups/extensions/

# Check disk space
df -h
```

**Fix:**
```bash
# Fix permissions
chmod -R 775 storage/backups/
chown -R www-data:www-data storage/backups/
```

---

## 📊 Comparison: Fix vs Fresh Install

| Feature | Fix Extension | Fresh Install |
|---------|---------------|---------------|
| **Data Loss** | ❌ None | ✅ Complete |
| **Base Config** | ✅ Restored | ✅ Restored |
| **Custom Records** | ✅ Preserved | ❌ Deleted |
| **User Data** | ✅ Preserved | ❌ Deleted |
| **Migrations** | ✅ Pending only | ✅ All migrations |
| **Backup** | ✅ Auto-created | ✅ Auto-created |
| **Use Case** | Repair config | Complete reset |
| **Danger Level** | 🟢 Safe | 🔴 Destructive |

---

## 🎓 Best Practices

### For Extension Developers

1. **Use fixed IDs** for all base records in seeders
2. **Document ID ranges** clearly in comments
3. **Test Fix Extension** thoroughly before releasing
4. **Separate demo data** from base seeders
5. **Keep DatabaseSeeder minimal** (essential data only)

### For Users

1. **Try Fix Extension first** before Fresh Install
2. **Backup manually** if dealing with critical data
3. **Test on staging** if possible
4. **Note custom changes** before running Fix Extension
5. **Check backup created** after operation

---

## 📚 Related Documentation

- **Seeders Guide:** [SEEDERS-BEST-PRACTICES.md](SEEDERS-BEST-PRACTICES.md)
- **Fresh Install:** [FRESH-INSTALL-SYSTEM.md](FRESH-INSTALL-SYSTEM.md)
- **Backup System:** [BACKUP-RECOVERY.md](BACKUP-RECOVERY.md)
- **AI Instructions:** [../COPILOT/AI-AGENT-INSTRUCTIONS.md](../COPILOT/AI-AGENT-INSTRUCTIONS.md)

---

**Last Updated:** 15 de noviembre de 2025  
**Version:** 1.0.0
