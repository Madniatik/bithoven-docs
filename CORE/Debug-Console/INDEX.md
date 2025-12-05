# 🐞 Debug Console System Documentation

**Complete documentation for BITHOVEN Debug Console System v1.0**

---

## 📚 Documentation Index

### Quick Start

- **[README.md](./README.md)** - Main documentation, quick start, usage guide
- **[AI-AGENT-INSTRUCTIONS.md](./AI-AGENT-INSTRUCTIONS.md)** - Integration guide for extensions

### Complete Reference

- **[API-REFERENCE.md](./API-REFERENCE.md)** - Complete API specifications (JavaScript, Configuration)
- **[EXAMPLES.md](./EXAMPLES.md)** - Real-world integration examples and patterns
- **[CONFIGURATION.md](./CONFIGURATION.md)** - Configuration files, environment variables, deployment

---

## 🎯 What is Debug Console System?

Debug Console is a **core BITHOVEN system** that provides:

- ✅ **Centralized logging** - Single console interface for app + all extensions
- ✅ **Owner isolation** - Each extension/module has independent logger with own config
- ✅ **Level filtering** - debug/info/warn/error with configurable thresholds
- ✅ **Zero overhead** - Completely disabled in production, no performance impact
- ✅ **Vanilla-friendly** - Global availability (window.AppLogger, window.MonitorLogger)
- ✅ **Auto-registration** - Extensions register automatically via Service Provider

**Use Cases:**
- Extension development and debugging
- Feature troubleshooting in development
- Performance analysis (timers)
- Grouped related logs (collapsible sections)
- Table data inspection

---

## 🚀 Quick Links

### For Developers

- [JavaScript API](./README.md#-javascript-api) - All logging methods
- [Integration Pattern](./AI-AGENT-INSTRUCTIONS.md#-integration-steps) - 3-file setup
- [Configuration](./README.md#%EF%B8%8F-configuration) - Enable/disable per owner
- [Common Patterns](./EXAMPLES.md#common-patterns) - Copy-paste examples

### For Extension Developers

- [Extension Integration](./AI-AGENT-INSTRUCTIONS.md) - Step-by-step guide
- [Service Provider Setup](./EXAMPLES.md#extension-registration) - Auto-injection pattern
- [Config File Pattern](./EXAMPLES.md#config-file-pattern) - debug_console section
- [View Composer Pattern](./EXAMPLES.md#view-composer-pattern) - Auto-registration

### For System Admins

- [Enable/Disable Logging](./CONFIGURATION.md#enabledisable-debug-console) - Per-owner control
- [Log Levels](./CONFIGURATION.md#log-levels) - Filtering configuration
- [Production Safety](./CONFIGURATION.md#production-deployment) - Zero overhead guarantee
- [Troubleshooting](./CONFIGURATION.md#troubleshooting) - Common issues

---

## 📋 System Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                     Browser Console                             │
├─────────────────────────────────────────────────────────────────┤
│  [app] [INFO] 12:34:56 Debug Console System initialized         │
│  [llm-manager] [INFO] 12:34:56 LLM Manager registered           │
│  [tickets] [DEBUG] 12:35:01 Fetching tickets list...            │
│  [app] [WARN] 12:35:10 Cache miss for key: users                │
└─────────────────────────────────────────────────────────────────┘
                               ▲
                               │
        ┌──────────────────────┴──────────────────────┐
        │                                             │
┌───────┴────────┐              ┌────────────────────┴──────┐
│  window.       │              │  window.                  │
│  DebugConsole  │              │  DEBUG_CONSOLE_CONFIG     │
│  (Factory)     │              │  (Configuration)          │
└───────┬────────┘              └───────────────────────────┘
        │                                     │
        │  .create('app')                     │
        │  .create('llm-manager')             │  app: {enabled: true, level: 'debug'}
        │                                     │  llm-manager: {enabled: true, ...}
        ▼                                     │
┌────────────────┐                            │
│  Loggers:      │◄───────────────────────────┘
│  - AppLogger   │
│  - MonitorLogger│
│  - TicketsLogger│
└────────────────┘
```

**Global Loading Pattern:**

1. **DebugConsole.js** → Loaded ONCE in `config/settings.php` (global assets)
2. **Core App Registration** → `debug-console-init.blade.php` in `master.blade.php`
3. **Extension Registration** → `@stack('debug-console-extensions')` via View Composer
4. **Available Everywhere** → All loggers work on ANY page

---

## 🔑 Key Features

### 1. Owner Isolation
```javascript
// Each owner has independent configuration
window.AppLogger.debug('App message');        // Shows if app.level >= debug
window.MonitorLogger.info('LLM message');     // Shows if llm-manager.level >= info
window.TicketsLogger.error('Ticket error');   // Always shows if enabled
```

### 2. Level Filtering
```javascript
// Levels (lowest to highest): debug < info < warn < error
// If level = 'info', shows: info, warn, error (NO debug)
MonitorLogger.debug('Hidden');   // ❌ Not shown (below threshold)
MonitorLogger.info('Shown');     // ✅ Shown
MonitorLogger.warn('Shown');     // ✅ Shown
MonitorLogger.error('Shown');    // ✅ Shown
```

### 3. Zero Overhead When Disabled
```javascript
// If enabled = false, early return (no console calls)
if (!this.enabled) return;  // ← In every method
```

### 4. Timestamps & Prefixes
```javascript
// Output format: [owner] [LEVEL] HH:MM:SS message
[llm-manager] [INFO] 14:30:45 Chat session started
[app] [WARN] 14:31:02 Deprecated method used
```

---

## 📖 Documentation Structure

This documentation follows the **BITHOVEN documentation standard** used in other core components (Monitor, Extension Manager):

1. **INDEX.md** (this file) - Navigation hub
2. **README.md** - Main documentation, API overview
3. **AI-AGENT-INSTRUCTIONS.md** - Integration guide for AI assistants
4. **API-REFERENCE.md** - Complete technical reference
5. **EXAMPLES.md** - Real-world code examples
6. **CONFIGURATION.md** - Deployment and config guide

**Last Updated:** 5 de diciembre de 2025  
**Version:** 1.0.0  
**Status:** Production Ready ✅
