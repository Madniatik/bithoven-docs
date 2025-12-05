# 📚 Debug Console API Reference

**Complete technical reference for Debug Console System v1.0**

---

## 📖 Table of Contents

1. [JavaScript API](#-javascript-api)
2. [Configuration API](#%EF%B8%8F-configuration-api)
3. [Blade API](#-blade-api)
4. [PHP Integration](#-php-integration)
5. [Type Definitions](#-type-definitions)

---

## 🔧 JavaScript API

### Factory Class: `DebugConsole`

Global singleton instance available at `window.DebugConsole`.

---

#### `create(owner: string): ConsoleLogger`

Create logger instance for specific owner.

**Parameters:**
- `owner` (string) - Owner identifier (e.g., 'app', 'llm-manager', 'tickets')

**Returns:** `ConsoleLogger` instance configured for owner

**Example:**
```javascript
const MyLogger = window.DebugConsole.create('my-module');
MyLogger.info('Module initialized');
```

**Behavior:**
- Reads config from `window.DEBUG_CONSOLE_CONFIG[owner]`
- If owner not registered, creates logger with `enabled: false`
- Logger inherits `enabled` and `level` from config

---

#### `register(owner: string, config: OwnerConfig): void`

Register new owner with configuration.

**Parameters:**
- `owner` (string) - Owner identifier
- `config` (OwnerConfig) - Configuration object

**OwnerConfig Type:**
```typescript
{
    enabled: boolean,   // Enable/disable logging
    level: LogLevel     // Minimum log level ('debug'|'info'|'warn'|'error')
}
```

**Example:**
```javascript
window.DebugConsole.register('my-extension', {
    enabled: true,
    level: 'debug'
});
```

**Behavior:**
- Adds owner to `window.DEBUG_CONSOLE_CONFIG`
- Logs registration if `enabled: true`
- Can be called multiple times (updates config)

---

#### `getConfig(): Record<string, OwnerConfig>`

Get current configuration for all owners.

**Returns:** Object mapping owner names to configurations

**Example:**
```javascript
const config = window.DebugConsole.getConfig();
console.log(config);
// {
//   app: {enabled: true, level: 'debug'},
//   'llm-manager': {enabled: true, level: 'info'}
// }
```

---

#### `getOwners(): string[]`

List all registered owners.

**Returns:** Array of owner names

**Example:**
```javascript
const owners = window.DebugConsole.getOwners();
console.log(owners);  // ['app', 'llm-manager', 'tickets']
```

---

### Logger Class: `ConsoleLogger`

Instance created by `DebugConsole.create(owner)`.

**Available as:**
- `window.AppLogger` (core app)
- `window.MonitorLogger` (LLM Manager)
- `window.TicketsLogger` (Tickets extension)
- Any custom logger created via `.create()`

---

#### `debug(...args: any[]): void`

Log debug information (development only).

**Parameters:**
- `...args` - Any number of arguments to log

**Level:** `0` (lowest)  
**Shown when:** `level = 'debug'`

**Output Format:**
```
[owner] [DEBUG] HH:MM:SS message args...
```

**Example:**
```javascript
AppLogger.debug('Processing user:', user);
AppLogger.debug('Step 1', 'Step 2', 'Step 3');
```

**Behavior:**
- Early return if `enabled: false`
- Early return if current level > debug
- Calls `console.log()` with formatted prefix

---

#### `info(...args: any[]): void`

Log informational messages.

**Parameters:**
- `...args` - Any number of arguments to log

**Level:** `1`  
**Shown when:** `level = 'debug' | 'info'`

**Output Format:**
```
[owner] [INFO] HH:MM:SS message args...
```

**Example:**
```javascript
MonitorLogger.info('Chat session started');
MonitorLogger.info('User logged in:', username);
```

**Behavior:**
- Early return if `enabled: false`
- Early return if current level > info
- Calls `console.info()` with formatted prefix

---

#### `warn(...args: any[]): void`

Log warnings.

**Parameters:**
- `...args` - Any number of arguments to log

**Level:** `2`  
**Shown when:** `level = 'debug' | 'info' | 'warn'`

**Output Format:**
```
[owner] [WARN] HH:MM:SS message args...
```

**Example:**
```javascript
AppLogger.warn('Deprecated method used');
MonitorLogger.warn('API rate limit approaching');
```

**Behavior:**
- Early return if `enabled: false`
- Early return if current level > warn
- Calls `console.warn()` with formatted prefix

---

#### `error(...args: any[]): void`

Log errors (critical).

**Parameters:**
- `...args` - Any number of arguments to log

**Level:** `3` (highest)  
**Shown when:** Always (if enabled)

**Output Format:**
```
[owner] [ERROR] HH:MM:SS message args...
```

**Example:**
```javascript
AppLogger.error('Database connection failed');
MonitorLogger.error('API request failed:', error);
```

**Behavior:**
- Early return if `enabled: false`
- Always shown (highest level)
- Calls `console.error()` with formatted prefix

---

#### `group(label: string): void`

Start collapsible log group.

**Parameters:**
- `label` (string) - Group title

**Example:**
```javascript
MonitorLogger.group('Processing 100 items');
for (let i = 0; i < 100; i++) {
    MonitorLogger.debug(`Item ${i}`);
}
MonitorLogger.groupEnd();
```

**Behavior:**
- Early return if `enabled: false`
- Calls `console.group()` with formatted prefix
- Groups can be nested
- Must be closed with `groupEnd()`

---

#### `groupEnd(): void`

End collapsible log group.

**Example:**
```javascript
AppLogger.group('Outer');
AppLogger.group('Inner');
AppLogger.info('Nested message');
AppLogger.groupEnd();  // Close Inner
AppLogger.groupEnd();  // Close Outer
```

**Behavior:**
- Early return if `enabled: false`
- Calls `console.groupEnd()`

---

#### `table(data: object | any[]): void`

Display data in table format.

**Parameters:**
- `data` (object | array) - Data to display (objects, arrays of objects)

**Example:**
```javascript
const users = [
    {id: 1, name: 'John', role: 'Admin'},
    {id: 2, name: 'Jane', role: 'User'}
];
AppLogger.table(users);
```

**Output:** Beautiful table in browser console

**Behavior:**
- Early return if `enabled: false`
- Logs prefix, then calls `console.table(data)`
- Best for arrays of objects

---

#### `time(label: string): void`

Start performance timer.

**Parameters:**
- `label` (string) - Timer identifier

**Example:**
```javascript
MonitorLogger.time('Fetch Data');
await fetchData();
MonitorLogger.timeEnd('Fetch Data');
```

**Output:**
```
llm-manager Fetch Data: 234.56ms
```

**Behavior:**
- Early return if `enabled: false`
- Calls `console.time(owner + ' ' + label)`
- Timer runs until `timeEnd()` called

---

#### `timeEnd(label: string): void`

Stop performance timer and log duration.

**Parameters:**
- `label` (string) - Timer identifier (must match `time()` call)

**Example:**
```javascript
AppLogger.time('Database Query');
const result = await db.query('SELECT * FROM users');
AppLogger.timeEnd('Database Query');
// Output: app Database Query: 45.23ms
```

**Behavior:**
- Early return if `enabled: false`
- Calls `console.timeEnd(owner + ' ' + label)`
- Logs elapsed time in milliseconds

---

## ⚙️ Configuration API

### PHP Configuration

#### Core App Config

**File:** `config/app.php`

```php
return [
    'debug' => env('APP_DEBUG', false),
    'env' => env('APP_ENV', 'production'),
];
```

**Logic:**
```php
// In debug-console-init.blade.php
$enabled = config('app.debug', false);
$level = config('app.env') === 'local' ? 'debug' : 'error';
```

---

#### Extension Config

**File:** `config/{extension}.php`

```php
return [
    'debug_console' => [
        'enabled' => env('{EXTENSION}_DEBUG_CONSOLE', false),
        'level' => env('{EXTENSION}_DEBUG_LEVEL', 'info'),
    ],
];
```

**Access:**
```php
$enabled = config('llm-manager.debug_console.enabled', false);
$level = config('llm-manager.debug_console.level', 'info');
```

---

### Environment Variables

#### Core App

```env
APP_DEBUG=true
APP_ENV=local  # or production, staging, etc.
```

**Effect:**
- `APP_DEBUG=true` → `enabled: true`
- `APP_ENV=local` → `level: 'debug'`
- `APP_ENV=production` → `level: 'error'`

---

#### Extension Variables

```env
# LLM Manager
LLM_DEBUG_CONSOLE=true
LLM_DEBUG_LEVEL=debug

# Tickets
TICKETS_DEBUG_CONSOLE=true
TICKETS_DEBUG_LEVEL=info
```

**Pattern:**
```env
{EXTENSION}_DEBUG_CONSOLE=true|false
{EXTENSION}_DEBUG_LEVEL=debug|info|warn|error
```

---

### JavaScript Configuration Object

**Structure:**
```javascript
window.DEBUG_CONSOLE_CONFIG = {
    'app': {
        enabled: true,
        level: 'debug'
    },
    'llm-manager': {
        enabled: true,
        level: 'info'
    },
    'tickets': {
        enabled: false,
        level: 'warn'
    }
};
```

**Access:**
```javascript
const appConfig = window.DEBUG_CONSOLE_CONFIG.app;
const llmConfig = window.DEBUG_CONSOLE_CONFIG['llm-manager'];
```

---

## 🎨 Blade API

### Core App Registration

**File:** `resources/views/partials/debug-console-init.blade.php`

**Template:**
```blade
<script>
    window.DEBUG_CONSOLE_CONFIG = window.DEBUG_CONSOLE_CONFIG || {};
    window.DEBUG_CONSOLE_CONFIG.app = {
        enabled: {{ config('app.debug', false) ? 'true' : 'false' }},
        level: '{{ config('app.env') === 'local' ? 'debug' : 'error' }}'
    };
    
    document.addEventListener('DOMContentLoaded', function() {
        if (window.DebugConsole) {
            window.AppLogger = window.DebugConsole.create('app');
            
            @if(config('app.debug', false))
                AppLogger.info('Debug Console System initialized');
            @endif
        }
    });
</script>
```

**Usage:** Included once in `master.blade.php`

---

### Extension Registration

**File:** `resources/views/partials/debug-console-registration.blade.php`

**Template:**
```blade
@push('debug-console-extensions')
<script>
    window.DEBUG_CONSOLE_CONFIG = window.DEBUG_CONSOLE_CONFIG || {};
    window.DEBUG_CONSOLE_CONFIG['{extension-slug}'] = {
        enabled: {{ config('{extension}.debug_console.enabled', false) ? 'true' : 'false' }},
        level: '{{ config('{extension}.debug_console.level', 'info') }}'
    };
    
    document.addEventListener('DOMContentLoaded', function() {
        if (window.DebugConsole) {
            window.{Logger}Logger = window.DebugConsole.create('{extension-slug}');
            
            @if(config('{extension}.debug_console.enabled', false))
                {Logger}Logger.info('{Extension Name} Debug Console registered');
            @endif
        }
    });
</script>
@endpush
```

**Usage:** Auto-injected via View Composer

---

### Stack Directive

**File:** `resources/views/layouts/master.blade.php`

```blade
@include('partials.debug-console-init')
@stack('debug-console-extensions')
```

**Behavior:**
- Core app registration runs first
- Extensions push to stack via `@push('debug-console-extensions')`
- Stack resolves at render time (after all @push calls)

---

## 🐘 PHP Integration

### View Composer Pattern

**File:** `src/{Extension}ServiceProvider.php`

```php
use Illuminate\Support\Facades\View;

public function register()
{
    View::composer('*', function ($view) {
        static $registered = false;
        if (!$registered) {
            $registered = true;
            $view->with('__{extension}DebugConsoleRegistration', 
                view('{extension}::partials.debug-console-registration')->render()
            );
        }
    });
}
```

**Behavior:**
- Runs on EVERY view render
- Static flag ensures registration ONCE per request
- Injects rendered partial into all views
- Partial uses @push to add to stack

---

### Config Helper Usage

```php
// Check if debug console enabled
if (config('llm-manager.debug_console.enabled', false)) {
    // Do something debug-specific
}

// Get level
$level = config('llm-manager.debug_console.level', 'info');
```

---

## 📐 Type Definitions

### TypeScript Definitions (Reference)

```typescript
// Owner configuration
interface OwnerConfig {
    enabled: boolean;
    level: LogLevel;
}

// Log levels
type LogLevel = 'debug' | 'info' | 'warn' | 'error';

// Log level priorities
enum LogLevelPriority {
    debug = 0,
    info = 1,
    warn = 2,
    error = 3
}

// Debug Console Factory
class DebugConsole {
    config: Record<string, OwnerConfig>;
    
    create(owner: string): ConsoleLogger;
    register(owner: string, config: OwnerConfig): void;
    getConfig(): Record<string, OwnerConfig>;
    getOwners(): string[];
}

// Console Logger Instance
class ConsoleLogger {
    owner: string;
    enabled: boolean;
    level: LogLevel;
    
    debug(...args: any[]): void;
    info(...args: any[]): void;
    warn(...args: any[]): void;
    error(...args: any[]): void;
    
    group(label: string): void;
    groupEnd(): void;
    table(data: object | any[]): void;
    
    time(label: string): void;
    timeEnd(label: string): void;
}

// Global Window Extensions
declare global {
    interface Window {
        DEBUG_CONSOLE_CONFIG: Record<string, OwnerConfig>;
        DebugConsole: DebugConsole;
        AppLogger: ConsoleLogger;
        MonitorLogger?: ConsoleLogger;
        TicketsLogger?: ConsoleLogger;
    }
}
```

---

## 📊 Level System Reference

### Level Hierarchy

```
debug (0)  ← Lowest (most verbose)
  ↓
info (1)
  ↓
warn (2)
  ↓
error (3)  ← Highest (least verbose)
```

### Level Matrix

| Config Level | .debug() | .info() | .warn() | .error() |
|--------------|----------|---------|---------|----------|
| `debug`      | ✅       | ✅      | ✅      | ✅       |
| `info`       | ❌       | ✅      | ✅      | ✅       |
| `warn`       | ❌       | ❌      | ✅      | ✅       |
| `error`      | ❌       | ❌      | ❌      | ✅       |

---

## 🔍 Internal Implementation

### Level Comparison Logic

```javascript
_shouldLog(level) {
    if (!this.enabled) return false;
    
    const levels = {
        debug: 0,
        info: 1,
        warn: 2,
        error: 3
    };
    
    return levels[level] >= levels[this.level];
}
```

**Example:**
```javascript
// Config: {enabled: true, level: 'info'}
// levels['info'] = 1

_shouldLog('debug')  // 0 >= 1 ? false ❌
_shouldLog('info')   // 1 >= 1 ? true  ✅
_shouldLog('warn')   // 2 >= 1 ? true  ✅
_shouldLog('error')  // 3 >= 1 ? true  ✅
```

---

### Prefix Formatting

```javascript
_formatPrefix(level) {
    const timestamp = new Date().toLocaleTimeString();
    return `[${this.owner}] [${level.toUpperCase()}] ${timestamp}`;
}
```

**Output:**
```
[llm-manager] [INFO] 14:30:45
[app] [DEBUG] 09:15:23
[tickets] [ERROR] 18:42:10
```

---

## 📖 Related Documentation

- **[README.md](./README.md)** - Main documentation, usage guide
- **[AI-AGENT-INSTRUCTIONS.md](./AI-AGENT-INSTRUCTIONS.md)** - Integration guide
- **[EXAMPLES.md](./EXAMPLES.md)** - Real-world examples
- **[CONFIGURATION.md](./CONFIGURATION.md)** - Deployment guide

---

**Last Updated:** 5 de diciembre de 2025  
**Version:** 1.0.0  
**API Status:** Stable
