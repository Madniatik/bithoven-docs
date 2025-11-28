# Extension Installation & Configuration Flow

**Version:** 2.0.0  
**Last Updated:** 17 de noviembre de 2025  
**Status:** PRODUCTION

---

## 📁 Configuration Files Overview

El sistema de extensiones utiliza 3 archivos de configuración complementarios:

| Archivo | Propósito | Scope | Manejado por |
|---------|-----------|-------|--------------|
| `composer.json` | Package dependencies & repositories | Global project | Composer |
| `config/bithoven-extensions.php` | Installed & active extensions list | Extension system | PHP cache |
| `storage/app/extension-settings.json` | Per-extension configuration | Extension system | JSON file |

---

## 🔄 Installation Flow

### 1. Pre-Installation Validation

**Trigger:** `php artisan bithoven:extension:install tickets --local`

**Step 1.1: Validate Extension Source**
```php
// For VCS mode
$validator->validateGitHubRepo($name);
// Checks:
// - Repository exists
// - extension.json accessible
// - Valid JSON schema

// For Local mode
$validator->validateLocalPath($path);
// Checks:
// - Directory exists
// - extension.json exists
// - Valid JSON schema
// - Migration files exist
```

**Step 1.2: Check Dependencies**
```php
$extensionJson = json_decode(file_get_contents('extension.json'));

foreach ($extensionJson->dependencies as $package => $version) {
    if (!$this->packageExists($package, $version)) {
        throw new DependencyException("Missing: {$package} {$version}");
    }
}
```

**Step 1.3: Check Conflicts**
```php
// Check if already installed
if ($this->isInstalled($name)) {
    throw new InstallationException("Extension already installed");
}

// Check table name conflicts
$existingTables = Schema::getAllTables();
$extensionTables = $this->getExtensionTables($name);

if ($conflicts = array_intersect($existingTables, $extensionTables)) {
    throw new ConflictException("Table conflicts: " . implode(', ', $conflicts));
}
```

### 2. Configuration Setup

**Step 2.1: Build Extension Config**
```php
protected function buildExtensionConfig(string $name, array $options): array
{
    $type = $options['type'] ?? 'vcs';
    
    if ($type === 'local') {
        return [
            'type' => 'local',
            'path' => $options['path'] ?? "../EXTENSIONS/bithoven-extension-{$name}",
        ];
    }
    
    return [
        'type' => 'vcs',
        'url' => $options['url'] ?? "https://github.com/Madniatik/bithoven-extension-{$name}.git",
    ];
}
```

**Step 2.2: Save to extension-settings.json**
```php
// CRITICAL: Save config BEFORE composer operations
$config = $this->buildExtensionConfig($name, $options);
setExtensionConfig($name, $config);

Log::info("Extension config saved", [
    'extension' => $name,
    'config' => $config
]);
```

**File Update:**
```json
// Before
{
  "cache_ttl": 60,
  "extensions": {}
}

// After
{
  "cache_ttl": 60,
  "extensions": {
    "tickets": {
      "type": "local",
      "path": "../EXTENSIONS/bithoven-extension-tickets"
    }
  }
}
```

### 3. Composer Repository Configuration

**Step 3.1: Read Extension Config**
```php
public function ensureRepositoryExists(string $name): void
{
    // Read saved config
    $config = getExtensionConfig($name);
    
    if (!$config) {
        // Fallback to VCS if no config
        $config = [
            'type' => 'vcs',
            'url' => "https://github.com/Madniatik/bithoven-extension-{$name}.git"
        ];
    }
    
    $this->addRepository($name, $config);
}
```

**Step 3.2: Build Repository Array**
```php
protected function buildRepository(string $name, array $config): array
{
    if ($config['type'] === 'local') {
        return [
            'type' => 'path',
            'url' => $config['path'],
            'options' => [
                'symlink' => true  // ALWAYS true for local
            ]
        ];
    }
    
    return [
        'type' => 'vcs',
        'url' => $config['url']
    ];
}
```

