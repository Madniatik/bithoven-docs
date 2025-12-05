# Settings System - Troubleshooting Guide

**Version:** 1.0.0  
**Last Updated:** December 5, 2025

---

## Common Issues & Solutions

### 🔴 Issue 1: Settings Not Saving to Database

**Symptoms:**
- Form submits successfully (200 OK)
- Success toast message appears
- Values don't persist in database
- Refreshing page shows old values

**Root Cause:**
This was caused by multiple issues:
1. Checkboxes not sending `false` when unchecked
2. Cache serving stale values
3. Type detection failure for form data

**Solution Implemented:**

1. **Hidden Inputs for Checkboxes:**
```blade
<!-- BEFORE (broken) -->
<input type="checkbox" name="settings[group][key]" value="1" />

<!-- AFTER (working) -->
<input type="hidden" name="settings[group][key]" value="0" />
<input type="checkbox" name="settings[group][key]" value="1" />
```

2. **Enhanced Cache Clearing:**
```php
// In Setting::set()
Cache::forget("setting.{$group}.{$key}");      // Individual setting
Cache::forget("settings.group.{$group}");      // Entire group
```

3. **Type Detection from DB:**
```php
// Controller checks existing setting type in database
$existingSetting = Setting::where('group', $group)->where('key', $key)->first();

if ($existingSetting && $existingSetting->type === 'boolean') {
    $convertedValue = $value === '1' || $value === 1 || $value === true;
    $type = 'boolean';
}
```

**Manual Fix (if still broken):**
```bash
# Clear all caches
php artisan cache:clear
php artisan view:clear
php artisan config:clear

# Check database directly
mysql -u root -p'password' bithoven_laravel -e "SELECT * FROM settings WHERE \`group\` = 'app'"
```

---

### 🔴 Issue 2: UI Not Reflecting Saved Values

**Symptoms:**
- Settings saved correctly in database (verified with SQL)
- UI shows old/default values after page refresh
- Helper function `setting()` returns cached old values

**Root Cause:**
Laravel cache serving stale values. The `Setting::getGroup()` method caches results for 1 hour, and cache wasn't being invalidated properly on update.

**Solution Implemented:**

```php
// app/Models/Setting.php - set() method
public static function set(string $group, string $key, $value, string $type = 'string'): bool
{
    $stringValue = static::valueToString($value, $type);
    
    $updated = static::updateOrCreate(
        ['group' => $group, 'key' => $key],
        ['value' => $stringValue, 'type' => $type]
    );
    
    // Clear BOTH individual and group cache
    Cache::forget("setting.{$group}.{$key}");
    Cache::forget("settings.group.{$group}");  // ← This was missing!
    
    return $updated->wasRecentlyCreated || $updated->wasChanged();
}
```

**Verification:**
```bash
# Test cache clearing works
cd /path/to/project
php -r "
require __DIR__.'/vendor/autoload.php';
\$app = require_once __DIR__.'/bootstrap/app.php';
\$kernel = \$app->make(Illuminate\Contracts\Console\Kernel::class);
\$kernel->bootstrap();

echo 'BEFORE: ' . App\Models\Setting::where('group', 'app')->where('key', 'name')->first()->value . PHP_EOL;
App\Models\Setting::set('app', 'name', 'Updated Test', 'string');
echo 'AFTER: ' . App\Models\Setting::where('group', 'app')->where('key', 'name')->first()->value . PHP_EOL;
"
```

---

### 🔴 Issue 3: Checkbox Always Checked/Unchecked

**Symptoms:**
- Checkbox toggle doesn't work
- Always shows as checked (or unchecked) regardless of database value
- Changing state and saving doesn't update

**Root Cause:**
HTML forms don't send anything for unchecked checkboxes. If you don't include a hidden input, the controller receives no data for that field when unchecked.

**Incorrect Implementation:**
```blade
<!-- BAD: Unchecking this sends NOTHING to server -->
<input type="checkbox" name="settings[debug][enabled]" value="1" {{ $enabled ? 'checked' : '' }} />
```

**Correct Implementation:**
```blade
<!-- GOOD: Hidden input sends "0" when checkbox unchecked -->
<input type="hidden" name="settings[debug][enabled]" value="0" />
<input type="checkbox" name="settings[debug][enabled]" value="1" {{ $enabled ? 'checked' : '' }} />
```

