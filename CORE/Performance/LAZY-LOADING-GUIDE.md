# Lazy Loading & Eager Loading Guide

**Última actualización:** 28 de noviembre de 2025  
**Versión:** v1.8.0

---

## 🎯 Problema: N+1 Queries

### ¿Qué es N+1?
Ejecutar 1 query inicial + N queries adicionales para relaciones.

### Ejemplo del Problema
```php
// Controller
$users = User::all(); // 1 query: SELECT * FROM users (100 usuarios)

// Vista
@foreach($users as $user)
    {{ $user->role->name }} <!-- +100 queries: SELECT * FROM roles WHERE id = ? -->
@endforeach

// TOTAL: 101 queries para 100 usuarios
```

**Resultado:** Página de 100 usuarios tarda **500ms** en cargar.

---

## ✅ Solución: Eager Loading

### Mismo Ejemplo Optimizado
```php
// Controller
$users = User::with('role')->get(); // 2 queries:
// 1. SELECT * FROM users
// 2. SELECT * FROM roles WHERE id IN (1,2,3,...)

// Vista (sin cambios)
@foreach($users as $user)
    {{ $user->role->name }} <!-- Sin query, ya está cargado -->
@endforeach

// TOTAL: 2 queries para 100 usuarios
```

**Resultado:** Página tarda **30ms** = **16x más rápido**

---

## 📊 Estado Actual del Proyecto

### ✅ Eager Loading Implementado

#### 1. **UsersDataTable**
```php
// app/DataTables/UsersDataTable.php
public function query(User $model): QueryBuilder
{
    return $model->newQuery()
        ->with(['roles', 'permissions']); // ← Eager loading
}
```

**Evita:** N+1 en lista de usuarios (roles y permisos)

#### 2. **AccessReportsDataTable**
```php
// app/DataTables/AccessReportsDataTable.php
public function query(AccessReport $model): QueryBuilder
{
    return $model->newQuery()
        ->with('user'); // ← Eager loading
}
```

**Evita:** N+1 en reportes de acceso (usuario por cada registro)

#### 3. **NotificationsDataTable**
```php
// app/DataTables/NotificationsDataTable.php
public function query(Notification $model): QueryBuilder
{
    return $model->newQuery()
        ->with('user'); // ← Eager loading
}
```

**Evita:** N+1 en notificaciones

---

### 🔍 Áreas a Auditar (Potencial N+1)

#### 1. **Security Logs**
```bash
# Buscar en vistas
grep -r "\$log->user" resources/views/
```

**Riesgo:** Si SecurityLog muestra usuario en cada fila.

**Fix:**
```php
// app/DataTables/SecurityLogDataTable.php
public function query(SecurityLog $model): QueryBuilder
{
    return $model->newQuery()
        ->with('user'); // ← Agregar eager loading
}
```

#### 2. **Extension Dependencies**
```bash
grep -r "\$extension->dependencies" resources/views/
```

**Riesgo:** Si extensión carga dependencias en loop.

**Fix:**
```php
// app/Services/ExtensionManager.php
public function getInstalled(): array
{
    return Extension::with('dependencies')->get(); // Si hay relación
}
```

---

## 🎨 Patrones de Eager Loading

### Pattern 1: Simple With
```php
// Cargar una relación
$users = User::with('role')->get();

// Cargar múltiples relaciones
$users = User::with(['role', 'permissions', 'profile'])->get();
```

### Pattern 2: Nested Eager Loading
```php
// Cargar relaciones anidadas
$users = User::with(['role.permissions'])->get();

// Acceso en vista
$user->role->permissions->pluck('name');
```

### Pattern 3: Eager Loading Condicional
```php
// Cargar solo si se va a usar
$users = User::all();

if ($needsRoles) {
    $users->load('role'); // Lazy eager loading
}
```

### Pattern 4: Eager Loading con Constraints
```php
// Cargar solo relaciones específicas
$users = User::with([
    'notifications' => function ($query) {
        $query->where('read_at', null)->latest()->limit(5);
    }
])->get();
```

### Pattern 5: Eager Loading en Modelos
```php
// app/Models/User.php
class User extends Model
{
    // SIEMPRE cargar role (muy común)
    protected $with = ['role'];
    
    // Deshabilitar en casos específicos
    $users = User::without('role')->get();
}
```

---

## 🔍 Detección de N+1

### Método 1: Laravel Debugbar (Recomendado)
```bash
composer require barryvdh/laravel-debugbar --dev
```

**Uso:**
1. Navegar a la página sospechosa
2. Abrir Debugbar (bottom toolbar)
3. Tab "Queries" → Ver lista de queries
4. **Red Flag:** Queries idénticas con solo el WHERE diferente

**Ejemplo de N+1 detectado:**
```
SELECT * FROM roles WHERE id = 1
SELECT * FROM roles WHERE id = 2
SELECT * FROM roles WHERE id = 1  ← Duplicado!
SELECT * FROM roles WHERE id = 3
... (97 más)
```