**Step 3.3: Update composer.json**
```php
$composerJson = json_decode(file_get_contents('composer.json'), true);

// Check if repository already exists
$exists = false;
foreach ($composerJson['repositories'] ?? [] as $repo) {
    if ($repo['url'] === $repository['url']) {
        $exists = true;
        break;
    }
}

if (!$exists) {
    $composerJson['repositories'][] = $repository;
    file_put_contents('composer.json', json_encode($composerJson, JSON_PRETTY_PRINT));
}
```

**File Update - VCS Mode:**
```json
{
  "repositories": [
    {
      "type": "vcs",
      "url": "https://github.com/Madniatik/bithoven-extension-tickets.git"
    }
  ]
}
```

**File Update - Local Mode:**
```json
{
  "repositories": [
    {
      "type": "path",
      "url": "../EXTENSIONS/bithoven-extension-tickets",
      "options": {
        "symlink": true
      }
    }
  ]
}
```

### 4. Composer Require

**Step 4.1: Execute Composer**
```php
$command = "composer require bithoven/{$name}:* --no-scripts --no-plugins 2>&1";
exec($command, $output, $returnCode);
```

**What Happens:**
1. Composer reads `repositories` array
2. Finds package `bithoven/{name}`
3. For `path` type: Creates symlink to local directory
4. For `vcs` type: Clones from GitHub
5. Updates `composer.lock`
6. Updates `vendor/` directory

**Step 4.2: Handle Failures**
```php
if ($returnCode !== 0) {
    // Rollback: Remove config
    removeExtensionConfig($name);
    
    // Rollback: Remove repository from composer.json
    $this->removeRepository($name);
    
    Log::error("Composer install failed", [
        'extension' => $name,
        'output' => $output
    ]);
    
    throw new ComposerException("Failed to install package");
}
```

### 5. Extension Registration

**Step 5.1: Update bithoven-extensions.php**
```php
$config = config('bithoven-extensions');

// Add to installed list
$config['installed'][] = $name;

// Add to active list
$config['active'][] = $name;

// Save to config file
file_put_contents(
    config_path('bithoven-extensions.php'),
    '<?php return ' . var_export($config, true) . ';'
);
```

**File Update:**
```php
<?php

return array (
  'installed' => 
  array (
    0 => 'tickets',
  ),
  'active' => 
  array (
    0 => 'tickets',
  ),
  'settings' => 
  array (
  ),
);
```

**Step 5.2: Clear Config Cache**
```php
Artisan::call('config:clear');
Log::info("Config cache cleared");
```

### 6. Database Migrations

**Step 6.1: Check Migration Requirement**
```php
$extensionJson = $this->getExtensionJson($name);

if ($extensionJson->migrations->required && !$options['skip-migrate']) {
    $this->runMigrations($name);
}
```

**Step 6.2: Execute Migrations**
```php
protected function runMigrations(string $name): void
{
    $path = "vendor/bithoven/{$name}/database/migrations";
    
    Artisan::call('migrate', [
        '--path' => $path,
        '--force' => true
    ]);
    
    Log::info("Migrations executed", [
        'extension' => $name,
        'path' => $path
    ]);
}
```

### 7. Run Seeders

**Step 7.1: Core Seeders**
```php
$extensionJson = $this->getExtensionJson($name);

foreach ($extensionJson->seeders->core as $seederClass) {
    $fullClass = "Bithoven\\{$name}\\Database\\Seeders\\{$seederClass}";
    
    Artisan::call('db:seed', [
        '--class' => $fullClass,
        '--force' => true
    ]);
    
    Log::info("Core seeder executed", [
        'extension' => $name,
        'seeder' => $seederClass
    ]);
}
```

