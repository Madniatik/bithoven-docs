# Cache Strategy - BITHOVEN CPANEL

**Última actualización:** 28 de noviembre de 2025  
**Versión:** v1.8.0

---

## 🎯 Principios de Cache

### 1. **Cache en Producción, NO en Desarrollo**
```php
if (app()->environment('production')) {
    return Cache::remember('key', $ttl, fn() => $expensiveOperation());
}
return $expensiveOperation(); // Sin cache en development
```

### 2. **Invalidar Cache cuando cambian datos**
```php
public function update(User $user, array $data): void
{
    $user->update($data);
    Cache::forget("user.{$user->id}.profile");
    Cache::tags(['users'])->flush(); // Si usas tags
}
```

### 3. **TTL (Time To Live) apropiado**
- **Datos estáticos:** Forever o 24h (roles, permissions)
- **Datos semi-estáticos:** 1h (marketplace, extensions)
- **Datos dinámicos:** 5-10min (notifications, stats)
- **Datos en tiempo real:** NO cachear (live counters)

---

## 📊 Cache Layers Implementadas

### Layer 1: Application Cache (Laravel Cache)

#### **Extension Manager**
```php
// app/Services/ExtensionManager.php
public function getInstalled(): array
{
    return Cache::remember('extensions.installed', 86400, function () {
        return $this->scanInstalledExtensions(); // Filesystem scan: 200ms
    });
}

public function getMarketplaceExtensions(): array
{
    return Cache::remember('extensions.marketplace', 3600, function () {
        return $this->githubApi->searchExtensions(); // API call: 2-5s
    });
}

// Invalidación
public function install(string $name): void
{
    // ... install logic
    Cache::forget('extensions.installed');
    Cache::forget('extensions.marketplace');
}
```

**Beneficio:** 2000ms → 2ms (API/filesystem) = **1000x**

#### **Notifications**
```php
// app/Livewire/NotificationDropdown.php
public function loadNotifications(NotificationService $service): void
{
    $user = auth()->user();
    
    $this->notifications = Cache::remember(
        "user.{$user->id}.notifications.latest",
        300, // 5 minutos
        fn() => $service->getLatestForUser($user, 10)
    );
}

// Invalidación cuando se crea/lee notificación
public function markAsRead(int $notificationId): void
{
    // ... mark as read
    Cache::forget("user.{auth()->id()}.notifications.latest");
}
```

**Beneficio:** 100ms → 2ms (query) = **50x**

#### **Theme Assets**
```php
// app/Core/Theme.php
public function getAssets(): array
{
    if (app()->environment('production')) {
        return Cache::rememberForever('theme.assets.compiled', 
            fn() => $this->compileAssets()
        );
    }
    return $this->compileAssets(); // Sin cache en dev
}
```

**Beneficio:** 50ms → 1ms = **50x**

---

### Layer 2: Framework Cache (Laravel Optimize)

#### **Route Cache**
```bash
php artisan route:cache
```
- Compila routes/web.php a un solo archivo
- **Beneficio:** 20ms → 2ms por request = **10x**
- **Cuando:** Solo en producción
- **Invalidar:** `php artisan route:clear` o re-deploy

#### **View Cache**
```bash
php artisan view:cache
```
- Pre-compila todas las vistas Blade
- **Beneficio:** 15ms → 3ms por vista = **5x**
- **Cuando:** Solo en producción
- **Invalidar:** `php artisan view:clear`

