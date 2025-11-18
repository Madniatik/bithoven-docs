# Extension Configuration System

**Version:** 2.0.0  
**Last Updated:** 16 de noviembre de 2025  
**Status:** Production Ready

---

## 📋 Overview

El sistema de configuración de extensiones de Bithoven gestiona dos modos de instalación (Local y VCS), autenticación con GitHub, y configuración global del sistema de extensiones.

---

## 🗂️ Archivos de Configuración

### 1. `storage/app/extension-settings.json`

**Propósito:** Configuración global del sistema de extensiones (Single Source of Truth)

**Ubicación:** `CPANEL/storage/app/extension-settings.json`

**Estructura:**
```json
{
    "repo_mode": "vcs",
    "cache_ttl": 60,
    "use_symlink": true,
    "updated_at": "2025-11-16T05:30:00+00:00",
    "updated_by": 1
}
```

**Campos:**

| Campo | Tipo | Valores | Default | Descripción |
|-------|------|---------|---------|-------------|
| `repo_mode` | string | `local`, `vcs` | `vcs` | Modo de repositorio Composer |
| `cache_ttl` | integer | 5-1440 | `60` | Tiempo de caché en minutos |
| `use_symlink` | boolean | `true`, `false` | `true` | Usar symlinks en modo local |
| `updated_at` | string | ISO 8601 | - | Fecha de última actualización |
| `updated_by` | integer | User ID | - | Usuario que actualizó |

**Gestión:**
- **Lectura:** `ExtensionManager::getSettings()`, `ExtensionController::settings`
- **Escritura:** Panel UI (`/app/extensions`), `ExtensionController::saveSystemSettings()`
- **Auto-creación:** Si no existe, se crea desde defaults en primera carga

---

### 2. `storage/app/extension-settings-defaults.json`

**Propósito:** Valores por defecto del sistema (immutable reference)

**Ubicación:** `CPANEL/storage/app/extension-settings-defaults.json`

**Estructura:**
```json
{
    "repo_mode": "vcs",
    "cache_ttl": 60,
    "use_symlink": true
}
```

**Uso:**
- Inicialización de `extension-settings.json` si no existe
- Reset to Defaults desde UI
- Referencia para validación

**⚠️ NO modificar directamente:** Cambiar defaults requiere actualización de código

---

### 3. `~/.composer/auth.json`

**Propósito:** Autenticación global de Composer (incluye GitHub token)

**Ubicación:** `~/.composer/auth.json` (sistema, fuera del proyecto)

**Estructura:**
```json
{
    "bitbucket-oauth": {},
    "github-oauth": {
        "github.com": "github_pat_11ADJKN7Y0..."
    },
    "gitlab-oauth": {},
    "gitlab-token": {},
    "http-basic": {},
    "bearer": {}
}
```

**GitHub Token:**
- **Formato:** `github_pat_*` (fine-grained) o `ghp_*` (classic)
- **Permisos necesarios:**
  - Repository access: `bithoven-extension-*` o All repositories
  - Permissions: `Contents: Read-only`
- **Gestión:** Panel UI → GitHub Authentication → Add/Change Token
- **Alcance:** Global para TODOS los proyectos Composer del sistema

**Ventajas:**
- ✅ Single Source of Truth para credenciales Git
- ✅ Compartido entre todos los proyectos
- ✅ Fuera del repositorio (seguridad)
- ✅ Estándar Composer

---

### 4. `composer.json` (CPANEL)

**Propósito:** Definición de repositorios de extensiones

**Ubicación:** `CPANEL/composer.json`

**Sección relevante:**
```json
{
    "repositories": [
        {
            "type": "vcs",
            "url": "https://github.com/Madniatik/bithoven-extension-dummy"
        },
        {
            "type": "vcs",
            "url": "https://github.com/Madniatik/bithoven-extension-tickets"
        }
    ]
}
```

**Gestión automática:**
- **Creación:** `ExtensionManager::ensureRepositoryExists()` al instalar
- **Actualización:** `ExtensionController::updateComposerRepositories()` al cambiar modo
- **Eliminación:** `ExtensionManager::removeRepository()` al desinstalar

**⚠️ NO editar manualmente:** El sistema lo gestiona automáticamente

---

## 🔄 Modos de Repositorio

### Modo VCS (Production Mode)