**Step 7.2: Demo Seeders (Optional)**
```php
if ($options['seed'] ?? false) {
    foreach ($extensionJson->seeders->demo as $seederClass) {
        $fullClass = "Bithoven\\{$name}\\Database\\Seeders\\{$seederClass}";
        
        Artisan::call('db:seed', [
            '--class' => $fullClass,
            '--force' => true
        ]);
        
        Log::info("Demo seeder executed", [
            'extension' => $name,
            'seeder' => $seederClass
        ]);
    }
}
```

### 8. Permission Registration

**Step 8.1: Create Permissions**
```php
$extensionJson = $this->getExtensionJson($name);

foreach ($extensionJson->permissions as $permissionName) {
    Permission::firstOrCreate([
        'name' => $permissionName,
        'guard_name' => 'web'
    ]);
    
    Log::info("Permission created", [
        'extension' => $name,
        'permission' => $permissionName
    ]);
}
```

**Step 8.2: Assign to Admin Role**
```php
$adminRole = Role::where('name', 'admin')->first();

if ($adminRole) {
    $adminRole->givePermissionTo($extensionJson->permissions);
    
    Log::info("Permissions assigned to admin role", [
        'extension' => $name,
        'permissions' => count($extensionJson->permissions)
    ]);
}
```

### 9. Menu Registration

**Step 9.1: Register Menu Items**
```php
if ($extensionJson->menu->enabled ?? false) {
    foreach ($extensionJson->menu->items as $item) {
        $this->registerMenuItem($name, $item);
    }
}
```

**Step 9.2: Cache Menu Structure**
```php
Cache::forget('app_menu_structure');
Cache::rememberForever('app_menu_structure', function () {
    return $this->buildMenuFromExtensions();
});
```

### 10. Post-Installation

**Step 10.1: Fire Event**
```php
event(new ExtensionInstalled($name, $version));

Log::info("Extension installed successfully", [
    'extension' => $name,
    'version' => $version,
    'type' => $config['type']
]);
```

**Step 10.2: Clear Caches**
```php
Artisan::call('cache:clear');
Artisan::call('view:clear');
Artisan::call('route:clear');
```

**Step 10.3: Activity Log**
```php
activity()
    ->causedBy(auth()->user())
    ->performedOn($extension)
    ->withProperties([
        'extension' => $name,
        'version' => $version,
        'type' => $config['type']
    ])
    ->log("Extension installed: {$name}");
```

---

## 🗑️ Uninstallation Flow

### 1. Pre-Uninstall Validation

**Step 1.1: Check if Installed**
```php
if (!$this->isInstalled($name)) {
    throw new UninstallException("Extension not installed");
}
```

**Step 1.2: Check Dependencies**
```php
// Check if other extensions depend on this one
$dependents = $this->getDependentExtensions($name);

if (!empty($dependents) && !$options['force']) {
    throw new DependencyException(
        "Cannot uninstall. Required by: " . implode(', ', $dependents)
    );
}
```

### 2. Disable Extension

**Step 2.1: Remove from Active List**
```php
$config = config('bithoven-extensions');

$config['active'] = array_filter($config['active'], function($ext) use ($name) {
    return $ext !== $name;
});

file_put_contents(
    config_path('bithoven-extensions.php'),
    '<?php return ' . var_export($config, true) . ';'
);
```

### 3. Remove Permissions

**Step 3.1: Revoke from Roles**
```php
$extensionJson = $this->getExtensionJson($name);

foreach ($extensionJson->permissions as $permissionName) {
    $permission = Permission::where('name', $permissionName)->first();
    
    if ($permission) {
        // Revoke from all roles
        foreach ($permission->roles as $role) {
            $role->revokePermissionTo($permission);
        }
        
        // Delete permission
        $permission->delete();
        
        Log::info("Permission removed", [
            'extension' => $name,
            'permission' => $permissionName
        ]);
    }
}
```

### 4. Remove Menu Items

