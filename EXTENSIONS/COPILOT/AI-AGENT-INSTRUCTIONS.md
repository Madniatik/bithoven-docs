# AI Agent Instructions - Bithoven Extensions

**Target Agents:** GitHub Copilot, Claude, GPT-4, and other AI coding assistants  
**Version:** 1.1.0  
**Last Updated:** 16 de noviembre de 2025

---

## 🤖 Quick Context Loading

When working on Bithoven extensions, load context in this order:

### 1. Essential Context (ALWAYS load first)
```bash
# Main README
read_file('/Users/madniatik/CODE/LARAVEL/BITHOVEN/EXTENSIONS/DOCUMENTATION/README.md')

# Critical seeder patterns
read_file('/Users/madniatik/CODE/LARAVEL/BITHOVEN/EXTENSIONS/DOCUMENTATION/guides/SEEDERS-BEST-PRACTICES.md')

# This file
read_file('/Users/madniatik/CODE/LARAVEL/BITHOVEN/EXTENSIONS/DOCUMENTATION/COPILOT/AI-AGENT-INSTRUCTIONS.md')
```

### 2. Topic-Specific Context (load as needed)
```bash
# When working on extension structure
read_file('/Users/madniatik/CODE/LARAVEL/BITHOVEN/EXTENSIONS/DOCUMENTATION/guides/EXTENSION-STRUCTURE.md')

# When working on migrations
read_file('/Users/madniatik/CODE/LARAVEL/BITHOVEN/EXTENSIONS/DOCUMENTATION/guides/MIGRATIONS-GUIDELINES.md')

# When working on Fix/Fresh Install

# When working on configuration, repos, or auth
read_file('/Users/madniatik/CODE/LARAVEL/BITHOVEN/EXTENSIONS/DOCUMENTATION/guides/CONFIGURATION-SYSTEM.md')
read_file('/Users/madniatik/CODE/LARAVEL/BITHOVEN/EXTENSIONS/DOCUMENTATION/guides/FIX-EXTENSION-SYSTEM.md')
```

---

## ⚙️ Configuration System Quick Reference

### Extension Manager Location
**URL:** `/app/extensions` (moved from `/admin/extensions`)  
**Routes:** `app.extensions.*` (changed from `admin.extensions.*`)

### Configuration Files

**1. extension-settings.json** (storage/app/) - System settings
```json
{
    "repo_mode": "vcs",        // "local" or "vcs"
    "cache_ttl": 60,           // minutes
    "use_symlink": true,       // for local mode
    "updated_at": "ISO 8601",
    "updated_by": 1
}
```

**2. ~/.composer/auth.json** - GitHub token (global)
```json
{
    "github-oauth": {
        "github.com": "github_pat_..."
    }
}
```

**3. composer.json** - Repository definitions (auto-managed)

### Repository Modes

**VCS Mode (Production):**
- Uses GitHub repositories
- Requires GitHub token for private repos
- Safe for end users

**Local Mode (Development):**
- Uses `../EXTENSIONS/bithoven-extension-{name}`
- Optional symlinks for live changes
- Requires local extension folders

**For detailed configuration docs:**
```bash
read_file('/Users/madniatik/CODE/LARAVEL/BITHOVEN/EXTENSIONS/DOCUMENTATION/guides/CONFIGURATION-SYSTEM.md')
```


## ⚠️ CRITICAL RULES - NON-NEGOTIABLE

### Rule #1: Seeder Pattern with Fixed IDs

**ALWAYS use this pattern for base seeders:**

```php
// ✅ CORRECT
$baseRecords = [
    ['id' => 1, 'name' => 'Item 1', 'slug' => 'item-1', ...],
    ['id' => 2, 'name' => 'Item 2', 'slug' => 'item-2', ...],
];

foreach ($baseRecords as $record) {
    Model::updateOrCreate(
        ['id' => $record['id']],  // ✅ MUST use ID
        $record
    );
}
```