**Why This Works:**
1. Checkbox unchecked → Hidden input sends `0` → Controller receives `"0"`
2. Checkbox checked → Checkbox overrides hidden input → Controller receives `"1"`
3. Controller converts `"0"` → `false`, `"1"` → `true`

**Files Fixed:**
- `resources/views/app/settings/pages/debug.blade.php` (3 checkboxes)
- `resources/views/app/settings/pages/security.blade.php` (2 checkboxes)
- `resources/views/app/settings/pages/email.blade.php` (2 checkboxes)
- `resources/views/app/settings/pages/storage.blade.php` (1 checkbox)
- `resources/views/app/settings/pages/advanced.blade.php` (1 checkbox)

---

### 🔴 Issue 4: Success Message Shows "0 settings updated"

**Symptoms:**
- Form submits successfully
- Message: "0 settings updated successfully."
- But settings ARE saved in database

**Root Cause:**
The `Setting::set()` method returns `false` when values haven't changed:
```php
return $updated->wasRecentlyCreated || $updated->wasChanged();
```

If you submit the form with the same values, no database change occurs, so `wasChanged()` = `false`.

**Solution Implemented:**

Changed success message to distinguish between actual changes vs. no changes:

```php
// app/Http/Controllers/Settings/SettingsController.php
$message = $saved > 0 
    ? "{$saved} setting(s) updated successfully." 
    : "Settings saved successfully (no changes detected).";

return redirect()->back()->with('success', $message);
```

**Behavior:**
- Changed values: "5 setting(s) updated successfully."
- No changes: "Settings saved successfully (no changes detected)."

---

### 🔴 Issue 5: Type Casting Errors

**Symptoms:**
- Boolean setting saved as string `"true"` or `"1"`
- Retrieved as string instead of boolean
- Conditional checks fail: `if (setting('debug.enabled'))` always truthy

**Root Cause:**
Form data is always strings. `is_bool("1")` returns `false`, not `true`.

**Solution Implemented:**

Controller detects type from existing DB record:

```php
$existingSetting = Setting::where('group', $group)->where('key', $key)->first();

if ($existingSetting && $existingSetting->type === 'boolean') {
    // Convert string "0"/"1" to boolean false/true
    $convertedValue = $value === '1' || $value === 1 || $value === true;
    $type = 'boolean';
} else {
    // Auto-detect for new settings
    $type = match (true) {
        is_bool($value) => 'boolean',
        is_numeric($value) => 'integer',
        is_array($value) => 'json',
        default => 'string',
    };
}
```

**Verification:**
```php
// Check types in database
Setting::where('type', 'boolean')->get(['group', 'key', 'value', 'type']);

// Verify casting works
$enabled = setting('debug_console.enabled'); // Should be TRUE/FALSE, not "true"/"1"
var_dump($enabled); // bool(true) or bool(false)
```

---

### 🔴 Issue 6: Validation Errors on Submit

**Symptoms:**
- Form rejects valid data
- Error: "The settings field is required"
- Or: "The settings.app.name field is required"

**Root Cause:**
Validation rules don't match form structure.

**Current Validation (working):**
```php
$validated = $request->validate([
    'settings' => 'required|array',
    'settings.*.*' => 'nullable',  // Allow any value in multi-dimensional array
]);
```

**If You Want Specific Validation:**
```php
$validated = $request->validate([
    'settings' => 'required|array',
    'settings.app.name' => 'required|string|max:255',
    'settings.app.timezone' => 'required|string|in:' . implode(',', timezone_identifiers_list()),
    'settings.auth.session_timeout' => 'required|integer|min:1|max:1440',
    // ... other specific rules
]);
```

---

### 🔴 Issue 7: Debug Console Not Updating

**Symptoms:**
- Changed `debug_console.enabled` to `false` in settings
- Debug Console still appears on page
- Or changed `level` but console shows old level

**Root Cause:**
1. Cache not cleared
2. Browser cached JavaScript
3. View not checking latest setting

**Solution:**

1. **Clear Laravel cache:**
```bash
php artisan cache:clear
```

2. **Hard refresh browser:**
- Chrome/Firefox: `Cmd + Shift + R` (Mac) or `Ctrl + Shift + R` (Windows)

3. **Verify blade logic:**
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

---

## Debugging Tools

### Check Actual Database Values

