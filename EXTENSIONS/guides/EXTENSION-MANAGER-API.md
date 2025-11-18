# Extension Manager API Reference

**Version:** 2.0.0  
**Last Updated:** 17 de noviembre de 2025  
**Status:** STABLE

---

## 🎯 Overview

Complete API reference for `App\Services\ExtensionManager` - the core service managing all extension operations.

**Location:** `app/Services/ExtensionManager.php`

---

## 📋 Class Overview

```php
<?php

namespace App\Services;

class ExtensionManager
{
    // Installation
    public function install(string $slug, array $options = []): bool
    public function uninstall(string $slug, array $options = []): bool
    
    // Status Management
    public function enable(string $slug): bool
    public function disable(string $slug): bool
    public function isInstalled(string $slug): bool
    public function isActive(string $slug): bool
    
    // Information
    public function getInstalled(): array
    public function getActive(): array
    public function getInfo(string $slug): ?array
    public function list(): Collection
    
    // Configuration
    public function getConfig(string $slug): ?array
    public function saveConfig(string $slug, array $config): bool
    
    // Utilities
    public function refresh(): void
    public function validate(string $slug): bool
}
```

---

## 🔧 Installation Methods

### install()

**Installs an extension.**

```php
public function install(string $slug, array $options = []): bool
```

**Parameters:**
- `$slug` (string) - Extension slug (e.g., 'tasks')
- `$options` (array) - Optional configuration
  - `type` (string) - 'vcs' or 'local'
  - `url` (string) - GitHub URL (for vcs)
  - `path` (string) - Local path (for local)
  - `skip_migrations` (bool) - Skip running migrations
  - `skip_seeders` (bool) - Skip running seeders

**Returns:** `bool` - Success status

**Example:**
```php
use App\Services\ExtensionManager;

$manager = app(ExtensionManager::class);

// VCS install
$manager->install('tasks', [
    'type' => 'vcs',
    'url' => 'https://github.com/user/bithoven-extension-tasks'
]);

// Local install
$manager->install('tasks', [
    'type' => 'local',
    'path' => '../EXTENSIONS/bithoven-extension-tasks'
]);
```

**Process:**
1. Validate extension exists
2. Check not already installed
3. Save configuration
4. Update composer.json
5. Run `composer require`
6. Update bithoven-extensions.php
7. Run migrations
8. Run seeders
9. Register permissions
10. Register menu items

---

### uninstall()

**Uninstalls an extension.**

```php
public function uninstall(string $slug, array $options = []): bool
```

**Parameters:**
- `$slug` (string) - Extension slug
- `$options` (array) - Optional configuration
  - `keep_data` (bool) - Don't drop tables
  - `force` (bool) - Force uninstall even if errors

**Returns:** `bool` - Success status

**Example:**
```php
// Standard uninstall
$manager->uninstall('tasks');

// Keep data
$manager->uninstall('tasks', ['keep_data' => true]);

// Force uninstall
$manager->uninstall('tasks', ['force' => true]);
```

**Process:**
1. Validate extension installed
2. Disable extension
3. Remove permissions
4. Remove menu items
5. Rollback migrations (unless keep_data)
6. Run `composer remove`
7. Update config files
8. Clear caches

---

## 📊 Status Management

### enable()

**Enables a disabled extension.**

```php
public function enable(string $slug): bool
```

**Example:**
```php
$manager->enable('tasks');
```

**Effect:** Routes, views, and features become available

---

### disable()

**Disables an installed extension.**

```php
public function disable(string $slug): bool
```

**Example:**
```php
$manager->disable('tasks');
```

**Effect:** Routes 404, features inaccessible (but data preserved)

---

### isInstalled()

**Check if extension is installed.**

```php
public function isInstalled(string $slug): bool
```

**Example:**
```php
if ($manager->isInstalled('tasks')) {
    echo "Tasks extension is installed";
}
```

---

### isActive()

**Check if extension is active.**

```php
public function isActive(string $slug): bool
```

**Example:**
```php
if ($manager->isActive('tasks')) {
    echo "Tasks extension is active";
}
```

---

## 📖 Information Methods

### getInstalled()

**Get list of installed extensions.**

```php
public function getInstalled(): array
```

**Returns:** Array of slugs

**Example:**
```php
$installed = $manager->getInstalled();
// ['tasks', 'tickets', 'dummy']
```

---

### getActive()

**Get list of active extensions.**

```php
public function getActive(): array
```

**Returns:** Array of slugs

**Example:**
```php
$active = $manager->getActive();
// ['tasks', 'tickets']
```

---

### getInfo()

**Get extension information from extension.json.**

```php
public function getInfo(string $slug): ?array
```

**Returns:** Array of extension metadata or null

**Example:**
```php
$info = $manager->getInfo('tasks');

/*
[
    'name' => 'Task Manager',
    'slug' => 'tasks',
    'version' => '1.0.0',
    'description' => 'Task management system',
    'category' => 'Productivity',
    'migrations' => [...],
    'seeders' => [...],
    'permissions' => [...]
]
*/
```