**NEVER use these patterns:**
```php
// ❌ WRONG - breaks Fix Extension
Model::updateOrCreate(['name' => ...], [...]);
Model::updateOrCreate(['slug' => ...], [...]);
Model::updateOrCreate(['shortcut' => ...], [...]);
```

**Why:** Fix Extension restores edited base records. If you match by editable fields (name, slug), you create duplicates.

---

### Rule #2: Separate Demo Data

**DatabaseSeeder.php** - Only essential data:
```php
public function run(): void
{
    $this->call([
        CategorySeeder::class,      // ✅ Essential
        TemplateSeeder::class,      // ✅ Essential
        // NO demo data here
    ]);
}
```

**DemoSeeder.php** - Demo/test data:
```php
public function run(): void
{
    // Demo tickets, comments, etc.
}
```

---

### Rule #3: Document ID Ranges

**ALWAYS add this comment at the top of seeders:**
```php
/**
 * ID Ranges:
 * - IDs 1-N: Base records (restored by Fix Extension)
 * - IDs > N: Custom user records (preserved)
 */
```

---

## 🎯 Extension Development Workflow

### Creating a New Extension

```bash
# 1. Create directory structure
mkdir -p bithoven-extension-{name}/{src,database/{migrations,seeders},config,resources/views,routes,tests}

# 2. Create composer.json
# See template: /EXTENSIONS/DOCUMENTATION/templates/composer.json

# 3. Create extension.json
# See template: /EXTENSIONS/DOCUMENTATION/templates/extension.json

# 4. Create ServiceProvider
# See template: /EXTENSIONS/DOCUMENTATION/templates/ServiceProvider.php

# 5. Create seeders with FIXED IDs
# See: SEEDERS-BEST-PRACTICES.md
```

---

### Modifying Existing Seeders

**When you see this pattern (LEGACY - MUST FIX):**
```php
// ❌ LEGACY PATTERN - breaks Fix Extension
Model::updateOrCreate(['name' => 'Item'], [...]);
```

**Convert to:**
```php
// ✅ CORRECT PATTERN
$baseItems = [
    ['id' => 1, 'name' => 'Item', ...],
];

foreach ($baseItems as $item) {
    Model::updateOrCreate(['id' => $item['id']], $item);
}
```

**Steps:**
1. Identify all base records
2. Assign fixed IDs (1, 2, 3, ...)
3. Add ID to data array
4. Change updateOrCreate key to `['id' => ...]`
5. Document ID ranges in comments
6. Test Fix Extension

---

## 📋 Code Generation Templates

### Seeder Template

```php
<?php

namespace VendorName\ExtensionName\Database\Seeders;

use VendorName\ExtensionName\Models\ModelName;
use Illuminate\Database\Seeder;

/**
 * Base [ModelName] Seeder
 * 
 * IMPORTANT: Uses fixed IDs for base records to allow
 * Fix Extension to properly restore edited values.
 * 
 * ID Ranges:
 * - IDs 1-{N}: Base records (restored by Fix Extension)
 * - IDs > {N}: Custom user records (preserved)
 */
class ModelNameSeeder extends Seeder
{
    public function run(): void
    {
        $baseRecords = [
            [
                'id' => 1,
                'name' => 'Record 1',
                'slug' => 'record-1',
                'description' => 'Description here',
                'is_active' => true,
                'sort_order' => 1,
            ],
            [
                'id' => 2,
                'name' => 'Record 2',
                'slug' => 'record-2',
                'description' => 'Description here',
                'is_active' => true,
                'sort_order' => 2,
            ],
            // Add more base records with sequential IDs
        ];

        foreach ($baseRecords as $record) {
            ModelName::updateOrCreate(
                ['id' => $record['id']],
                $record
            );
        }
    }
}
```

---

