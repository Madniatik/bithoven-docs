# Service Providers for BITHOVEN Extensions

**Version:** 2.0.0  
**Last Updated:** 17 de noviembre de 2025  
**Status:** STABLE

---

## 🎯 Overview

El Service Provider es el punto de entrada de tu extensión. Aquí se registran:
- Routes
- Views
- Migrations
- Config files
- Event listeners
- Policies
- Commands

---

## 📋 Basic Structure

```php
<?php

namespace Bithoven\Tasks;

use Illuminate\Support\ServiceProvider;

class TasksServiceProvider extends ServiceProvider
{
    /**
     * Register services.
     */
    public function register(): void
    {
        // Merge config
        $this->mergeConfigFrom(
            __DIR__.'/../config/tasks.php', 
            'tasks'
        );
    }

    /**
     * Bootstrap services.
     */
    public function boot(): void
    {
        // Load migrations
        $this->loadMigrationsFrom(__DIR__.'/../database/migrations');
        
        // Load routes
        $this->loadRoutesFrom(__DIR__.'/../routes/web.php');
        
        // Load views
        $this->loadViewsFrom(__DIR__.'/../resources/views', 'tasks');
        
        // Publish assets
        $this->publishes([
            __DIR__.'/../config/tasks.php' => config_path('tasks.php'),
        ], 'tasks-config');
    }
}
```

---

## 🔧 Register vs Boot

### register() - Para Bindings

**Cuándo:** Registrar servicios en el contenedor IoC

```php
public function register(): void
{
    // Merge config
    $this->mergeConfigFrom(__DIR__.'/../config/tasks.php', 'tasks');
    
    // Register singleton
    $this->app->singleton(TaskService::class, function ($app) {
        return new TaskService();
    });
    
    // Bind interface to implementation
    $this->app->bind(TaskRepositoryInterface::class, TaskRepository::class);
}
```

### boot() - Para Bootstrapping

**Cuándo:** Acciones después de que todos los servicios estén registrados

```php
public function boot(): void
{
    // Load resources
    $this->loadMigrationsFrom(__DIR__.'/../database/migrations');
    $this->loadRoutesFrom(__DIR__.'/../routes/web.php');
    $this->loadViewsFrom(__DIR__.'/../resources/views', 'tasks');
    
    // Register policies
    Gate::policy(Task::class, TaskPolicy::class);
    
    // Register events
    Event::listen(TaskCreated::class, SendTaskNotification::class);
    
    // Publish assets
    $this->publishes([...]);
}
```

---

## 📦 Loading Resources

### Load Migrations

```php
public function boot(): void
{
    $this->loadMigrationsFrom(__DIR__.'/../database/migrations');
}
```

**Result:** Migrations auto-discovered by `php artisan migrate`

### Load Routes

```php
public function boot(): void
{
    // Web routes
    $this->loadRoutesFrom(__DIR__.'/../routes/web.php');
    
    // API routes (optional)
    if (file_exists(__DIR__.'/../routes/api.php')) {
        $this->loadRoutesFrom(__DIR__.'/../routes/api.php');
    }
}
```

### Load Views

```php
public function boot(): void
{
    $this->loadViewsFrom(__DIR__.'/../resources/views', 'tasks');
}
```

**Usage:**
```php
return view('tasks::index'); // resources/views/index.blade.php
```

### Load Translations

```php
public function boot(): void
{
    $this->loadTranslationsFrom(__DIR__.'/../resources/lang', 'tasks');
}
```

**Usage:**
```php
__('tasks::messages.welcome'); // resources/lang/en/messages.php
```

---

## 🎨 Publishing Assets

### Publish Config

```php
public function boot(): void
{
    $this->publishes([
        __DIR__.'/../config/tasks.php' => config_path('tasks.php'),
    ], 'tasks-config');
}
```

**User runs:**
```bash
php artisan vendor:publish --tag=tasks-config
```

### Publish Views

