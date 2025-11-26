# Namespace Conventions & Extension Manager Integration

**Version:** 1.0.0  
**Last Updated:** 18 de noviembre de 2025  
**Status:** ⚠️ CRITICAL - Extension Manager Compatibility

---

## 🎯 Core Principle

> **Extension Manager reads namespaces from `composer.json` PSR-4 autoload, NOT from slug transformation**

The Extension Manager automatically resolves seeder class namespaces by reading your extension's `composer.json` file, ensuring compatibility with ANY slug naming convention (simple or complex).

---

## 📦 composer.json PSR-4 Configuration

### Required Configuration

Your extension's `composer.json` MUST include PSR-4 autoload configuration:

```json
{
    "name": "bithoven/{slug}",
    "autoload": {
        "psr-4": {
            "Bithoven\\{YourNamespace}\\": "src/",
            "Bithoven\\{YourNamespace}\\Database\\Seeders\\": "database/seeders/"
        }
    }
}
```

### Examples

#### Simple Slug (tasks, dummy, tickets)

```json
{
    "name": "bithoven/tasks",
    "autoload": {
        "psr-4": {
            "Bithoven\\Tasks\\": "src/",
            "Bithoven\\Tasks\\Database\\Seeders\\": "database/seeders/"
        }
    }
}
```

**Resulting namespace:** `Bithoven\Tasks\Database\Seeders\{SeederClass}`

#### Complex Slug (llm-manager, user-management)

```json
{
    "name": "bithoven/llm-manager",
    "autoload": {
        "psr-4": {
            "Bithoven\\LLMManager\\": "src/",
            "Bithoven\\LLMManager\\Database\\Seeders\\": "database/seeders/"
        }
    }
}
```

**Resulting namespace:** `Bithoven\LLMManager\Database\Seeders\{SeederClass}`

---

## 🔍 How Extension Manager Resolves Namespaces

### Automatic Resolution Process

When Extension Manager needs to run seeders, it:

1. **Reads** `vendor/bithoven/{slug}/composer.json`
2. **Parses** `autoload.psr-4` section
3. **Finds** namespace pointing to `"src/"`
4. **Constructs** full seeder class: `{namespace}\Database\Seeders\{SeederClass}`

### Implementation Reference

```php
// ExtensionSeederManager.php
protected function getExtensionNamespace(string $name): ?string
{
    $composerPath = base_path("vendor/bithoven/{$name}/composer.json");
    
    if (!file_exists($composerPath)) {
        return null;
    }
    
    $composer = json_decode(file_get_contents($composerPath), true);
    $psr4 = $composer['autoload']['psr-4'] ?? [];
    
    // Find namespace pointing to "src/"
    foreach ($psr4 as $namespace => $path) {
        if ($path === 'src/') {
            return rtrim($namespace, '\\');
        }
    }
    
    return null;
}
```

### Why This Matters

**Before (broken for complex slugs):**
```php
// ❌ OLD: Used ucfirst() on slug
$namespace = 'Bithoven\\' . ucfirst($name);
// "llm-manager" → "Llm-manager" ❌ WRONG
```

**After (works for ALL slugs):**
```php
// ✅ NEW: Reads from composer.json
$namespace = $this->getExtensionNamespace($name);
// "llm-manager" → "LLMManager" ✅ CORRECT
```

---

## ✅ Naming Conventions

### Slug Naming

**Allowed formats:**
- `tasks` (simple)
- `user-management` (hyphenated)
- `llm-manager` (with acronym)
- `my-awesome-extension` (multi-word)

**Rules:**
- Lowercase only
- Alphanumeric + hyphens
- 2-30 characters
- Must match `composer.json` package name

### Namespace Naming (PSR-4)

**Format:** `Bithoven\{StudlyCase}\`

**Examples:**

| Slug | Namespace | Valid |
|------|-----------|-------|
| `tasks` | `Bithoven\Tasks\` | ✅ |
| `user-management` | `Bithoven\UserManagement\` | ✅ |
| `llm-manager` | `Bithoven\LLMManager\` | ✅ |
| `my-extension` | `Bithoven\MyExtension\` | ✅ |

**Rules:**
- PascalCase/StudlyCase
- No spaces or special characters
- Follows PSR-4 standard
- MUST match actual directory structure in `src/`

---

## 📋 extension.json Seeders Configuration

### Core vs Demo Seeders

Extension Manager distinguishes between two types of seeders:

```json
{
    "seeders": {
        "core": [
            "ConfigurationSeeder",
            "CategorySeeder",
            "TemplateSeeder"
        ],
        "demo": [
            "DemoSeeder"
        ]
    }
}
```

### Seeder Class Names (without namespace)

**Format:** Just the class name, Extension Manager adds namespace automatically

**Examples:**

```json
{
    "seeders": {
        "core": [
            "LLMConfigurationSeeder",      // ✅ Correct
            "LLMToolDefinitionsSeeder",    // ✅ Correct
            "LLMMCPConnectorsSeeder"       // ✅ Correct
        ],
        "demo": [
            "LLMDemoSeeder"                // ✅ Correct
        ]
    }
}
```

**Extension Manager will construct:**
- `Bithoven\LLMManager\Database\Seeders\LLMConfigurationSeeder`
- `Bithoven\LLMManager\Database\Seeders\LLMToolDefinitionsSeeder`
- `Bithoven\LLMManager\Database\Seeders\LLMMCPConnectorsSeeder`
- `Bithoven\LLMManager\Database\Seeders\LLMDemoSeeder`

---

## 🧪 Testing Namespace Resolution

### Verify Composer Autoload

```bash
# Check composer autoload
composer dump-autoload -o

