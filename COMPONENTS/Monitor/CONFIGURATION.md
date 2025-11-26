# 📡 Monitor Component - Configuration Guide

**Complete configuration reference for Monitor Protocol v3.0**

---

## 📋 Configuration Files

### config/monitor.php

```php
<?php

return [
    /*
    |--------------------------------------------------------------------------
    | Monitor System Enabled
    |--------------------------------------------------------------------------
    |
    | Enable or disable the entire monitor system. When disabled, monitor
    | components will still render but recording features will be disabled.
    |
    */
    'enabled' => env('MONITOR_ENABLED', true),

    /*
    |--------------------------------------------------------------------------
    | Storage Path
    |--------------------------------------------------------------------------
    |
    | Base directory for storing monitor sessions. Path is relative to
    | storage/app/ directory.
    |
    */
    'storage_path' => 'monitors',

    /*
    |--------------------------------------------------------------------------
    | Session Cleanup
    |--------------------------------------------------------------------------
    |
    | Automatic cleanup of old monitor sessions to prevent storage bloat.
    |
    */
    'cleanup' => [
        'enabled' => env('MONITOR_CLEANUP_ENABLED', true),
        'retention_days' => env('MONITOR_RETENTION_DAYS', 30),
        'schedule' => 'daily', // daily, weekly, monthly
    ],

    /*
    |--------------------------------------------------------------------------
    | Recording Configuration
    |--------------------------------------------------------------------------
    |
    | Settings for real-time recording feature.
    |
    */
    'recording' => [
        'batch_interval' => 2000,  // Batch flush interval (milliseconds)
        'max_batch_size' => 100,   // Maximum logs per batch
        'timeout' => 10000,        // Request timeout (milliseconds)
        'retry_attempts' => 3,     // Retry failed requests
    ],

    /*
    |--------------------------------------------------------------------------
    | Export Configuration
    |--------------------------------------------------------------------------
    |
    | Settings for session export functionality.
    |
    */
    'export' => [
        'enabled' => true,
        'formats' => ['txt', 'json', 'csv'],
        'default_format' => 'txt',
        'storage_path' => 'exports',
        'auto_delete' => true, // Delete file after download
    ],

    /*
    |--------------------------------------------------------------------------
    | UI Defaults
    |--------------------------------------------------------------------------
    |
    | Default settings for monitor component UI.
    |
    */
    'ui' => [
        'default_title' => '📡 Monitor',
        'default_height' => '300px',
        'max_height' => '600px',
        'enable_copy' => true,
        'enable_download' => true,
        'enable_clear' => true,
        'enable_recording' => false, // Disabled by default for security
    ],

    /*
    |--------------------------------------------------------------------------
    | Metadata Configuration
    |--------------------------------------------------------------------------
    |
    | Automatic metadata to include in all sessions.
    |
    */
    'auto_metadata' => [
        'app_name' => env('APP_NAME', 'BITHOVEN'),
        'app_env' => env('APP_ENV', 'production'),
        'app_version' => '1.0.0', // Your app version
    ],

    /*
    |--------------------------------------------------------------------------
    | Security
    |--------------------------------------------------------------------------
    |
    | Security settings for monitor system.
    |
    */
    'security' => [
        'require_auth' => true,     // Require authentication for recording
        'allowed_origins' => ['*'], // CORS allowed origins
        'rate_limit' => 60,         // Requests per minute
    ],
];
```

---

## 🌍 Environment Variables

### .env Configuration

```env
# Monitor System
MONITOR_ENABLED=true
MONITOR_CLEANUP_ENABLED=true
MONITOR_RETENTION_DAYS=30

# Security
MONITOR_REQUIRE_AUTH=true
MONITOR_RATE_LIMIT=60

# Storage
MONITOR_STORAGE_PATH=monitors
MONITOR_EXPORT_PATH=exports
```

---

## 🔧 Service Provider Setup

### app/Providers/AppServiceProvider.php

```php
<?php

namespace App\Providers;

use Illuminate\Support\ServiceProvider;
use App\Contracts\MonitorServiceInterface;
use App\Services\MonitorService;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Register any application services.
     */
    public function register(): void
    {
        // Bind Monitor Service
        $this->app->singleton(MonitorServiceInterface::class, MonitorService::class);
    }

    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        // Monitor configuration
        if (config('monitor.enabled')) {
            // Load monitor routes
            // Load monitor views
            // etc.
        }
    }
}
```