```php
public function boot(): void
{
    $this->publishes([
        __DIR__.'/../resources/views' => resource_path('views/vendor/tasks'),
    ], 'tasks-views');
}
```

### Publish Multiple Assets

```php
public function boot(): void
{
    // Config
    $this->publishes([
        __DIR__.'/../config/tasks.php' => config_path('tasks.php'),
    ], 'tasks-config');
    
    // Views
    $this->publishes([
        __DIR__.'/../resources/views' => resource_path('views/vendor/tasks'),
    ], 'tasks-views');
    
    // Migrations (if user wants to customize)
    $this->publishes([
        __DIR__.'/../database/migrations' => database_path('migrations'),
    ], 'tasks-migrations');
}
```

**User runs:**
```bash
# Publish all
php artisan vendor:publish --provider="Bithoven\Tasks\TasksServiceProvider"

# Publish specific
php artisan vendor:publish --tag=tasks-config
php artisan vendor:publish --tag=tasks-views
```

---

## 🔐 Register Policies

```php
use Illuminate\Support\Facades\Gate;
use Bithoven\Tasks\Models\Task;
use Bithoven\Tasks\Policies\TaskPolicy;

public function boot(): void
{
    // Register policy
    Gate::policy(Task::class, TaskPolicy::class);
}
```

---

## 📡 Register Events

```php
use Illuminate\Support\Facades\Event;
use Bithoven\Tasks\Events\TaskCreated;
use Bithoven\Tasks\Listeners\SendTaskNotification;

public function boot(): void
{
    Event::listen(
        TaskCreated::class,
        SendTaskNotification::class
    );
}
```

**Alternative:** Use `$listen` property in EventServiceProvider

---

## 🎯 Register Commands

```php
use Bithoven\Tasks\Console\Commands\TaskCleanupCommand;

public function boot(): void
{
    if ($this->app->runningInConsole()) {
        $this->commands([
            TaskCleanupCommand::class,
        ]);
    }
}
```

---

## 🔧 Register Middleware

```php
use Bithoven\Tasks\Http\Middleware\TaskOwnerMiddleware;

public function boot(): void
{
    // Register middleware
    $this->app['router']->aliasMiddleware('task.owner', TaskOwnerMiddleware::class);
}
```

**Usage:**
```php
Route::get('/tasks/{task}', [TaskController::class, 'show'])
    ->middleware('task.owner');
```

---

## 📚 Complete Example

```php
<?php

namespace Bithoven\Tasks;

use Illuminate\Support\ServiceProvider;
use Illuminate\Support\Facades\Gate;
use Illuminate\Support\Facades\Event;
use Bithoven\Tasks\Models\Task;
use Bithoven\Tasks\Policies\TaskPolicy;
use Bithoven\Tasks\Events\TaskCreated;
use Bithoven\Tasks\Listeners\SendTaskNotification;
use Bithoven\Tasks\Console\Commands\TaskCleanupCommand;
use Bithoven\Tasks\Services\TaskService;

class TasksServiceProvider extends ServiceProvider
{
    /**
     * Register services.
     */
    public function register(): void
    {
        // Merge config
        $this->mergeConfigFrom(
            __DIR__.'/../config/tasks.php',
            'tasks'
        );
        
        // Register services
        $this->app->singleton(TaskService::class, function ($app) {
            return new TaskService();
        });
    }

    /**
     * Bootstrap services.
     */
    public function boot(): void
    {
        // Load migrations
        $this->loadMigrationsFrom(__DIR__.'/../database/migrations');
        
        // Load routes
        $this->loadRoutesFrom(__DIR__.'/../routes/web.php');
        
        if (file_exists(__DIR__.'/../routes/api.php')) {
            $this->loadRoutesFrom(__DIR__.'/../routes/api.php');
        }
        
        // Load views
        $this->loadViewsFrom(__DIR__.'/../resources/views', 'tasks');
        
        // Load translations
        $this->loadTranslationsFrom(__DIR__.'/../resources/lang', 'tasks');
        
        // Register policies
        Gate::policy(Task::class, TaskPolicy::class);
        
        // Register events
        Event::listen(TaskCreated::class, SendTaskNotification::class);
        
        // Register commands
        if ($this->app->runningInConsole()) {
            $this->commands([
                TaskCleanupCommand::class,
            ]);
        }
        
        // Publish assets
        $this->publishes([
            __DIR__.'/../config/tasks.php' => config_path('tasks.php'),
        ], 'tasks-config');
        
        $this->publishes([
            __DIR__.'/../resources/views' => resource_path('views/vendor/tasks'),
        ], 'tasks-views');
    }
}
```