#### **Config Cache**
```bash
php artisan config:cache
```
- Merge todos los config/* en un archivo
- **Beneficio:** 30ms → 1ms = **30x**
- **Cuando:** Solo en producción
- **⚠️ IMPORTANTE:** `env()` NO funciona fuera de config/

---

## 🎨 Patrones de Cache

### Pattern 1: Simple Remember
```php
$users = Cache::remember('active.users', 3600, function () {
    return User::where('is_active', true)->get();
});
```

### Pattern 2: Cache con Tags (Redis/Memcached required)
```php
// Guardar
Cache::tags(['users', 'active'])->put('user.list', $users, 3600);

// Invalidar por tag
Cache::tags(['users'])->flush(); // Limpia todo cache de users
```

### Pattern 3: Cache condicional
```php
public function getDashboardStats(): array
{
    if (!app()->environment('production')) {
        return $this->calculateStats(); // Sin cache en dev
    }
    
    return Cache::remember('dashboard.stats', 600, 
        fn() => $this->calculateStats()
    );
}
```

### Pattern 4: Cache con invalidación manual
```php
class UserService
{
    private function getCacheKey(int $userId): string
    {
        return "user.{$userId}.profile";
    }
    
    public function getProfile(int $userId): User
    {
        return Cache::remember(
            $this->getCacheKey($userId),
            3600,
            fn() => User::with('roles', 'permissions')->findOrFail($userId)
        );
    }
    
    public function updateProfile(int $userId, array $data): void
    {
        $user = User::findOrFail($userId);
        $user->update($data);
        
        Cache::forget($this->getCacheKey($userId)); // Invalidar
    }
}
```

---

## 📋 Cache Inventory (Estado Actual)

### ✅ Implementado
| Área | Key | TTL | Size | Hit Rate |
|------|-----|-----|------|----------|
| Extensions Installed | `extensions.installed` | 24h | ~5KB | 99% |
| Extensions Marketplace | `extensions.marketplace` | 1h | ~50KB | 95% |
| User Notifications | `user.{id}.notifications.latest` | 5min | ~2KB | 90% |
| Theme Assets | `theme.assets.compiled` | Forever | ~10KB | 100% |

### 🔄 Pendiente de Implementar
| Área | Key Propuesta | TTL | Prioridad |
|------|---------------|-----|-----------|
| Dashboard Stats | `dashboard.stats` | 10min | Alta |
| Roles List | `roles.all` | 1h | Alta |
| Permissions List | `permissions.all` | 1h | Alta |
| Access Report Stats | `access_reports.stats.{period}` | 10min | Media |
| User Count | `users.count.active` | 10min | Baja |

---

## 🚀 Plan de Implementación

### Fase 1: Dashboard Stats (Alta Prioridad)
```php
// app/Http/Controllers/DashboardController.php
public function index()
{
    $stats = Cache::remember('dashboard.stats', 600, function () {
        return [
            'users' => User::count(),
            'active_users' => User::where('is_active', true)->count(),
            'notifications' => Notification::unread()->count(),
            'extensions' => $this->extensionManager->getInstalled(),
        ];
    });
    
    return view('dashboard', compact('stats'));
}
```

### Fase 2: Roles & Permissions (Alta Prioridad)
```php
// app/Http/Controllers/Admin/UserManagementController.php
public function create()
{
    $roles = Cache::remember('roles.all', 3600, 
        fn() => Role::all()
    );
    
    return view('admin.users.create', compact('roles'));
}

// Invalidación en RoleController
public function store(Request $request)
{
    $role = Role::create($request->validated());
    Cache::forget('roles.all');
    
    return redirect()->route('admin.roles.index');
}
```

---

## 🛠️ Cache Drivers

### Desarrollo
```php
// .env
CACHE_DRIVER=file
```
- Simple, sin dependencias
- Perfecto para desarrollo local

### Producción (Recomendado)
```php
// .env
CACHE_DRIVER=redis
REDIS_HOST=127.0.0.1
REDIS_PASSWORD=null
REDIS_PORT=6379
```
- Alta performance
- Soporta tags
- Compartido entre workers

### Alternativa
```php
// .env
CACHE_DRIVER=memcached
```
- Similar a Redis
- Buena performance

---

## 📊 Monitoring

### Verificar Cache Hits
```php
// AppServiceProvider.php
use Illuminate\Support\Facades\Cache;

public function boot(): void
{
    if (app()->environment('local')) {
        Cache::extend('instrumented', function ($app, $config) {
            return new InstrumentedStore(
                Cache::store('file')->getStore()
            );
        });
    }
}
```

### Log de Cache Operations
```php
Cache::macro('rememberWithLog', function ($key, $ttl, $callback) {
    $value = Cache::remember($key, $ttl, $callback);
    
    Log::info('Cache operation', [
        'key' => $key,
        'hit' => Cache::has($key),
        'ttl' => $ttl,
    ]);
    
    return $value;
});
```

---

## ⚠️ Caveats

### 1. **env() NO funciona con config:cache**
```php
// ❌ MALO
$apiKey = env('GITHUB_API_KEY');

// ✅ BUENO
$apiKey = config('services.github.api_key');
```

### 2. **Cache::remember es blocking**
```php
// Si 2 requests llegan simultáneamente y cache expiró,
// ambos ejecutan el callback (race condition)

// Solución: Cache::lock()
$lock = Cache::lock('extensions.refresh', 10);

if ($lock->get()) {
    Cache::put('extensions.list', $this->fetchExtensions(), 3600);
    $lock->release();
}
```

### 3. **Serialization de objetos**
```php
// Cachear solo arrays/scalars, NO objetos complejos
Cache::put('user', $user->toArray()); // ✅ OK
Cache::put('user', $user); // ⚠️ Puede fallar en deserialización
```

---

## 🔗 Referencias

- Laravel Cache: https://laravel.com/docs/11.x/cache
- Redis Configuration: https://laravel.com/docs/11.x/redis
- Performance: https://laravel.com/docs/11.x/optimization
