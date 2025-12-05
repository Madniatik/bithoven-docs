# ⚙️ Debug Console Configuration

**Complete configuration and deployment guide for Debug Console System**

---

## 📖 Table of Contents

1. [Environment Configuration](#-environment-configuration)
2. [Enable/Disable Debug Console](#-enabledisable-debug-console)
3. [Log Levels](#-log-levels)
4. [Production Deployment](#-production-deployment)
5. [Performance Considerations](#-performance-considerations)
6. [Troubleshooting](#-troubleshooting)
7. [Security](#-security)

---

## 🌍 Environment Configuration

### Core App Configuration

**File:** `.env`

```env
# Core Application
APP_DEBUG=true
APP_ENV=local

# When APP_DEBUG=true:
# - Debug Console enabled globally
# - Core app logger (AppLogger) active

# APP_ENV values affect default level:
# - local → level: 'debug' (most verbose)
# - staging → level: 'error' (errors only)
# - production → level: 'error' (errors only)
```

**Logic:**
```php
// In debug-console-init.blade.php
$enabled = config('app.debug', false);  // APP_DEBUG
$level = config('app.env') === 'local' ? 'debug' : 'error';  // APP_ENV
```

---

### Extension Configuration

**Pattern:** `{EXTENSION}_DEBUG_CONSOLE` and `{EXTENSION}_DEBUG_LEVEL`

#### LLM Manager Extension

```env
LLM_DEBUG_CONSOLE=true
LLM_DEBUG_LEVEL=debug
```

**Config File:** `config/llm-manager.php`
```php
'debug_console' => [
    'enabled' => env('LLM_DEBUG_CONSOLE', false),
    'level' => env('LLM_DEBUG_LEVEL', 'info'),
],
```

---

#### Tickets Extension

```env
TICKETS_DEBUG_CONSOLE=true
TICKETS_DEBUG_LEVEL=info
```

**Config File:** `config/tickets.php`
```php
'debug_console' => [
    'enabled' => env('TICKETS_DEBUG_CONSOLE', false),
    'level' => env('TICKETS_DEBUG_LEVEL', 'info'),
],
```

---

### Environment Examples

#### Development Environment

```env
# .env.development

# Core App - Full debugging
APP_DEBUG=true
APP_ENV=local

# LLM Manager - Verbose
LLM_DEBUG_CONSOLE=true
LLM_DEBUG_LEVEL=debug

# Tickets - Verbose
TICKETS_DEBUG_CONSOLE=true
TICKETS_DEBUG_LEVEL=debug
```

**Result:**
- All loggers enabled
- All log levels shown (debug, info, warn, error)
- Maximum verbosity for development

---

#### Staging Environment

```env
# .env.staging

# Core App - Errors only
APP_DEBUG=false
APP_ENV=staging

# LLM Manager - Info and above
LLM_DEBUG_CONSOLE=true
LLM_DEBUG_LEVEL=info

# Tickets - Warnings and errors
TICKETS_DEBUG_CONSOLE=true
TICKETS_DEBUG_LEVEL=warn
```

**Result:**
- Core app disabled (production-like)
- LLM Manager shows info/warn/error (no debug)
- Tickets shows warn/error only

---

#### Production Environment

```env
# .env.production

# Core App - DISABLED
APP_DEBUG=false
APP_ENV=production

# LLM Manager - DISABLED
LLM_DEBUG_CONSOLE=false

# Tickets - DISABLED
TICKETS_DEBUG_CONSOLE=false
```

**Result:**
- **All loggers disabled**
- **Zero console output**
- **Zero performance overhead** (early return in all methods)

---

## 🔌 Enable/Disable Debug Console

### Enable Debug Console

#### Per Owner (Extension)

**Step 1:** Set environment variable
```env
LLM_DEBUG_CONSOLE=true
```

**Step 2:** Reload configuration (if running)
```bash
php artisan config:clear
```

**Step 3:** Verify in browser console
```javascript
window.DebugConsole.getConfig();
// Should show: 'llm-manager': {enabled: true, ...}
```

---

#### All Owners (Global)

```env
# Core App
APP_DEBUG=true

# Extensions (enable individually)
LLM_DEBUG_CONSOLE=true
TICKETS_DEBUG_CONSOLE=true
```

**Important:** Core app uses `APP_DEBUG`, extensions use own vars

---

### Disable Debug Console

#### Per Owner

**Step 1:** Set environment variable
```env
LLM_DEBUG_CONSOLE=false
```

**Step 2:** Clear config cache
```bash
php artisan config:clear
```

**Step 3:** Verify in browser console
```javascript
window.DebugConsole.getConfig();
// Should show: 'llm-manager': {enabled: false, ...}

// Try logging (should be silent)
window.MonitorLogger.info('Test');  // No output
```

---

#### All Owners (Global Disable)

```env
# Core App
APP_DEBUG=false

# Extensions
LLM_DEBUG_CONSOLE=false
TICKETS_DEBUG_CONSOLE=false
```

**Result:** All loggers disabled, zero overhead

---

## 📊 Log Levels

### Level Definitions

| Level | Priority | When to Use | Shows |
|-------|----------|-------------|-------|
| `debug` | 0 (lowest) | Development, step-by-step tracing | debug, info, warn, error |
| `info` | 1 | Normal operations, tracking | info, warn, error |
| `warn` | 2 | Non-critical issues, deprecations | warn, error |
| `error` | 3 (highest) | Critical failures only | error |

---

### Set Log Level

#### Via Environment Variable

```env
# Development (show everything)
LLM_DEBUG_LEVEL=debug

# Staging (show info and above)
LLM_DEBUG_LEVEL=info

# Production-like (errors only)
LLM_DEBUG_LEVEL=error
```

---

#### Per Environment

**Development:**
```env
APP_ENV=local
LLM_DEBUG_LEVEL=debug
TICKETS_DEBUG_LEVEL=debug
```

**Staging:**
```env
APP_ENV=staging
LLM_DEBUG_LEVEL=info
TICKETS_DEBUG_LEVEL=warn
```

**Production:**
```env
APP_ENV=production
LLM_DEBUG_LEVEL=error  # Or disabled
TICKETS_DEBUG_LEVEL=error  # Or disabled
```

---

### Level Examples

#### Level: `debug` (Development)

```javascript
// Config: {enabled: true, level: 'debug'}

MonitorLogger.debug('Step 1');   // ✅ Shown
MonitorLogger.info('Started');   // ✅ Shown
MonitorLogger.warn('Slow');      // ✅ Shown
MonitorLogger.error('Failed');   // ✅ Shown
```

**Output:**
```
[llm-manager] [DEBUG] 14:30:45 Step 1
[llm-manager] [INFO] 14:30:46 Started
[llm-manager] [WARN] 14:30:47 Slow
[llm-manager] [ERROR] 14:30:48 Failed
```

---

#### Level: `info` (Staging)

```javascript
// Config: {enabled: true, level: 'info'}

MonitorLogger.debug('Step 1');   // ❌ Hidden (below info)
MonitorLogger.info('Started');   // ✅ Shown
MonitorLogger.warn('Slow');      // ✅ Shown
MonitorLogger.error('Failed');   // ✅ Shown
```

**Output:**
```
[llm-manager] [INFO] 14:30:46 Started
[llm-manager] [WARN] 14:30:47 Slow
[llm-manager] [ERROR] 14:30:48 Failed
```

---

#### Level: `warn` (Pre-Production)

```javascript
// Config: {enabled: true, level: 'warn'}

MonitorLogger.debug('Step 1');   // ❌ Hidden
MonitorLogger.info('Started');   // ❌ Hidden
MonitorLogger.warn('Slow');      // ✅ Shown
MonitorLogger.error('Failed');   // ✅ Shown
```

**Output:**
```
[llm-manager] [WARN] 14:30:47 Slow
[llm-manager] [ERROR] 14:30:48 Failed
```

---

#### Level: `error` (Production)

```javascript
// Config: {enabled: true, level: 'error'}

MonitorLogger.debug('Step 1');   // ❌ Hidden
MonitorLogger.info('Started');   // ❌ Hidden
MonitorLogger.warn('Slow');      // ❌ Hidden
MonitorLogger.error('Failed');   // ✅ Shown
```

**Output:**
```
[llm-manager] [ERROR] 14:30:48 Failed
```

---

## 🚀 Production Deployment

### Production Configuration

**File:** `.env.production`

```env
# Core App - DISABLED
APP_DEBUG=false
APP_ENV=production

# Extensions - DISABLED
LLM_DEBUG_CONSOLE=false
TICKETS_DEBUG_CONSOLE=false

# Optional: Enable for specific extension (errors only)
# LLM_DEBUG_CONSOLE=true
# LLM_DEBUG_LEVEL=error
```

---

### Deployment Checklist

#### Pre-Deployment

- [ ] ✅ Review `.env` file
- [ ] ✅ Ensure `APP_DEBUG=false`
- [ ] ✅ Ensure all `{EXTENSION}_DEBUG_CONSOLE=false`
- [ ] ✅ Test with production config locally

```bash
# Test production config
cp .env .env.backup
cp .env.production .env
php artisan config:clear
php artisan serve

# Verify in browser console
# window.DebugConsole.getConfig() should show all enabled: false
```

---

#### Deployment

```bash
# 1. Pull latest code
git pull origin main

# 2. Install dependencies
composer install --no-dev --optimize-autoloader

# 3. Clear all caches
php artisan config:clear
php artisan route:clear
php artisan view:clear
php artisan optimize

# 4. Verify environment
grep DEBUG .env
# Should show: APP_DEBUG=false, *_DEBUG_CONSOLE=false

# 5. Restart services
sudo systemctl restart php8.3-fpm
sudo systemctl restart nginx
```

---

#### Post-Deployment Verification

```bash
# 1. Check application logs (should be silent)
tail -f storage/logs/laravel.log

# 2. Open production site in browser
# 3. Open Developer Tools → Console
# 4. Verify NO debug logs appear

# Expected: Clean console (only app logs, NO Debug Console output)
```

---

### Production Safety Features

#### 1. Early Return Pattern

```javascript
// In every logger method
debug(...args) {
    if (!this.enabled) return;  // ← Immediate exit
    if (this._shouldLog('debug')) {
        console.log(...);
    }
}
```

**Result:** When `enabled: false`, zero console calls

---

#### 2. Default to Disabled

```php
// In config files
'enabled' => env('LLM_DEBUG_CONSOLE', false),  // ← Default false
```

**Result:** If env var missing, logger disabled

---

#### 3. Zero Overhead

```javascript
// Production config: {enabled: false, level: 'error'}
MonitorLogger.debug('Expensive operation:', complexObject);
// ↓
// Early return (line 1 of method)
// complexObject never evaluated
// console.log() never called
```

**Result:** No performance impact

---

## ⚡ Performance Considerations

### Development Performance

**Impact:** Negligible
- Loggers active, console calls frequent
- Browser DevTools rendering logs
- **Acceptable for development**

---

### Production Performance

**Impact:** Zero
- All loggers disabled (`enabled: false`)
- Early return in every method (line 1)
- No console calls, no object serialization
- **No overhead**

---

### Memory Usage

**Development:**
- Logs accumulate in browser console
- Large objects logged can consume memory
- **Solution:** Use grouping, clear console periodically

**Production:**
- No logs generated
- No memory usage
- **No impact**

---

### Network Traffic

**Impact:** None
- Debug Console is client-side only
- No server requests
- No network overhead

---

## 🐛 Troubleshooting

### Issue: Logs Not Appearing

#### Checklist

1. ✅ Check `enabled`:
   ```javascript
   window.DebugConsole.getConfig();
   // Check: {owner}: {enabled: true, ...}
   ```

2. ✅ Check `level`:
   ```javascript
   window.DebugConsole.getConfig();
   // If level: 'error', debug/info/warn won't show
   ```

3. ✅ Check environment variables:
   ```bash
   grep DEBUG .env
   # Should show: {EXTENSION}_DEBUG_CONSOLE=true
   ```

4. ✅ Clear config cache:
   ```bash
   php artisan config:clear
   ```

5. ✅ Check browser console filter:
   - DevTools → Console tab
   - Ensure filter set to "All levels"
   - Try searching for `[owner]`

---

### Issue: Wrong Level Shown

**Symptom:** Debug logs shown when level is `info`

**Solution:**

1. Check config:
   ```javascript
   window.DebugConsole.getConfig();
   // Verify level value
   ```

2. Check environment variable:
   ```bash
   echo $LLM_DEBUG_LEVEL
   ```

3. Update `.env`:
   ```env
   LLM_DEBUG_LEVEL=info  # Not 'debug'
   ```

4. Clear cache:
   ```bash
   php artisan config:clear
   ```

---

### Issue: Production Logs Showing

**Symptom:** Logs appear in production environment

**Critical Fix:**

1. **Immediately disable:**
   ```env
   APP_DEBUG=false
   LLM_DEBUG_CONSOLE=false
   TICKETS_DEBUG_CONSOLE=false
   ```

2. **Clear caches:**
   ```bash
   php artisan config:clear
   php artisan optimize
   ```

3. **Verify:**
   ```bash
   php artisan tinker
   >>> config('app.debug')
   => false
   >>> config('llm-manager.debug_console.enabled')
   => false
   ```

4. **Restart services:**
   ```bash
   sudo systemctl restart php8.3-fpm
   ```

---

### Issue: Config Changes Not Applied

**Symptom:** Changed `.env`, but config not updated

**Solution:**

1. **Clear config cache:**
   ```bash
   php artisan config:clear
   ```

2. **Verify environment loaded:**
   ```bash
   php artisan tinker
   >>> env('LLM_DEBUG_CONSOLE')
   => "true"
   ```

3. **Hard reload browser:**
   - Chrome/Firefox: `Ctrl+Shift+R` (Windows/Linux)
   - Chrome/Firefox: `Cmd+Shift+R` (macOS)

4. **Check cached config:**
   ```bash
   ls -la bootstrap/cache/config.php
   # If exists, delete:
   rm bootstrap/cache/config.php
   ```

---

## 🔒 Security

### Production Security

#### 1. Disable Debug Console

**Critical:**
```env
APP_DEBUG=false
LLM_DEBUG_CONSOLE=false
TICKETS_DEBUG_CONSOLE=false
```

**Why:** Prevents leaking sensitive data in browser console

---

#### 2. Never Log Sensitive Data

**❌ BAD:**
```javascript
AppLogger.debug('User password:', password);
AppLogger.info('API key:', apiKey);
MonitorLogger.debug('Credit card:', cardNumber);
```

**✅ GOOD:**
```javascript
AppLogger.debug('User authenticated:', user.id);
AppLogger.info('API request successful');
MonitorLogger.debug('Payment processed:', transactionId);
```

---

#### 3. Log Sanitization

```javascript
// Sanitize before logging
function logUser(user) {
    const sanitized = {
        id: user.id,
        name: user.name,
        email: user.email.replace(/@.+/, '@***')  // Partial hide
        // ❌ Do NOT log: password, token, apiKey
    };
    AppLogger.debug('User data:', sanitized);
}
```

---

### Development Security

#### 1. Don't Commit `.env`

```bash
# .gitignore
.env
.env.local
.env.production
```

---

#### 2. Use `.env.example`

```env
# .env.example (safe to commit)

APP_DEBUG=false
LLM_DEBUG_CONSOLE=false
LLM_DEBUG_LEVEL=info
```

**Developers copy to `.env` and modify**

---

#### 3. Review Console Before Screenshots

- Before taking screenshots/demos, disable debug console
- Or clear console output
- Prevent accidentally leaking internal data

---

## 📖 Related Documentation

- **[README.md](./README.md)** - Main documentation, API overview
- **[AI-AGENT-INSTRUCTIONS.md](./AI-AGENT-INSTRUCTIONS.md)** - Integration guide
- **[API-REFERENCE.md](./API-REFERENCE.md)** - Complete API specifications
- **[EXAMPLES.md](./EXAMPLES.md)** - Real-world examples

---

**Last Updated:** 5 de diciembre de 2025  
**Version:** 1.0.0  
**Production Ready:** ✅