---

## 📅 Scheduled Tasks

### app/Console/Kernel.php

```php
<?php

namespace App\Console;

use Illuminate\Console\Scheduling\Schedule;
use Illuminate\Foundation\Console\Kernel as ConsoleKernel;

class Kernel extends ConsoleKernel
{
    /**
     * Define the application's command schedule.
     */
    protected function schedule(Schedule $schedule): void
    {
        // Clean old monitor sessions daily at 2 AM
        if (config('monitor.cleanup.enabled')) {
            $days = config('monitor.cleanup.retention_days', 30);
            
            $schedule->call(function () use ($days) {
                app(\App\Services\MonitorService::class)->cleanOldSessions($days);
            })->daily()->at('02:00');
        }
    }

    /**
     * Register the commands for the application.
     */
    protected function commands(): void
    {
        $this->load(__DIR__.'/Commands');

        require base_path('routes/console.php');
    }
}
```

---

## 🛣️ Routes Configuration

### routes/api.php

```php
<?php

use Illuminate\Support\Facades\Route;
use App\Http\Controllers\Api\MonitorController;

/*
|--------------------------------------------------------------------------
| Monitor API Routes
|--------------------------------------------------------------------------
*/

if (config('monitor.enabled')) {
    Route::prefix('monitor')
        ->name('api.monitor.')
        ->middleware(['api', 'auth:sanctum']) // Add your auth middleware
        ->group(function () {
            // Save logs batch
            Route::post('/logs', [MonitorController::class, 'saveLogs'])
                ->name('logs');
            
            // Get session data
            Route::get('/session/{sessionId}', [MonitorController::class, 'getSession'])
                ->name('session');
            
            // List sessions for monitor
            Route::get('/sessions/{monitorId}', [MonitorController::class, 'listSessions'])
                ->name('sessions');
            
            // Export session
            Route::get('/export/{sessionId}', [MonitorController::class, 'exportSession'])
                ->name('export');
            
            // Download exported file
            Route::get('/download/{sessionId}', [MonitorController::class, 'downloadSession'])
                ->name('download');
        });
}
```

---

## 🔒 Middleware Configuration

### Rate Limiting

```php
// app/Http/Kernel.php
protected $middlewareGroups = [
    'api' => [
        'throttle:monitor', // Use monitor rate limit
        \Illuminate\Routing\Middleware\SubstituteBindings::class,
    ],
];

// app/Providers/RouteServiceProvider.php
protected function configureRateLimiting()
{
    RateLimiter::for('monitor', function (Request $request) {
        return Limit::perMinute(config('monitor.security.rate_limit', 60))
            ->by($request->user()?->id ?: $request->ip());
    });
}
```

### CORS Configuration

```php
// config/cors.php
'paths' => ['api/*', 'api/monitor/*'],

'allowed_origins' => config('monitor.security.allowed_origins', ['*']),
```

---

## 💾 Storage Structure

### Directory Setup

```bash
# Create storage directories
mkdir -p storage/app/monitors
mkdir -p storage/app/exports

# Set permissions
chmod -R 775 storage/app/monitors
chmod -R 775 storage/app/exports
```

### Storage Organization

```
storage/app/
├── monitors/
│   ├── extension-install-monitor/
│   │   ├── session-1700000000000.json
│   │   ├── session-1700001000000.json
│   │   └── session-1700002000000.json
│   ├── llm-test-monitor/
│   │   └── session-1700003000000.json
│   └── import-csv-monitor/
│       └── session-1700004000000.json
└── exports/
    ├── session-1700000000000.txt
    ├── session-1700000000000.json
    └── session-1700000000000.csv
```

---

## 🎨 UI Customization

### Custom CSS

```blade
{{-- resources/views/layouts/app.blade.php --}}
@push('styles')
<style>
    .monitor-container {
        background: #1e1e1e !important; /* Dark theme */
        color: #d4d4d4 !important;
        font-family: 'Fira Code', 'Courier New', monospace !important;
        font-size: 14px !important;
        line-height: 1.6 !important;
    }
    
    .monitor-container .text-success {
        color: #4ec9b0 !important; /* Custom green */
    }
    
    .monitor-container .text-danger {
        color: #f48771 !important; /* Custom red */
    }
</style>
@endpush
```

### Custom JavaScript Colors