**Características:**
- Instala desde GitHub (repositorios remotos)
- Requiere GitHub token para repos privados
- Ideal para usuarios finales y producción
- No requiere carpeta `../EXTENSIONS/`

**Configuración en composer.json:**
```json
{
    "type": "vcs",
    "url": "https://github.com/Madniatik/bithoven-extension-dummy"
}
```

**Flujo de instalación:**
```
Usuario instala extensión
    ↓
Composer consulta GitHub API (con token)
    ↓
Descarga release/tag desde GitHub
    ↓
Instala en vendor/bithoven/extension-{name}
    ↓
Ejecuta migrations, seeders, etc.
```

**Rate Limits:**
- Sin token: 60 requests/hora
- Con token: 5000 requests/hora

---

### Modo Local (Development Mode)

**Características:**
- Instala desde directorio local `../EXTENSIONS/bithoven-extension-{name}`
- Ideal para desarrollo de extensiones
- Cambios en código se reflejan inmediatamente (con symlink)
- Requiere estructura de carpetas específica

**Configuración en composer.json:**
```json
{
    "type": "path",
    "url": "../EXTENSIONS/bithoven-extension-dummy",
    "options": {
        "symlink": true
    }
}
```

**Estructura requerida:**
```
/Users/madniatik/CODE/LARAVEL/BITHOVEN/
├── CPANEL/                           # Proyecto principal
└── EXTENSIONS/                       # Extensiones locales
    ├── bithoven-extension-dummy/
    ├── bithoven-extension-tickets/
    └── bithoven-extension-{name}/
```

**Symlink:**
- **Enabled (true):** Enlace simbólico → cambios inmediatos
- **Disabled (false):** Copia de archivos → requiere `composer update` tras cambios

---

## 🔀 Flujos del Sistema

### Flujo 1: Primera Instalación

```
1. Usuario accede a /app/extensions
    ↓
2. Sistema verifica extension-settings.json
    ↓
3. Si NO existe:
    ├─ Lee extension-settings-defaults.json
    ├─ Crea extension-settings.json con defaults
    └─ Guarda: { repo_mode: "vcs", cache_ttl: 60, use_symlink: true }
    ↓
4. UI renderiza con modo VCS
    ↓
5. ExtensionMarketplace::getAvailableExtensions()
    ├─ Verifica token en ~/.composer/auth.json
    ├─ Consulta GitHub API
    └─ Cachea resultados (60 min)
```

### Flujo 2: Instalación de Extensión

```
1. Usuario click "Install" en UI
    ↓
2. ExtensionManager::install('dummy')
    ↓
3. ensureRepositoryExists('dummy')
    ├─ Lee extension-settings.json → repo_mode: "vcs"
    ├─ Verifica si repo existe en composer.json
    └─ Si NO existe:
        ├─ Agrega: { type: "vcs", url: "https://..." }
        └─ Guarda composer.json
    ↓
4. Ejecuta: composer require bithoven/extension-dummy
    ├─ Composer lee ~/.composer/auth.json
    ├─ Consulta GitHub con token
    └─ Descarga e instala
    ↓
5. Ejecuta migrations y seeders
    ↓
6. Registra en config('bithoven.extensions.installed')
```

### Flujo 3: Cambio de Modo (VCS → Local)

```
1. Usuario cambia a "Local (Path)" en UI
    ↓
2. ExtensionController::saveSystemSettings()
    ├─ Valida: repo_mode = "local"
    └─ Guarda en extension-settings.json
    ↓
3. updateComposerRepositories('local', true)
    ├─ Lee composer.json
    ├─ Itera TODAS las extensiones bithoven-extension-*
    └─ Para cada una:
        ├─ Cambia: type: "vcs" → type: "path"
        ├─ Cambia: url: "https://..." → url: "../EXTENSIONS/..."
        └─ Agrega: options: { symlink: true }
    ↓
4. Guarda composer.json
    ↓
5. Usuario ejecuta: composer update
```

### Flujo 4: Desinstalación de Extensión

```
1. Usuario click "Uninstall" en UI
    ↓
2. ExtensionManager::uninstall('dummy')
    ↓
3. Crea backup automático
    ↓
4. Ejecuta: composer remove bithoven/extension-dummy
    ↓
5. Rollback migrations (opcional)
    ↓
6. removeRepository('dummy')
    ├─ Lee composer.json
    ├─ Filtra entrada de repositorio
    └─ Guarda composer.json limpio
    ↓
7. Elimina de config('bithoven.extensions.installed')
```