```php
Cache::forget('app_menu_structure');
Log::info("Menu cache cleared for extension", ['extension' => $name]);
```

### 5. Rollback Migrations (Optional)

**Step 5.1: Ask User Confirmation**
```php
if ($options['rollback-migrations'] ?? false) {
    $this->rollbackMigrations($name);
}
```

**Step 5.2: Execute Rollback**
```php
protected function rollbackMigrations(string $name): void
{
    $extensionJson = $this->getExtensionJson($name);
    
    // Get migration files in reverse order
    $migrations = array_reverse($extensionJson->migrations->details);
    
    foreach ($migrations as $migration) {
        Artisan::call('migrate:rollback', [
            '--path' => "vendor/bithoven/{$name}/database/migrations/{$migration}.php",
            '--force' => true
        ]);
    }
    
    Log::info("Migrations rolled back", [
        'extension' => $name,
        'count' => count($migrations)
    ]);
}
```

**⚠️ Warning:** Never rollback system tables!
```php
// Check system_tables before rollback
if (in_array($tableName, $extensionJson->system_tables ?? [])) {
    Log::warning("Skipping system table", ['table' => $tableName]);
    continue;
}
```

### 6. Composer Remove

**Step 6.1: Remove Package**
```php
$command = "composer remove bithoven/{$name} --no-scripts --no-plugins 2>&1";
exec($command, $output, $returnCode);

if ($returnCode !== 0) {
    Log::error("Composer remove failed", [
        'extension' => $name,
        'output' => $output
    ]);
}
```

**Step 6.2: Remove Repository from composer.json**
```php
$composerJson = json_decode(file_get_contents('composer.json'), true);

$composerJson['repositories'] = array_filter(
    $composerJson['repositories'] ?? [],
    function($repo) use ($name) {
        // Remove VCS repos
        if (str_contains($repo['url'], "bithoven-extension-{$name}")) {
            return false;
        }
        // Remove path repos
        if (str_contains($repo['url'], $name)) {
            return false;
        }
        return true;
    }
);

file_put_contents('composer.json', json_encode($composerJson, JSON_PRETTY_PRINT));
```

### 7. Remove from bithoven-extensions.php

```php
$config = config('bithoven-extensions');

$config['installed'] = array_filter($config['installed'], function($ext) use ($name) {
    return $ext !== $name;
});

file_put_contents(
    config_path('bithoven-extensions.php'),
    '<?php return ' . var_export($config, true) . ';'
);
```

### 8. Remove Extension Configuration

**Step 8.1: Remove from extension-settings.json**
```php
removeExtensionConfig($name);

Log::info("Extension config removed", ['extension' => $name]);
```

**File Update:**
```json
// Before
{
  "extensions": {
    "tickets": {...},
    "dummy": {...}
  }
}

// After
{
  "extensions": {
    "dummy": {...}
  }
}
```

### 9. Post-Uninstall Cleanup

**Step 9.1: Clear Caches**
```php
Artisan::call('cache:clear');
Artisan::call('view:clear');
Artisan::call('route:clear');
Artisan::call('config:clear');
```

**Step 9.2: Fire Event**
```php
event(new ExtensionUninstalled($name));

Log::info("Extension uninstalled successfully", ['extension' => $name]);
```

**Step 9.3: Activity Log**
```php
activity()
    ->causedBy(auth()->user())
    ->withProperties(['extension' => $name])
    ->log("Extension uninstalled: {$name}");
```

---

## 📊 Configuration File States

### Initial State (Fresh Install)

**composer.json:**
```json
{
  "repositories": [],
  "require": {}
}
```

**bithoven-extensions.php:**
```php
return [
  'installed' => [],
  'active' => [],
  'settings' => []
];
```

**extension-settings.json:**
```json
{
  "cache_ttl": 60,
  "extensions": {}
}
```

### After Installing Extension (VCS Mode)

