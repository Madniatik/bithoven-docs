# 🔧 Extension Manager Controller - Plan de Refactorización

**Fecha:** 27 de noviembre de 2025  
**Archivo:** `app/Http/Controllers/Apps/ExtensionManagerController.php`  
**Líneas actuales:** 2,090 líneas  
**Estado:** CRÍTICO - Requiere refactorización urgente

---

## 📊 Análisis del Problema

### Métricas Actuales
- **Tamaño:** 2,090 líneas (God Object anti-pattern)
- **Métodos:** 39 métodos públicos
- **Responsabilidades:** 8 dominios diferentes
- **Complejidad ciclomática:** ALTA (métodos con >100 líneas)
- **Violaciones SOLID:** Múltiples (SRP, OCP, DIP)

### Problemas Identificados

#### 1. **Violación Single Responsibility Principle (SRP)**
El controlador maneja:
- Vistas (overview, marketplace, settings)
- Instalación/Desinstalación
- Actualización de versiones
- Gestión de backups
- Configuración de extensiones
- Dev-mode (symlinks)
- GitHub API integration
- Composer operations

#### 2. **Código Duplicado**
- Validación de extensiones: 8 repeticiones
- Respuestas JSON: patrón `['success' => ..., 'message' => ...]` en 25+ lugares
- Logs: patrón repetido en casi todos los métodos
- Activity tracking: duplicado en 10+ métodos

#### 3. **Métodos Demasiado Largos**
| Método | Líneas | Complejidad |
|--------|--------|-------------|
| `overview()` | 123 | ALTA |
| `show()` | 89 | ALTA |
| `install()` | 54 | MEDIA |
| `installLocal()` | 108 | MUY ALTA |
| `updateSettings()` | 64 | MEDIA |
| `getUpdateInfo()` | 81 | ALTA |

#### 4. **Dependencias Excesivas**
Constructor inyecta 3 servicios, pero métodos usan 7+ servicios adicionales vía `app()`.

#### 5. **Mezclado de Concerns**
- Validación mezclada con lógica de negocio
- Formateo de respuestas mezclado con operaciones
- Logging mezclado en todos lados

---

## 🏗️ Estructura de Refactorización Propuesta

### Fase 1: Separación por Responsabilidades

```
app/Http/Controllers/Apps/Extensions/
├── ExtensionViewController.php          (Vistas: overview, show, settings)
├── ExtensionInstallController.php       (Install, reinstall, uninstall)
├── ExtensionUpdateController.php        (Update, checkVersion, getUpdateInfo)
├── ExtensionBackupController.php        (Backups CRUD)
├── ExtensionConfigController.php        (Settings CRUD, reset)
├── ExtensionDevModeController.php       (Dev-mode enable/disable)
├── ExtensionMarketplaceController.php   (Marketplace API, details)
├── ExtensionSystemController.php        (GitHub token, rate limit, cache)
```

### Fase 2: Request Objects (Form Requests)

```
app/Http/Requests/Extensions/
├── InstallExtensionRequest.php
├── ReinstallExtensionRequest.php
├── UpdateExtensionRequest.php
├── UpdateSettingsRequest.php
├── CreateBackupRequest.php
├── RestoreBackupRequest.php
├── EnableDevModeRequest.php
└── UpdateGitHubTokenRequest.php
```

### Fase 3: Resource Objects (API Responses)

```
app/Http/Resources/Extensions/
├── ExtensionResource.php
├── ExtensionDetailResource.php
├── ExtensionListResource.php
├── BackupResource.php
└── MarketplaceExtensionResource.php
```

### Fase 4: DTOs (Data Transfer Objects)

```
app/Services/Extensions/DTOs/
├── ExtensionInfo.php
├── InstallOptions.php
├── UpdateInfo.php
├── MigrationInfo.php
├── BackupOptions.php
└── DevModeConfig.php
```

### Fase 5: Action Classes (Single Purpose)

```
app/Actions/Extensions/
├── Installation/
│   ├── InstallExtensionAction.php
│   ├── InstallLocalExtensionAction.php
│   ├── UninstallExtensionAction.php
│   └── ReinstallExtensionAction.php
├── Updates/
│   ├── UpdateExtensionAction.php
│   ├── CheckVersionAction.php
│   └── GetUpdateInfoAction.php
├── Backups/
│   ├── CreateBackupAction.php
│   ├── RestoreBackupAction.php
│   └── DeleteBackupAction.php
├── Configuration/
│   ├── UpdateSettingsAction.php
│   ├── ResetSettingsAction.php
│   └── ProcessConfigDataAction.php
└── DevMode/
    ├── EnableDevModeAction.php
    └── DisableDevModeAction.php
```

### Fase 6: Traits para Funcionalidad Común

```
app/Http/Controllers/Apps/Extensions/Concerns/
├── ValidatesExtensions.php          (extension existence, state)
├── RespondsWithJson.php             (unified JSON responses)
├── LogsExtensionActivity.php        (activity logging)
└── ClearsCaches.php                 (cache invalidation)
```

---

## 📋 Clasificación de Métodos por Tipo

### 1. **VIEW Controllers (Render HTML)**
| Método Actual | Nuevo Controlador | Líneas | Prioridad |
|---------------|-------------------|--------|-----------|
| `overview()` | `ExtensionViewController::overview()` | 123 | 🔴 ALTA |
| `index()` | `ExtensionViewController::index()` | 3 | 🟢 BAJA (redirect) |
| `show()` | `ExtensionViewController::show()` | 89 | 🔴 ALTA |
| `settings()` | `ExtensionViewController::settings()` | 28 | 🟡 MEDIA |
| `backups()` | `ExtensionViewController::backups()` | 21 | 🟡 MEDIA |
| `marketplace()` | `ExtensionMarketplaceController::index()` | 38 | 🟡 MEDIA |
| `extensionSettings()` | `ExtensionViewController::systemSettings()` | 39 | 🟡 MEDIA |