### Flujo 5: Reset to Defaults

```
1. Usuario click "Reset to Defaults" en UI
    ↓
2. Modal confirma valores default (VCS, 60, true)
    ↓
3. Frontend envía: { reset_to_defaults: true }
    ↓
4. ExtensionController::saveSystemSettings()
    ├─ Detecta flag reset_to_defaults
    ├─ Lee extension-settings-defaults.json
    └─ Sobrescribe extension-settings.json
    ↓
5. updateComposerRepositories('vcs', true)
    └─ Actualiza TODAS las extensiones a modo VCS
    ↓
6. Recarga página con configuración default
```

---

## 🔐 Sistema de Autenticación GitHub

### Token Storage

**Ubicación única:** `~/.composer/auth.json`

**Lectura en código:**
```php
// ExtensionMarketplace.php
private function getComposerToken(): ?string
{
    $authFile = getenv('HOME') . '/.composer/auth.json';
    
    if (!file_exists($authFile)) {
        return null;
    }
    
    $authData = json_decode(file_get_contents($authFile), true);
    return $authData['github-oauth']['github.com'] ?? null;
}

private function getGitHubToken(): ?string
{
    // Priority 1: Laravel config (if set)
    $configToken = config('services.github.token');
    if ($configToken) {
        return $configToken;
    }

    // Priority 2: Composer auth.json (standard)
    return $this->getComposerToken();
}
```

**Uso:**
- **ExtensionMarketplace:** Listar extensiones desde GitHub API
- **Composer:** Instalar extensiones desde repos privados
- **ExtensionVersionChecker:** Verificar actualizaciones

### Token Management UI

**Panel:** `/app/extensions` → Extension System Settings → GitHub Authentication

**Operaciones:**
- **Add Token:** Guarda en `~/.composer/auth.json`
- **Change Token:** Sobrescribe token existente
- **View Token:** Muestra versión enmascarada (primeros 20 + últimos 4 caracteres)
- **Remove Token:** Vacía entrada github-oauth

**Validación:**
- Formato: `github_pat_*` o `ghp_*`
- Longitud mínima: 20 caracteres
- Test de autenticidad: Consulta GitHub API rate limit

---

## 📊 Variables de Estado Global

### Runtime Configuration

**Cargadas en memoria:**
```php
// config/bithoven.php
return [
    'extensions' => [
        'installed' => [
            'dummy' => [
                'enabled' => true,
                'version' => '1.7.0',
                'installed_at' => '2025-11-16T05:30:00+00:00'
            ],
            'tickets' => [
                'enabled' => false,
                'version' => '1.1.1',
                'installed_at' => '2025-11-16T06:00:00+00:00'
            ]
        ]
    ]
];
```

**Acceso:**
```php
// Verificar si extensión está instalada
if (extension_installed('dummy')) { ... }

// Verificar si extensión está activa
if (extension_enabled('dummy')) { ... }

// Obtener versión instalada
$version = extension_version('dummy'); // "1.7.0"
```

### Cache System

**Laravel Cache:**
- **Marketplace extensions:** `marketplace_extensions` (60 min)
- **Extension details:** `marketplace_extension_{slug}` (60 min)
- **Version checks:** Dinámico según `cache_ttl`

**Invalidación:**
- Panel UI → "Clear All Caches" → `Cache::flush()`
- Cambio de settings → Auto-invalidación
- `php artisan cache:clear` (manual)

---

## 🛠️ API de Configuración

### ExtensionController

```php
// Guardar configuración global
POST /app/extensions/settings/save
Body: {
    "repo_mode": "vcs",
    "cache_ttl": 60,
    "use_symlink": true
}

// Reset a defaults
POST /app/extensions/settings/save
Body: {
    "reset_to_defaults": true
}

// Guardar GitHub token
POST /app/extensions/settings/save-token
Body: {
    "github_token": "github_pat_..."
}

// Refrescar rate limit
GET /app/extensions/settings/refresh
Response: {
    "success": true,
    "rate_limit": {
        "limit": 5000,
        "remaining": 4998,
        "reset": 1700000000
    }
}

// Limpiar cachés
POST /app/extensions/clear-cache
Body: {
    "clear_all": true
}
```