Edit `resources/js/custom/monitor/monitor.js`:

```javascript
const colors = {
    success: 'text-success',
    error: 'text-danger',
    warning: 'text-warning',
    info: 'text-info',
    debug: 'text-muted'
};

const icons = {
    success: '✓',
    error: '✗',
    warning: '⚠',
    info: 'ℹ',
    debug: '▸'
};
```

---

## 🔧 Artisan Commands

### monitor:clean

Clean old monitor sessions.

```bash
# Clean sessions older than default (30 days)
php artisan monitor:clean

# Clean sessions older than 7 days
php artisan monitor:clean --days=7

# Force without confirmation
php artisan monitor:clean --force

# Dry run (show what would be deleted)
php artisan monitor:clean --dry-run
```

### monitor:list

List all monitor sessions.

```bash
# List all sessions
php artisan monitor:list

# List sessions for specific monitor
php artisan monitor:list --monitor=extension-install-monitor

# List with limit
php artisan monitor:list --limit=10
```

### monitor:export

Export monitor session.

```bash
# Export as TXT (default)
php artisan monitor:export session-1700000000000

# Export as JSON
php artisan monitor:export session-1700000000000 --format=json

# Export as CSV
php artisan monitor:export session-1700000000000 --format=csv
```

---

## 📊 Database Configuration (Optional)

If you prefer database storage over JSON files:

### Migration

```php
// database/migrations/xxxx_create_monitor_sessions_table.php
Schema::create('monitor_sessions', function (Blueprint $table) {
    $table->id();
    $table->string('session_id')->unique();
    $table->string('monitor_id')->index();
    $table->json('metadata')->nullable();
    $table->integer('total_logs')->default(0);
    $table->timestamp('started_at');
    $table->timestamps();
});

Schema::create('monitor_logs', function (Blueprint $table) {
    $table->id();
    $table->foreignId('session_id')->constrained('monitor_sessions')->onDelete('cascade');
    $table->timestamp('timestamp');
    $table->string('type'); // success, error, warning, info, debug
    $table->text('message');
    $table->timestamps();
});
```

### Update config/monitor.php

```php
'storage' => [
    'driver' => env('MONITOR_STORAGE_DRIVER', 'file'), // file|database
    'connection' => env('DB_CONNECTION', 'mysql'),
],
```

---

## 🔐 Security Best Practices

### 1. Require Authentication

```php
// config/monitor.php
'security' => [
    'require_auth' => true,
],

// routes/api.php
Route::middleware(['auth:sanctum'])->group(function () {
    // Monitor routes
});
```

### 2. Rate Limiting

```php
'security' => [
    'rate_limit' => 60, // requests per minute
],
```

### 3. Sanitize Metadata

```php
// In MonitorController::saveLogs()
$metadata = array_filter($request->metadata, function ($key) {
    return in_array($key, ['extension', 'user_id', 'context', 'action']);
}, ARRAY_FILTER_USE_KEY);
```

### 4. File Permissions

```bash
chmod 755 storage/app/monitors
chown www-data:www-data storage/app/monitors
```

---

## 📈 Performance Optimization

### 1. Batch Configuration

```php
'recording' => [
    'batch_interval' => 2000,  // Increase for less frequent saves
    'max_batch_size' => 100,   // Increase for larger batches
],
```

### 2. Storage Cleanup

```php
'cleanup' => [
    'retention_days' => 7, // Reduce for smaller storage
],
```

### 3. Database Indexes

```php
Schema::table('monitor_sessions', function (Blueprint $table) {
    $table->index('monitor_id');
    $table->index('started_at');
    $table->index(['monitor_id', 'started_at']);
});
```

---

## 🧪 Testing Configuration

### phpunit.xml

```xml
<php>
    <env name="MONITOR_ENABLED" value="false"/>
    <env name="MONITOR_STORAGE_DRIVER" value="array"/>
</php>
```

### Feature Test

```php
public function test_monitor_recording_requires_auth()
{
    $response = $this->postJson('/api/monitor/logs', [
        'session_id' => 'test-session',
        'monitor_id' => 'test-monitor',
        'logs' => [],
    ]);

    $response->assertUnauthorized();
}
```

---

**For more details, see:**
- [README.md](./README.md) - Main documentation
- [API-REFERENCE.md](./API-REFERENCE.md) - API specifications
- [EXAMPLES.md](./EXAMPLES.md) - Integration examples