### 2. **API Controllers (Return JSON)**

#### Installation Domain
| Método Actual | Nuevo Controlador | Líneas | Prioridad |
|---------------|-------------------|--------|-----------|
| `install()` | `ExtensionInstallController::install()` | 54 | 🔴 ALTA |
| `installLocal()` | `ExtensionInstallController::installLocal()` | 108 | 🔴 ALTA |
| `reinstall()` | `ExtensionInstallController::reinstall()` | 68 | 🔴 ALTA |
| `uninstall()` | `ExtensionInstallController::uninstall()` | 19 | 🟡 MEDIA |

#### Update Domain
| Método Actual | Nuevo Controlador | Líneas | Prioridad |
|---------------|-------------------|--------|-----------|
| `update()` | `ExtensionUpdateController::update()` | 32 | 🔴 ALTA |
| `checkVersion()` | `ExtensionUpdateController::checkVersion()` | 49 | 🔴 ALTA |
| `getUpdateInfo()` | `ExtensionUpdateController::getUpdateInfo()` | 81 | 🔴 ALTA |

#### Backup Domain
| Método Actual | Nuevo Controlador | Líneas | Prioridad |
|---------------|-------------------|--------|-----------|
| `backup()` | `ExtensionBackupController::create()` | 59 | 🟡 MEDIA |
| `restoreBackup()` | `ExtensionBackupController::restore()` | 32 | 🟡 MEDIA |
| `deleteBackup()` | `ExtensionBackupController::destroy()` | 26 | 🟡 MEDIA |

#### Configuration Domain
| Método Actual | Nuevo Controlador | Líneas | Prioridad |
|---------------|-------------------|--------|-----------|
| `updateSettings()` | `ExtensionConfigController::update()` | 64 | 🔴 ALTA |
| `resetSettings()` | `ExtensionConfigController::reset()` | 44 | 🟡 MEDIA |
| `updateExtensionSettings()` | `ExtensionSystemController::updateSettings()` | 61 | 🟡 MEDIA |

#### Dev Mode Domain
| Método Actual | Nuevo Controlador | Líneas | Prioridad |
|---------------|-------------------|--------|-----------|
| `enableDevMode()` | `ExtensionDevModeController::enable()` | 43 | 🟡 MEDIA |
| `disableDevMode()` | `ExtensionDevModeController::disable()` | 36 | 🟡 MEDIA |

#### Marketplace Domain
| Método Actual | Nuevo Controlador | Líneas | Prioridad |
|---------------|-------------------|--------|-----------|
| `marketplaceApi()` | `ExtensionMarketplaceController::api()` | 45 | 🟡 MEDIA |
| `marketplaceDetails()` | `ExtensionMarketplaceController::details()` | 30 | 🟡 MEDIA |

#### System Domain
| Método Actual | Nuevo Controlador | Líneas | Prioridad |
|---------------|-------------------|--------|-----------|
| `enable()` | `ExtensionSystemController::enable()` | 17 | 🟢 BAJA |
| `disable()` | `ExtensionSystemController::disable()` | 17 | 🟢 BAJA |
| `seedDemo()` | `ExtensionSystemController::seedDemo()` | 24 | 🟢 BAJA |
| `forceCleanup()` | `ExtensionSystemController::forceCleanup()` | 37 | 🟡 MEDIA |
| `clearCache()` | `ExtensionSystemController::clearCache()` | 34 | 🟢 BAJA |
| `saveGithubToken()` | `ExtensionSystemController::saveGithubToken()` | 39 | 🟡 MEDIA |
| `updateGitHubToken()` | `ExtensionSystemController::updateGitHubToken()` | 55 | 🟡 MEDIA |
| `getRateLimit()` | `ExtensionSystemController::getRateLimit()` | 18 | 🟢 BAJA |
| `testGitHubConnection()` | `ExtensionSystemController::testGitHubConnection()` | 78 | 🟡 MEDIA |
| `refreshSettings()` | `ExtensionSystemController::refreshSettings()` | 16 | 🟢 BAJA |
| `saveSystemSettings()` | `ExtensionSystemController::saveSystemSettings()` | 48 | 🟡 MEDIA (DEPRECATED) |

### 3. **PROTECTED/PRIVATE Helpers (Extract to Services)**
| Método Actual | Destino | Líneas | Uso |
|---------------|---------|--------|-----|
| `processConfigData()` | `ProcessConfigDataAction` | 25 | Settings processing |
| `varExportFormatted()` | `ConfigFormatterService` | 38 | Config file formatting |
| `detectPendingMigrations()` | `MigrationDetectionService` | 41 | Migration analysis |
| `areMigrationsRequired()` | `MigrationDetectionService` | 20 | Migration validation |
| `updateComposerRepositories()` | ❌ ELIMINAR | 5 | DEPRECATED |

---

## 🔍 Redundancias y Código Obsoleto

### 1. **Código Duplicado Crítico**

#### Patrón: Validación de Extensión
```php
// Se repite 8 veces
$validator = Validator::make($request->all(), [
    'name' => 'required|string',
]);

if ($validator->fails()) {
    return response()->json([
        'success' => false,
        'message' => 'Invalid extension name',
    ], 422);
}
```
**Solución:** Form Requests + Trait `ValidatesExtensions`