---

### list()

**Get detailed list of all available extensions.**

```php
public function list(): Collection
```

**Returns:** Collection of extension objects

**Example:**
```php
$extensions = $manager->list();

foreach ($extensions as $ext) {
    echo "{$ext->name} ({$ext->slug}) - ";
    echo $ext->is_installed ? 'Installed' : 'Not Installed';
    echo "\n";
}
```

**Object structure:**
```php
[
    'slug' => 'tasks',
    'name' => 'Task Manager',
    'version' => '1.0.0',
    'description' => '...',
    'is_installed' => true,
    'is_active' => true,
    'type' => 'vcs', // or 'local'
    'path' => 'vendor/bithoven/tasks'
]
```

---

## ⚙️ Configuration Methods

### getConfig()

**Get extension configuration.**

```php
public function getConfig(string $slug): ?array
```

**Returns:** Configuration array or null

**Example:**
```php
$config = $manager->getConfig('tasks');

/*
[
    'type' => 'vcs',
    'url' => 'https://github.com/user/bithoven-extension-tasks'
]
*/
```

---

### saveConfig()

**Save extension configuration.**

```php
public function saveConfig(string $slug, array $config): bool
```

**Example:**
```php
$manager->saveConfig('tasks', [
    'type' => 'local',
    'path' => '../EXTENSIONS/bithoven-extension-tasks'
]);
```

---

## 🔧 Utility Methods

### refresh()

**Refresh extension cache and state.**

```php
public function refresh(): void
```

**Example:**
```php
$manager->refresh();
```

**When to use:**
- After manual config changes
- After composer operations
- To clear stale cache

---

### validate()

**Validate extension structure.**

```php
public function validate(string $slug): bool
```

**Returns:** `bool` - Validation result

**Example:**
```php
if ($manager->validate('tasks')) {
    echo "Extension structure valid";
} else {
    echo "Extension has validation errors";
}
```

**Checks:**
- extension.json exists
- composer.json exists
- Service provider exists
- Required directories exist

---

## 🎯 Usage Examples

### Example 1: Install & Enable

```php
$manager = app(ExtensionManager::class);

// Install from GitHub
$success = $manager->install('tasks', [
    'type' => 'vcs',
    'url' => 'https://github.com/user/bithoven-extension-tasks'
]);

if ($success) {
    echo "✅ Tasks extension installed";
    
    // Already enabled by default
    if ($manager->isActive('tasks')) {
        echo "✅ Tasks extension active";
    }
}
```

### Example 2: Disable Temporarily

```php
// Disable for maintenance
$manager->disable('tasks');

// Perform maintenance
// ...

// Re-enable
$manager->enable('tasks');
```

### Example 3: List All Extensions

```php
$extensions = $manager->list();

foreach ($extensions as $ext) {
    echo str_pad($ext->name, 20);
    echo str_pad($ext->version, 10);
    echo $ext->is_installed ? '● Installed' : '○ Available';
    echo "\n";
}
```

### Example 4: Conditional Features

```php
// In blade
@if(app(ExtensionManager::class)->isActive('tasks'))
    <a href="{{ route('tasks.index') }}">My Tasks</a>
@endif

// In controller
public function index()
{
    $manager = app(ExtensionManager::class);
    
    if (!$manager->isActive('tasks')) {
        abort(404, 'Tasks extension not available');
    }
    
    // ...
}
```

---

## 🚨 Error Handling

### Installation Errors

```php
try {
    $manager->install('tasks');
} catch (\Exception $e) {
    if (str_contains($e->getMessage(), 'already installed')) {
        echo "Extension already installed";
    } elseif (str_contains($e->getMessage(), 'not found')) {
        echo "Extension not found";
    } else {
        echo "Installation failed: " . $e->getMessage();
    }
}
```

### Validation Errors

```php
if (!$manager->validate('tasks')) {
    $errors = $manager->getValidationErrors('tasks');
    foreach ($errors as $error) {
        echo "❌ {$error}\n";
    }
}
```

---

## 📋 API Checklist

**Before using ExtensionManager:**
- [ ] Understand install vs enable
- [ ] Know difference between VCS and local mode
- [ ] Prepared for potential errors
- [ ] Have backup plan

**Common Operations:**
- [ ] Install: `install()`
- [ ] Check status: `isInstalled()`, `isActive()`
- [ ] List available: `list()`
- [ ] Enable/disable: `enable()`, `disable()`
- [ ] Uninstall: `uninstall()`

---

## 🔗 Related Documentation

- **[Architecture](./ARCHITECTURE.md)** - System overview
- **[Installation Flow](../../CPANEL/docs/extensions/EXTENSION-INSTALLATION-FLOW.md)** - Detailed process
- **[Configuration System](./CONFIGURATION-SYSTEM.md)** - Config management

---

**Remember:** ExtensionManager is the single source of truth for all extension operations!
