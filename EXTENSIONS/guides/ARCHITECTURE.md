# BITHOVEN Extension System Architecture

**Version:** 2.0.0  
**Last Updated:** 17 de noviembre de 2025  
**Status:** STABLE

---

## 🎯 Overview

El sistema de extensiones de BITHOVEN permite modularizar funcionalidades mediante paquetes Composer que se integran perfectamente con el core.

**Principios:**
- **Modular:** Cada extensión es independiente
- **Reversible:** Install/uninstall sin romper el sistema
- **Configurable:** VCS (GitHub) o Local (desarrollo)
- **Compatible:** Múltiples extensiones coexisten

---

## 🏗️ System Components

### 1. Core System (CPANEL)

```
app/
├── Core/
│   ├── Theme.php           # Metronic theme config
│   └── Bootstrap.php       # System initialization
└── Services/
    └── ExtensionManager.php # Extension orchestration
```

**ExtensionManager** es el cerebro del sistema:
- Install/uninstall extensions
- Manage configuration
- Run migrations/seeders
- Register permissions/menu items

### 2. Extension Package

```
bithoven-extension-{slug}/
├── composer.json           # Package definition
├── extension.json         # BITHOVEN metadata
├── src/
│   └── {Slug}ServiceProvider.php
├── database/
│   ├── migrations/
│   └── seeders/
└── resources/views/
```

### 3. Configuration Files

**Three-file system:**

#### a) `composer.json` (CPANEL)
```json
{
  "repositories": [
    {
      "type": "vcs",
      "url": "https://github.com/user/bithoven-extension-tasks"
    }
  ],
  "require": {
    "bithoven/tasks": "^1.0"
  }
}
```

#### b) `config/bithoven-extensions.php` (CPANEL)
```php
return [
    'installed' => ['tasks', 'tickets'],
    'active' => ['tasks', 'tickets'],
];
```

#### c) `storage/app/extension-settings.json` (CPANEL)
```json
{
  "tasks": {
    "type": "vcs",
    "url": "https://github.com/user/bithoven-extension-tasks"
  },
  "tickets": {
    "type": "local",
    "path": "../EXTENSIONS/bithoven-extension-tickets"
  }
}
```

---

## 🔄 Installation Flow

### Step 1: User Initiates Install

```bash
php artisan bithoven:extension:install tasks
```

**OR**

Via Admin UI: `/admin/extensions` → "Install tasks"

### Step 2: ExtensionManager Process

```
┌─────────────────────────────────────┐
│ 1. Validate extension.json exists  │
│ 2. Check if already installed       │
│ 3. Build configuration              │
└──────────────┬──────────────────────┘
               │
               ▼
┌─────────────────────────────────────┐
│ 4. Save to extension-settings.json │ ← FIRST (critical order)
└──────────────┬──────────────────────┘
               │
               ▼
┌─────────────────────────────────────┐
│ 5. Update composer.json             │
│    - Add repository (vcs/path)      │
│    - Add to require{}               │
└──────────────┬──────────────────────┘
               │
               ▼
┌─────────────────────────────────────┐
│ 6. Run composer require             │
│    Downloads & installs package     │
└──────────────┬──────────────────────┘
               │
               ▼
┌─────────────────────────────────────┐
│ 7. Update bithoven-extensions.php   │
│    - Add to installed[]             │
│    - Add to active[]                │
└──────────────┬──────────────────────┘
               │
               ▼
┌─────────────────────────────────────┐
│ 8. Run migrations                   │
└──────────────┬──────────────────────┘
               │
               ▼
┌─────────────────────────────────────┐
│ 9. Run seeders (core + demo)        │
└──────────────┬──────────────────────┘
               │
               ▼
┌─────────────────────────────────────┐
│ 10. Register permissions & menu     │
└─────────────────────────────────────┘
```

---

## 🗑️ Uninstallation Flow

```
┌─────────────────────────────────────┐
│ 1. Validate extension is installed │
└──────────────┬──────────────────────┘
               │
               ▼
┌─────────────────────────────────────┐
│ 2. Disable extension                │
│    - Remove from active[]           │
└──────────────┬──────────────────────┘
               │
               ▼
┌─────────────────────────────────────┐
│ 3. Remove permissions               │
└──────────────┬──────────────────────┘
               │
               ▼
┌─────────────────────────────────────┐
│ 4. Remove menu items                │
└──────────────┬──────────────────────┘
               │
               ▼
┌─────────────────────────────────────┐
│ 5. Rollback migrations (optional)   │
└──────────────┬──────────────────────┘
               │
               ▼
┌─────────────────────────────────────┐
│ 6. Run composer remove              │
└──────────────┬──────────────────────┘
               │
               ▼
┌─────────────────────────────────────┐
│ 7. Remove from composer.json        │
└──────────────┬──────────────────────┘
               │
               ▼
┌─────────────────────────────────────┐
│ 8. Remove from bithoven-extensions  │
└──────────────┬──────────────────────┘
               │
               ▼
┌─────────────────────────────────────┐
│ 9. Remove from extension-settings   │
└─────────────────────────────────────┘
```

---

## 🔧 Extension Modes

### VCS Mode (Production)

**For:** Extensions hosted on GitHub