#### Patrón: Respuesta JSON
```php
// Se repite 25+ veces
return response()->json([
    'success' => true/false,
    'message' => '...',
], $statusCode);
```
**Solución:** Trait `RespondsWithJson` con métodos helper:
- `successResponse($message, $data = [], $code = 200)`
- `errorResponse($message, $code = 400)`

#### Patrón: Activity Logging
```php
// Se repite 10+ veces
activity()
    ->causedBy(Auth::user())
    ->withProperties([...])
    ->log('...');
```
**Solución:** Trait `LogsExtensionActivity` con método `logExtensionActivity($action, $properties)`

#### Patrón: Cache Clear
```php
// Se repite 6 veces
\Artisan::call('cache:clear');
clearstatcache(true);
```
**Solución:** Trait `ClearsCaches` con método `clearAllCaches()`

### 2. **Código Obsoleto (ELIMINAR)**

```php
/**
 * @deprecated No longer used - repositories managed per-extension
 */
protected function updateComposerRepositories(string $repoMode, bool $useSymlink = true): void
{
    // DEPRECATED: Global repository updates removed
    Log::warning('updateComposerRepositories called but deprecated');
}
```

```php
/**
 * @deprecated Global settings deprecated - now using per-extension configuration
 */
public function saveSystemSettings(Request $request)
{
    // ... 48 líneas de código legacy
}
```

**Total líneas a eliminar:** ~53 líneas

### 3. **Lógica Redundante en Servicios**

Estos métodos delegan 100% al servicio, sin agregar valor:
- `enable()` → `$this->manager->enable()`
- `disable()` → `$this->manager->disable()`
- `seedDemo()` → `$this->manager->seed()`

**Solución:** Considerar eliminar estos métodos y usar rutas directas al servicio, o mantenerlos como thin wrappers si necesitamos middleware/logging específico.

---

## 📐 Plan de Refactorización (5 Fases)

### **FASE 1: Preparación (Sin Breaking Changes)** ⏱️ 2-3 horas

#### 1.1. Crear Estructura de Directorios
```bash
mkdir -p app/Http/Controllers/Apps/Extensions/Concerns
mkdir -p app/Http/Requests/Extensions
mkdir -p app/Http/Resources/Extensions
mkdir -p app/Services/Extensions/DTOs
mkdir -p app/Actions/Extensions/{Installation,Updates,Backups,Configuration,DevMode}
```

#### 1.2. Crear Traits Base
- [x] `ValidatesExtensions.php` ✅ COMPLETADO
- [x] `RespondsWithJson.php` ✅ COMPLETADO
- [x] `LogsExtensionActivity.php` ✅ COMPLETADO
- [x] `ClearsCaches.php` ✅ COMPLETADO

#### 1.3. Crear DTOs Base
- [x] `ExtensionInfo.php` ✅ COMPLETADO
- [x] `InstallOptions.php` ✅ COMPLETADO
- [x] `UpdateInfo.php` ✅ COMPLETADO
- [x] `MigrationInfo.php` ✅ COMPLETADO
- [x] `BackupOptions.php` ✅ COMPLETADO

#### 1.4. Crear Form Requests
- [x] `InstallExtensionRequest.php` ✅ COMPLETADO
- [x] `UpdateExtensionRequest.php` ✅ COMPLETADO
- [x] `UpdateSettingsRequest.php` ✅ COMPLETADO
- [x] `CreateBackupRequest.php` ✅ COMPLETADO

**Validación:** ✅ FASE 1 COMPLETADA - Commit 414ac94 - 13 archivos nuevos - Sin breaking changes

---

### **FASE 2: Extracción de Actions** ⏱️ 4-5 horas

#### 2.1. Installation Actions (PRIORIDAD ALTA 🔴)
- [x] `InstallExtensionAction.php` (líneas 177-234) ✅ COMPLETADO
- [x] `InstallLocalExtensionAction.php` (líneas 1461-1609) ✅ COMPLETADO
- [x] `ReinstallExtensionAction.php` (líneas 236-306) ✅ COMPLETADO
- [x] `UninstallExtensionAction.php` (líneas 343-369) ✅ COMPLETADO

#### 2.2. Update Actions (PRIORIDAD ALTA 🔴)
- [x] `UpdateExtensionAction.php` (líneas 690-731) ✅ COMPLETADO
- [x] `CheckVersionAction.php` (líneas 733-795) ✅ COMPLETADO
- [x] `GetUpdateInfoAction.php` (líneas 797-881) ✅ COMPLETADO

#### 2.3. Configuration Actions (PRIORIDAD ALTA 🔴)
- [x] `UpdateSettingsAction.php` (líneas 509-566) ✅ COMPLETADO
- [x] `ProcessConfigDataAction.php` (líneas 568-591) ✅ COMPLETADO
- [x] `ResetSettingsAction.php` (líneas 633-690) ✅ COMPLETADO

#### 2.4. Backup Actions (PRIORIDAD MEDIA 🟡)
- [x] `CreateBackupAction.php` (líneas 1003-1077) ✅ COMPLETADO
- [x] `RestoreBackupAction.php` (líneas 1079-1122) ✅ COMPLETADO
- [x] `DeleteBackupAction.php` (líneas 1124-1162) ✅ COMPLETADO

#### 2.5. DevMode Actions (PRIORIDAD MEDIA 🟡)
- [x] `EnableDevModeAction.php` (líneas 1362-1410) ✅ COMPLETADO
- [x] `DisableDevModeAction.php` (líneas 1412-1451) ✅ COMPLETADO

