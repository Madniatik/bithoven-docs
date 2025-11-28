# Extension.json - Schema & Conventions

**Version:** 1.0.0  
**Last Updated:** 17 de noviembre de 2025  
**Last Reviewed:** 26 de noviembre de 2025  
**Proyecto:** BITHOVEN v1.7.0  
**Status:** STABLE

> **Nota:** Este schema fue estandarizado en 3 extensiones (LLM Manager v1.0.2, Tickets v1.2.2, Dummy v1.7.2). Ver `dev/copilot/tasks/EXTENSION-SCHEMA-IMPLEMENTATION-PLAN.md` para validadores futuros.

---

## 📋 Complete Schema

```json
{
  "name": "Extension Name",
  "slug": "extension-slug",
  "version": "1.0.0",
  "description": "Extension description",
  "category": "Category Name",
  "icon": "ki-icon-name",
  "featured": false,
  "tags": ["tag1", "tag2"],
  "migrations": {
    "required": true,
    "count": 3,
    "can_skip": false,
    "details": [
      "2025_01_01_000001_create_table_name",
      "2025_01_01_000002_create_another_table"
    ],
    "description": "Migration description"
  },
  "breaking_changes": false,
  "min_version": "1.0.0",
  "dependencies": {
    "bithoven/core": "^1.4.0",
    "another/package": "^2.0"
  },
  "seeders": {
    "core": ["CoreSeeder1", "CoreSeeder2"],
    "demo": ["DemoSeeder1"]
  },
  "system_tables": ["notifications", "users"],
  "permissions": [
    "view-resource",
    "create-resource",
    "edit-resource",
    "delete-resource"
  ],
  "menu": {
    "enabled": true,
    "items": [
      {
        "label": "Dashboard",
        "route": "extension.dashboard",
        "icon": "ki-home",
        "permission": "view-resource"
      }
    ]
  },
  "changelog": {
    "v1.0.0": {
      "date": "2025-01-01",
      "changes": ["Initial release"],
      "migration_notes": "Creates 3 tables",
      "breaking_changes": false,
      "components": ["code", "database"]
    }
  }
}
```

---

## 🔑 Field Definitions

### Required Fields

#### `name` (string)
Human-readable name displayed in UI.

```json
"name": "Tickets System"
```

**Rules:**
- 3-50 characters
- Can contain spaces, letters, numbers
- Displayed in marketplace cards

#### `slug` (string)
Unique identifier, used in package name `bithoven/{slug}`.

```json
"slug": "tickets"
```

**Rules:**
- 2-30 characters
- Lowercase, alphanumeric, hyphens only
- Must match composer package name
- Used in routes, database prefixes