---

## 🎯 Auto-Discovery in composer.json

```json
{
  "extra": {
    "laravel": {
      "providers": [
        "Bithoven\\Tasks\\TasksServiceProvider"
      ]
    }
  }
}
```

**Result:** Laravel auto-registers provider during `composer install`

---

## ⚠️ Common Mistakes

### ❌ Wrong: Loading Routes in register()

```php
// ❌ WRONG - Routes not ready yet
public function register(): void
{
    $this->loadRoutesFrom(__DIR__.'/../routes/web.php');
}

// ✅ CORRECT - Load in boot()
public function boot(): void
{
    $this->loadRoutesFrom(__DIR__.'/../routes/web.php');
}
```

### ❌ Wrong: Missing View Namespace

```php
// ❌ WRONG - No namespace
$this->loadViewsFrom(__DIR__.'/../resources/views');

// ✅ CORRECT - With namespace
$this->loadViewsFrom(__DIR__.'/../resources/views', 'tasks');
```

### ❌ Wrong: Hardcoded Paths

```php
// ❌ WRONG - Hardcoded path
$this->loadMigrationsFrom('/var/www/vendor/bithoven/tasks/database/migrations');

// ✅ CORRECT - Relative path
$this->loadMigrationsFrom(__DIR__.'/../database/migrations');
```

---

## 🧪 Testing Service Provider

```php
namespace Bithoven\Tasks\Tests;

use Tests\TestCase;
use Bithoven\Tasks\TasksServiceProvider;

class ServiceProviderTest extends TestCase
{
    public function test_service_provider_is_registered()
    {
        $providers = $this->app->getLoadedProviders();
        
        $this->assertArrayHasKey(TasksServiceProvider::class, $providers);
    }
    
    public function test_config_is_merged()
    {
        $this->assertNotNull(config('tasks.defaults.priority'));
    }
    
    public function test_routes_are_loaded()
    {
        $this->assertTrue(\Route::has('tasks.index'));
    }
    
    public function test_views_are_loaded()
    {
        $this->assertTrue(view()->exists('tasks::index'));
    }
}
```

---

## 📋 Checklist: Service Provider

- [ ] Namespace matches `Bithoven\{Slug}\{Slug}ServiceProvider`
- [ ] `register()` merges config with `mergeConfigFrom()`
- [ ] `boot()` loads migrations with `loadMigrationsFrom()`
- [ ] `boot()` loads routes with `loadRoutesFrom()`
- [ ] `boot()` loads views with `loadViewsFrom()` + namespace
- [ ] Policies registered with `Gate::policy()`
- [ ] Events registered with `Event::listen()`
- [ ] Commands registered with `$this->commands()`
- [ ] Assets publishable with `publishes()`
- [ ] Auto-discovery configured in composer.json
- [ ] Relative paths used (`__DIR__.'/../'`)

---

## 🔗 Related Documentation

- **[Extension Structure](./EXTENSION-STRUCTURE.md)** - Directory organization
- **[Developer Guide](../DEVELOPER-GUIDE.md)** - Complete workflow
- **[Laravel Service Providers](https://laravel.com/docs/11.x/providers)** - Official docs

---

**Remember:** The Service Provider is your extension's entry point - keep it clean and organized!