**Validación:** ✅ FASE 2 COMPLETADA - Commit 04581fa - 15 Actions - 1,293 líneas extraídas

---

### **FASE 3: Extracción de Controladores** ⏱️ 6-8 horas

#### 3.1. ExtensionInstallController (PRIORIDAD ALTA 🔴)
```php
<?php

namespace App\Http\Controllers\Apps\Extensions;

use App\Http\Controllers\Controller;
use App\Http\Requests\Extensions\InstallExtensionRequest;
use App\Http\Requests\Extensions\ReinstallExtensionRequest;
use App\Actions\Extensions\Installation\InstallExtensionAction;
use App\Actions\Extensions\Installation\InstallLocalExtensionAction;
use App\Actions\Extensions\Installation\ReinstallExtensionAction;
use App\Actions\Extensions\Installation\UninstallExtensionAction;
use App\Http\Controllers\Apps\Extensions\Concerns\{
    ValidatesExtensions,
    RespondsWithJson,
    LogsExtensionActivity
};

class ExtensionInstallController extends Controller
{
    use ValidatesExtensions, RespondsWithJson, LogsExtensionActivity;

    public function __construct(
        private InstallExtensionAction $installAction,
        private InstallLocalExtensionAction $installLocalAction,
        private ReinstallExtensionAction $reinstallAction,
        private UninstallExtensionAction $uninstallAction
    ) {
        $this->middleware(['auth', 'permission:core:marketplace:view']);
    }

    public function install(InstallExtensionRequest $request)
    {
        $result = $this->installAction->execute(
            $request->validated()
        );

        $this->logExtensionActivity('installed', [
            'extension' => $request->name,
            'type' => $request->type,
        ]);

        return $this->successResponse(
            $result->message,
            ['extension' => $result->extension]
        );
    }

    // ... otros métodos
}
```

#### 3.2. Controladores a Crear (en orden de prioridad)
1. [x] `ExtensionInstallController.php` (4 métodos) ✅ COMPLETADO
2. [x] `ExtensionUpdateController.php` (3 métodos) ✅ COMPLETADO
3. [x] `ExtensionConfigController.php` (2 métodos) ✅ COMPLETADO
4. [x] `ExtensionViewController.php` (6 métodos) ✅ COMPLETADO
5. [x] `ExtensionBackupController.php` (3 métodos) ✅ COMPLETADO
6. [x] `ExtensionDevModeController.php` (2 métodos) ✅ COMPLETADO
7. [x] `ExtensionMarketplaceController.php` (3 métodos) ✅ COMPLETADO
8. [x] `ExtensionSystemController.php` (11 métodos) ✅ COMPLETADO

**Validación:** ✅ FASE 3 COMPLETADA - Commit 4ada46d - 8 Controllers - 1,234 líneas

---

### **FASE 4: Actualización de Rutas** ⏱️ 2-3 horas

#### 4.1. Backup de Rutas Actuales
```bash
cp routes/web.php routes/web.php.backup-20251127
```

#### 4.2. Crear Archivo de Rutas Nuevo
`routes/extensions.php`:
```php
<?php

use App\Http\Controllers\Apps\Extensions\{
    ExtensionViewController,
    ExtensionInstallController,
    ExtensionUpdateController,
    ExtensionBackupController,
    ExtensionConfigController,
    ExtensionDevModeController,
    ExtensionMarketplaceController,
    ExtensionSystemController
};

Route::prefix('app/extensions')
    ->name('app.extensions.')
    ->middleware(['auth', 'permission:core:marketplace:view'])
    ->group(function () {

    // Views
    Route::get('/', [ExtensionViewController::class, 'overview'])->name('overview');
    Route::get('/{name}', [ExtensionViewController::class, 'show'])->name('show');
    Route::get('/{name}/backups', [ExtensionViewController::class, 'backups'])->name('backups');
    Route::get('/{name}/config', [ExtensionViewController::class, 'settings'])->name('config');
    Route::get('/settings', [ExtensionViewController::class, 'systemSettings'])->name('settings');

    // Marketplace
    Route::prefix('marketplace')->name('marketplace.')->group(function () {
        Route::get('/', [ExtensionMarketplaceController::class, 'index'])->name('index');
        Route::get('/api', [ExtensionMarketplaceController::class, 'api'])->name('api');
        Route::get('/{slug}', [ExtensionMarketplaceController::class, 'details'])->name('details');
    });

    // Installation
    Route::post('/install', [ExtensionInstallController::class, 'install'])->name('install');
    Route::post('/install-local', [ExtensionInstallController::class, 'installLocal'])->name('install-local');
    Route::post('/reinstall', [ExtensionInstallController::class, 'reinstall'])->name('reinstall');
    Route::post('/uninstall', [ExtensionInstallController::class, 'uninstall'])->name('uninstall');

    // Updates
    Route::post('/update', [ExtensionUpdateController::class, 'update'])->name('update');
    Route::get('/{name}/check-version', [ExtensionUpdateController::class, 'checkVersion'])->name('check-version');
    Route::get('/{name}/update-info', [ExtensionUpdateController::class, 'getUpdateInfo'])->name('update-info');

    // Configuration
    Route::put('/{name}/config', [ExtensionConfigController::class, 'update'])->name('config.update');
    Route::post('/{name}/config/reset', [ExtensionConfigController::class, 'reset'])->name('config.reset');

    // Backups
    Route::post('/{name}/backup', [ExtensionBackupController::class, 'create'])->name('backup');
    Route::post('/backups/restore', [ExtensionBackupController::class, 'restore'])->name('backups.restore');
    Route::delete('/backups/delete', [ExtensionBackupController::class, 'destroy'])->name('backups.destroy');

    // Dev Mode
    Route::post('/{name}/dev-mode/enable', [ExtensionDevModeController::class, 'enable'])->name('dev-mode.enable');
    Route::post('/{name}/dev-mode/disable', [ExtensionDevModeController::class, 'disable'])->name('dev-mode.disable');

    // System
    Route::post('/enable', [ExtensionSystemController::class, 'enable'])->name('enable');
    Route::post('/disable', [ExtensionSystemController::class, 'disable'])->name('disable');
    Route::post('/seed-demo', [ExtensionSystemController::class, 'seedDemo'])->name('seed-demo');
    Route::post('/force-cleanup/{name}', [ExtensionSystemController::class, 'forceCleanup'])->name('force-cleanup');
    Route::post('/clear-cache', [ExtensionSystemController::class, 'clearCache'])->name('clear-cache');

    // GitHub Integration
    Route::post('/test-connection', [ExtensionSystemController::class, 'testGitHubConnection'])->name('test-connection');
    Route::post('/update-token', [ExtensionSystemController::class, 'updateGitHubToken'])->name('update-token');
    Route::get('/rate-limit', [ExtensionSystemController::class, 'getRateLimit'])->name('rate-limit');
    Route::post('/settings/refresh', [ExtensionSystemController::class, 'refreshSettings'])->name('settings.refresh');
    Route::put('/settings', [ExtensionSystemController::class, 'updateSettings'])->name('settings.update');
});
```

