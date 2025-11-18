# Seeders Best Practices - CRITICAL GUIDE

**Version:** 1.0.0  
**Last Updated:** 15 de noviembre de 2025  
**Status:** ⚠️ MANDATORY - Failure to follow causes Fix Extension to fail

---

## 🎯 Core Principle

> **If users can edit a field, DO NOT use that field as the key in `updateOrCreate()`**

Fix Extension restores edited base records to original values. If you match by fields that users edit (name, slug, description), you create duplicates instead of updating.

---

## ❌ The Problem

### What Happens When You Do It Wrong

```php
// ❌ WRONG: Using editable field as key
TicketCategory::updateOrCreate(
    ['slug' => 'tecnico'],  // User edits "Técnico" → "Técnico Editado"
    [                        // Slug becomes "tecnico-editado"
        'name' => 'Técnico',
        'slug' => 'tecnico',
        // ...
    ]
);
```

**When Fix Extension runs:**
1. Looks for `slug = 'tecnico'`
2. Doesn't find it (user changed it to 'tecnico-editado')
3. **Creates NEW record** with slug='tecnico'
4. **Result: DUPLICATE** - both "Técnico" and "Técnico Editado" exist

---

## ✅ The Solution

### Use Fixed IDs for Base Records

```php
// ✅ CORRECT: Using immutable ID as key
$baseCategories = [
    [
        'id' => 1,
        'name' => 'Técnico',
        'slug' => 'tecnico',
        'description' => 'Problemas técnicos...',
        // ...
    ],
    [
        'id' => 2,
        'name' => 'Facturación',
        'slug' => 'facturacion',
        // ...
    ],
    // ... IDs 3-8 for other base categories
];

foreach ($baseCategories as $category) {
    TicketCategory::updateOrCreate(
        ['id' => $category['id']],  // ✅ ID never changes
        $category                     // All fields including edited ones get restored
    );
}
```

**When Fix Extension runs:**
1. Looks for `id = 1`
2. **Always finds it** (ID never changes)
3. **Updates existing record** with original values
4. **Result: RESTORATION** - edited record restored, no duplicates

---

## 📋 Complete Seeder Template

### CategorySeeder.php (Example)

```php
<?php

namespace YourExtension\Database\Seeders;

use YourExtension\Models\Category;
use Illuminate\Database\Seeder;

/**
 * Base Category Seeder
 * 
 * IMPORTANT: Uses fixed IDs (1-8) for base categories to allow
 * Fix Extension to properly restore edited values.
 * 
 * ID Ranges:
 * - IDs 1-8: Base categories (restored by Fix Extension)
 * - IDs > 8: Custom user categories (preserved untouched)
 */
class CategorySeeder extends Seeder
{
    public function run(): void
    {
        $baseCategories = [
            [
                'id' => 1,
                'name' => 'Category 1',
                'slug' => 'category-1',
                'description' => 'Description for category 1',
                'is_active' => true,
                'sort_order' => 1,
            ],
            [
                'id' => 2,
                'name' => 'Category 2',
                'slug' => 'category-2',
                'description' => 'Description for category 2',
                'is_active' => true,
                'sort_order' => 2,
            ],
            // ... IDs 3-8 for remaining base categories
        ];

        foreach ($baseCategories as $category) {
            Category::updateOrCreate(
                ['id' => $category['id']], // ✅ Match by ID
                $category                   // Update all fields
            );
        }
    }
}
```

---

## 🔢 ID Range Guidelines

### Define Clear Ranges

Document ID ranges at the top of each seeder:

```php
/**
 * ID Ranges:
 * - IDs 1-10: Base items (restored by Fix Extension)
 * - IDs > 10: Custom user items (preserved untouched)
 */
```

### How to Choose Range Size

- **Categories:** 8-15 base items typical
- **Templates:** 10-20 base items typical
- **Quick Responses:** 20-30 base items typical
- **Automation Rules:** 5-10 base items typical

**Rule of thumb:** Allocate 2-3x what you initially need to allow for future additions.

### Real Examples

From **bithoven-extension-tickets**:

| Seeder | Base IDs | Custom IDs | Count |
|--------|----------|------------|-------|
| CategorySeeder | 1-8 | >8 | 8 categories |
| TemplatesSeeder | 1-10 | >10 | 10 templates |
| CannedResponseSeeder | 1-24 | >24 | 24 responses |
| AutomationRulesSeeder | 1-6 | >6 | 6 rules |

---

## 🗂️ DatabaseSeeder vs DemoSeeder

### DatabaseSeeder.php - Essential Data Only

```php
<?php

namespace YourExtension\Database\Seeders;

use Illuminate\Database\Seeder;

/**
 * Base Database Seeder
 * 
 * Runs automatically on:
 * - Fresh Install
 * - Fix Extension
 * 
 * Contains ONLY essential data for extension functionality.
 */
class DatabaseSeeder extends Seeder
{
    public function run(): void
    {
        $this->call([
            CategorySeeder::class,        // Essential: base categories
            TemplateSeeder::class,         // Essential: base templates
            AutomationRulesSeeder::class,  // Essential: default rules
        ]);
    }
}
```

### DemoSeeder.php - Example Data

```php
<?php

namespace YourExtension\Database\Seeders;

use Illuminate\Database\Seeder;

/**
 * Demo Data Seeder
 * 
 * Runs ONLY when explicitly requested by user.
 * Contains example/testing data.
 * 
 * NOT executed by Fix Extension or Fresh Install.
 */
class DemoSeeder extends Seeder
{
    public function run(): void
    {
        // Create demo tickets
        // Create demo comments
        // Create demo attachments
        // etc.
    }
}
```