```bash
# Connect to MySQL
mysql -u root -p'password' bithoven_laravel

# View all settings
SELECT `group`, `key`, value, type FROM settings ORDER BY `group`, `key`;

# View specific group
SELECT `key`, value, type FROM settings WHERE `group` = 'app';

# Check if setting exists
SELECT * FROM settings WHERE `group` = 'debug_console' AND `key` = 'enabled';
```

### Check Cache

```bash
# Clear all caches
php artisan cache:clear

# Check if cache driver is working
php artisan tinker
>>> Cache::put('test', 'value', 60);
>>> Cache::get('test');
=> "value"
>>> Cache::forget('test');
>>> Cache::get('test');
=> null
```

### Test Setting Model Directly

```bash
php artisan tinker

# Get setting
>>> App\Models\Setting::get('app', 'name');
=> "Laravel"

# Set setting
>>> App\Models\Setting::set('app', 'name', 'Test App', 'string');
=> true

# Verify
>>> App\Models\Setting::get('app', 'name');
=> "Test App"

# Get group
>>> App\Models\Setting::getGroup('app');
=> [
     "name" => "Test App",
     "timezone" => "UTC",
     ...
   ]
```

### Check Logs

```bash
# Watch logs in real-time
tail -f storage/logs/laravel.log

# Filter for settings-related entries
tail -100 storage/logs/laravel.log | grep -i "setting"

# Check last 20 lines
tail -20 storage/logs/laravel.log
```

### Debug Controller

Add logging to see actual POST data:

```php
// app/Http/Controllers/Settings/SettingsController.php
public function update(Request $request)
{
    \Log::debug('Settings update request', [
        'raw_request' => $request->all(),
    ]);

    $validated = $request->validate([...]);
    
    \Log::debug('Validated settings', $validated);
    
    // ... rest of method
}
```

Then check `storage/logs/laravel.log` after submitting form.

---

## Prevention Checklist

Before deploying settings changes:

- [ ] All checkboxes have hidden inputs before them
- [ ] Setting::set() clears both individual and group cache
- [ ] Controller logs requests for debugging
- [ ] Validation rules match form structure
- [ ] Default values provided for all setting() calls
- [ ] Types specified correctly (boolean, integer, string, json)
- [ ] Seeder includes all new settings
- [ ] UI tests performed (toggle checkboxes, change values, verify persistence)
- [ ] Cache cleared after deployment
- [ ] Database backup created before major changes

---

## Emergency Recovery

If settings system is completely broken:

### 1. Reset Settings Table

```bash
# Backup current settings
php artisan db:seed --class=SettingsSeeder --no-interaction

# Or manually backup
mysqldump -u root -p'password' bithoven_laravel settings > settings_backup.sql

# Drop and recreate
php artisan migrate:fresh --path=database/migrations/2025_12_04_000001_create_settings_table.php

# Reseed
php artisan db:seed --class=SettingsSeeder
```

### 2. Clear All Caches

```bash
php artisan cache:clear
php artisan config:clear
php artisan view:clear
php artisan route:clear
php artisan optimize:clear
```

### 3. Restore from Backup

```bash
# Restore settings table only
mysql -u root -p'password' bithoven_laravel < settings_backup.sql
```

### 4. Manual Database Fix

```sql
-- Fix type for boolean settings
UPDATE settings 
SET type = 'boolean', value = CASE WHEN value IN ('1', 'true', 'yes') THEN 'true' ELSE 'false' END 
WHERE `key` IN ('enabled', 'two_factor_required', 'cors_enabled', 'email_enabled', 'database_enabled', 'auto_thumbnails', 'expire_on_close');

-- Fix type for integer settings
UPDATE settings 
SET type = 'integer' 
WHERE `key` IN ('session_timeout', 'password_min_length', 'max_login_attempts', 'rate_limit', 'max_upload_size', 'compression_quality', 'lifetime', 'default_ttl', 'max_files');
```

---

## Support

If you encounter issues not covered here:

1. Check main documentation: `/DOCS/CORE/System-Settings/README.md`
2. Review code: `app/Models/Setting.php`, `app/Http/Controllers/Settings/SettingsController.php`
3. Check logs: `storage/logs/laravel.log`
4. Verify database: `SELECT * FROM settings`
5. Clear all caches: `php artisan optimize:clear`

---

**Last Updated:** December 5, 2025
