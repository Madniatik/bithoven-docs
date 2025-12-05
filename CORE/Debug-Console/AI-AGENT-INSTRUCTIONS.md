# 🤖 AI Agent Instructions - Debug Console Integration

**Complete guide for integrating Debug Console into BITHOVEN extensions**

---

## 📖 Table of Contents

1. [Integration Overview](#-integration-overview)
2. [Integration Steps](#-integration-steps)
3. [File Templates](#-file-templates)
4. [Validation Checklist](#-validation-checklist)
5. [Common Patterns](#-common-patterns)
6. [Troubleshooting](#-troubleshooting)

---

## 🎯 Integration Overview

Debug Console integration requires **3 files** in your extension:

1. **Config file** (`config/{extension}.php`) - Add `debug_console` section
2. **Registration partial** (`resources/views/partials/debug-console-registration.blade.php`) - Register with @push
3. **Service Provider** (`src/{Extension}ServiceProvider.php`) - Add View Composer

**Time:** ~5 minutes  
**Complexity:** Low (copy-paste pattern)

---

## 🚀 Integration Steps

### Step 1: Add Config Section

**File:** `config/{extension-name}.php`

Add `debug_console` section to your extension config:

```php
<?php

return [
    
    // ... existing config ...
    
    /*
    |--------------------------------------------------------------------------
    | Debug Console
    |--------------------------------------------------------------------------
    |
    | Enable/disable Debug Console for this extension.
    | - enabled: Controls whether logs appear in browser console
    | - level: Minimum log level to display (debug|info|warn|error)
    |
    */
    
    'debug_console' => [
        'enabled' => env('{EXTENSION}_DEBUG_CONSOLE', false),
        'level' => env('{EXTENSION}_DEBUG_LEVEL', 'info'), // debug|info|warn|error
    ],
    
];
```

**Replace `{EXTENSION}` with your extension prefix:**
- LLM Manager: `LLM_DEBUG_CONSOLE`, `LLM_DEBUG_LEVEL`
- Tickets: `TICKETS_DEBUG_CONSOLE`, `TICKETS_DEBUG_LEVEL`
- Dummy: `DUMMY_DEBUG_CONSOLE`, `DUMMY_DEBUG_LEVEL`

**Environment Variables (.env):**
```env
LLM_DEBUG_CONSOLE=true
LLM_DEBUG_LEVEL=debug
```

---

### Step 2: Create Registration Partial

**File:** `resources/views/partials/debug-console-registration.blade.php`

```blade
{{--
    Debug Console Registration - {Extension Name}
    
    Se inyecta automáticamente via View Composer en TODAS las páginas
    Registra la extensión en el sistema de Debug Console global
    
    @see CPANEL/resources/views/partials/debug-console-init.blade.php
    @see CPANEL/resources/views/layouts/master.blade.php (@stack('debug-console-extensions'))
--}}

@push('debug-console-extensions')
<script>
    // Auto-registro de {Extension Name} en Debug Console
    window.DEBUG_CONSOLE_CONFIG = window.DEBUG_CONSOLE_CONFIG || {};
    window.DEBUG_CONSOLE_CONFIG['{extension-slug}'] = {
        enabled: {{ config('{extension-name}.debug_console.enabled', false) ? 'true' : 'false' }},
        level: '{{ config('{extension-name}.debug_console.level', 'info') }}'
    };
    
    // Crear logger para la extensión (cuando DebugConsole esté disponible)
    document.addEventListener('DOMContentLoaded', function() {
        if (window.DebugConsole) {
            window.{Logger}Logger = window.DebugConsole.create('{extension-slug}');
            
            @if(config('{extension-name}.debug_console.enabled', false))
                {Logger}Logger.info('{Extension Name} Debug Console registered');
            @endif
        }
    });
</script>
@endpush
```

**Replacements:**

| Placeholder | Example (LLM Manager) | Example (Tickets) |
|-------------|----------------------|-------------------|
| `{Extension Name}` | LLM Manager | Tickets System |
| `{extension-slug}` | llm-manager | tickets |
| `{extension-name}` | llm-manager | tickets |
| `{Logger}` | Monitor | Tickets |

**Result:** Creates global `window.MonitorLogger` or `window.TicketsLogger`

---

### Step 3: Add View Composer to Service Provider

**File:** `src/{Extension}ServiceProvider.php`

Add this **after** component registration in `register()` or `boot()` method:

```php
<?php

namespace Bithoven\{Extension};

use Illuminate\Support\ServiceProvider;
use Illuminate\Support\Facades\View;

class {Extension}ServiceProvider extends ServiceProvider
{
    public function register()
    {
        // ... existing registrations ...
        
        // Register Debug Console for extension (global via view share)
        View::composer('*', function ($view) {
            static $registered = false;
            if (!$registered) {
                $registered = true;
                $view->with('__{extension}DebugConsoleRegistration', view('{extension}::partials.debug-console-registration')->render());
            }
        });
    }
}
```

**Replacements:**

| Placeholder | Example (LLM Manager) | Example (Tickets) |
|-------------|----------------------|-------------------|
| `{Extension}` | LLMManager | Tickets |
| `{extension}` | llm | tickets |

**Example (LLM Manager):**
```php
View::composer('*', function ($view) {
    static $registered = false;
    if (!$registered) {
        $registered = true;
        $view->with('__llmDebugConsoleRegistration', view('llm-manager::partials.debug-console-registration')->render());
    }
});
```

**Why static flag?**
- View Composer runs on EVERY view render
- Static flag ensures registration runs ONCE per request
- Prevents duplicate @push calls

---

### Step 4: Use Logger in JavaScript

**In your extension's JavaScript files:**

```javascript
// Available globally after registration
if (window.{Logger}Logger) {
    {Logger}Logger.info('Extension feature initialized');
    {Logger}Logger.debug('Configuration:', config);
    {Logger}Logger.warn('Deprecated method used');
    {Logger}Logger.error('Operation failed:', error);
}
```

**Example (LLM Manager):**
```javascript
// In monitor-api.blade.php or any JS
if (window.MonitorLogger) {
    MonitorLogger.info('Chat session started');
    MonitorLogger.debug('Message sent:', message);
}
```

**Example (Tickets):**
```javascript
// In tickets.js
if (window.TicketsLogger) {
    TicketsLogger.info('Ticket created:', ticketId);
    TicketsLogger.debug('Form data:', formData);
}
```

---

## 📄 File Templates

### Template: Config File Addition

```php
// Add to config/{extension}.php

'debug_console' => [
    'enabled' => env('{EXTENSION}_DEBUG_CONSOLE', false),
    'level' => env('{EXTENSION}_DEBUG_LEVEL', 'info'),
],
```

---

### Template: Registration Partial (Complete)

```blade
{{-- resources/views/partials/debug-console-registration.blade.php --}}

@push('debug-console-extensions')
<script>
    window.DEBUG_CONSOLE_CONFIG = window.DEBUG_CONSOLE_CONFIG || {};
    window.DEBUG_CONSOLE_CONFIG['{slug}'] = {
        enabled: {{ config('{name}.debug_console.enabled', false) ? 'true' : 'false' }},
        level: '{{ config('{name}.debug_console.level', 'info') }}'
    };
    
    document.addEventListener('DOMContentLoaded', function() {
        if (window.DebugConsole) {
            window.{Logger}Logger = window.DebugConsole.create('{slug}');
            
            @if(config('{name}.debug_console.enabled', false))
                {Logger}Logger.info('{Title} Debug Console registered');
            @endif
        }
    });
</script>
@endpush
```

---

### Template: View Composer (Complete)

```php
// In {Extension}ServiceProvider.php register() or boot()

use Illuminate\Support\Facades\View;

View::composer('*', function ($view) {
    static $registered = false;
    if (!$registered) {
        $registered = true;
        $view->with('__{prefix}DebugConsoleRegistration', view('{namespace}::partials.debug-console-registration')->render());
    }
});
```

---

## ✅ Validation Checklist

After integration, verify:

### 1. Config Added
```bash
# Check config file has debug_console section
grep -A 5 "debug_console" config/{extension}.php
```

**Expected:**
```php
'debug_console' => [
    'enabled' => env('...', false),
    'level' => env('...', 'info'),
],
```

---

### 2. Registration Partial Created
```bash
# Check partial exists
ls -la resources/views/partials/debug-console-registration.blade.php
```

**Expected:**
```blade
@push('debug-console-extensions')
<script>
    window.DEBUG_CONSOLE_CONFIG['{extension}'] = {...};
    window.{Logger}Logger = ...
</script>
@endpush
```

---

### 3. View Composer Added
```bash
# Check Service Provider has View::composer
grep -A 8 "View::composer" src/*ServiceProvider.php
```

**Expected:**
```php
View::composer('*', function ($view) {
    static $registered = false;
    if (!$registered) {
        $registered = true;
        $view->with('__debugRegistration', ...);
    }
});
```

---

### 4. Environment Variables Set
```bash
# Check .env has debug console vars
grep "DEBUG_CONSOLE" .env
```

**Expected:**
```env
{EXTENSION}_DEBUG_CONSOLE=true
{EXTENSION}_DEBUG_LEVEL=debug
```

---

### 5. Browser Verification

**Open any page** in browser, check console:

```javascript
// 1. Check logger exists
console.log(window.{Logger}Logger);  // Should be object

// 2. Check configuration
window.DebugConsole.getConfig();
// Should show: {extension}: {enabled: true, level: 'debug'}

// 3. Test logging
window.{Logger}Logger.info('Test message');
// Should output: [{extension}] [INFO] HH:MM:SS Test message
```

---

## 🔧 Common Patterns

### Pattern 1: Conditional Logging

```javascript
// Only log if debug mode
if (window.MonitorLogger) {
    MonitorLogger.debug('Detailed info:', data);
}

// Always try to log (safe if undefined)
window.MonitorLogger?.info('Something happened');
```

---

### Pattern 2: Grouped Operations

```javascript
if (window.TicketsLogger) {
    TicketsLogger.group('Processing 50 tickets');
    
    tickets.forEach(ticket => {
        TicketsLogger.debug('Processing ticket:', ticket.id);
        processTicket(ticket);
    });
    
    TicketsLogger.groupEnd();
    TicketsLogger.info('All tickets processed');
}
```

---

### Pattern 3: Performance Monitoring

```javascript
if (window.MonitorLogger) {
    MonitorLogger.time('API Request');
    
    try {
        const response = await fetch('/api/chat');
        MonitorLogger.timeEnd('API Request');
        MonitorLogger.info('Request successful');
    } catch (error) {
        MonitorLogger.timeEnd('API Request');
        MonitorLogger.error('Request failed:', error);
    }
}
```

---

### Pattern 4: Table Display

```javascript
if (window.TicketsLogger) {
    const tickets = await fetchTickets();
    
    TicketsLogger.info(`Fetched ${tickets.length} tickets`);
    TicketsLogger.table(tickets.map(t => ({
        id: t.id,
        title: t.title,
        status: t.status
    })));
}
```

---

## 🐛 Troubleshooting

### Issue: Logger Not Available

**Symptom:**
```javascript
window.{Logger}Logger is undefined
```

**Checklist:**

1. ✅ Config file has `debug_console` section?
2. ✅ Registration partial created?
3. ✅ View Composer added to Service Provider?
4. ✅ Composer updated: `composer update bithoven/{extension}`?
5. ✅ Views cleared: `php artisan view:clear`?
6. ✅ Browser console shows registration log?

**Debug:**
```javascript
// Check DebugConsole exists
console.log(window.DebugConsole);  // Should be object

// Check config has your extension
window.DebugConsole.getConfig();
// Should include: {extension}: {enabled: ..., level: ...}

// Check owners list
window.DebugConsole.getOwners();
// Should include: '{extension}'
```

---

### Issue: Logs Not Showing

**Symptom:** Logger exists, but logs don't appear

**Solutions:**

1. **Check enabled:**
   ```javascript
   window.DebugConsole.getConfig()['{extension}'];
   // {enabled: false}  ← Should be true
   ```
   
   **Fix:** Set `{EXTENSION}_DEBUG_CONSOLE=true` in .env

2. **Check level:**
   ```javascript
   window.DebugConsole.getConfig()['{extension}'];
   // {enabled: true, level: 'error'}  ← Too high for .debug() or .info()
   ```
   
   **Fix:** Set `{EXTENSION}_DEBUG_LEVEL=debug` in .env

3. **Check browser console filter:**
   - Open DevTools → Console tab
   - Check filter settings (should show all levels)
   - Try searching for `[{extension}]` in console

---

### Issue: Duplicate Registration

**Symptom:** Multiple registration logs in console

**Cause:** Missing `static $registered` flag in View Composer

**Fix:**
```php
// ❌ WRONG - Runs on every view
View::composer('*', function ($view) {
    $view->with('__debugRegistration', ...);
});

// ✅ CORRECT - Runs once per request
View::composer('*', function ($view) {
    static $registered = false;  // ← ADD THIS
    if (!$registered) {
        $registered = true;
        $view->with('__debugRegistration', ...);
    }
});
```

---

### Issue: Extension Not in Config

**Symptom:**
```javascript
window.DebugConsole.getOwners();
// ['app']  ← Missing your extension
```

**Checklist:**

1. ✅ Registration partial uses `@push('debug-console-extensions')`?
2. ✅ `master.blade.php` has `@stack('debug-console-extensions')`?
3. ✅ View Composer runs on `'*'` (all views)?
4. ✅ Composer updated and views cleared?

**Debug:**
```bash
# Check master.blade.php has stack
grep "debug-console-extensions" resources/views/layouts/master.blade.php

# Check registration partial has @push
grep "@push" vendor/bithoven/{extension}/resources/views/partials/debug-console-registration.blade.php

# Update and clear
composer update bithoven/{extension}
php artisan view:clear
```

---

## 📖 Related Documentation

- **[README.md](./README.md)** - Main documentation, API overview
- **[API-REFERENCE.md](./API-REFERENCE.md)** - Complete API specifications
- **[EXAMPLES.md](./EXAMPLES.md)** - Real-world integration examples
- **[CONFIGURATION.md](./CONFIGURATION.md)** - Environment variables, deployment

---

**Last Updated:** 5 de diciembre de 2025  
**Version:** 1.0.0  
**For:** AI Agents (Claude, GPT, etc.) + Human Developers