### Migration Template

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
        Schema::create('extension_table_name', function (Blueprint $table) {
            $table->id();
            $table->string('name');
            $table->string('slug')->unique();
            $table->text('description')->nullable();
            $table->boolean('is_active')->default(true);
            $table->integer('sort_order')->default(0);
            $table->timestamps();
            $table->softDeletes(); // Optional
            
            // Indexes
            $table->index('slug');
            $table->index('is_active');
        });
    }

    /**
     * Reverse the migrations.
     */
    public function down(): void
    {
        Schema::dropIfExists('extension_table_name');
    }
};
```

---

## 🔍 Debugging Guide

### Issue: "Extension already installed" on Fresh Install

**Cause:** Config cache not refreshed after uninstall  
**Location:** `ExtensionManager::freshInstall()`  
**Fix:** Already implemented in v1.0.0

```php
// After uninstall, refresh config
app('config')->set("{$this->configPath}.installed", $this->readConfigFromFile('installed'));
app('config')->set("{$this->configPath}.active", $this->readConfigFromFile('active'));
```

---

### Issue: Fix Extension creates duplicates

**Cause:** Seeder using editable field as updateOrCreate key  
**Location:** Extension seeders  
**Fix:** Use ID as key (see Rule #1)

```php
// ❌ WRONG - creates duplicates
Model::updateOrCreate(['slug' => $item['slug']], $item);

// ✅ CORRECT - restores properly
Model::updateOrCreate(['id' => $item['id']], $item);
```

---

### Issue: Custom records overwritten by Fix Extension

**Cause:** Custom records have IDs within base range  
**Location:** Seeder ID ranges  
**Fix:** Expand base ID range

```php
// If custom record has ID=9 but base range is 1-10:
// ❌ Custom record will be overwritten

// Solution: Expand base range to 1-15
/**
 * ID Ranges:
 * - IDs 1-15: Base records ← increased
 * - IDs > 15: Custom records ← safe zone
 */
```

---

## 🧪 Testing Checklist

### Before Submitting Code

**Seeder Testing:**
- [ ] All base records have explicit `id` field
- [ ] updateOrCreate uses `['id' => ...]` as key
- [ ] ID ranges documented in comments
- [ ] DatabaseSeeder only calls essential seeders
- [ ] DemoSeeder separated from DatabaseSeeder

**Fix Extension Testing:**
- [ ] Install extension fresh
- [ ] Edit base records (change names/slugs)
- [ ] Create custom records
- [ ] Run Fix Extension
- [ ] Verify: base restored, custom preserved, no duplicates

**Fresh Install Testing:**
- [ ] Run Fresh Install
- [ ] Verify: all data deleted
- [ ] Verify: backup created
- [ ] Verify: reinstall successful
- [ ] Verify: base seeders ran

---

## 📚 Reference Files

### Working Examples

**Tickets Extension (Complete Implementation):**
- CategorySeeder: `/EXTENSIONS/bithoven-extension-tickets/database/seeders/CategorySeeder.php`
- TemplatesResponsesSeeder: `/EXTENSIONS/bithoven-extension-tickets/database/seeders/TemplatesResponsesSeeder.php`
- AutomationRulesSeeder: `/EXTENSIONS/bithoven-extension-tickets/database/seeders/AutomationRulesSeeder.php`

**Dummy Extension (Minimal Example):**
- Structure: `/EXTENSIONS/bithoven-extension-dummy/`

---

### Core System Files

**Extension Manager:**
- Main class: `/CPANEL/app/Services/Extensions/ExtensionManager.php`
- Methods: `install()`, `uninstall()`, `fixExtension()`, `freshInstall()`

**Controllers:**
- Admin UI: `/CPANEL/app/Http/Controllers/Admin/ExtensionController.php`

**Views:**
- Extensions index: `/CPANEL/resources/views/admin/extensions/index.blade.php`

---

## 🎯 Quick Decision Tree

### "Should I use fixed IDs in this seeder?"

```
Is this seeder called by DatabaseSeeder?
├─ YES → Are these base/essential records?
│  ├─ YES → ✅ USE FIXED IDs (id: 1, 2, 3...)
│  └─ NO → Move to DemoSeeder
└─ NO → Is this DemoSeeder?
   ├─ YES → ❌ NO FIXED IDs needed
   └─ NO → Should this seeder exist?