### ExtensionManager Service

```php
use App\Services\Extensions\ExtensionManager;

$manager = app(ExtensionManager::class);

// Obtener configuración actual
$settings = $manager->getSettings();
// ['repo_mode' => 'vcs', 'cache_ttl' => 60, ...]

// Verificar modo
$isLocal = $settings['repo_mode'] === 'local';
$isVCS = $settings['repo_mode'] === 'vcs';

// Asegurar repositorio existe
$manager->ensureRepositoryExists('dummy');

// Eliminar repositorio
$manager->removeRepository('dummy');
```

---

## ⚙️ Configuración Avanzada

### Cambiar Defaults del Sistema

**Archivo:** `storage/app/extension-settings-defaults.json`

**Ejemplo: Cambiar TTL default a 120 minutos:**
```json
{
    "repo_mode": "vcs",
    "cache_ttl": 120,
    "use_symlink": true
}
```

**⚠️ Impacto:**
- Afecta nuevas instalaciones
- Reset to Defaults usará nuevos valores
- Instalaciones existentes NO se modifican automáticamente

### Modo Mixto (No recomendado)

Es posible tener extensiones en diferentes modos editando `composer.json` manualmente:

```json
{
    "repositories": [
        {
            "type": "path",
            "url": "../EXTENSIONS/bithoven-extension-dummy",
            "options": { "symlink": true }
        },
        {
            "type": "vcs",
            "url": "https://github.com/Madniatik/bithoven-extension-tickets"
        }
    ]
}
```

**⚠️ Advertencias:**
- UI no refleja estado mixto correctamente
- `updateComposerRepositories()` sobrescribirá configuración manual
- Solo para casos especiales de desarrollo

---

## 🐛 Troubleshooting

### Problema: "Rate limit exceeded"

**Causa:** GitHub API sin token (60 requests/hora)

**Solución:**
```bash
# 1. Generar token en GitHub
# 2. Agregar token en UI
# 3. Verificar en ~/.composer/auth.json
cat ~/.composer/auth.json | grep github-oauth
```

### Problema: "Repository not found" al instalar

**Causa:** Repositorio no agregado o modo incorrecto

**Solución:**
```php
// Verificar composer.json
php artisan tinker
>>> $composer = json_decode(file_get_contents('composer.json'), true);
>>> print_r($composer['repositories']);

// Forzar re-creación de repo
$manager = app(\App\Services\Extensions\ExtensionManager::class);
$manager->ensureRepositoryExists('dummy');
```

### Problema: Cambios en extensión local no se reflejan

**Causa:** Symlink deshabilitado

**Solución:**
```bash
# 1. Cambiar en UI: use_symlink = true
# 2. O ejecutar:
composer update bithoven/extension-dummy --prefer-source
```

### Problema: Extensión no aparece en Marketplace

**Causa:** Caché desactualizada o token inválido

**Solución:**
```bash
# 1. Limpiar caché
php artisan cache:clear

# 2. Verificar token
curl -H "Authorization: token github_pat_..." https://api.github.com/rate_limit

# 3. Refrescar marketplace
# Click "Refresh" en UI o esperar 60 minutos
```

---

## 📚 Referencias

### Archivos Relacionados

**Core System:**
- `app/Http/Controllers/Apps/ExtensionManagerController.php`
- `app/Services/Extensions/ExtensionManager.php`
- `app/Services/Extensions/ExtensionMarketplace.php`
- `app/Services/Extensions/ExtensionVersionChecker.php`

**Vistas:**
- `resources/views/app/extension-manager/index.blade.php`
- `resources/views/app/extension-manager/partials/settings-panel.blade.php`
- `resources/views/app/extension-manager/partials/scripts/settings-panel.blade.php`

**Configuración:**
- `storage/app/extension-settings.json`
- `storage/app/extension-settings-defaults.json`
- `~/.composer/auth.json`
- `composer.json` (sección repositories)

### Documentación Externa

- **Composer Repositories:** https://getcomposer.org/doc/05-repositories.md
- **GitHub API:** https://docs.github.com/en/rest
- **GitHub Tokens:** https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens

---

**Última Actualización:** 16 de noviembre de 2025, 06:15  
**Versión:** 2.0.0  
**Estado:** Production Ready