#### 4.3. Incluir en `routes/web.php`
```php
// Extension Management Routes
require __DIR__.'/extensions.php';
```

**Validación:** `php artisan route:list` muestra todas las rutas correctamente

---

### **FASE 5: Limpieza y Deprecación** ⏱️ 1-2 horas

#### 5.1. Marcar Controlador Original como Deprecated
```php
/**
 * @deprecated Use new Controllers in App\Http\Controllers\Apps\Extensions namespace
 * This controller will be removed in v2.0.0
 */
class ExtensionManagerController extends Controller
{
    // Keep for backward compatibility (1-2 releases)
}
```

#### 5.2. Eliminar Código Obsoleto
- [x] `updateComposerRepositories()` (5 líneas)
- [x] `saveSystemSettings()` (48 líneas)
- Total: **53 líneas eliminadas**

#### 5.3. Crear Guía de Migración
`docs/migrations/EXTENSION-CONTROLLER-MIGRATION.md`

**Validación:** 
- Tests completos pasan (100%)
- Documentación actualizada
- CHANGELOG.md actualizado

---

## 🧪 Plan de Testing

### Estructura de Tests

```
tests/Feature/Extensions/
├── Installation/
│   ├── InstallExtensionTest.php
│   ├── InstallLocalExtensionTest.php
│   ├── ReinstallExtensionTest.php
│   └── UninstallExtensionTest.php
├── Updates/
│   ├── UpdateExtensionTest.php
│   ├── CheckVersionTest.php
│   └── GetUpdateInfoTest.php
├── Configuration/
│   ├── UpdateSettingsTest.php
│   └── ResetSettingsTest.php
├── Backups/
│   ├── CreateBackupTest.php
│   ├── RestoreBackupTest.php
│   └── DeleteBackupTest.php
├── DevMode/
│   ├── EnableDevModeTest.php
│   └── DisableDevModeTest.php
└── Marketplace/
    ├── MarketplaceApiTest.php
    └── MarketplaceDetailsTest.php

tests/Unit/Actions/Extensions/
├── Installation/
│   ├── InstallExtensionActionTest.php
│   ├── InstallLocalExtensionActionTest.php
│   └── ...
├── Updates/
│   └── ...
└── ...
```

### Coverage Mínimo Requerido

| Componente | Coverage |
|------------|----------|
| Actions | 95%+ |
| Controllers | 90%+ |
| Requests | 100% |
| DTOs | 85%+ |
| Traits | 90%+ |

### Test Suite Automatizado

```bash
# Crear script de testing
cat > tests/run-refactoring-tests.sh << 'EOF'
#!/bin/bash

echo "🧪 Running Extension Manager Refactoring Tests..."

# Unit Tests
echo "📦 Unit Tests (Actions)..."
php artisan test --filter="Actions\\Extensions" --coverage

# Feature Tests
echo "🔧 Feature Tests (Controllers)..."
php artisan test --filter="Extensions\\" --coverage

# Integration Tests
echo "🌐 Integration Tests..."
php artisan test --testsuite=Feature --filter="Extension" --coverage

# Generate Coverage Report
php artisan test --coverage-html=reports/coverage/extensions

echo "✅ Tests completed! Check reports/coverage/extensions/index.html"
EOF

chmod +x tests/run-refactoring-tests.sh
```

### Tests Críticos a Crear

#### 1. **Installation Flow** (PRIORIDAD 🔴)
```php
public function test_install_vcs_extension_successfully()
{
    // Given: Extension not installed
    // When: Install via VCS
    // Then: Extension installed, migrations run, enabled
}

public function test_install_local_extension_with_dev_mode()
{
    // Given: Local extension path exists
    // When: Install with dev-mode enabled
    // Then: Symlink created, extension active
}

public function test_reinstall_extension_with_fresh_mode()
{
    // Given: Extension installed with data
    // When: Reinstall with fresh mode
    // Then: Data wiped, migrations re-run, seeders executed
}
```

