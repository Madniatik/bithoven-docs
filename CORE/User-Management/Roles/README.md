# Roles - Sistema de Gestión de Roles

**Última actualización:** 26 de noviembre de 2025  
**Versión:** 1.7.0

## 📋 Índice

- [Visión General](#visión-general)
- [Convenciones de Nomenclatura](#convenciones-de-nomenclatura)
- [Estructura de Roles](#estructura-de-roles)
- [Roles del Sistema](#roles-del-sistema)
- [Gestión de Roles](#gestión-de-roles)
- [API Reference](#api-reference)
- [Mejores Prácticas](#mejores-prácticas)

---

## Visión General

El sistema de roles de Bithoven utiliza **Spatie Laravel Permission** para gestionar roles y permisos de usuarios. Cada rol define un conjunto de permisos que determinan qué acciones puede realizar un usuario en el sistema.

### Características Principales

- ✅ Roles basados en **Spatie Permission**
- ✅ Roles de sistema **protegidos** (no eliminables)
- ✅ Asignación dinámica de permisos
- ✅ Sincronización automática con extensiones
- ✅ Interfaz gráfica completa (CRUD + asignación de permisos)
- ✅ Políticas de autorización integradas

---

## Convenciones de Nomenclatura

### Formato de Nombres

**System Name (DB):**
```
slug-format-lowercase
```

**Display Name (UI):**
```
Title Case Format
```

**Ejemplos:**
```php
// System Name → Display Name
'administrator' → 'Administrator'
'content-editor' → 'Content Editor'
'customer-support' → 'Customer Support'
```

### Reglas de Nomenclatura

1. **System Name:**
   - Solo minúsculas
   - Palabras separadas por guiones (`-`)
   - Sin espacios ni caracteres especiales
   - Máximo 255 caracteres

2. **Display Name (Alias):**
   - Primera letra de cada palabra en mayúscula
   - Puede contener espacios
   - Máximo 255 caracteres

3. **Description:**
   - Descripción clara del propósito del rol
   - Máximo 500 caracteres
   - Opcional pero recomendado

---

## Estructura de Roles

### Modelo de Base de Datos

**Tabla:** `roles`

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `id` | bigint | ID único autoincrementable |
| `name` | varchar(255) | System name (unique, slug format) |
| `alias` | varchar(255) | Display name (nullable) |
| `description` | text | Descripción del rol (nullable) |
| `guard_name` | varchar(255) | Guard (default: 'web') |
| `created_at` | timestamp | Fecha de creación |
| `updated_at` | timestamp | Última actualización |

### Relaciones

```php
// Relación con usuarios
$role->users; // Usuarios asignados al rol

// Relación con permisos
$role->permissions; // Permisos asignados al rol
```

### Computed Properties

```php
// Display name calculado
$role->display_name; // Retorna alias si existe, sino capitaliza name
```

---

## Roles del Sistema

### Roles Protegidos

Estos roles **NO pueden ser eliminados** por seguridad:

```php
'administrator'  // Super admin con todos los permisos
'admin'          // Administrador general
'user'           // Usuario básico
```

### Verificación de Roles del Sistema

```php
$systemRoles = ['administrator', 'admin', 'user'];
$isSystemRole = in_array($role->name, $systemRoles);
```

---

## Gestión de Roles

### Interfaz Web

**URL:** `/user-management/roles`

#### Vista Index (Lista de Roles)
- **Ruta:** `GET /user-management/roles`
- **Vista:** `app.user-management.roles.list`
- **Componente:** Livewire DataTable
- **Acciones:**
  - Ver detalles de rol
  - Editar rol (modal)
  - Eliminar rol (si no es sistema)
  - Crear nuevo rol

#### Vista Show (Detalles del Rol)
- **Ruta:** `GET /user-management/roles/{role}`
- **Vista:** `app.user-management.roles.show`
- **Tabs:**
  - **Overview:** Información general + muestra de permisos
  - **Edit Role:** Edición inline (name, alias, description)
  - **Users:** DataTable de usuarios asignados
  - **Permissions:** Editor de permisos con tabs Core/Extensions

#### Características de la UI

**Header:**
- Icono de shield
- Display name + badge "System Role" (si aplica)
- Descripción
- Botón "Delete Role" (si no es sistema)

**Estadísticas:**
- Users Assigned
- Permissions (total)
- System Name
- Created Date

**Tab Overview:**
- Grid de 2 columnas: Role Information + Permission Sample
- Muestra información básica + 8 permisos aleatorios

**Tab Edit Role:**
- Formulario inline con 3 campos
- Guardado con AJAX
- Transformación automática de `name` a slug format
- Actualización en tiempo real del header

**Tab Users:**
- DataTable con búsqueda
- Asignar usuarios (modal)
- Remover usuarios (confirmación SweetAlert)
- Búsqueda por nombre/email

**Tab Permissions:**
- Sub-tabs: Core Permissions / Extensions
- "Select All" por sección
- "Select All" por cada extensión
- Checkboxes por fila (categoría/scope)
- Columnas: View, Create, Edit, Delete, Manage, Other
- Guardado con AJAX
- Botón "Reset" para deshacer cambios

---

## API Reference

### Controlador

**Clase:** `App\Core\Foundation\Controllers\RoleManagementController`

#### Métodos Principales

```php
// Listar roles
public function index()

// Ver detalles de un rol
public function show(Role $role, UsersAssignedRoleDataTable $usersDataTable)

// Actualizar rol
public function update(Request $request, Role $role)

// Eliminar rol
public function destroy(Role $role)

// Asignar usuarios al rol
public function assignUsers(Request $request, Role $role)

// Remover usuario del rol
public function removeUser(Role $role, User $user)

// DataTable de usuarios asignados
public function usersDataTable(Role $role, UsersAssignedRoleDataTable $dataTable)
```

### Validaciones

**Creación/Actualización de Rol:**

```php
[
    'name' => ['required', 'string', 'max:255', 'unique:roles,name,{id}'],
    'alias' => ['nullable', 'string', 'max:255'],
    'description' => ['nullable', 'string', 'max:500'],
]
```

**Transformación Automática:**
```php
// Backend
$roleName = strtolower(trim($validated['name']));
$roleName = preg_replace('/\s+/', '-', $roleName);

// Frontend (JavaScript)
roleNameInput.addEventListener('blur', function() {
    this.value = this.value.toLowerCase().trim().replace(/\s+/g, '-');
});
```

### Políticas de Autorización

**Clase:** `App\Policies\RolePolicy`

```php
viewAny(User $user)    // Ver lista de roles
view(User $user, Role $role)       // Ver detalles de un rol
create(User $user)     // Crear nuevo rol
update(User $user, Role $role)     // Editar rol
delete(User $user, Role $role)     // Eliminar rol
```

**Uso en Controladores:**
```php
$this->authorize('update', $role);
```

**Uso en Vistas:**
```blade
@can('update', $role)
    <!-- Contenido solo si tiene permiso -->
@endcan
```

---

## Mejores Prácticas

### 1. Nombrado de Roles

✅ **Hacer:**
```php
'customer-support'
'content-editor'
'sales-manager'
```

❌ **No Hacer:**
```php
'CustomerSupport'    // No usar CamelCase
'customer_support'   // No usar snake_case
'Customer Support'   // No usar espacios
```

### 2. Asignación de Permisos

✅ **Hacer:**
```php
// Sincronizar permisos (reemplaza todos)
$role->syncPermissions(['core:users:view', 'core:users:create']);

// Dar permiso individual
$role->givePermissionTo('core:users:edit');

// Revocar permiso individual
$role->revokePermissionTo('core:users:delete');
```

❌ **No Hacer:**
```php
// No asignar permisos directamente a la tabla pivot
DB::table('role_has_permissions')->insert(...); // ❌
```

### 3. Verificación de Roles

✅ **Hacer:**
```php
// Verificar si es rol del sistema
$systemRoles = ['administrator', 'admin', 'user'];
if (in_array($role->name, $systemRoles)) {
    // Es rol del sistema
}

// Usar políticas
$this->authorize('delete', $role);
```

❌ **No Hacer:**
```php
// No eliminar roles sin verificar
$role->delete(); // ❌ Puede eliminar roles del sistema
```

### 4. Display Names

✅ **Hacer:**
```php
// Usar display_name computed property
echo $role->display_name; // "Content Editor"

// Capitalizar manualmente si es necesario
echo ucwords($role->name); // Fallback
```

❌ **No Hacer:**
```php
// No mostrar el system name directamente en la UI
echo $role->name; // ❌ Muestra "content-editor"
```

### 5. Creación de Roles

✅ **Hacer:**
```php
$role = Role::create([
    'name' => 'customer-support',
    'alias' => 'Customer Support',
    'description' => 'Handles customer inquiries and support tickets'
]);

// Asignar permisos después de crear
$role->syncPermissions([...]);
```

❌ **No Hacer:**
```php
// No crear roles sin alias
Role::create(['name' => 'customer-support']); // ❌ Falta alias

// No olvidar asignar permisos
Role::create([...]); // ❌ Sin permisos asignados
```

---

## Extensiones

Cada extensión puede declarar roles específicos en su `extension.json`:

```json
{
  "roles": [
    {
      "name": "ticket-agent",
      "alias": "Ticket Agent",
      "description": "Manages support tickets",
      "permissions": [
        "extensions:tickets:base:view",
        "extensions:tickets:base:create",
        "extensions:tickets:base:edit"
      ]
    }
  ]
}
```

**Comportamiento:**
- Los roles de extensión se crean automáticamente al instalar
- Se eliminan automáticamente al desinstalar la extensión
- Los permisos se asignan automáticamente al rol

**Ver más:** [Extension Manager - Roles](../../Extension-Manager/guides/ROLES-CONVENTIONS.md)

---

## Archivos Relacionados

### Controladores
- `app/Core/Foundation/Controllers/RoleManagementController.php`

### Modelos
- `app/Models/Role.php`

### Políticas
- `app/Policies/RolePolicy.php`

### Vistas
- `resources/views/app/user-management/roles/`
  - `list.blade.php` - Lista de roles
  - `show.blade.php` - Detalles del rol
  - `partials/structure/_header.blade.php` - Header con stats
  - `partials/tabs/_overview.blade.php` - Tab Overview
  - `partials/tabs/_edit.blade.php` - Tab Edit Role
  - `partials/tabs/_users.blade.php` - Tab Users
  - `partials/tabs/_permissions.blade.php` - Tab Permissions

### Componentes Livewire
- `app/Livewire/Permission/RoleModal.php` - Modal CRUD roles
- `app/Livewire/Permission/RolePermissionsEditor.php` - Editor permisos

### DataTables
- `app/DataTables/UsersAssignedRoleDataTable.php`

### Rutas
- `routes/web.php` - Resource routes para roles

---

## Soporte

Para más información sobre permisos, consulta:
- [Permissions Documentation](../../Permissions/README.md)
- [Extension Manager](../../Extension-Manager/README.md)
- [User Management Overview](../README.md)