```

### "What ID range should I use?"

```
How many base records do you have now?
├─ N records → Use IDs 1-N
└─ Allocate room for growth → Use IDs 1-(N*2 or N*3)

Example:
├─ 8 categories now → IDs 1-8 (or 1-15 for future)
├─ 10 templates now → IDs 1-10 (or 1-20 for future)
└─ 24 responses now → IDs 1-24 (or 1-30 for future)
```

---

## 💬 Common Questions

### Q: Can I change base record names in seeders?

**A:** Yes! That's the point. Fix Extension will restore them.

```php
// You CAN update these values freely:
['id' => 1, 'name' => 'New Name', 'slug' => 'new-slug', ...]

// Fix Extension will update existing record ID=1 with new values
```

---

### Q: What if I need to add a new base record?

**A:** Add it with the next sequential ID.

```php
// Existing base records: IDs 1-8
// New base record: ID 9
$baseRecords = [
    // ... existing IDs 1-8
    ['id' => 9, 'name' => 'New Category', ...], // ✅ Add as ID 9
];
```

**Update documentation:**
```php
/**
 * ID Ranges:
 * - IDs 1-9: Base records (was 1-8, added ID 9) ← Update this
 * - IDs > 9: Custom records (was >8, now >9) ← Update this
 */
```

---

### Q: What if user created custom record with ID=5?

**A:** If your base range is 1-8, Fix Extension will overwrite it.

**Solutions:**
1. **Prevention:** Start base range higher (e.g., 100-108) to avoid conflicts
2. **Documentation:** Clearly document ID ranges in UI/docs
3. **Validation:** Add validation to prevent user creating IDs in base range

**Current approach:** Base uses low IDs (1-N), assumes users won't manually set IDs. Most UIs use auto-increment, so no conflict.

---

## 🚀 Performance Tips

### Batch Updates

For large seeders (>100 records), use batch upsert:

```php
// Instead of loop with updateOrCreate:
foreach ($records as $record) {
    Model::updateOrCreate(['id' => $record['id']], $record);
}

// Use upsert for better performance:
Model::upsert(
    $records,
    ['id'],  // Unique key
    ['name', 'slug', 'description', 'updated_at']  // Fields to update
);
```

**Note:** Test thoroughly - upsert behavior differs from updateOrCreate.

---

## 🔗 Related Documentation

- **Main README:** `../README.md`
- **Seeders Guide:** `../guides/SEEDERS-BEST-PRACTICES.md`
- **Extension Structure:** `../guides/EXTENSION-STRUCTURE.md`
- **Fix Extension System:** `../guides/FIX-EXTENSION-SYSTEM.md`
- **Main Project Copilot:** `/CPANEL/.github/copilot-instructions.md`

---

## ✅ Pre-Commit Checklist

Before committing extension code:

### Code Quality
- [ ] All seeders use `updateOrCreate(['id' => ...])`
- [ ] ID ranges documented
- [ ] Demo data in separate DemoSeeder
- [ ] Migrations have proper down() methods
- [ ] No hardcoded paths or credentials

### Testing
- [ ] Fix Extension tested with edited records
- [ ] Fix Extension tested with custom records
- [ ] Fresh Install tested
- [ ] Demo data tested (if applicable)
- [ ] No duplicates created

### Documentation
- [ ] extension.json updated
- [ ] CHANGELOG.md updated
- [ ] README.md reflects changes
- [ ] ID ranges documented in seeder comments

---

**Agent Signature Required:**
When implementing seeders, confirm understanding by including this comment:

```php
/**
 * Seeder Pattern v1.0.0
 * Uses fixed IDs with updateOrCreate(['id' => ...]) for Fix Extension compatibility
 * AI Agent: [Your Name] - Verified SEEDERS-BEST-PRACTICES.md compliance
 */
```

---

**Last Updated:** 16 de noviembre de 2025  
**Maintained by:** Madniatik + AI Agents  
**Version:** 1.1.0