#### 2. **Update Flow** (PRIORIDAD 🔴)
```php
public function test_update_extension_with_required_migrations()
{
    // Given: Extension v1.0.0 installed
    // When: Update to v1.1.0 (has migrations)
    // Then: Migrations run, version updated
}

public function test_update_blocked_when_migrations_required_but_skipped()
{
    // Given: Extension v1.0.0
    // When: Update to v1.1.0 with migrate=false
    // Then: 400 error with migrations_required=true
}
```

#### 3. **Configuration Flow** (PRIORIDAD 🔴)
```php
public function test_update_settings_handles_checkboxes_correctly()
{
    // Given: Config with boolean values
    // When: Update with unchecked checkbox
    // Then: Boolean set to false (not deleted)
}

public function test_reset_settings_restores_vendor_defaults()
{
    // Given: Modified settings
    // When: Reset to defaults
    // Then: Vendor config copied, cache cleared
}
```

---

## 🔬 Seguimiento de Residuos

### Checklist de Limpieza Post-Refactorización

#### Código a Eliminar (Después de 2 releases)
- [ ] `ExtensionManagerController.php` (todo el archivo)
- [ ] Rutas antiguas en `routes/web.php`
- [ ] Vistas que referencien rutas antiguas

#### Configuración a Actualizar
- [ ] `config/bithoven-extensions.php` (si aplica)
- [ ] `.env.example` (documentar nuevas variables)

#### Documentación a Actualizar
- [ ] `DOCS/CORE/Extension-Manager/README.md`
- [ ] `DOCS/CORE/Extension-Manager/guides/EXTENSION-MANAGER-API.md`
- [ ] `.github/copilot-core/CORE-SYSTEM.md`

#### Assets JavaScript a Actualizar
- [ ] `resources/js/custom/extension-manager.js` (rutas API)
- [ ] `resources/views/app/extension-manager/**/*.blade.php` (rutas)

#### Tests Antiguos a Eliminar
- [ ] `tests/Feature/ExtensionManagerControllerTest.php` (si existe)

---

## 💡 Sugerencias de Buenas Prácticas

### 1. **Command Pattern para Operaciones Complejas**

En lugar de Actions simples, considerar Commands para operaciones transaccionales:

```php
// app/Commands/Extensions/InstallExtensionCommand.php
class InstallExtensionCommand
{
    public function __construct(
        public string $slug,
        public InstallOptions $options
    ) {}
}

// app/Handlers/Extensions/InstallExtensionHandler.php
class InstallExtensionHandler
{
    public function handle(InstallExtensionCommand $command): ExtensionInfo
    {
        DB::beginTransaction();
        try {
            // 1. Validate
            $this->validator->validate($command->slug);
            
            // 2. Install via Composer
            $this->composer->require($command->slug);
            
            // 3. Register
            $this->config->register($command->slug);
            
            // 4. Migrate
            if ($command->options->migrate) {
                $this->migrator->run($command->slug);
            }
            
            // 5. Seed
            $this->seeder->runBase($command->slug);
            
            // 6. Enable
            if ($command->options->enable) {
                $this->manager->enable($command->slug);
            }
            
            DB::commit();
            return ExtensionInfo::fromArray($this->manager->getInfo($command->slug));
            
        } catch (\Exception $e) {
            DB::rollBack();
            throw new ExtensionInstallationException($e->getMessage());
        }
    }
}
```

**Ventajas:**
- Transaccionalidad garantizada
- Rollback automático en errores
- Testing más fácil (mock del handler)
- Logs centralizados

### 2. **Events & Listeners para Side Effects**

```php
// app/Events/Extensions/ExtensionInstalled.php
class ExtensionInstalled
{
    public function __construct(
        public ExtensionInfo $extension,
        public User $installedBy
    ) {}
}

// app/Listeners/Extensions/ClearExtensionCaches.php
class ClearExtensionCaches
{
    public function handle(ExtensionInstalled $event): void
    {
        Cache::tags(['extensions', $event->extension->slug])->flush();
        Artisan::call('config:clear');
    }
}

// app/Listeners/Extensions/NotifyAdminsOfInstallation.php
class NotifyAdminsOfInstallation
{
    public function handle(ExtensionInstalled $event): void
    {
        Notification::send(
            User::admins()->get(),
            new ExtensionInstalledNotification($event->extension)
        );
    }
}

// EventServiceProvider.php
protected $listen = [
    ExtensionInstalled::class => [
        ClearExtensionCaches::class,
        NotifyAdminsOfInstallation::class,
        LogExtensionActivity::class,
    ],
];
```

**Ventajas:**
- Desacoplamiento total
- Side effects desacoplados de lógica core
- Fácil agregar/remover listeners

### 3. **Repository Pattern para Configuración**

```php
// app/Repositories/ExtensionConfigRepository.php
interface ExtensionConfigRepository
{
    public function get(string $slug): ?ExtensionConfig;
    public function save(string $slug, array $config): bool;
    public function reset(string $slug): bool;
    public function getDefault(string $slug): ?array;
}

// app/Repositories/FileExtensionConfigRepository.php
class FileExtensionConfigRepository implements ExtensionConfigRepository
{
    public function get(string $slug): ?ExtensionConfig
    {
        $configKey = str_replace('-', '_', $slug);
        $data = config($configKey);
        
        return $data ? ExtensionConfig::fromArray($data) : null;
    }
    
    public function save(string $slug, array $config): bool
    {
        $configKey = str_replace('-', '_', $slug);
        $configFile = config_path("{$configKey}.php");
        
        $content = "<?php\n\nreturn " . $this->formatter->format($config) . ";\n";
        
        File::put($configFile, $content);
        $this->clearCache();
        
        return true;
    }
}
```

