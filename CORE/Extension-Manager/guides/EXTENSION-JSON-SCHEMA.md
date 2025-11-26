# Extension JSON Schema Reference

**Versión:** 1.0.0  
**Última Actualización:** 26 de noviembre de 2025  
**Schema File:** `/DOCS/CORE/Extension-Manager/schemas/extension-schema.json`

---

## 📋 Tabla de Contenidos

1. [Introducción](#introducción)
2. [Campos Requeridos](#campos-requeridos)
3. [Campos Opcionales](#campos-opcionales)
4. [Ejemplos Completos](#ejemplos-completos)
5. [Campos Deprecated](#campos-deprecated)
6. [Validación](#validación)
7. [Migration Guide](#migration-guide)

---

## Introducción

El archivo `extension.json` es el **manifiesto oficial** de cada extensión BITHOVEN. Define metadata, dependencias, permisos, y configuración necesaria para la instalación y gestión de extensiones.

### Source of Truth

**`extension.json` es la única fuente de verdad para:**
- ✅ Versión de la extensión
- ✅ Permisos y roles
- ✅ Seeders (core vs demo)
- ✅ Metadata de marketplace

**`composer.json` NO se usa para versioning** (solo metadata Composer estándar).

---

## Campos Requeridos

Estos campos **DEBEN** estar presentes en todo `extension.json`:

### `slug` (string, required)
**Identificador único** de la extensión (lowercase, hyphens allowed).

```json
{
  "slug": "llm-manager"
}
```

**Pattern:** `^[a-z0-9-]+$`  
**Usado en:** 
- Rutas de instalación (`vendor/bithoven/{slug}`)
- Nombres de tablas (`{slug}_*`)
- Prefijos de permisos (`extensions:{slug}:*`)

---

### `name` (string, required)
**Nombre para mostrar** en UI (user-facing).

```json
{
  "name": "LLM Manager"
}
```

**Longitud:** 1-100 caracteres  
**Usado en:** Marketplace cards, modales, breadcrumbs

---

### `version` (string, required)
**Versión semántica** de la extensión (X.Y.Z).

```json
{
  "version": "1.0.2"
}
```

**Pattern:** `^\d+\.\d+\.\d+$`  
**⚠️ SOURCE OF TRUTH:** Esta es la versión real (no `composer.json`)  
**Usado en:** Version checking, updates, changelogs

---

### `description` (string, required)
**Descripción corta** de funcionalidad.

```json
{
  "description": "Multi-provider LLM orchestration platform with streaming support"
}
```

**Longitud:** 10-255 caracteres  
**Usado en:** Marketplace search, modales

---

### `author` (string, required)
**Autor o equipo** responsable.

```json
{
  "author": "BITHOVEN Team"
}
```

**Usado en:** Credits, about section

---

### `created_at` (string, required)
**Timestamp de creación** (ISO 8601).

```json
{
  "created_at": "2025-01-15T10:30:00+00:00"
}
```

**Format:** `YYYY-MM-DDTHH:MM:SS+00:00`  
**🆕 Nuevo en v1.0.0:** Tracking de antigüedad

---

## Campos Opcionales

### Metadata y Links

#### `updated_at` (string, optional)
**Última actualización** (ISO 8601).

```json
{
  "updated_at": "2025-11-26T21:30:00+00:00"
}
```

**Auto-actualizar:** Al crear nuevas versiones

---

#### `homepage` (string, optional)
**URL de homepage** (docs, GitHub, etc.).

```json
{
  "homepage": "https://github.com/bithoven/llm-manager"
}
```

**Usado en:** Links en marketplace modal

---

#### `repository` (object, optional)
**VCS repository info** - CRÍTICO para dev-mode.

```json
{
  "repository": {
    "type": "vcs",
    "url": "https://github.com/bithoven/llm-manager.git"
  }
}
```

**Properties:**
- `type` (enum): `"vcs"` o `"git"`
- `url` (string): Git clone URL

**Usado en:** Development mode (symlink management)

---

### Marketplace UI

#### `category` (string, optional)
**Categoría de marketplace**.

```json
{
  "category": "AI & Machine Learning"
}
```

**Valores permitidos:**
- `"Productivity"`
- `"Development"`
- `"AI & Machine Learning"`
- `"Communication"`
- `"Analytics"`
- `"Security"`
- `"Integration"`
- `"Utilities"`
- `"Other"`

**Usado en:** Marketplace filters, badges

---

#### `icon` (string, optional)
**Keenicons CSS class**.

```json
{
  "icon": "ki-abstract-26"
}
```

**Pattern:** `^ki-[a-z0-9-]+$`  
**Usado en:** Marketplace cards, sidebar menus

---

#### `featured` (boolean, optional)
**Flag de destacado**.

```json
{
  "featured": true
}
```

**Default:** `false`  
**Usado en:** Marketplace sorting (featured first)

---

#### `tags` (array, optional)
**Tags para búsqueda**.

```json
{
  "tags": ["ai", "llm", "openai", "streaming"]
}
```

**Max items:** 10  
**Item length:** 2-30 caracteres  
**Usado en:** Search, badges en modal

---

### Sistema BITHOVEN

#### `permissions` (array, optional but recommended)
**Permisos auto-creados** durante instalación - CRÍTICO.

```json
{
  "permissions": [
    "extensions:llm-manager:conversations:view",
    "extensions:llm-manager:conversations:create",
    "extensions:llm-manager:conversations:edit",
    "extensions:llm-manager:conversations:delete",
    "extensions:llm-manager:settings:manage"
  ]
}
```

**Pattern:** `^extensions:[a-z-]+:[a-z-]+:(view|create|edit|delete|manage)$`  
**⚠️ IMPORTANTE:** Estos permisos se crean automáticamente al instalar

---

#### `system_tables` (array, optional)
**Tablas del sistema** que la extensión usa.

```json
{
  "system_tables": ["users", "permissions", "roles", "notifications"]
}
```

**Usado en:** Validación de dependencias

---

### Base de Datos

#### `migrations` (object, optional)
**Configuración de migraciones**.

```json
{
  "migrations": {
    "required": true,
    "path": "database/migrations"
  }
}
```

**Properties:**
- `required` (boolean): Si requiere migrations
- `path` (string): Ruta custom (default: `database/migrations`)

**⚠️ NO incluir:** `count`, `can_skip`, `details` (redundantes - se calculan auto)

---

#### `seeders` (object, CRITICAL)
**Seeders core vs demo** - CRÍTICO.

```json
{
  "seeders": {
    "core": [
      "LLMPermissionsSeeder",
      "LLMProvidersSeeder",
      "LLMPromptTemplatesSeeder"
    ],
    "demo": [
      "LLMDemoSeeder"
    ]
  }
}
```

**Core seeders:**
- Ejecutan en instalación automática
- Datos esenciales (categorías, templates, config)

**Demo seeders:**
- Ejecutan bajo demanda
- Datos de prueba/ejemplo

---

### Requirements (Futuro - Validación Pendiente)

#### `requirements` (object, optional)
**Versiones requeridas** de PHP, Laravel, etc.

```json
{
  "requirements": {
    "php": "^8.2",
    "laravel": "^11.0",
    "node": "^18.0",
    "python": "^3.9"
  }
}
```

**⏳ Estado:** Definido pero NO validado automáticamente (Prioridad 3)

---

#### `min_version` (string, optional)
**Versión mínima de CPANEL** requerida.

```json
{
  "min_version": "1.3.0"
}
```

**⏳ Estado:** Definido pero NO validado (Prioridad 3)

---

#### `dependencies` (object, optional)
**Dependencias de otras extensiones**.

```json
{
  "dependencies": {
    "bithoven/core": "^1.4.0",
    "bithoven/tickets": "^1.2.0"
  }
}
```

**⏳ Estado:** Definido pero NO validado (Prioridad 3)

---

### Changelog

#### `changelog` (object, optional but recommended)
**Historial de versiones** - Mostrado en marketplace.

```json
{
  "changelog": {
    "v1.0.2": {
      "date": "2025-11-26",
      "changes": [
        "Fixed seeder architecture (core vs demo separation)",
        "Added LLMPromptTemplatesSeeder with 5 essential templates",
        "Added LLMKnowledgeBaseSeeder with 2 documentation articles"
      ],
      "migration_notes": "No database changes",
      "breaking_changes": false,
      "components": ["code", "database"]
    },
    "v1.0.1": {
      "date": "2025-11-20",
      "changes": ["Initial release"],
      "migration_notes": "Creates 8 tables",
      "breaking_changes": false,
      "components": ["code", "database", "config"]
    }
  }
}
```

**Version key pattern:** `^v\d+\.\d+\.\d+$`  
**Required per version:**
- `date` (string): YYYY-MM-DD
- `changes` (array): Lista de cambios

**Optional per version:**
- `migration_notes` (string): Notas de migrations
- `breaking_changes` (boolean): Flag de breaking
- `components` (array): `["code", "database", "config", "assets", "dependencies"]`

---

## Campos Deprecated

### ❌ NO usar estos campos

| Campo | Razón | Alternativa |
|-------|-------|-------------|
| `menu` | Menú hardcoded en CPANEL | Definir en `__menu.blade.php` |
| `post_install.commands` | No se ejecutan (riesgo seguridad) | Documentar en README |
| `post_install.messages` | No se muestran | Usar logs |
| `migrations.count` | Redundante (auto-calculado) | Sistema cuenta archivos |
| `migrations.can_skip` | No implementado | - |
| `migrations.details` | Redundante | Sistema lista archivos |
| `migrations.description` | Innecesario | Usar README |
| `breaking_changes` (top-level) | Duplicado | Solo en `changelog.vX.X.X.breaking_changes` |
| `features` | Solo documentación | Usar README |
| `display_name` | Duplicado de `name` | Usar `name` |

---

## Ejemplos Completos

### Ejemplo Mínimo (Required Fields Only)

```json
{
  "slug": "my-extension",
  "name": "My Extension",
  "version": "1.0.0",
  "description": "A simple BITHOVEN extension for demonstration",
  "author": "Your Name",
  "created_at": "2025-11-26T21:30:00+00:00",
  "seeders": {
    "core": [],
    "demo": []
  }
}
```

### Ejemplo Completo (Production Ready)

```json
{
  "slug": "llm-manager",
  "name": "LLM Manager",
  "version": "1.0.2",
  "description": "Multi-provider LLM orchestration platform with streaming support",
  "author": "BITHOVEN Team",
  "created_at": "2025-01-15T10:30:00+00:00",
  "updated_at": "2025-11-26T21:30:00+00:00",
  
  "homepage": "https://github.com/bithoven/llm-manager",
  "repository": {
    "type": "vcs",
    "url": "https://github.com/Madniatik/bithoven-extension-llm-manager.git"
  },
  
  "category": "AI & Machine Learning",
  "icon": "ki-abstract-26",
  "featured": true,
  "tags": ["ai", "llm", "openai", "anthropic", "streaming"],
  
  "permissions": [
    "extensions:llm-manager:conversations:view",
    "extensions:llm-manager:conversations:create",
    "extensions:llm-manager:conversations:edit",
    "extensions:llm-manager:conversations:delete",
    "extensions:llm-manager:providers:view",
    "extensions:llm-manager:providers:manage",
    "extensions:llm-manager:prompts:view",
    "extensions:llm-manager:prompts:manage",
    "extensions:llm-manager:knowledge-base:view",
    "extensions:llm-manager:knowledge-base:manage",
    "extensions:llm-manager:settings:view",
    "extensions:llm-manager:settings:manage"
  ],
  
  "system_tables": ["users", "permissions", "roles"],
  
  "migrations": {
    "required": true,
    "path": "database/migrations"
  },
  
  "seeders": {
    "core": [
      "LLMPermissionsSeeder",
      "LLMProvidersSeeder",
      "LLMModelsSeeder",
      "LLMStatusesSeeder",
      "LLMPromptTemplatesSeeder",
      "LLMKnowledgeBaseSeeder"
    ],
    "demo": [
      "LLMDemoSeeder"
    ]
  },
  
  "requirements": {
    "php": "^8.2",
    "laravel": "^11.0",
    "node": "^18.0"
  },
  
  "changelog": {
    "v1.0.2": {
      "date": "2025-11-26",
      "changes": [
        "Fixed seeder architecture",
        "Added prompt templates seeder",
        "Added knowledge base seeder"
      ],
      "migration_notes": "No database changes",
      "breaking_changes": false,
      "components": ["code", "database"]
    },
    "v1.0.1": {
      "date": "2025-11-20",
      "changes": ["Initial production release"],
      "migration_notes": "Creates 8 tables for LLM management",
      "breaking_changes": false,
      "components": ["code", "database", "config"]
    }
  }
}
```

---

## Validación

### Manual Validation

```bash
# Validar contra schema JSON
cat vendor/bithoven/llm-manager/extension.json | \
  jq -s '.[0] as $schema | .[1] | validate($schema)' \
  /DOCS/CORE/Extension-Manager/schemas/extension-schema.json -
```

### Automatic Validation (Futuro - Prioridad 1)

```bash
# Comando Artisan (pendiente implementación)
php artisan bithoven:extension:validate llm-manager
```

**Validará:**
- ✅ Campos requeridos presentes
- ✅ Patterns correctos (slug, version, permissions)
- ✅ Formatos (ISO 8601 para timestamps, URLs)
- ✅ Enums (category, repository.type)

---

## Migration Guide

### De Schema Antiguo a v1.0.0

#### 1. Añadir Campos Nuevos

```json
{
  "created_at": "2025-01-15T10:30:00+00:00",  // Fecha creación real
  "updated_at": "2025-11-26T21:30:00+00:00"   // Fecha última versión
}
```

**Cómo obtener `created_at`:**
```bash
# Fecha del primer commit de la extensión
git log --reverse --format="%aI" | head -1
```

**Cómo obtener `updated_at`:**
```bash
# Fecha del último commit
git log -1 --format="%aI"
```

#### 2. Normalizar Campos Existentes

```json
// ❌ ANTES
{
  "name": "Tickets System",  // Usado como display_name
  "display_name": "Tickets"  // Redundante
}

// ✅ DESPUÉS
{
  "name": "Tickets System"  // Solo uno, es display_name
}
```

#### 3. Mover Permisos

```json
// ❌ ANTES (composer.json)
{
  "extra": {
    "bithoven": {
      "permissions": [...]
    }
  }
}

// ✅ DESPUÉS (extension.json)
{
  "permissions": [
    "extensions:tickets:tickets:view",
    "extensions:tickets:tickets:create"
  ]
}
```

#### 4. Eliminar Campos Deprecated

```json
// ❌ ELIMINAR
{
  "menu": {...},                    // No usado
  "post_install": {...},            // No implementado
  "migrations": {
    "count": 8,                     // Redundante
    "can_skip": false,              // No usado
    "details": [...]                // Redundante
  },
  "breaking_changes": false,        // Solo en changelog
  "features": {...}                 // Solo docs
}

// ✅ MANTENER
{
  "migrations": {
    "required": true,
    "path": "database/migrations"   // Solo si custom
  }
}
```

#### 5. Limpiar composer.json

```json
// ❌ ELIMINAR de composer.json
{
  "version": "1.2.1",         // Usar extension.json
  "extra": {
    "bithoven": {             // Eliminar sección completa
      "name": "...",
      "slug": "...",
      "version": "...",
      "permissions": [...]
    }
  }
}

// ✅ MANTENER solo
{
  "name": "bithoven/tickets",
  "description": "...",       // OK duplicar de extension.json
  "type": "library",
  "license": "MIT",
  "authors": [...],
  "require": {...},
  "autoload": {...},
  "extra": {
    "laravel": {              // Solo Laravel service providers
      "providers": [...],
      "aliases": [...]
    }
  }
}
```

---

## Referencias

- **Schema JSON:** `/DOCS/CORE/Extension-Manager/schemas/extension-schema.json`
- **Quick Start:** `/DOCS/CORE/Extension-Manager/QUICK-START.md`
- **Análisis Completo:** `/reports/chat/20251126-2110-extension-json-analysis.md`
- **Implementation Plan:** `/dev/copilot/tasks/EXTENSION-SCHEMA-IMPLEMENTATION-PLAN.md`

---

**Última Actualización:** 26 de noviembre de 2025, 21:40  
**Versión Documento:** 1.0.0
