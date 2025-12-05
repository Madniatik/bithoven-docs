# System Settings - Complete Documentation

**Version:** 1.0.0  
**Date:** December 5, 2025  
**Status:** Production Ready ✅

---

## 📖 Table of Contents

1. [Overview](#overview)
2. [Architecture](#architecture)
3. [Database Schema](#database-schema)
4. [Model & Helpers](#model--helpers)
5. [Controllers & Routes](#controllers--routes)
6. [UI Components](#ui-components)
7. [Usage Examples](#usage-examples)
8. [Troubleshooting](#troubleshooting)
9. [Best Practices](#best-practices)

---

## Overview

The System Settings module provides a centralized, database-driven configuration system for the Laravel application. It replaces hardcoded config values with dynamic settings that can be modified through a user-friendly web interface.

### Key Features

✅ **Categorized Settings** - 7 organized tabs (General, Debug, Security, Email, Storage, Advanced, API Keys)  
✅ **Type Safety** - Automatic type casting (string, integer, boolean, json)  
✅ **Caching** - 1-hour cache with automatic invalidation  
✅ **Global Helpers** - Simple `setting()` and `setting_set()` functions  
✅ **UI Components** - Professional Metronic-based forms with validation  
✅ **Debug Console Control** - Global + per-extension granular control  
✅ **Extensible** - Easy to add new settings and categories

---

## Architecture

### Component Overview

```
┌─────────────────────────────────────────────────────┐
│                  User Interface                     │
│  (7 Categorized Tabs: General, Debug, Security...) │
└─────────────────┬───────────────────────────────────┘
                  │
                  ▼
┌─────────────────────────────────────────────────────┐
│            SettingsController                       │
│  - general(), debug(), security(), email()...       │
│  - update() with validation & error handling        │
└─────────────────┬───────────────────────────────────┘
                  │
                  ▼
┌─────────────────────────────────────────────────────┐
│              Setting Model                          │
│  - get($group, $key, $default)                      │
│  - set($group, $key, $value, $type)                 │
│  - getGroup($group)                                 │
│  - Cache: 1 hour TTL, auto-invalidation             │
└─────────────────┬───────────────────────────────────┘
                  │
                  ▼
┌─────────────────────────────────────────────────────┐
│          Database: settings table                   │
│  - group, key, value, type, description             │
│  - Unique constraint: (group, key)                  │
└─────────────────────────────────────────────────────┘
```

### Request Flow

1. **User submits form** → `PUT /settings/update`
2. **Controller validates** → Multi-dimensional array `settings[group][key]`
3. **Type detection** → Check existing setting type in DB, convert value
4. **Model updates** → `Setting::set()` with `updateOrCreate()`
5. **Cache invalidated** → Individual setting + group cache cleared
6. **Redirect back** → Success message with change count

---

## Database Schema

### Migration: `2025_12_04_000001_create_settings_table.php`

```php
Schema::create('settings', function (Blueprint $table) {
    $table->id();
    $table->string('group', 100)->index();
    $table->string('key', 255);
    $table->text('value');
    $table->string('type', 50)->default('string'); // string|integer|boolean|json
    $table->text('description')->nullable();
    $table->boolean('is_public')->default(false);
    $table->timestamps();

    $table->unique(['group', 'key']);
});
```

### Seeded Settings (40+ default values)

**Groups:**
- `debug_console` - Global Debug Console control
- `app` - Application name, timezone, formats, language
- `debug` - Error display, detailed errors
- `auth` - 2FA, session timeout, password policy, login attempts
- `api` - Rate limiting, CORS settings
- `mail` - Driver, from address, from name
- `notifications` - Email/database notifications, admin email
- `storage` - Default disk, storage path
- `media` - Upload limits, allowed types, thumbnails, compression
- `session` - Driver, lifetime, expire on close
- `cache` - Driver, prefix, default TTL
- `logs` - Channel, level, max files

### Example Records

```sql
+---------------+---------------------+----------+---------+
| group         | key                 | value    | type    |
+---------------+---------------------+----------+---------+
| debug_console | enabled             | true     | boolean |
| debug_console | level               | debug    | string  |
| app           | name                | Laravel  | string  |
| app           | timezone            | UTC      | string  |
| auth          | two_factor_required | false    | boolean |
| auth          | session_timeout     | 120      | integer |
+---------------+---------------------+----------+---------+
```

---

## Model & Helpers

### Setting Model: `app/Models/Setting.php`

#### Static Methods

**`get(string $group, string $key, $default = null)`**
```php
// Get a setting value with caching (1 hour TTL)
$appName = Setting::get('app', 'name', 'Default App');
// Returns: string|int|bool|array (auto-casted to type)
```

**`set(string $group, string $key, $value, string $type = 'string'): bool`**
```php
// Set a setting value, clears cache
Setting::set('app', 'name', 'My Application', 'string');
// Returns: true if created/changed, false if no change
```

**`getGroup(string $group): array`**
```php
// Get all settings in a group as associative array
$appSettings = Setting::getGroup('app');
// Returns: ['name' => 'Laravel', 'timezone' => 'UTC', ...]
```

**`getPublic(): Collection`**
```php
// Get all public settings (is_public = true)
$publicSettings = Setting::getPublic();
```

#### Type Casting

Automatic type conversion based on `type` column:

| Type      | Storage  | Retrieved As         |
|-----------|----------|----------------------|
| `string`  | text     | `string`             |
| `integer` | text     | `int`                |
| `boolean` | text     | `bool` (true/false)  |
| `json`    | text     | `array` (decoded)    |

### Global Helpers: `app/helpers.php`

**`setting(string|null $key = null, $default = null)`**
```php
// Get single setting
setting('app.name', 'Default');              // "Laravel"

// Get all settings in a group
setting('app');                               // ['name' => '...', 'timezone' => '...']

// Get all settings (grouped)
setting();                                    // ['app' => [...], 'debug' => [...], ...]
```

**`setting_set(string $key, $value, string $type = 'string'): bool`**
```php
// Set a setting value (group.key notation)
setting_set('app.name', 'New Name', 'string');          // true
setting_set('debug_console.enabled', true, 'boolean');  // true
```

---

## Controllers & Routes

### SettingsController: `app/Http/Controllers/Settings/SettingsController.php`

#### Methods

**`general()`** - General application settings  
**`debug()`** - Debug Console & error reporting  
**`security()`** - Authentication, password policy, API security  
**`email()`** - Mail configuration & notifications  
**`storage()`** - Storage paths, media settings  
**`advanced()`** - Session, cache, logs  
**`apiKeys()`** - API key management (existing)  

**`update(Request $request)`** - Save all settings with validation

#### Update Method Logic

```php
public function update(Request $request)
{
    // 1. Validate multi-dimensional array
    $validated = $request->validate([
        'settings' => 'required|array',
        'settings.*.*' => 'nullable',
    ]);

    // 2. Loop through groups and keys
    foreach ($validated['settings'] as $group => $keys) {
        foreach ($keys as $key => $value) {
            // 3. Detect type from existing setting
            $existingSetting = Setting::where('group', $group)
                ->where('key', $key)->first();
            
            // 4. Convert value (especially checkboxes: '0'/'1' → false/true)
            if ($existingSetting && $existingSetting->type === 'boolean') {
                $convertedValue = $value === '1' || $value === 1 || $value === true;
                $type = 'boolean';
            } else {
                $type = match (true) {
                    is_bool($value) => 'boolean',
                    is_numeric($value) => 'integer',
                    is_array($value) => 'json',
                    default => 'string',
                };
            }
            
            // 5. Save with error handling
            try {
                Setting::set($group, $key, $convertedValue, $type);
                $saved++;
            } catch (\Exception $e) {
                $errors[] = "{$group}.{$key}: {$e->getMessage()}";
            }
        }
    }

    // 6. Return with informative message
    return redirect()->back()->with('success', 
        $saved > 0 
            ? "{$saved} setting(s) updated successfully." 
            : "Settings saved successfully (no changes detected)."
    );
}
```

### Routes: `routes/web.php`

```php
Route::middleware(['auth'])->prefix('settings')->group(function () {
    Route::get('/', fn() => redirect()->route('settings.general'));
    Route::get('/general', [SettingsController::class, 'general'])->name('settings.general');
    Route::get('/debug', [SettingsController::class, 'debug'])->name('settings.debug');
    Route::get('/security', [SettingsController::class, 'security'])->name('settings.security');
    Route::get('/email', [SettingsController::class, 'email'])->name('settings.email');
    Route::get('/storage', [SettingsController::class, 'storage'])->name('settings.storage');
    Route::get('/advanced', [SettingsController::class, 'advanced'])->name('settings.advanced');
    Route::get('/api-keys', [SettingsController::class, 'apiKeys'])->name('settings.api-keys');
    
    Route::put('/update', [SettingsController::class, 'update'])->name('settings.update');
});
```

---

## UI Components

### Layout Structure

All settings pages extend `app.settings.index`:

```blade
@extends('app.settings.index')

@section('settings-content')
    <div class="card">
        <div class="card-header">
            <h3>Category Name</h3>
        </div>

        <form method="POST" action="{{ route('settings.update') }}" class="form">
            @csrf
            @method('PUT')

            <div class="card-body">
                <!-- Input groups -->
            </div>

            <div class="card-footer d-flex justify-content-end py-6 px-9">
                <button type="reset" class="btn btn-light btn-active-light-primary me-2">Discard</button>
                <button type="submit" class="btn btn-primary">Save Changes</button>
            </div>
        </form>
    </div>
@endsection
```

### Form Input Patterns

**Text Input:**
```blade
<input type="text" 
       name="settings[app][name]" 
       class="form-control form-control-sm form-control-solid" 
       value="{{ $settings['name'] ?? config('app.name') }}" />
```

**Select Dropdown:**
```blade
<select name="settings[app][timezone]" 
        class="form-select form-select-solid form-select-sm" 
        data-control="select2">
    @foreach(timezone_identifiers_list() as $tz)
        <option value="{{ $tz }}" {{ ($settings['timezone'] ?? config('app.timezone')) === $tz ? 'selected' : '' }}>
            {{ $tz }}
        </option>
    @endforeach
</select>
```

**Checkbox (Critical Pattern!):**
```blade
<!-- ALWAYS include hidden input BEFORE checkbox -->
<input type="hidden" name="settings[debug_console][enabled]" value="0" />
<input class="form-check-input" 
       type="checkbox" 
       name="settings[debug_console][enabled]"
       value="1" 
       id="debug_console_enabled"
       {{ $settings['enabled'] ?? true ? 'checked' : '' }} />
```

**Why Hidden Input?**  
Unchecked checkboxes don't send any value in POST data. The hidden input ensures a `0` is sent when unchecked, which the controller converts to `false`.

**Number Input:**
```blade
<input type="number" 
       name="settings[auth][session_timeout]" 
       class="form-control form-control-lg form-control-solid" 
       value="{{ $settings['session_timeout'] ?? 120 }}" 
       min="1" max="1440" />
```

### Navigation Tabs

Located in `resources/views/app/settings/index.blade.php`:

```blade
<ul class="nav nav-tabs nav-line-tabs mb-5 fs-6">
    <li class="nav-item">
        <a class="nav-link {{ request()->routeIs('settings.general') ? 'active' : '' }}" 
           href="{{ route('settings.general') }}">General</a>
    </li>
    <!-- ... other tabs ... -->
</ul>

@yield('settings-content')
```

---

## Usage Examples

### Example 1: Get Application Name

```php
// In controller
$appName = setting('app.name', 'Default App');

// In view
{{ setting('app.name') }}

// Using model directly
$name = Setting::get('app', 'name', 'Default');
```

### Example 2: Update Debug Console Settings

```php
// Enable Debug Console globally
setting_set('debug_console.enabled', true, 'boolean');

// Set log level
setting_set('debug_console.level', 'info', 'string');

// Using model directly
Setting::set('debug_console', 'enabled', true, 'boolean');
Setting::set('debug_console', 'level', 'debug', 'string');
```

### Example 3: Get All App Settings

```php
// Get all app settings as array
$appSettings = setting('app');
// Returns: ['name' => 'Laravel', 'timezone' => 'UTC', ...]

// Or using model
$appSettings = Setting::getGroup('app');
```

### Example 4: Check Debug Console in Blade

```blade
@if(setting('debug_console.enabled', true) && setting('debug_console.level', 'debug') !== 'none')
    <script>
        window.DEBUG_CONSOLE_CONFIG = {
            enabled: true,
            level: '{{ setting('debug_console.level', 'debug') }}'
        };
    </script>
@endif
```

### Example 5: Conditional Feature Based on Setting

```php
// In controller
if (setting('auth.two_factor_required', false)) {
    // Require 2FA for login
    return redirect()->route('two-factor.challenge');
}

// Check max login attempts
$maxAttempts = setting('auth.max_login_attempts', 5);
if ($user->login_attempts >= $maxAttempts) {
    // Lock account
}
```

---

## Troubleshooting

### Issue 1: Settings Not Saving

**Symptoms:**
- Form submits successfully
- Success message appears
- Values don't persist in database or UI

**Causes:**
1. Cache not being cleared
2. Missing hidden inputs for checkboxes
3. Type detection failure

**Solution:**
```bash
# Clear cache manually
php artisan cache:clear
php artisan view:clear

# Check logs
tail -50 storage/logs/laravel.log | grep "Settings"
```

**Prevention:**
- Ensure `Setting::set()` clears both individual and group cache
- Always add hidden inputs before checkboxes
- Use correct type when calling `setting_set()`

### Issue 2: Checkbox Values Not Working

**Symptoms:**
- Unchecking checkbox doesn't save `false`
- Checkbox always remains checked after save

**Solution:**
Always include hidden input BEFORE checkbox:
```blade
<input type="hidden" name="settings[group][key]" value="0" />
<input type="checkbox" name="settings[group][key]" value="1" ... />
```

### Issue 3: Cache Not Invalidating

**Symptoms:**
- Changes saved to database
- UI shows old values
- `setting()` helper returns stale data

**Solution:**
The `Setting::set()` method now clears both caches:
```php
Cache::forget("setting.{$group}.{$key}");      // Individual
Cache::forget("settings.group.{$group}");      // Group
```

If still having issues:
```bash
php artisan cache:clear
```

### Issue 4: Type Casting Errors

**Symptoms:**
- Boolean setting saved as string "1" or "0"
- Integer setting retrieved as string

**Solution:**
Always specify correct type:
```php
// Correct
setting_set('debug_console.enabled', true, 'boolean');
setting_set('auth.session_timeout', 120, 'integer');

// Incorrect
setting_set('debug_console.enabled', 'true', 'string'); // ❌
```

The controller auto-detects type from existing settings in DB.

---

## Best Practices

### 1. Naming Conventions

**Groups:** Lowercase, singular, descriptive
```php
'app', 'debug', 'auth', 'mail', 'storage', 'cache', 'session'
```

**Keys:** Snake_case, descriptive
```php
'name', 'timezone', 'two_factor_required', 'session_timeout'
```

**Avoid:** CamelCase, PascalCase, hyphens in keys

### 2. Default Values

Always provide sensible defaults:
```php
setting('app.name', 'Laravel Application');
setting('auth.session_timeout', 120);
setting('debug_console.enabled', true);
```

### 3. Type Safety

Use explicit types when setting values:
```php
Setting::set('auth', 'two_factor_required', true, 'boolean');
Setting::set('auth', 'session_timeout', 120, 'integer');
Setting::set('app', 'name', 'My App', 'string');
```

### 4. Validation

Add specific validation in controller for critical settings:
```php
$request->validate([
    'settings.app.name' => 'required|string|max:255',
    'settings.app.timezone' => 'required|string|in:' . implode(',', timezone_identifiers_list()),
    'settings.auth.session_timeout' => 'required|integer|min:1|max:1440',
]);
```

### 5. Seeder Organization

Group related settings together in seeder:
```php
// Authentication settings
Setting::updateOrCreate(
    ['group' => 'auth', 'key' => 'two_factor_required'],
    ['value' => 'false', 'type' => 'boolean', 'description' => 'Require 2FA for all users']
);

Setting::updateOrCreate(
    ['group' => 'auth', 'key' => 'session_timeout'],
    ['value' => '120', 'type' => 'integer', 'description' => 'Session timeout in minutes']
);
```

### 6. Public Settings

Mark settings as `is_public = true` if they need to be accessed by frontend JavaScript:
```php
Setting::updateOrCreate(
    ['group' => 'app', 'key' => 'name'],
    ['value' => 'Laravel', 'type' => 'string', 'is_public' => true]
);
```

Then retrieve:
```php
$publicSettings = Setting::getPublic();
```

### 7. Cache Strategy

- **Read-heavy settings:** Use cache (default 1 hour)
- **Frequently changed:** Consider shorter TTL
- **Critical settings:** Clear cache after update (already implemented)

### 8. UI Organization

Group related settings in same tab:
- **General:** App identity, locale, formats
- **Debug:** Development tools, error reporting
- **Security:** Authentication, authorization, API
- **Email:** SMTP, notifications
- **Storage:** Disks, media processing
- **Advanced:** Session, cache, logs

---

## Debug Console Integration

The Settings system provides centralized control for the Debug Console feature.

### Global Control

```php
// Master switch (turns Debug Console on/off globally)
setting('debug_console.enabled', true);

// Core app log level
setting('debug_console.level', 'debug'); // 'none'|'debug'|'info'|'warn'|'error'
```

### Extension-Specific Control

Each extension has its own `debug_console.level` config:

```php
// llm-manager/config/llm-manager.php
'debug_console' => [
    'level' => env('LLM_MANAGER_DEBUG_LEVEL', 'debug'),
],
```

### Registration Logic

```blade
@if(setting('debug_console.enabled', true) && config('llm-manager.debug_console.level', 'none') !== 'none')
    <script>
        window.DEBUG_CONSOLE_CONFIG['llm-manager'] = {
            enabled: true,
            level: '{{ config('llm-manager.debug_console.level', 'debug') }}'
        };
    </script>
@endif
```

**Behavior:**
- If global `enabled = false` → No Debug Console at all
- If global `enabled = true` + extension `level = 'none'` → Extension hidden from Debug Console
- If global `enabled = true` + extension `level != 'none'` → Extension appears in Debug Console with specified level

---

## Migration Guide

### Adding New Setting

1. **Add to seeder** (`database/seeders/SettingsSeeder.php`):
```php
Setting::updateOrCreate(
    ['group' => 'my_group', 'key' => 'my_setting'],
    [
        'value' => 'default_value',
        'type' => 'string',
        'description' => 'Description of this setting',
        'is_public' => false,
    ]
);
```

2. **Run seeder**:
```bash
php artisan db:seed --class=SettingsSeeder
```

3. **Add to UI** (create/update view):
```blade
<input type="text" 
       name="settings[my_group][my_setting]" 
       value="{{ $settings['my_setting'] ?? 'default' }}" />
```

4. **Use in code**:
```php
$value = setting('my_group.my_setting', 'default');
```

### Creating New Settings Tab

1. **Create view** (`resources/views/app/settings/pages/my-category.blade.php`)
2. **Add controller method**:
```php
public function myCategory()
{
    $settings = Setting::getGroup('my_group');
    return view('app.settings.pages.my-category', compact('settings'));
}
```
3. **Add route**:
```php
Route::get('/my-category', [SettingsController::class, 'myCategory'])
    ->name('settings.my-category');
```
4. **Add tab to navigation** (`app/settings/index.blade.php`)

---

## API Reference

### Setting Model

| Method | Parameters | Returns | Description |
|--------|------------|---------|-------------|
| `get()` | `string $group, string $key, $default = null` | `mixed` | Get setting value (cached) |
| `set()` | `string $group, string $key, $value, string $type = 'string'` | `bool` | Set setting value, clear cache |
| `getGroup()` | `string $group` | `array` | Get all settings in group |
| `getPublic()` | - | `Collection` | Get all public settings |

### Global Helpers

| Function | Parameters | Returns | Description |
|----------|------------|---------|-------------|
| `setting()` | `string\|null $key = null, $default = null` | `mixed` | Get setting(s) |
| `setting_set()` | `string $key, $value, string $type = 'string'` | `bool` | Set setting value |

---

## Change Log

### Version 1.0.0 (December 5, 2025)

**Initial Release:**
- ✅ Settings table migration
- ✅ Setting model with caching
- ✅ Global helpers (`setting`, `setting_set`)
- ✅ 7 categorized settings tabs
- ✅ SettingsController with validation
- ✅ 40+ default settings seeded
- ✅ Debug Console global + per-extension control
- ✅ Checkbox handling with hidden inputs
- ✅ Cache invalidation on update
- ✅ Error handling and detailed logging
- ✅ Informative success/no-change messages

**Known Issues:**
- None (all bugs fixed during development)

---

## Support & Contribution

### Reporting Issues

If you encounter issues with the Settings system:

1. Check `storage/logs/laravel.log` for errors
2. Verify cache is cleared: `php artisan cache:clear`
3. Check database for actual saved values
4. Review this documentation for usage patterns

### Future Enhancements

Potential improvements:
- [ ] Setting history/audit log
- [ ] Import/export settings as JSON
- [ ] Settings validation rules in database
- [ ] Role-based setting permissions
- [ ] API endpoints for settings
- [ ] Settings search functionality
- [ ] Settings diff/comparison tool

---

**Documentation Last Updated:** December 5, 2025  
**Maintained By:** Bithoven Development Team