# Test class loading
php artisan tinker
>>> class_exists('Bithoven\LLMManager\Database\Seeders\LLMDemoSeeder');
=> true
```

### Verify Extension Manager Resolution

```bash
# Test seeder execution
php artisan db:seed --class="Bithoven\LLMManager\Database\Seeders\LLMDemoSeeder"

# Or via Extension Manager UI
# Navigate to: /app/extensions
# Click "Load Demo Data" button
```

### Common Issues & Solutions

#### Issue: "Class not found"

**Symptom:**
```
Target class [Bithoven\Llm-manager\Database\Seeders\LLMDemoSeeder] does not exist
```

**Cause:** Old Extension Manager using `ucfirst()` on slug

**Solution:** Update CPANEL to latest version (includes namespace resolution fix)

#### Issue: "Namespace not found in composer.json"

**Symptom:**
```
Cannot resolve namespace for llm-manager
```

**Cause:** Missing or incorrect PSR-4 configuration

**Solution:** Add proper PSR-4 autoload in `composer.json`:

```json
{
    "autoload": {
        "psr-4": {
            "Bithoven\\LLMManager\\": "src/"
        }
    }
}
```

---

## 📖 Complete Example

### Directory Structure

```
bithoven-extension-llm-manager/
├── composer.json
├── extension.json
├── database/
│   └── seeders/
│       ├── LLMConfigurationSeeder.php
│       ├── LLMToolDefinitionsSeeder.php
│       └── LLMDemoSeeder.php
└── src/
    ├── LLMServiceProvider.php
    └── Database/
        └── Seeders/ (optional - can use root database/seeders)
```

### composer.json

```json
{
    "name": "bithoven/llm-manager",
    "description": "LLM Orchestration Platform",
    "type": "library",
    "autoload": {
        "psr-4": {
            "Bithoven\\LLMManager\\": "src/",
            "Bithoven\\LLMManager\\Database\\Seeders\\": "database/seeders/"
        }
    }
}
```

### extension.json

```json
{
    "name": "LLM Manager",
    "slug": "llm-manager",
    "version": "1.0.0",
    "seeders": {
        "core": [
            "LLMConfigurationSeeder",
            "LLMToolDefinitionsSeeder",
            "LLMMCPConnectorsSeeder"
        ],
        "demo": [
            "LLMDemoSeeder"
        ]
    }
}
```

### Seeder File

```php
<?php

namespace Bithoven\LLMManager\Database\Seeders;

use Illuminate\Database\Seeder;

class LLMConfigurationSeeder extends Seeder
{
    public function run(): void
    {
        // Seeder logic
    }
}
```

### Execution

When Extension Manager runs seeders, it:

1. Reads `composer.json` → Finds `Bithoven\LLMManager\`
2. Reads `extension.json` → Finds `LLMConfigurationSeeder`
3. Constructs: `Bithoven\LLMManager\Database\Seeders\LLMConfigurationSeeder`
4. Executes: `php artisan db:seed --class="Bithoven\LLMManager\Database\Seeders\LLMConfigurationSeeder"`

---

## 🚨 Critical Warnings

### ❌ Don't: Hardcode Full Namespace in extension.json

```json
{
    "seeders": {
        "core": [
            "Bithoven\\LLMManager\\Database\\Seeders\\LLMConfigurationSeeder"  // ❌ WRONG
        ]
    }
}
```

**Why:** Extension Manager adds namespace automatically

### ✅ Do: Use Class Name Only

```json
{
    "seeders": {
        "core": [
            "LLMConfigurationSeeder"  // ✅ CORRECT
        ]
    }
}
```

---

### ❌ Don't: Use Inconsistent Naming

```json
// composer.json
{
    "autoload": {
        "psr-4": {
            "Bithoven\\LLMManager\\": "src/"  // LLMManager
        }
    }
}
```

```php
// Seeder file
namespace Bithoven\LlmManager\Database\Seeders;  // ❌ WRONG - Inconsistent case
```

### ✅ Do: Match Exactly

```php
// Seeder file
namespace Bithoven\LLMManager\Database\Seeders;  // ✅ CORRECT - Matches composer.json
```

---

## 📚 Related Documentation

- [EXTENSION-STRUCTURE.md](EXTENSION-STRUCTURE.md) - Complete extension structure guide
- [SEEDERS-BEST-PRACTICES.md](SEEDERS-BEST-PRACTICES.md) - Seeder implementation patterns
- [DEVELOPMENT-WORKFLOW.md](DEVELOPMENT-WORKFLOW.md) - Complete development workflow
- [QUICK-START.md](QUICK-START.md) - Getting started guide

---

## 🎓 Summary

**Key Takeaways:**

1. **Extension Manager reads namespaces from `composer.json`** - No slug transformation
2. **PSR-4 autoload is REQUIRED** - Must point to `"src/"`
3. **extension.json uses class names only** - Extension Manager adds namespace
4. **Works with ANY slug format** - Simple or complex, doesn't matter
5. **Namespace must match directory structure** - PSR-4 standard

**Best Practice:**
- Keep slug simple when possible: `tasks`, `tickets`, `users`
- Use hyphens for multi-word: `user-management`, `llm-manager`
- Use StudlyCase in namespace: `UserManagement`, `LLMManager`
- Always include PSR-4 autoload in `composer.json`

---

**Questions?** See [../COPILOT/AI-AGENT-INSTRUCTIONS.md](../COPILOT/AI-AGENT-INSTRUCTIONS.md) for AI-specific guidance.
