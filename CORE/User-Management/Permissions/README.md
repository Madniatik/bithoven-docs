# Permissions - Sistema de Permisos

**Última actualización:** 26 de noviembre de 2025  
**Versión:** 1.7.0  
**Migration:** v2.0 COMPLETADA ✅ (69 permisos totales: 49 core + 20 extensions)

## 📋 Índice

- [Visión General](#visión-general)
- [Convenciones de Nomenclatura](#convenciones-de-nomenclatura)
- [Estructura de Permisos](#estructura-de-permisos)
- [Permisos Core](#permisos-core)
- [Permisos de Extensiones](#permisos-de-extensiones)
- [Gestión Automática](#gestión-automática)
- [API Reference](#api-reference)
- [Mejores Prácticas](#mejores-prácticas)

---

## Visión General

El sistema de permisos de Bithoven utiliza **Spatie Laravel Permission** con una estructura jerárquica de tres niveles que permite granularidad y escalabilidad.

### Principios Fundamentales

1. **Estructura Jerárquica:** `prefix:area:action`
2. **Dos Tipos:** Core (`core:*`) y Extensions (`extensions:*`)
3. **Auto-gestión:** Creación/eliminación automática con extensiones
4. **Cache:** Automático con Spatie Permission

### Características Principales

- ✅ **69 permisos totales** (49 core + 20 extensions)
- ✅ Estructura consistente `prefix:area:action` o `prefix:area:scope:action`
- ✅ Auto-creación de permisos de extensiones
- ✅ Auto-asignación a super-admin
- ✅ Auto-eliminación al desinstalar extensión
- ✅ Interfaz gráfica completa (Edit Role + Livewire Permissions Editor)
- ✅ Validación con regex
- ✅ Sistema de backup/rollback integrado

---

## Convenciones de Nomenclatura

### Formato Estándar

```
{prefix}:{area}:{scope}:{action}
```

### Ejemplos por Tipo

**Core Permissions:**
```
core:users:view
core:users:create
core:users:edit
core:users:delete
core:roles:manage
```

**Extension Permissions:**
```
extensions:tickets:base:view
extensions:tickets:base:create
extensions:tickets:categories:manage
extensions:llm-manager:base:view
extensions:llm-manager:models:manage
```

### Componentes del Nombre

| Componente | Descripción | Ejemplos |
|------------|-------------|----------|
| **Prefix** | Tipo de permiso | `core`, `extensions` |
| **Area** | Módulo/recurso | `users`, `roles`, `tickets`, `llm-manager` |
| **Scope** | Subcategoría (solo extensions) | `base`, `categories`, `models`, `providers` |
| **Action** | Acción permitida | `view`, `create`, `edit`, `delete`, `manage` |

### Acciones Estándar

| Acción | Descripción | Uso |
|--------|-------------|-----|
| `view` | Ver/listar recursos | READ |
| `create` | Crear nuevos recursos | CREATE |
| `edit` | Modificar recursos existentes | UPDATE |
| `delete` | Eliminar recursos | DELETE |
| `manage` | Gestión avanzada | Admin-level operations |

### Acciones Especiales

Además de las CRUD estándar, pueden existir acciones específicas:

```
core:users:impersonate
core:activity:view-logs
extensions:tickets:base:assign
extensions:tickets:base:close
```

---

## Estructura de Permisos

### Modelo de Base de Datos

**Tabla:** `permissions`

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `id` | bigint | ID único autoincrementable |
| `name` | varchar(255) | Nombre del permiso (unique) |
| `guard_name` | varchar(255) | Guard (default: 'web') |
| `created_at` | timestamp | Fecha de creación |
| `updated_at` | timestamp | Última actualización |

### Relaciones

```php
// Relación con roles
$permission->roles; // Roles que tienen este permiso

// Relación con usuarios (permisos directos)
$permission->users; // Usuarios con permiso directo
```

---

## Permisos Core

### Lista Completa (26 permisos)

#### User Management (13 permisos)

**Users:**
```
core:users:view
core:users:create
core:users:edit
core:users:delete
core:users:impersonate
```

**Roles:**
```
core:roles:view
core:roles:create
core:roles:edit
core:roles:delete
```

**Permissions:**
```
core:permissions:view
core:permissions:create
core:permissions:edit
core:permissions:delete
```

#### Activity Logs (1 permiso)

```
core:activity:view-logs
```

#### Extension Management (4 permisos)

```
core:extensions:view
core:extensions:install
core:extensions:manage
core:extensions:uninstall
```

#### Developer Tools (8 permisos)

```
core:developer:view-dashboard
core:developer:view-bugs
core:developer:manage-bugs
core:developer:view-tasks
core:developer:manage-tasks
core:developer:view-sessions
core:developer:view-logs
core:developer:manage-system
```

---

## Permisos de Extensiones

### Estructura de Scopes

Las extensiones organizan permisos en **scopes** (subcategorías):

```
extensions:{extension-name}:{scope}:{action}
```

### Ejemplos por Extensión

#### Tickets Extension (14 permisos)

**Base Scope:**
```
extensions:tickets:base:view
extensions:tickets:base:create
extensions:tickets:base:edit
extensions:tickets:base:delete
extensions:tickets:base:assign
extensions:tickets:base:close
```

**Categories Scope:**
```
extensions:tickets:categories:view
extensions:tickets:categories:create
extensions:tickets:categories:edit
extensions:tickets:categories:delete
extensions:tickets:categories:manage
```

**Settings Scope:**
```
extensions:tickets:settings:view
extensions:tickets:settings:edit
extensions:tickets:settings:manage
```

#### LLM Manager Extension (12 permisos)

**Base Scope:**
```
extensions:llm-manager:base:view
extensions:llm-manager:base:create
extensions:llm-manager:base:edit
extensions:llm-manager:base:delete
```

**Models Scope:**
```
extensions:llm-manager:models:view
extensions:llm-manager:models:manage
```

**Providers Scope:**
```
extensions:llm-manager:providers:view
extensions:llm-manager:providers:manage
```

**API Keys Scope:**
```
extensions:llm-manager:api-keys:view
extensions:llm-manager:api-keys:manage
```

**Settings Scope:**
```
extensions:llm-manager:settings:view
extensions:llm-manager:settings:manage
```

### Declaración en extension.json

```json
{
  "name": "tickets",
  "permissions": [
    "extensions:tickets:base:view",
    "extensions:tickets:base:create",
    "extensions:tickets:base:edit",
    "extensions:tickets:base:delete",
    "extensions:tickets:categories:view",
    "extensions:tickets:categories:manage"
  ]
}
```

---

## Gestión Automática

### Ciclo de Vida de Permisos de Extensiones

#### 1. Instalación

**Automático al instalar extensión:**

```php
// ExtensionManager::install()
1. Parsea extension.json
2. Extrae array permissions
3. Crea permisos en DB (si no existen)
4. Asigna a rol super-admin automáticamente
```

**Regex de Validación:**
```regex
/^extensions:([a-z0-9-]+):([a-z0-9-]+):([a-z0-9-]+)$/
```

#### 2. Durante Uso

- Los permisos se asignan a roles via UI
- Se verifican en controllers con policies
- Se verifican en routes con middleware
- Cache automático de Spatie

#### 3. Desinstalación

**Automático al desinstalar extensión:**

```php
// ExtensionManager::uninstall()
1. Obtiene permisos de extension.json
2. Revoca permisos de todos los roles
3. Elimina permisos de DB
4. Limpia cache de permisos
```

### Soporte para Múltiples Métodos de Instalación

El sistema funciona con:

- ✅ **Local (symlink):** Detecta extension.json en vendor/bithoven/
- ✅ **VCS (GitHub):** Parsea extension.json después de clonar
- ✅ **Composer:** Lee extension.json post-install

### Validación Automática

```php
// Valida formato del permiso
if (!preg_match('/^extensions:([a-z0-9-]+):([a-z0-9-]+):([a-z0-9-]+)$/', $permission)) {
    throw new \Exception("Invalid permission format: {$permission}");
}

// Valida que empiece con extensions:
if (!str_starts_with($permission, 'extensions:')) {
    throw new \Exception("Extension permissions must start with 'extensions:'");
}
```

---

## API Reference

### Verificación de Permisos

**En Controllers:**

```php
// Via Policy
$this->authorize('update', $user); // Verifica can('edit', User)

// Directo
if (!$user->can('core:users:edit')) {
    abort(403);
}

// Múltiples permisos (ANY)
if (!$user->hasAnyPermission(['core:users:view', 'core:users:edit'])) {
    abort(403);
}

// Múltiples permisos (ALL)
if (!$user->hasAllPermissions(['core:users:view', 'core:users:edit'])) {
    abort(403);
}
```

**En Rutas:**

```php
// Middleware permission
Route::middleware('permission:core:users:view')
    ->get('/users', [UserController::class, 'index']);

// Middleware role or permission
Route::middleware('role_or_permission:administrator|core:users:view')
    ->get('/users', [UserController::class, 'index']);
```

**En Vistas:**

```blade
@can('core:users:edit')
    <button>Edit User</button>
@endcan

@cannot('core:users:delete')
    <p>You cannot delete users</p>
@endcannot

{{-- Con modelo específico --}}
@can('update', $user)
    <button>Edit</button>
@endcan
```

### Asignación de Permisos

**A Roles:**

```php
// Asignar permiso individual
$role->givePermissionTo('core:users:view');

// Asignar múltiples
$role->givePermissionTo(['core:users:view', 'core:users:edit']);

// Sincronizar (reemplaza todos)
$role->syncPermissions(['core:users:view', 'core:users:edit']);

// Revocar permiso
$role->revokePermissionTo('core:users:delete');
```

**A Usuarios (directo):**

```php
// Asignar permiso directo a usuario
$user->givePermissionTo('core:users:impersonate');

// Verificar si tiene permiso (via rol o directo)
$user->hasPermissionTo('core:users:view');

// Verificar solo permisos directos
$user->hasDirectPermission('core:users:impersonate');
```

### Creación de Permisos

**Manual (solo para Core):**

```php
use Spatie\Permission\Models\Permission;

// Crear permiso único
Permission::create(['name' => 'core:new-feature:view']);

// Crear múltiples
$permissions = [
    'core:new-feature:view',
    'core:new-feature:create',
    'core:new-feature:edit',
];

foreach ($permissions as $permission) {
    Permission::firstOrCreate(['name' => $permission]);
}
```

**Automático (para Extensions):**

Los permisos de extensiones se crean **automáticamente** al instalar la extensión. NO se deben crear manualmente.

---

## Mejores Prácticas

### 1. Nombrado de Permisos

✅ **Hacer:**
```php
'core:users:view'
'core:roles:manage'
'extensions:tickets:base:create'
'extensions:llm-manager:models:manage'
```

❌ **No Hacer:**
```php
'view_users'              // ❌ Formato antiguo
'core:users-view'         // ❌ Guión en action
'extensions-tickets-view' // ❌ Guiones en lugar de dos puntos
'CORE:USERS:VIEW'         // ❌ Mayúsculas
```

### 2. Organización por Scopes

✅ **Hacer:**
```json
{
  "permissions": [
    "extensions:tickets:base:view",
    "extensions:tickets:base:create",
    "extensions:tickets:categories:view",
    "extensions:tickets:categories:manage"
  ]
}
```

❌ **No Hacer:**
```json
{
  "permissions": [
    "extensions:tickets:view",        // ❌ Falta scope
    "extensions:tickets:base-create"  // ❌ Formato incorrecto
  ]
}
```

### 3. Uso de Policies vs. Permisos Directos

✅ **Hacer:**
```php
// En Controller - usar Policies
$this->authorize('update', $user);

// En Vistas - usar Policies
@can('update', $user)

// Solo usar permisos directos si no hay policy
if ($user->can('core:activity:view-logs')) {
    //...
}
```

❌ **No Hacer:**
```php
// No verificar permisos directos si hay policy
if ($user->can('core:users:edit')) { // ❌ Usar policy
    //...
}
```

### 4. Caché de Permisos

✅ **Hacer:**
```php
// Limpiar cache después de cambios masivos
app()[\Spatie\Permission\PermissionRegistrar::class]->forgetCachedPermissions();

// Spatie lo hace automáticamente en:
// - syncPermissions()
// - givePermissionTo()
// - revokePermissionTo()
```

❌ **No Hacer:**
```php
// No modificar directamente la tabla pivot
DB::table('role_has_permissions')->insert(...); // ❌ No limpia cache
```

### 5. Extensiones - Declaración de Permisos

✅ **Hacer:**
```json
{
  "permissions": [
    "extensions:my-extension:base:view",
    "extensions:my-extension:base:create"
  ]
}
```

❌ **No Hacer:**
```php
// No crear permisos de extensión manualmente en seeders
Permission::create([
    'name' => 'extensions:my-extension:base:view'
]); // ❌ Usar extension.json
```

### 6. Super-Admin

El rol `administrator` recibe **automáticamente** todos los permisos de extensiones al instalar.

✅ **Hacer:**
```php
// El sistema lo hace automático
// No requiere acción manual
```

❌ **No Hacer:**
```php
// No asignar manualmente permisos a super-admin
$admin = Role::where('name', 'administrator')->first();
$admin->givePermissionTo('extensions:tickets:base:view'); // ❌ Automático
```

---

## Interfaz de Usuario

### Vista de Permisos en Roles

**URL:** `/user-management/roles/{role}` → Tab "Permissions"

**Características:**

1. **Tabs Organizados:**
   - Core Permissions
   - Extensions (una sección por extensión)

2. **Select All Granular:**
   - "Select All Core Permissions" → Todos los permisos core
   - "Select All" por extensión → Solo permisos de esa extensión

3. **Tabla de Permisos:**
   - Columnas: Category/Scope + View/Create/Edit/Delete/Manage + Other
   - Row toggles: Checkbox por categoría/scope
   - Títulos clickeables

4. **Guardado:**
   - Botón "Save Changes" con AJAX
   - Botón "Reset" para deshacer
   - Toast notifications
   - Actualización automática de contadores

### Gestión de Permisos

**URL:** `/user-management/permissions`

- Lista completa de permisos
- DataTable con búsqueda
- Muestra roles asignados por permiso
- CRUD completo (crear/editar/eliminar permisos)

**Nota:** Solo para permisos Core. Permisos de extensiones se gestionan via `extension.json`.

---

## Migración v2.0

### Resumen de Cambios

**Antes (v1.0):** 73 permisos  
**Ahora (v2.0):** 52 permisos  
**Reducción:** 29% (21 permisos eliminados)

### Cambios de Formato

**Antes:**
```
view users
create-roles
edit_permissions
manage-extensions
```

**Ahora:**
```
core:users:view
core:roles:create
core:permissions:edit
core:extensions:manage
```

### Permisos Eliminados

Se eliminaron permisos obsoletos de módulos no existentes:
- LLM Chat (módulo eliminado)
- AI Agents (no implementado)
- Permisos duplicados o redundantes

### Permisos Migrados

Todos los permisos activos se migraron al nuevo formato con:
- Prefijo `core:` o `extensions:`
- Estructura jerárquica clara
- Scopes para extensiones

### Archivos de Migración

- `database/seeders/RolesPermissionsSeeder.php` - Seeder actualizado
- `docs/PERMISSIONS-MIGRATION-REPORT.md` - Reporte detallado (obsoleto, ver DOCS/)

---

## Archivos Relacionados

### Core
- `app/Models/Permission.php` - Modelo Spatie
- `app/Policies/*Policy.php` - Políticas de autorización

### Seeders
- `database/seeders/RolesPermissionsSeeder.php` - Seeder de permisos core

### Extension Manager
- `app/Services/ExtensionManager.php` - Auto-gestión de permisos
- `app/Services/PermissionParser.php` - Parser de extension.json

### Vistas
- `resources/views/livewire/permission/role-permissions-editor.blade.php`
- `resources/views/app/user-management/permissions/`

---

## Soporte

Para más información:
- [Roles Documentation](../Roles/README.md)
- [Extension Manager](../../Extension-Manager/README.md)
- [Spatie Permission Docs](https://spatie.be/docs/laravel-permission)