**Ventajas:**
- Abstracción del storage (fácil cambiar a DB/Redis)
- Testing con mock repository
- Validación centralizada

### 4. **Pipeline Pattern para Instalación Multi-Step**

```php
// app/Pipelines/Extensions/InstallExtensionPipeline.php
class InstallExtensionPipeline
{
    protected array $steps = [
        ValidateExtensionStep::class,
        CheckDependenciesStep::class,
        InstallViaComposerStep::class,
        RegisterExtensionStep::class,
        RunMigrationsStep::class,
        RunBaseSeedersStep::class,
        EnableExtensionStep::class,
        ClearCachesStep::class,
    ];
    
    public function execute(InstallContext $context): InstallContext
    {
        return Pipeline::via('handle')
            ->send($context)
            ->through($this->steps)
            ->thenReturn();
    }
}

// app/Pipelines/Extensions/Steps/ValidateExtensionStep.php
class ValidateExtensionStep
{
    public function handle(InstallContext $context, Closure $next): InstallContext
    {
        if ($this->manager->isInstalled($context->slug)) {
            throw new ExtensionAlreadyInstalledException($context->slug);
        }
        
        return $next($context);
    }
}
```

**Ventajas:**
- Pasos independientes y testables
- Fácil agregar/remover/reordenar pasos
- Logs detallados por paso

### 5. **Value Objects para Tipos Complejos**

```php
// app/ValueObjects/Extensions/Version.php
class Version
{
    private function __construct(
        public readonly int $major,
        public readonly int $minor,
        public readonly int $patch,
        public readonly ?string $preRelease = null
    ) {}
    
    public static function fromString(string $version): self
    {
        // Parse "1.2.3-beta" -> new Version(1, 2, 3, 'beta')
    }
    
    public function isGreaterThan(Version $other): bool
    {
        // Semantic version comparison
    }
    
    public function toString(): string
    {
        return "{$this->major}.{$this->minor}.{$this->patch}" .
               ($this->preRelease ? "-{$this->preRelease}" : '');
    }
}

// Uso
$localVersion = Version::fromString('1.0.0');
$remoteVersion = Version::fromString('1.1.0');

if ($remoteVersion->isGreaterThan($localVersion)) {
    // Update available
}
```

**Ventajas:**
- Type safety
- Lógica de comparación encapsulada
- Inmutabilidad

### 6. **Service Container Bindings para DI**

```php
// app/Providers/ExtensionServiceProvider.php
public function register(): void
{
    // Bind interfaces
    $this->app->bind(
        ExtensionConfigRepository::class,
        FileExtensionConfigRepository::class
    );
    
    // Singleton services
    $this->app->singleton(ExtensionManager::class);
    $this->app->singleton(ExtensionMarketplace::class);
    
    // Context-aware bindings
    $this->app->when(InstallExtensionAction::class)
        ->needs('$backupEnabled')
        ->giveConfig('bithoven-extensions.auto_backup');
}
```

**Ventajas:**
- DI automática
- Fácil swap de implementaciones
- Testing con mocks

### 7. **API Resources para Respuestas Consistentes**

```php
// app/Http/Resources/Extensions/ExtensionResource.php
class ExtensionResource extends JsonResource
{
    public function toArray($request): array
    {
        return [
            'slug' => $this->slug,
            'name' => $this->name,
            'version' => $this->version,
            'description' => $this->description,
            'is_installed' => $this->installed,
            'is_active' => $this->active,
            'has_update' => $this->has_update,
            'remote_version' => $this->when($this->has_update, $this->remote_version),
            'migrations' => new MigrationInfoResource($this->whenLoaded('migrations')),
            'links' => [
                'self' => route('app.extensions.show', $this->slug),
                'config' => route('app.extensions.config', $this->slug),
                'backups' => route('app.extensions.backups', $this->slug),
            ],
        ];
    }
}

// Controller
public function show(string $slug)
{
    $extension = $this->manager->getInfo($slug);
    return new ExtensionResource($extension);
}
```

**Ventajas:**
- Estructura de respuesta consistente
- Conditional attributes (when/whenLoaded)
- HATEOAS links

### 8. **Rate Limiting para GitHub API**

```php
// app/Services/Extensions/RateLimitedGitHubClient.php
class RateLimitedGitHubClient
{
    public function get(string $url): Response
    {
        return Cache::remember(
            "github_api:{$url}",
            now()->addHour(),
            fn() => RateLimiter::attempt(
                "github_api:".auth()->id(),
                $perMinute = 60,
                fn() => Http::withToken($this->token)->get($url)
            )
        );
    }
}
```

**Ventajas:**
- Protección contra rate limits
- Cache automático
- Per-user limiting

### 9. **Atomic File Operations**

```php
// app/Services/Extensions/AtomicConfigWriter.php
class AtomicConfigWriter
{
    public function write(string $path, string $content): void
    {
        $tempPath = $path . '.tmp.' . uniqid();
        
        try {
            File::put($tempPath, $content);
            
            if (!File::exists($tempPath)) {
                throw new FileWriteException("Failed to write temp file");
            }
            
            // Atomic rename
            File::move($tempPath, $path);
            
            // Set permissions
            chmod($path, 0644);
            
            // Invalidate OPcache
            if (function_exists('opcache_invalidate')) {
                opcache_invalidate($path, true);
            }
            
        } catch (\Exception $e) {
            File::delete($tempPath);
            throw $e;
        }
    }
}
```