#### `version` (string)
Semantic version following [SemVer](https://semver.org/).

```json
"version": "1.2.1"
```

**Format:** `MAJOR.MINOR.PATCH`
- `MAJOR`: Breaking changes
- `MINOR`: New features (backward compatible)
- `PATCH`: Bug fixes

#### `description` (string)
Brief description (50-200 chars).

```json
"description": "Complete support ticket system with automation"
```

**Best Practices:**
- Concise but descriptive
- Mention key features
- Avoid marketing language

### Optional Fields

#### `category` (string)
Extension category for marketplace filtering.

```json
"category": "Productivity"
```

**Valid Categories:**
- `Productivity`
- `Development`
- `E-commerce`
- `Communication`
- `Analytics`
- `Security`
- `Utilities`

#### `icon` (string)
Keenicons icon class (without `ki-` prefix).

```json
"icon": "ticket"
```

**Usage:** Displayed in marketplace cards, menu items.

**Find Icons:** `demo7/html/resources/icons/duotone.html`

#### `featured` (boolean)
Whether extension appears in featured section.

```json
"featured": true
```

**Note:** Only core/official extensions should be featured.

#### `tags` (array)
Search keywords for marketplace.

```json
"tags": ["support", "helpdesk", "tickets", "automation"]
```

**Best Practices:**
- 3-8 relevant tags
- Lowercase
- Single words or compound terms
- Think "what would users search for?"

---

## 🗄️ Database Configuration

### `migrations` (object)

#### `required` (boolean)
Whether migrations MUST run during installation.

```json
"required": true
```

**Values:**
- `true`: Installer will fail if migrations not run
- `false`: Migrations optional (rare case)

#### `count` (integer)
Total number of migration files.

```json
"count": 8
```

**Purpose:**
- Validation: Verify all migrations listed
- UI: Progress indicator
- Logging: Track completion

#### `can_skip` (boolean)
Whether user can skip migrations in UI.

```json
"can_skip": false
```

**Values:**
- `false`: Required, cannot skip (most extensions)
- `true`: Optional, user can proceed without migrations

**⚠️ Warning:** Almost always `false`. Only set `true` for extensions with optional features.

#### `details` (array)
List of migration file names (without `.php`).

```json
"details": [
  "2025_01_01_000001_create_ticket_categories_table",
  "2025_01_01_000002_create_tickets_table"
]
```

**Purpose:**
- Documentation: What tables will be created
- Validation: Verify migration files exist
- UI: Display migration list

**Naming Convention:**
```
YYYY_MM_DD_HHMMSS_action_table_name
```

**Actions:**
- `create_`: New table
- `add_column_to_`: Add column
- `modify_`: Alter table structure
- `drop_`: Remove table/column

#### `description` (string)
Human-readable summary of what migrations do.

```json
"description": "Creates 8 tables: ticket_categories, tickets, ticket_comments, ..."
```

**Best Practices:**
- List table names
- Mention important relationships
- Note any system tables modified

### `system_tables` (array)
Core Laravel/Bithoven tables that extension **uses** (not creates).

```json
"system_tables": ["notifications", "users"]
```

**Purpose:**
- Documentation: Dependencies on core tables
- Validation: Verify tables exist before install
- Uninstall: DO NOT drop these tables

**Common System Tables:**
- `users`
- `roles`
- `permissions`
- `notifications`
- `activity_log`

**⚠️ Critical:** These tables are **NEVER** dropped during uninstall.

---

## 📦 Table Naming Conventions

### Prefix Rules

**Extension Tables MUST use slug prefix:**

```
{slug}_{table_name}
```

**Examples:**
```sql
-- ✅ CORRECT
tickets_categories
tickets_tickets
tickets_comments
tickets_attachments

-- ❌ WRONG (no prefix)
categories
tickets
comments

-- ❌ WRONG (wrong prefix)
ticket_categories  -- Missing 's'
support_tickets    -- Wrong slug
```

### Singular vs Plural

**Tables:** Plural (Laravel convention)
```sql
ticket_categories  -- ✅ 
ticket_comments    -- ✅
ticket_attachment  -- ❌ Should be 'attachments'
```

**Models:** Singular
```php
TicketCategory     -- ✅ Maps to 'ticket_categories'
TicketComment      -- ✅ Maps to 'ticket_comments'
```

### Pivot Tables

Format: `{slug}_{table1}_{table2}` (alphabetical order)

```sql
-- ✅ CORRECT
tickets_categories_users
tickets_tags_tickets

-- ❌ WRONG (not alphabetical)
tickets_users_categories
```

### System Table Modifications

**NEVER** create tables that conflict with system:
```sql
-- ❌ FORBIDDEN
notifications
users
permissions
activity_log
```

**Instead:** Use `system_tables` to declare usage.

---

## 🌱 Seeders Configuration

### `seeders` (object)

#### `core` (array)
Essential data required for extension to function.

```json
"core": [
  "CategorySeeder",
  "AutomationRulesSeeder"
]
```

**Purpose:**
- Create default categories
- Set up automation rules
- Configure initial settings
- Create essential records

**Rules:**
- **ALWAYS** run on install (unless user explicitly skips)
- Use actual class names (without namespace)
- Located in `database/seeders/`
- Must be **idempotent** (can run multiple times safely)

**Example Core Seeder:**
```php
class CategorySeeder extends Seeder
{
    public function run()
    {
        $categories = [
            ['name' => 'Technical Support', 'color' => 'primary'],
            ['name' => 'Billing', 'color' => 'success'],
        ];
        
        foreach ($categories as $cat) {
            TicketCategory::firstOrCreate(
                ['name' => $cat['name']], // Unique key
                $cat // All attributes
            );
        }
    }
}
```

**⚠️ Idempotency:** Use `firstOrCreate()`, `updateOrCreate()`, or check existence before insert.

#### `demo` (array)
Sample data for testing/demonstration.

```json
"demo": [
  "TicketsDemoSeeder"
]
```

**Purpose:**
- Create fake tickets
- Generate test users
- Populate with realistic data
- Help users understand features

**Rules:**
- **OPTIONAL** - user can skip
- Should use Faker for realistic data
- Use actual class names
- Located in `database/seeders/`

**Example Demo Seeder:**
```php
class TicketsDemoSeeder extends Seeder
{
    public function run()
    {
        $faker = Faker\Factory::create();
        
        // Create 50 demo tickets
        for ($i = 0; $i < 50; $i++) {
            Ticket::create([
                'title' => $faker->sentence(),
                'description' => $faker->paragraph(),
                'user_id' => User::inRandomOrder()->first()->id,
                // ...
            ]);
        }
    }
}
```

### Seeder Execution Order

**During Installation:**
1. Run migrations
2. Run `core` seeders (in array order)
3. Ask user if they want `demo` data
4. Run `demo` seeders if accepted (in array order)

**Command Line:**
```bash
# Install without demo data
php artisan bithoven:extension:install tickets

# Install with demo data
php artisan bithoven:extension:install tickets --seed
```

---

## 🔐 Permissions Configuration

### `permissions` (array)
List of permissions extension registers.

```json
"permissions": [
  "view-tickets",
  "create-tickets",
  "edit-tickets",
  "delete-tickets",
  "assign-tickets",
  "manage-ticket-categories"
]
```

**Naming Convention:**
```
{action}-{resource}
```

**Standard Actions:**
- `view-`: Read access
- `create-`: Create new records
- `edit-`: Update existing records
- `delete-`: Remove records
- `manage-`: Full admin access to specific area

**Examples:**
```json
[
  "view-tickets",           // Can see tickets
  "create-tickets",         // Can create new ticket
  "edit-own-tickets",       // Can edit their tickets
  "edit-all-tickets",       // Can edit any ticket
  "delete-tickets",         // Can delete tickets
  "assign-tickets",         // Can assign to agents
  "manage-ticket-settings"  // Can configure ticket system
]
```

**Auto-Registration:**
During installation, these permissions are automatically:
1. Created in `permissions` table
2. Assigned to admin role
3. Available in role management UI

**Cleanup:**
During uninstall, permissions are automatically removed.

---

## 🧭 Menu Configuration

### `menu` (object)

#### Complete Structure
```json
"menu": {
  "enabled": true,
  "parent": "apps",
  "items": [
    {
      "label": "Dashboard",
      "route": "tickets.dashboard",
      "icon": "ki-home",
      "permission": "view-tickets"
    },
    {
      "label": "My Tickets",
      "route": "tickets.index",
      "icon": "ki-ticket",
      "permission": null,
      "badge": {
        "type": "counter",
        "query": "Ticket::where('user_id', auth()->id())->count()"
      }
    },
    {
      "label": "Settings",
      "icon": "ki-setting",
      "permission": "manage-tickets",
      "children": [
        {
          "label": "Categories",
          "route": "tickets.categories.index"
        },
        {
          "label": "Automation",
          "route": "tickets.automation.index"
        }
      ]
    }
  ]
}
```

#### `enabled` (boolean)
Whether extension adds menu items.

```json
"enabled": true
```

**Values:**
- `true`: Menu items will be added to sidebar
- `false`: No menu items (extension accessed via other routes)

#### `parent` (string|null)
Parent menu section to nest under.

```json
"parent": "apps"
```

**Valid Parents:**
- `apps`: Applications section
- `admin`: Administration section
- `tools`: Tools section
- `null`: Top-level menu item

#### `items` (array)
Menu item definitions.

##### Single Item Structure
```json
{
  "label": "Dashboard",
  "route": "tickets.dashboard",
  "icon": "ki-home",
  "permission": "view-tickets",
  "badge": {
    "type": "counter",
    "query": "Model::count()"
  }
}
```

**Fields:**
- `label`: Display text
- `route`: Laravel route name (use named routes)
- `icon`: Keenicons icon (without `ki-` prefix)
- `permission`: Required permission (null = everyone)
- `badge`: Optional counter/indicator

##### Nested Menu Structure
```json
{
  "label": "Settings",
  "icon": "ki-setting",
  "permission": "manage-tickets",
  "children": [
    {
      "label": "Categories",
      "route": "tickets.categories.index"
    },
    {
      "label": "Templates",
      "route": "tickets.templates.index"
    }
  ]
}
```

**Nesting Rules:**
- Max 2 levels deep
- Parent must have `icon`
- Children inherit parent permission if not specified

---

## 📝 Changelog Configuration

### `changelog` (object)
Version history with semantic structure.

```json
"changelog": {
  "v1.2.0": {
    "date": "2025-11-16",
    "changes": [
      "Added automation rules system",
      "Fixed notification bug",
      "Improved performance"
    ],
    "migration_notes": "Creates 2 new tables. No data loss.",
    "breaking_changes": false,
    "components": ["code", "database", "views"]
  }
}
```

#### Version Entry Structure

##### `date` (string)
Release date in `YYYY-MM-DD` format.

```json
"date": "2025-11-16"
```

##### `changes` (array)
List of changes in this version.

```json
"changes": [
  "Added automation rules for ticket routing",
  "Fixed TypeError in email notifications",
  "Improved DataTables performance",
  "Updated UI components to Metronic 8.3"
]
```

**Best Practices:**
- Start with verb (Added, Fixed, Improved, Removed)
- Be specific but concise
- Group similar changes
- User-facing language (avoid technical jargon)

##### `migration_notes` (string)
Database changes description.

```json
"migration_notes": "Creates 2 new tables: ticket_automation_rules, ticket_automation_logs. Adds 3 columns to tickets table. No data loss on update."
```

**Include:**
- New tables created
- Columns added/modified
- Data migration details
- Backward compatibility notes

##### `breaking_changes` (boolean)
Whether update breaks existing code.

```json
"breaking_changes": false
```

**`true` when:**
- API changes (removed methods, changed signatures)
- Database changes (removed tables/columns)
- Config changes (removed settings)
- Route changes (removed/renamed routes)

**Requires:** Major version bump (1.x → 2.0)

##### `components` (array)
Which parts of extension changed.

```json
"components": ["code", "database", "views", "config"]
```

**Valid Components:**
- `code`: PHP classes, controllers, models
- `database`: Migrations, schema changes
- `views`: Blade templates, UI
- `config`: Configuration files
- `routes`: Route definitions
- `assets`: JS, CSS, images
- `lang`: Translations
- `docs`: Documentation

---

## 🔗 Dependencies

### `dependencies` (object)
Required packages and their versions.

```json
"dependencies": {
  "bithoven/core": "^1.4.0",
  "spatie/laravel-permission": "^6.0",
  "yajra/laravel-datatables": "^11.0"
}
```

**Format:** `"package/name": "version constraint"`

**Version Constraints:**
- `^1.4.0`: Compatible with 1.4.0 and higher (< 2.0)
- `~1.4.0`: Compatible with 1.4.x only
- `>=1.4.0`: Any version 1.4.0 or higher
- `1.4.*`: Any 1.4.x version

**Core Dependency:**
```json
"bithoven/core": "^1.4.0"
```

**Purpose:**
- Ensures core features available
- Validates compatibility
- Prevents install on incompatible versions

**Common Dependencies:**
- `bithoven/core`: Core framework features
- `spatie/laravel-permission`: Roles/permissions
- `yajra/laravel-datatables`: DataTables
- `livewire/livewire`: Reactive components

---

## ✅ Validation Rules

### File Validation (Installation)

**Installer validates:**
1. `extension.json` exists
2. Valid JSON syntax
3. Required fields present
4. Schema compliance
5. Version format valid
6. Migration files match `details` array
7. Seeder classes exist

**Validation Errors:**
```json
{
  "valid": false,
  "errors": [
    "Missing required field: slug",
    "Invalid version format: 1.2 (must be X.Y.Z)",
    "Migration file not found: 2025_01_01_000001_...",
    "Seeder class not found: CategorySeeder"
  ]
}
```

### Schema Validation

**Type Checks:**
- `name`: string, 3-50 chars
- `slug`: string, 2-30 chars, lowercase, alphanumeric + hyphens
- `version`: string, semantic version format
- `migrations.required`: boolean
- `migrations.count`: integer > 0
- `seeders.core`: array of strings
- `permissions`: array of strings

**Constraint Checks:**
- `slug` must match `bithoven/{slug}` in composer.json
- `migrations.count` must equal length of `details` array
- `version` must be ≥ `min_version`

---

## 📚 Complete Examples

### Minimal Extension (No Database)

```json
{
  "name": "Hello World",
  "slug": "hello-world",
  "version": "1.0.0",
  "description": "Simple greeting extension",
  "category": "Utilities",
  "migrations": {
    "required": false,
    "count": 0,
    "can_skip": true
  },
  "dependencies": {
    "bithoven/core": "^1.0.0"
  }
}
```

### Standard CRUD Extension

```json
{
  "name": "Products",
  "slug": "products",
  "version": "1.0.0",
  "description": "Product catalog management",
  "category": "E-commerce",
  "icon": "ki-package",
  "migrations": {
    "required": true,
    "count": 2,
    "can_skip": false,
    "details": [
      "2025_01_01_000001_create_products_categories_table",
      "2025_01_01_000002_create_products_table"
    ]
  },
  "seeders": {
    "core": ["ProductCategoriesSeeder"],
    "demo": ["ProductsSeeder"]
  },
  "permissions": [
    "view-products",
    "create-products",
    "edit-products",
    "delete-products"
  ],
  "dependencies": {
    "bithoven/core": "^1.4.0"
  }
}
```

---

## 🚨 Common Mistakes

### ❌ Wrong: Seeder Namespaces
```json
"seeders": {
  "core": ["Database\\Seeders\\CategorySeeder"]  // ❌ Don't include namespace
}
```

✅ **Correct:**
```json
"seeders": {
  "core": ["CategorySeeder"]  // ✅ Just class name
}
```

### ❌ Wrong: Migration Count Mismatch
```json
"migrations": {
  "count": 5,
  "details": [
    "2025_01_01_000001_create_table"
    // Only 1 migration listed, but count says 5
  ]
}
```

✅ **Correct:**
```json
"migrations": {
  "count": 1,  // Must match array length
  "details": ["2025_01_01_000001_create_table"]
}
```

### ❌ Wrong: Table Without Prefix
```sql
CREATE TABLE categories ...  -- ❌ Missing slug prefix
```

✅ **Correct:**
```sql
CREATE TABLE tickets_categories ...  -- ✅ Prefixed with slug
```

### ❌ Wrong: System Table in Migrations
```json
"migrations": {
  "details": [
    "2025_01_01_000001_create_notifications_table"  // ❌ System table
  ]
}
```

✅ **Correct:**
```json
"system_tables": ["notifications"],  // ✅ Declare usage
"migrations": {
  "details": []  // Don't create it
}
```

---

## 🔧 Development Workflow

### 1. Create extension.json
```bash
# Ver ejemplo completo en:
# ../../../DOCS/CORE/Extension-Manager/examples/extension.json
```

### 2. Edit Schema
Fill in required fields, add migrations, seeders, permissions.

### 3. Validate Locally
```bash
php artisan bithoven:extension:validate tickets --local
```

### 4. Create Migrations
```bash
php artisan make:migration create_tickets_categories_table \
  --path=database/migrations
```

### 5. Create Seeders
```bash
php artisan make:seeder CategorySeeder
```

### 6. Test Installation
```bash
php artisan bithoven:extension:install tickets --local --seed
```

### 7. Commit & Tag
```bash
git add extension.json
git commit -m "docs: update extension.json for v1.2.0"
git tag v1.2.0
git push origin develop --tags
```

---

**Related Documentation:**
- [Extension Development Guide](../../.github/copilot-core/EXTENSION-DEVELOPMENT.md)
- [Extension Installation Flow](./EXTENSION-INSTALLATION-FLOW.md)
- [Database Conventions](./DATABASE-CONVENTIONS.md)