---

## 🧪 Testing Checklist

### Before Considering Seeders Complete

- [ ] **Fixed IDs assigned** to all base records
- [ ] **ID ranges documented** in seeder comments
- [ ] **updateOrCreate uses ID** as key, not name/slug
- [ ] **DatabaseSeeder** contains only essential data
- [ ] **DemoSeeder** separated for optional data

### Testing Fix Extension

1. **Fresh Install** - Install extension clean
2. **Edit base records** - Change names, slugs, descriptions
3. **Create custom records** - Add new items (will get ID > base range)
4. **Run Fix Extension**
5. **Verify results:**
   - [ ] Base records (IDs 1-N) restored to original values
   - [ ] Custom records (IDs > N) preserved untouched
   - [ ] No duplicate records created
   - [ ] User data (tickets, comments, etc.) intact

---

## 🚨 Common Mistakes

### ❌ Mistake 1: Using Name/Slug as Key

```php
// ❌ WRONG
Model::updateOrCreate(['name' => 'Item'], [...]);
Model::updateOrCreate(['slug' => 'item'], [...]);
Model::updateOrCreate(['shortcut' => '/item'], [...]);
```

**Problem:** Users edit these fields → Fix Extension creates duplicates

**Fix:** Use ID as key
```php
// ✅ CORRECT
Model::updateOrCreate(['id' => 1], ['name' => 'Item', ...]);
```

---

### ❌ Mistake 2: No ID Ranges

```php
// ❌ WRONG: No way to distinguish base from custom
$categories = [
    ['name' => 'Category 1', 'slug' => 'cat-1'], // What ID will this get?
    ['name' => 'Category 2', 'slug' => 'cat-2'], // What ID will this get?
];
```

**Problem:** Custom user categories might get IDs 1-2 and get overwritten

**Fix:** Explicit ID assignment
```php
// ✅ CORRECT
$categories = [
    ['id' => 1, 'name' => 'Category 1', 'slug' => 'cat-1'],
    ['id' => 2, 'name' => 'Category 2', 'slug' => 'cat-2'],
];
```

---

### ❌ Mistake 3: Demo Data in DatabaseSeeder

```php
// ❌ WRONG: DatabaseSeeder.php
public function run(): void
{
    $this->call([
        CategorySeeder::class,  // ✅ OK - essential
        TemplateSeeder::class,  // ✅ OK - essential
        DemoTicketsSeeder::class, // ❌ WRONG - demo data!
    ]);
}
```

**Problem:** Demo data runs on every Fix Extension, polluting database

**Fix:** Separate demo data
```php
// ✅ CORRECT: DatabaseSeeder.php
public function run(): void
{
    $this->call([
        CategorySeeder::class,
        TemplateSeeder::class,
    ]);
    // Demo data in separate DemoSeeder.php
}
```

---

### ❌ Mistake 4: Forgetting to Include ID in Data Array

```php
// ❌ WRONG: ID in key but not in data
Category::updateOrCreate(
    ['id' => 1],
    [
        // Missing 'id' => 1 here!
        'name' => 'Category',
        'slug' => 'category',
    ]
);
```

**Problem:** ID gets nulled on update in some Laravel versions

**Fix:** Include ID in data array
```php
// ✅ CORRECT
Category::updateOrCreate(
    ['id' => 1],
    [
        'id' => 1,  // ✅ Explicit ID in data
        'name' => 'Category',
        'slug' => 'category',
    ]
);
```

---

## 📖 Real-World Example

See complete implementation in:
- **CategorySeeder:** `/EXTENSIONS/bithoven-extension-tickets/database/seeders/CategorySeeder.php`
- **TemplatesResponsesSeeder:** `/EXTENSIONS/bithoven-extension-tickets/database/seeders/TemplatesResponsesSeeder.php`
- **AutomationRulesSeeder:** `/EXTENSIONS/bithoven-extension-tickets/database/seeders/AutomationRulesSeeder.php`

---

## 🎓 Quick Reference

### Pattern Summary

```php
// Define base records with fixed IDs
$baseRecords = [
    ['id' => 1, 'field1' => 'value1', ...],
    ['id' => 2, 'field2' => 'value2', ...],
    // ... up to ID N
];

// Use ID as updateOrCreate key
foreach ($baseRecords as $record) {
    Model::updateOrCreate(
        ['id' => $record['id']],  // ✅ ID as key
        $record                    // All fields
    );
}

// Document ID ranges
/**
 * IDs 1-N: Base (Fix Extension restores these)
 * IDs > N: Custom (Fix Extension preserves these)
 */
```

---

## 💡 Why This Matters

**Fix Extension exists to:**
1. Restore corrupted base configuration
2. Reset edited base records to defaults
3. **WITHOUT deleting user's custom data**

**This ONLY works if:**
- Base records have fixed, known IDs (1-N)
- Seeders match by ID, not by editable fields
- Custom records use IDs > N and are never touched

**Failure to follow = Fix Extension creates duplicates instead of restoring**

---

**Next Steps:**
- Read [MIGRATIONS-GUIDELINES.md](MIGRATIONS-GUIDELINES.md) for database schema best practices
- See [EXTENSION-STRUCTURE.md](EXTENSION-STRUCTURE.md) for complete extension structure
- Check [QUICK-START.md](QUICK-START.md) to create your first extension

---

**Questions?** See [../COPILOT/AI-AGENT-INSTRUCTIONS.md](../COPILOT/AI-AGENT-INSTRUCTIONS.md) for AI-specific guidance.