### Método 2: Query Logging Manual
```php
// AppServiceProvider.php
use Illuminate\Support\Facades\DB;

public function boot(): void
{
    if (app()->environment('local')) {
        DB::listen(function ($query) {
            if ($query->time > 100) { // Queries > 100ms
                Log::warning('Slow query detected', [
                    'sql' => $query->sql,
                    'bindings' => $query->bindings,
                    'time' => $query->time . 'ms',
                ]);
            }
        });
    }
}
```

### Método 3: Count Queries en Tests
```php
// tests/Feature/UserListTest.php
public function test_user_list_has_no_n_plus_1()
{
    User::factory()->count(20)->create();
    
    DB::enableQueryLog();
    
    $this->get(route('users.index'));
    
    $queries = DB::getQueryLog();
    
    // Máximo 5 queries esperadas:
    // 1. Users
    // 2. Roles
    // 3. Permissions
    // 4. Session
    // 5. Auth
    $this->assertLessThan(6, count($queries));
}
```

---

## 📋 Eager Loading Audit Checklist

### DataTables (Alta Prioridad)
- [x] UsersDataTable - `with(['roles', 'permissions'])`
- [x] AccessReportsDataTable - `with('user')`
- [x] NotificationsDataTable - `with('user')`
- [ ] SecurityLogDataTable - ¿Usa `user`?
- [ ] PermissionsDataTable - ¿Usa `roles`?
- [ ] RolesDataTable - ¿Usa `permissions`?

### Controllers (Media Prioridad)
- [ ] UserManagementController::show() - ¿Carga roles/permissions?
- [ ] AccessReportController::index() - ¿Carga user en stats?
- [ ] ExtensionManagerController - ¿Carga dependencies?

### Livewire Components (Media Prioridad)
- [x] NotificationDropdown - OK (usa service con cache)
- [ ] UserProfileWidget - ¿Carga roles?

### API Endpoints (Baja Prioridad)
- [ ] /api/users - ¿Eager loading?
- [ ] /api/notifications - ¿Eager loading?

---

## 🚀 Plan de Auditoría

### Fase 1: Instalar Debugbar (5 min)
```bash
composer require barryvdh/laravel-debugbar --dev
php artisan vendor:publish --provider="Barryvdh\Debugbar\ServiceProvider"
```

### Fase 2: Navegación Manual (30 min)
1. Listar usuarios → Check queries
2. Ver perfil usuario → Check queries
3. Listar notificaciones → Check queries
4. Listar access reports → Check queries
5. Listar security logs → Check queries
6. Listar extensiones → Check queries

### Fase 3: Identificar N+1 (30 min)
- Buscar patterns: `SELECT * FROM table WHERE id = ?` repetido
- Anotar controlador/vista afectada
- Calcular impacto (cantidad de queries extra)

### Fase 4: Implementar Fixes (1-2 horas)
```php
// Para cada N+1 detectado:

// 1. Identificar relación
$users = User::all(); // ← Aquí
foreach ($users as $user) {
    $user->role->name; // ← Usa 'role'
}

// 2. Agregar with()
$users = User::with('role')->get(); // ← Fix

// 3. Verificar en Debugbar (queries deben bajar)
```

### Fase 5: Testing (30 min)
```php
// Agregar tests para prevenir regresiones
public function test_no_n_plus_1_in_user_list()
{
    User::factory()->count(50)->create();
    
    DB::enableQueryLog();
    $this->get(route('users.index'));
    
    $this->assertLessThan(10, count(DB::getQueryLog()));
}
```

---

## 📊 Impacto Estimado

| Área | Queries Antes | Queries Después | Mejora | Time Saved |
|------|---------------|-----------------|--------|------------|
| User List (100) | 201 | 3 | **67x** | 450ms |
| Access Reports (50) | 51 | 2 | **25x** | 200ms |
| Notifications (20) | 21 | 2 | **10x** | 80ms |
| Security Logs (100) | 101 | 2 | **50x** | 350ms |

**Total time saved:** ~1 segundo por página cargada

---

## ⚠️ Pitfalls Comunes

### 1. Over-Eager Loading
```php
// ❌ MALO: Cargar todo "por si acaso"
$users = User::with(['roles', 'permissions', 'profile', 'settings', 'logs'])->get();

// ✅ BUENO: Solo lo que se usa
$users = User::with('role')->get(); // Solo role se usa en la vista
```

### 2. Eager Loading en Loops
```php
// ❌ MALO
foreach ($users as $user) {
    $user->load('role'); // N queries
}

// ✅ BUENO
$users->load('role'); // 1 query para todos
```

### 3. Olvidar Relaciones Anidadas
```php
// ❌ MALO: N+1 oculto en segundo nivel
$users = User::with('role')->get();
foreach ($users as $user) {
    $user->role->permissions; // ← N+1 aquí!
}

// ✅ BUENO
$users = User::with('role.permissions')->get();
```

---

## 🔗 Referencias

- **Laravel Eager Loading:** https://laravel.com/docs/11.x/eloquent-relationships#eager-loading
- **N+1 Problem:** https://laravel-news.com/eloquent-eager-loading
- **Debugbar:** https://github.com/barryvdh/laravel-debugbar
- **Performance Best Practices:** https://laravel.com/docs/11.x/optimization