```json
{
  "tasks": {
    "type": "vcs",
    "url": "https://github.com/user/bithoven-extension-tasks"
  }
}
```

**Composer repository:**
```json
{
  "type": "vcs",
  "url": "https://github.com/user/bithoven-extension-tasks"
}
```

### Local Mode (Development)

**For:** Local development with symlink

```json
{
  "tickets": {
    "type": "local",
    "path": "../EXTENSIONS/bithoven-extension-tickets"
  }
}
```

**Composer repository:**
```json
{
  "type": "path",
  "url": "../EXTENSIONS/bithoven-extension-tickets",
  "options": {
    "symlink": true
  }
}
```

---

## 🎯 Extension Lifecycle

```
NOT INSTALLED → INSTALLING → INSTALLED & ACTIVE → DISABLED → UNINSTALLED
                    ↓             ↓                    ↓
               (migrations)  (fully working)      (inactive)
```

**States:**
- **Not Installed:** Extension not in system
- **Installing:** Process in progress
- **Installed & Active:** Fully operational
- **Disabled:** Installed but inactive
- **Uninstalled:** Removed from system

---

## 📊 Data Flow

### Installation Data Flow

```
GitHub/Local
     ↓
composer require
     ↓
vendor/bithoven/{slug}/
     ↓
ServiceProvider auto-discovered
     ↓
Migrations loaded
     ↓
Routes registered
     ↓
Views available
```

### Request Flow

```
User Request
     ↓
Route (loaded by ServiceProvider)
     ↓
Controller (extension's)
     ↓
Model (extension's tables)
     ↓
View (extension's views)
     ↓
Response
```

---

## 🔐 Permission System

### Registration

**extension.json:**
```json
{
  "permissions": [
    "view-tasks",
    "create-tasks",
    "edit-tasks",
    "delete-tasks"
  ]
}
```

### Installation

ExtensionManager creates permissions during install:
```php
foreach ($permissions as $permission) {
    Permission::create(['name' => $permission, 'guard_name' => 'web']);
}
```

### Usage

```php
// In controller
if (!auth()->user()->can('view-tasks')) {
    abort(403);
}

// In blade
@can('create-tasks')
    <a href="{{ route('tasks.create') }}">New Task</a>
@endcan

// In policy
public function view(User $user, Task $task): bool
{
    return $user->can('view-tasks') && $user->id === $task->user_id;
}
```

---

## 🎨 Menu System

**extension.json:**
```json
{
  "menu": {
    "label": "Tasks",
    "icon": "ki-task",
    "route": "tasks.index",
    "order": 10,
    "parent": null
  }
}
```

**Registration:** ExtensionManager adds to menu during install

---

## 🗄️ Database Integration

### System Tables

Extensions can reference system tables:

```json
{
  "system_tables": ["users", "permissions", "roles"]
}
```

### Extension Tables

**Naming:** `{slug}_tablename` or just `{slug}` for main table

```php
Schema::create('tasks', function (Blueprint $table) {
    $table->id();
    $table->foreignId('user_id')->constrained('users')->onDelete('cascade');
    // ...
});

Schema::create('tasks_categories', function (Blueprint $table) {
    // Related table
});
```

---

## 🔄 Version Management

**extension.json:**
```json
{
  "version": "1.2.0",
  "changelog": {
    "v1.2.0": {
      "date": "2025-01-15",
      "changes": ["Added categories", "Fixed bug #42"],
      "migration_notes": "Adds tasks_categories table",
      "breaking_changes": false,
      "components": ["code", "database"]
    },
    "v1.1.0": {
      "date": "2025-01-01",
      "changes": ["Initial release"]
    }
  }
}
```

---

## 🧩 Extension Interactions

### Extension A → System Tables

```php
// Extension uses system tables
use App\Models\User;

$task->user_id = auth()->id();
$task->save();
```

### Extension A → Extension B

**NOT RECOMMENDED** - Extensions should be independent

**If necessary:**
```json
{
  "dependencies": {
    "bithoven/tickets": "^1.0"
  }
}
```

---

## 📋 Architecture Patterns

### Repository Pattern (Optional)

```php
interface TaskRepositoryInterface
{
    public function all();
    public function find($id);
    public function create(array $data);
}

class TaskRepository implements TaskRepositoryInterface
{
    public function all()
    {
        return Task::all();
    }
}
```

**Register in ServiceProvider:**
```php
$this->app->bind(TaskRepositoryInterface::class, TaskRepository::class);
```

### Service Pattern (Recommended)

```php
class TaskService
{
    public function createTask(array $data): Task
    {
        // Business logic
        return Task::create($data);
    }
}
```

**Register in ServiceProvider:**
```php
$this->app->singleton(TaskService::class);
```

---

## 🔗 Related Documentation

- **[Extension Structure](./EXTENSION-STRUCTURE.md)** - Directory organization
- **[Developer Guide](../DEVELOPER-GUIDE.md)** - Creating extensions
- **[Configuration System](./CONFIGURATION-SYSTEM.md)** - Config management
- **[Installation Flow](../../CPANEL/docs/extensions/EXTENSION-INSTALLATION-FLOW.md)** - Detailed flow

---

**Key Takeaway:** El sistema está diseñado para máxima modularidad - cada extensión es un paquete Composer independiente que se integra limpiamente con BITHOVEN.
