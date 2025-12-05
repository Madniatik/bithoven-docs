# 🐞 Debug Console System

**Centralized logging system for BITHOVEN applications and extensions**

---

## 📖 Table of Contents

1. [Overview](#-overview)
2. [Quick Start](#-quick-start)
3. [JavaScript API](#-javascript-api)
4. [Configuration](#%EF%B8%8F-configuration)
5. [Architecture](#-architecture)
6. [Integration Guide](#-integration-guide)
7. [Best Practices](#-best-practices)
8. [Troubleshooting](#-troubleshooting)

---

## 🎯 Overview

Debug Console is a **vanilla-friendly logging system** that provides:

- **Owner Isolation** - Each app/extension has independent logger
- **Level Filtering** - debug, info, warn, error with configurable thresholds
- **Global Availability** - Works on ALL pages with zero configuration
- **Zero Overhead** - Completely disabled when `enabled: false`
- **Timestamps** - Automatic timestamp + owner prefix on every log

**Philosophy:** Load once, use everywhere (vanilla JavaScript pattern in Laravel)

---

## 🚀 Quick Start

### Using Existing Loggers

```javascript
// Core App Logger (always available)
window.AppLogger.info('Application started');
window.AppLogger.debug('Configuration loaded');
window.AppLogger.warn('Cache miss');
window.AppLogger.error('Fatal error occurred');

// Extension Loggers (if registered)
window.MonitorLogger.info('LLM Manager operation');
window.TicketsLogger.debug('Ticket created');
```

### Check Configuration

```javascript
// See all registered owners
window.DebugConsole.getConfig();
// {
//   app: {enabled: true, level: 'debug'},
//   'llm-manager': {enabled: true, level: 'info'}
// }

// List owners
window.DebugConsole.getOwners();
// ['app', 'llm-manager']
```

### Runtime Registration (Advanced)

```javascript
// Register new owner manually
window.DebugConsole.register('my-module', {
    enabled: true,
    level: 'debug'
});

// Create logger
const MyLogger = window.DebugConsole.create('my-module');
MyLogger.info('My module initialized');
```

---

## 📚 JavaScript API

### Factory Methods

#### `DebugConsole.create(owner)`
Create logger for specific owner.

```javascript
const MyLogger = window.DebugConsole.create('app');
```

**Parameters:**
- `owner` (string) - Owner identifier (e.g., 'app', 'llm-manager')

**Returns:** `ConsoleLogger` instance

---

#### `DebugConsole.register(owner, config)`
Register new owner with configuration.

```javascript
window.DebugConsole.register('tickets', {
    enabled: true,
    level: 'info'
});
```

**Parameters:**
- `owner` (string) - Owner identifier
- `config` (object) - `{enabled: boolean, level: string}`

---

#### `DebugConsole.getConfig()`
Get current configuration for all owners.

```javascript
const config = window.DebugConsole.getConfig();
// {app: {...}, 'llm-manager': {...}}
```

**Returns:** Object with all owner configurations

---

#### `DebugConsole.getOwners()`
List all registered owners.

```javascript
const owners = window.DebugConsole.getOwners();
// ['app', 'llm-manager', 'tickets']
```

**Returns:** Array of owner names

---

### Logger Methods

All logger instances (`AppLogger`, `MonitorLogger`, etc.) have these methods:

#### `logger.debug(...args)`
Log debug information (development only).

```javascript
AppLogger.debug('User data:', userData);
AppLogger.debug('Processing step 1');
```

**Shown when:** `level = 'debug'`

---

#### `logger.info(...args)`
Log informational messages.

```javascript
AppLogger.info('User logged in');
MonitorLogger.info('Chat session started');
```

**Shown when:** `level = 'debug' | 'info'`

---

#### `logger.warn(...args)`
Log warnings.

```javascript
AppLogger.warn('Deprecated method used');
MonitorLogger.warn('API rate limit approaching');
```

**Shown when:** `level = 'debug' | 'info' | 'warn'`

---

#### `logger.error(...args)`
Log errors (critical).

```javascript
AppLogger.error('Database connection failed');
MonitorLogger.error('API request failed:', error);
```

**Shown when:** `level = 'debug' | 'info' | 'warn' | 'error'` (always if enabled)

---

#### `logger.group(label)`
Start collapsible log group.

```javascript
MonitorLogger.group('Processing 100 items');
for (let i = 0; i < 100; i++) {
    MonitorLogger.debug(`Item ${i} processed`);
}
MonitorLogger.groupEnd();
```

---

#### `logger.groupEnd()`
End collapsible log group.

---

#### `logger.table(data)`
Display data in table format.

```javascript
const users = [
    {id: 1, name: 'John', role: 'Admin'},
    {id: 2, name: 'Jane', role: 'User'}
];
AppLogger.table(users);
```

---

#### `logger.time(label)`
Start performance timer.

```javascript
MonitorLogger.time('API Request');
await fetch('/api/data');
MonitorLogger.timeEnd('API Request');
// Output: llm-manager API Request: 234.56ms
```

---

#### `logger.timeEnd(label)`
Stop performance timer and log duration.

---

## ⚙️ Configuration

### Core App Configuration

**File:** `config/app.php`

```php
return [
    'debug' => env('APP_DEBUG', false),  // Controls Debug Console
    'env' => env('APP_ENV', 'production'),  // Controls level
];
```

**Logic:**
- `enabled`: `APP_DEBUG = true` → enabled
- `level`: `APP_ENV = local` → 'debug', otherwise 'error'

---

### Extension Configuration

**File:** `config/{extension-name}.php`

```php
return [
    'debug_console' => [
        'enabled' => env('LLM_DEBUG_CONSOLE', false),
        'level' => env('LLM_DEBUG_LEVEL', 'info'), // debug|info|warn|error
    ],
];
```

**Environment Variables:**

```env
# Core App
APP_DEBUG=true
APP_ENV=local

# LLM Manager Extension
LLM_DEBUG_CONSOLE=true
LLM_DEBUG_LEVEL=debug

# Tickets Extension
TICKETS_DEBUG_CONSOLE=true
TICKETS_DEBUG_LEVEL=info
```

---

### Log Levels

Levels in order (lowest to highest):

1. **debug** - Verbose (development only)
   - Shows: debug, info, warn, error
   - Use: Step-by-step tracing, variable inspection

2. **info** - Informational (default)
   - Shows: info, warn, error
   - Use: Normal operation tracking

3. **warn** - Warnings
   - Shows: warn, error
   - Use: Deprecation notices, non-critical issues

4. **error** - Errors only
   - Shows: error
   - Use: Production (critical issues only)

**Example:**

```javascript
// Config: {enabled: true, level: 'info'}

logger.debug('Step 1');   // ❌ Hidden (below info)
logger.info('Started');   // ✅ Shown
logger.warn('Slow');      // ✅ Shown
logger.error('Failed');   // ✅ Shown
```

---

## 🏗️ Architecture

### Global Loading Pattern

**1. DebugConsole.js loaded globally** (`config/settings.php`)
```php
'KT_THEME_ASSETS' => [
    'global' => [
        'js' => [
            'assets/metronic/plugins/global/plugins.bundle.js',
            'assets/metronic/js/scripts.bundle.js',
            'assets/js/custom/DebugConsole.js',  // ← Loads ONCE on ALL pages
        ],
    ],
],
```

**2. Core App Registration** (`master.blade.php`)
```blade
@include('partials.debug-console-init')        {{-- Core app --}}
@stack('debug-console-extensions')             {{-- Extensions --}}
```

**3. Extension Auto-Registration** (Service Provider + View Composer)
```php
View::composer('*', function ($view) {
    static $registered = false;
    if (!$registered) {
        $registered = true;
        $view->with('__debugConsoleRegistration', view('...')->render());
    }
});
```

**Result:** DebugConsole available on EVERY page, loggers created once via DOMContentLoaded.

---

### File Structure

```
CPANEL/
├── config/settings.php                         # DebugConsole.js in global assets
├── resources/
│   ├── js/custom/DebugConsole.js              # Core class (factory + logger)
│   └── views/
│       ├── layouts/master.blade.php           # @stack('debug-console-extensions')
│       └── partials/
│           └── debug-console-init.blade.php   # Core app registration
└── public/assets/js/custom/DebugConsole.js    # Compiled (auto-generated)

EXTENSIONS/{extension}/
├── config/{extension}.php                      # debug_console config
├── src/{Extension}ServiceProvider.php          # View Composer registration
└── resources/views/partials/
    └── debug-console-registration.blade.php   # @push to stack
```

---

## 🔌 Integration Guide

See **[AI-AGENT-INSTRUCTIONS.md](./AI-AGENT-INSTRUCTIONS.md)** for complete extension integration guide.

**Quick Summary:**

1. **Add config** to `config/{extension}.php`
2. **Create registration partial** with @push
3. **Add View Composer** in Service Provider
4. **Use logger** in JavaScript

**Time:** ~5 minutes per extension

---

## ✅ Best Practices

### 1. Use Appropriate Levels

```javascript
// ✅ GOOD
logger.debug('Processing item:', item);       // Development details
logger.info('User logged in');                // Normal operations
logger.warn('Cache miss, using fallback');    // Non-critical issues
logger.error('Database connection failed');   // Critical failures

// ❌ BAD
logger.info('Variable x = 123');              // Use .debug()
logger.error('User clicked button');          // Use .info()
```

---

### 2. Group Related Logs

```javascript
// ✅ GOOD - Collapsible
logger.group('Processing 100 users');
users.forEach(user => logger.debug('Processing:', user.name));
logger.groupEnd();

// ❌ BAD - Cluttered console
users.forEach(user => logger.debug('Processing:', user.name));
```

---

### 3. Use Timers for Performance

```javascript
// ✅ GOOD
logger.time('Fetch Users');
await fetchUsers();
logger.timeEnd('Fetch Users');  // Output: llm-manager Fetch Users: 234ms

// ❌ BAD
const start = Date.now();
await fetchUsers();
logger.info('Took:', Date.now() - start);
```

---

### 4. Table for Structured Data

```javascript
// ✅ GOOD
logger.table(users);  // Beautiful table in console

// ❌ BAD
logger.info('Users:', users);  // Hard to read nested objects
```

---

### 5. Production Safety

```php
// ✅ GOOD - Use env() with defaults
'enabled' => env('LLM_DEBUG_CONSOLE', false),  // Default OFF

// ❌ BAD - Always on
'enabled' => true,
```

**Production:** ALL loggers disabled (early return, zero overhead)

---

## 🐛 Troubleshooting

### Logger Not Available

**Symptom:**
```javascript
window.AppLogger is undefined
```

**Solutions:**

1. **Check DebugConsole.js loaded:**
   ```javascript
   console.log(window.DebugConsole);  // Should be object
   ```

2. **Check settings.php:**
   ```php
   // config/settings.php
   'global' => [
       'js' => [
           'assets/js/custom/DebugConsole.js',  // ← Present?
       ],
   ],
   ```

3. **Clear cache:**
   ```bash
   php artisan optimize:clear
   ```

4. **Check browser console** for JavaScript errors

---

### Logs Not Showing

**Symptom:** Logger exists, but logs don't appear

**Solutions:**

1. **Check enabled:**
   ```javascript
   window.DebugConsole.getConfig();
   // {app: {enabled: false, ...}}  ← Should be true
   ```

2. **Check level:**
   ```javascript
   // If level = 'info', .debug() won't show
   window.DebugConsole.getConfig();
   // {app: {enabled: true, level: 'error'}}  ← Too high
   ```

3. **Check environment:**
   ```bash
   # .env
   APP_DEBUG=true          # ← Must be true
   LLM_DEBUG_CONSOLE=true  # ← Extension-specific
   ```

4. **Check browser console filter** (Console tab settings)

---

### Extension Not Registered

**Symptom:**
```javascript
window.DebugConsole.getOwners();
// ['app']  ← Missing 'llm-manager'
```

**Solutions:**

1. **Check View Composer** in Service Provider:
   ```php
   View::composer('*', function ($view) {
       static $registered = false;
       if (!$registered) {
           $registered = true;
           $view->with('__debugRegistration', ...);
       }
   });
   ```

2. **Check @stack present** in master.blade.php:
   ```blade
   @stack('debug-console-extensions')
   ```

3. **Check registration partial** exists and uses @push

4. **Clear views:**
   ```bash
   php artisan view:clear
   ```

---

## 📖 Further Reading

- **[AI-AGENT-INSTRUCTIONS.md](./AI-AGENT-INSTRUCTIONS.md)** - Extension integration guide
- **[API-REFERENCE.md](./API-REFERENCE.md)** - Complete API documentation
- **[EXAMPLES.md](./EXAMPLES.md)** - Real-world code examples
- **[CONFIGURATION.md](./CONFIGURATION.md)** - Deployment and configuration

---

**Last Updated:** 5 de diciembre de 2025  
**Version:** 1.0.0  
**Maintained by:** BITHOVEN Team