**Ventajas:**
- Escritura atómica (no corrupción)
- Rollback automático en error
- OPcache invalidation

### 10. **Validation Rules Personalizadas**

```php
// app/Rules/ExtensionExists.php
class ExtensionExists implements Rule
{
    public function passes($attribute, $value): bool
    {
        return app(ExtensionManager::class)->exists($value);
    }
    
    public function message(): string
    {
        return 'The extension :input does not exist.';
    }
}

// app/Rules/ExtensionNotInstalled.php
class ExtensionNotInstalled implements Rule
{
    public function passes($attribute, $value): bool
    {
        return !app(ExtensionManager::class)->isInstalled($value);
    }
    
    public function message(): string
    {
        return 'The extension :input is already installed.';
    }
}

// Request
public function rules(): array
{
    return [
        'name' => ['required', 'string', new ExtensionExists, new ExtensionNotInstalled],
    ];
}
```

**Ventajas:**
- Validación reutilizable
- Mensajes de error claros
- Lógica de validación encapsulada

---

## 📊 Métricas de Éxito

### KPIs de Refactorización

| Métrica | Antes | Meta | Verificación |
|---------|-------|------|--------------|
| **Líneas por controlador** | 2,090 | <300 | `cloc` |
| **Métodos por controlador** | 39 | <15 | Manual |
| **Complejidad ciclomática** | ALTA | BAJA | PHPMetrics |
| **Code coverage** | ~60% | >90% | PHPUnit |
| **Duplication** | ~15% | <5% | PHPCPD |
| **Número de controladores** | 1 | 8 | Manual |
| **Número de actions** | 0 | 15+ | Manual |
| **SOLID violations** | Múltiples | 0 | Manual review |

### Comandos de Verificación

```bash
# Lines of Code
cloc app/Http/Controllers/Apps/Extensions/

# Complexity (install PHPMetrics)
composer require --dev phpmetrics/phpmetrics
vendor/bin/phpmetrics --report-html=reports/metrics app/Http/Controllers/Apps/Extensions/

# Duplication (install PHPCPD)
composer require --dev sebastian/phpcpd
vendor/bin/phpcpd app/Http/Controllers/Apps/Extensions/

# Coverage
php artisan test --coverage --min=90

# Static Analysis (install PHPStan)
composer require --dev phpstan/phpstan
vendor/bin/phpstan analyse app/Http/Controllers/Apps/Extensions/ --level=8
```

---

## 🎯 Roadmap de Ejecución

### Sprint 1 (Semana 1): Preparación + Actions
- **Días 1-2:** FASE 1 (Estructura + Traits + DTOs)
- **Días 3-5:** FASE 2 (Actions de Installation + Updates)

**Entregable:** Actions testeadas, sin breaking changes

### Sprint 2 (Semana 2): Controllers + Routes
- **Días 1-3:** FASE 3 (Controladores principales)
- **Días 4-5:** FASE 4 (Rutas + Testing integración)

**Entregable:** Controladores funcionales, rutas activas

### Sprint 3 (Semana 3): Testing + Limpieza
- **Días 1-3:** FASE 5 (Deprecación + Tests completos)
- **Días 4-5:** QA, Documentación, Code Review

**Entregable:** Sistema refactorizado 100% testeado

### Tiempo Total Estimado
- **Desarrollo:** 12-15 días laborales
- **QA:** 3 días
- **Total:** ~3 semanas

---

## 📝 Notas Finales

### Riesgos Identificados

1. **Breaking Changes en Rutas:**
   - **Mitigación:** Mantener ambos controladores 2 releases
   - **Deprecation warnings** en respuestas JSON

2. **Dependencias de Vistas:**
   - **Mitigación:** Buscar todas las referencias con grep
   - Actualizar assets JS con rutas nuevas

3. **Tests Faltantes:**
   - **Mitigación:** Coverage mínimo 90% antes de merge
   - Tests de regresión para funcionalidad crítica

4. **Performance de Actions:**
   - **Mitigación:** Benchmark antes/después
   - Cache estratégico en operations lentas

### Siguientes Pasos

1. **Crear issue en GitHub** con este plan
2. **Estimar tiempo** con el equipo
3. **Crear branch:** `refactor/extension-manager-controller`
4. **Ejecutar FASE 1** y validar con tests
5. **Iterar** hasta completar todas las fases

### Recursos Adicionales

- [Laravel Best Practices](https://github.com/alexeymezenin/laravel-best-practices)
- [Clean Code PHP](https://github.com/jupeter/clean-code-php)
- [SOLID Principles in PHP](https://github.com/wataridori/solid-php-examples)
- [Laravel Actions Package](https://laravelactions.com/)

---

**Documento creado:** 27 de noviembre de 2025  
**Autor:** AI Agent (Claude Sonnet 4.5)  
**Para:** Equipo de Desarrollo Bithoven CPANEL  
**Proyecto:** Extension Manager Controller Refactoring  
**Versión:** 1.0.0

---

## ✅ Checklist de Inicio

Antes de comenzar la refactorización, verificar:

- [x] Branch `refactor/extension-manager-controller` creada
- [x] Tests actuales pasan (baseline: 79.5%)
- [x] Backup del controlador actual realizado
- [x] Métricas actuales registradas (LOC, baseline documentado)
- [x] Equipo alineado con el plan
- [x] Tiempo estimado aprobado
- [x] Herramientas de análisis instaladas (PHPMetrics, PHPStan)
- [x] Este documento revisado por tech lead

**¡Buena suerte con la refactorización! 🚀**