**composer.json:**
```json
{
  "repositories": [
    {
      "type": "vcs",
      "url": "https://github.com/Madniatik/bithoven-extension-tickets.git"
    }
  ],
  "require": {
    "bithoven/tickets": "*"
  }
}
```

**bithoven-extensions.php:**
```php
return [
  'installed' => ['tickets'],
  'active' => ['tickets'],
  'settings' => []
];
```

**extension-settings.json:**
```json
{
  "cache_ttl": 60,
  "extensions": {
    "tickets": {
      "type": "vcs",
      "url": "https://github.com/Madniatik/bithoven-extension-tickets.git"
    }
  }
}
```

### After Installing Extension (Local Mode)

**composer.json:**
```json
{
  "repositories": [
    {
      "type": "path",
      "url": "../EXTENSIONS/bithoven-extension-tickets",
      "options": {
        "symlink": true
      }
    }
  ],
  "require": {
    "bithoven/tickets": "*"
  }
}
```

**bithoven-extensions.php:**
```php
return [
  'installed' => ['tickets'],
  'active' => ['tickets'],
  'settings' => []
];
```

**extension-settings.json:**
```json
{
  "cache_ttl": 60,
  "extensions": {
    "tickets": {
      "type": "local",
      "path": "../EXTENSIONS/bithoven-extension-tickets"
    }
  }
}
```

### Multiple Extensions (Mixed Mode)

**extension-settings.json:**
```json
{
  "cache_ttl": 60,
  "extensions": {
    "tickets": {
      "type": "local",
      "path": "../EXTENSIONS/bithoven-extension-tickets"
    },
    "dummy": {
      "type": "vcs",
      "url": "https://github.com/Madniatik/bithoven-extension-dummy.git"
    },
    "invoices": {
      "type": "vcs",
      "url": "https://github.com/Madniatik/bithoven-extension-invoices.git"
    }
  }
}
```

---

## 🔍 Troubleshooting

### Issue: Extension config not found after install

**Symptom:** `getExtensionConfig('tickets')` returns `null`

**Cause:** Config saved after composer operation (wrong order)

**Solution:** Save config BEFORE `composer require`
```php
// ✅ CORRECT ORDER
setExtensionConfig($name, $config);
$this->composer->require($name);

// ❌ WRONG ORDER
$this->composer->require($name);
setExtensionConfig($name, $config);  // Too late!
```

### Issue: Symlink not working

**Symptom:** Changes in local extension don't reflect

**Check:**
1. `composer.json` has `"symlink": true` in repository options
2. Run `ls -la vendor/bithoven/tickets` - should show `→` symbol
3. Check file permissions

**Fix:**
```bash
composer remove bithoven/tickets
rm -rf vendor/bithoven/tickets
composer require bithoven/tickets:*
```

### Issue: Migration already exists error

**Symptom:** `Base table or view already exists`

**Cause:** Migration ran but not in `migrations` table

**Fix:**
```php
// Check if migration already ran
$migrated = DB::table('migrations')
    ->where('migration', $migrationName)
    ->exists();

if ($migrated) {
    Log::info("Migration already ran", ['migration' => $migrationName]);
    continue;
}
```

### Issue: Repository not removed after uninstall

**Symptom:** `composer.json` still has repository entry

**Cause:** URL matching failed (different formats)

**Fix:** Match by extension name, not URL
```php
$composerJson['repositories'] = array_values(array_filter(
    $composerJson['repositories'] ?? [],
    function($repo) use ($name) {
        return !str_contains($repo['url'], "extension-{$name}");
    }
));
```

---

**Related Documentation:**
- [Extension JSON Schema](./EXTENSION-JSON-SCHEMA.md)
- [Architecture Change: Per-Extension Config](../ARCHITECTURE-CHANGE-PER-EXTENSION-CONFIG.md)
- [Extension Development Guide](../../.github/copilot-core/EXTENSION-DEVELOPMENT.md)
