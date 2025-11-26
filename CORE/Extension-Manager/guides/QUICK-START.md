# Quick Start Guide - Create Your First Extension

**Time to Complete:** 15-20 minutes  
**Difficulty:** Beginner  
**Prerequisites:** PHP 8.1+, Laravel 11, Composer

---

## 🎯 What You'll Build

A simple "Hello World" extension that:
- Registers in the Bithoven extension system
- Has a basic route and view
- Includes proper seeders with fixed IDs
- Is compatible with Fix Extension and Fresh Install

---

## 📦 Step 1: Create Directory Structure

```bash
cd /Users/madniatik/CODE/LARAVEL/BITHOVEN/EXTENSIONS

# Create extension directory
mkdir -p bithoven-extension-hello/{src,database/{migrations,seeders},config,resources/views,routes,tests}

cd bithoven-extension-hello
```

---

## 📄 Step 2: Create composer.json

```bash
cat > composer.json << 'EOF'
{
    "name": "bithoven/hello",
    "description": "Hello World Extension for Bithoven",
    "type": "library",
    "license": "MIT",
    "authors": [
        {
            "name": "Your Name",
            "email": "your.email@example.com"
        }
    ],
    "require": {
        "php": "^8.1",
        "laravel/framework": "^11.0"
    },
    "autoload": {
        "psr-4": {
            "Bithoven\\Hello\\": "src/"
        }
    },
    "extra": {
        "laravel": {
            "providers": [
                "Bithoven\\Hello\\HelloServiceProvider"
            ]
        }
    },
    "minimum-stability": "stable",
    "prefer-stable": true
}
EOF
```

---

## 📋 Step 3: Create extension.json

**⚠️ NEW SCHEMA v1.0.0** - See [EXTENSION-JSON-SCHEMA.md](EXTENSION-JSON-SCHEMA.md) for complete reference

```bash
cat > extension.json << 'EOF'
{
    "slug": "hello",
    "name": "Hello World",
    "version": "1.0.0",
    "description": "A simple Hello World extension for demonstrating BITHOVEN extension system",
    "author": "Your Name",
    "created_at": "2025-11-26T21:30:00+00:00",
    "updated_at": "2025-11-26T21:30:00+00:00",
    
    "homepage": "https://github.com/yourusername/bithoven-extension-hello",
    "repository": {
        "type": "vcs",
        "url": "https://github.com/yourusername/bithoven-extension-hello.git"
    },
    
    "category": "Development",
    "icon": "ki-code",
    "featured": false,
    "tags": ["demo", "hello-world", "template"],
    
    "permissions": [
        "extensions:hello:base:view"
    ],
    
    "seeders": {
        "core": [],
        "demo": ["HelloDemoSeeder"]
    },
    
    "changelog": {
        "v1.0.0": {
            "date": "2025-11-26",
            "changes": ["Initial release"],
            "migration_notes": "No migrations",
            "breaking_changes": false,
            "components": ["code"]
        }
    }
}
EOF
```

**📖 Required fields:**
- `slug` - Unique identifier (lowercase-hyphen)
- `name` - Display name
- `version` - Semantic version (X.Y.Z)
- `description` - Short description (10-255 chars)
- `author` - Your name or team
- `created_at` - ISO 8601 timestamp
- `seeders.core` - Core seeders (run on install)
- `seeders.demo` - Demo seeders (run on demand)

**🔗 See full documentation:** [EXTENSION-JSON-SCHEMA.md](EXTENSION-JSON-SCHEMA.md)

---

## 🔧 Step 4: Create Service Provider

```bash
cat > src/HelloServiceProvider.php << 'EOF'
<?php

namespace Bithoven\Hello;

use Illuminate\Support\ServiceProvider;

class HelloServiceProvider extends ServiceProvider
{
    /**
     * Register services.
     */
    public function register(): void
    {
        // Merge config
        $this->mergeConfigFrom(
            __DIR__.'/../config/hello.php', 'hello'
        );
    }

    /**
     * Bootstrap services.
     */
    public function boot(): void
    {
        // Load routes
        $this->loadRoutesFrom(__DIR__.'/../routes/web.php');

        // Load views
        $this->loadViewsFrom(__DIR__.'/../resources/views', 'hello');

        // Load migrations
        $this->loadMigrationsFrom(__DIR__.'/../database/migrations');

        // Publish config
        $this->publishes([
            __DIR__.'/../config/hello.php' => config_path('hello.php'),
        ], 'hello-config');

        // Publish views
        $this->publishes([
            __DIR__.'/../resources/views' => resource_path('views/vendor/hello'),
        ], 'hello-views');
    }
}
EOF
```

---

## ⚙️ Step 5: Create Configuration

```bash
cat > config/hello.php << 'EOF'
<?php

return [
    /*
    |--------------------------------------------------------------------------
    | Hello Extension Configuration
    |--------------------------------------------------------------------------
    */

    'enabled' => true,
    
    'greeting' => 'Hello, World!',
    
    'messages' => [
        'welcome' => 'Welcome to Hello Extension',
        'goodbye' => 'Goodbye from Hello Extension',
    ],
];
EOF
```

---

## 🗄️ Step 6: Create Migration

```bash
cat > database/migrations/2025_01_01_000001_create_hello_items_table.php << 'EOF'
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::create('hello_items', function (Blueprint $table) {
            $table->id();
            $table->string('name');
            $table->string('slug')->unique();
            $table->text('description')->nullable();
            $table->boolean('is_active')->default(true);
            $table->integer('sort_order')->default(0);
            $table->timestamps();
            
            $table->index('slug');
            $table->index('is_active');
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('hello_items');
    }
};
EOF
```

---

## 🌱 Step 7: Create Seeder (CRITICAL - Fixed IDs!)

```bash
cat > database/seeders/HelloSeeder.php << 'EOF'
<?php

namespace Bithoven\Hello\Database\Seeders;

use Illuminate\Database\Seeder;
use Illuminate\Support\Facades\DB;

/**
 * Hello Items Seeder
 * 
 * IMPORTANT: Uses fixed IDs (1-3) for base items to allow
 * Fix Extension to properly restore edited values.
 * 
 * ID Ranges:
 * - IDs 1-3: Base items (restored by Fix Extension)
 * - IDs > 3: Custom user items (preserved)
 */
class HelloSeeder extends Seeder
{
    public function run(): void
    {
        $baseItems = [
            [
                'id' => 1,
                'name' => 'Hello Item 1',
                'slug' => 'hello-item-1',
                'description' => 'First hello item',
                'is_active' => true,
                'sort_order' => 1,
                'created_at' => now(),
                'updated_at' => now(),
            ],
            [
                'id' => 2,
                'name' => 'Hello Item 2',
                'slug' => 'hello-item-2',
                'description' => 'Second hello item',
                'is_active' => true,
                'sort_order' => 2,
                'created_at' => now(),
                'updated_at' => now(),
            ],
            [
                'id' => 3,
                'name' => 'Hello Item 3',
                'slug' => 'hello-item-3',
                'description' => 'Third hello item',
                'is_active' => true,
                'sort_order' => 3,
                'created_at' => now(),
                'updated_at' => now(),
            ],
        ];

        foreach ($baseItems as $item) {
            DB::table('hello_items')->updateOrInsert(
                ['id' => $item['id']],  // ✅ CRITICAL: Match by ID
                $item
            );
        }
    }
}
EOF
```

---

## 🌱 Step 8: Create DatabaseSeeder

```bash
cat > database/seeders/DatabaseSeeder.php << 'EOF'
<?php

namespace Bithoven\Hello\Database\Seeders;

use Illuminate\Database\Seeder;

class DatabaseSeeder extends Seeder
{
    public function run(): void
    {
        $this->call([
            HelloSeeder::class,
        ]);
    }
}
EOF
```

---

## 🛣️ Step 9: Create Routes

```bash
cat > routes/web.php << 'EOF'
<?php

use Illuminate\Support\Facades\Route;

Route::middleware(['web', 'auth'])->prefix('hello')->name('hello.')->group(function () {
    Route::get('/', function () {
        return view('hello::index');
    })->name('index');
});
EOF
```

---

## 👁️ Step 10: Create View

```bash
cat > resources/views/index.blade.php << 'EOF'
<x-default-layout>
    @section('title', 'Hello Extension')
    @section('breadcrumbs')
        {{ Breadcrumbs::render('hello') }}
    @endsection

    <div class="card">
        <div class="card-header">
            <h3 class="card-title">Hello World Extension</h3>
        </div>
        <div class="card-body">
            <h1>{{ config('hello.greeting') }}</h1>
            <p>{{ config('hello.messages.welcome') }}</p>
            
            <h4 class="mt-5">Hello Items:</h4>
            <ul>
                @foreach(DB::table('hello_items')->get() as $item)
                    <li>{{ $item->name }} - {{ $item->description }}</li>
                @endforeach
            </ul>
        </div>
    </div>
</x-default-layout>
EOF
```

---

## 📝 Step 11: Create README

```bash
cat > README.md << 'EOF'
# Hello World Extension

A simple example extension for the Bithoven system.

## Installation

```bash
php artisan bithoven:extension:install hello
```

## Features

- Simple greeting page at `/hello`
- Database table with 3 base items
- Config file with customizable messages
- Compatible with Fix Extension and Fresh Install

## Configuration

Published config at `config/hello.php`:

```php
return [
    'greeting' => 'Hello, World!',
    'messages' => [
        'welcome' => 'Welcome to Hello Extension',
    ],
];
```

## Fix Extension Compatibility

Base items (IDs 1-3) will be restored to original values.
Custom items (IDs > 3) will be preserved.

## License

MIT
EOF
```

---

## 🚀 Step 12: Install Extension

```bash
cd /Users/madniatik/CODE/LARAVEL/BITHOVEN/CPANEL

# Install via Artisan
php artisan bithoven:extension:install hello

# Or via Admin UI
# Navigate to: http://localhost:8000/admin/extensions
# Click "Install" on Hello extension
```

---

## ✅ Step 13: Test Extension

### Basic Functionality
1. Visit `http://localhost:8000/hello`
2. Should see "Hello, World!" and list of 3 items

### Test Fix Extension Compatibility
1. Edit item ID=1 in database:
   ```sql
   UPDATE hello_items SET name='EDITED' WHERE id=1;
   ```
2. Run Fix Extension from Admin UI
3. Verify item ID=1 restored to "Hello Item 1"

### Test Custom Items Preservation
1. Add custom item:
   ```sql
   INSERT INTO hello_items (name, slug, description, is_active, sort_order, created_at, updated_at)
   VALUES ('Custom Item', 'custom-item', 'My custom item', 1, 10, NOW(), NOW());
   ```
2. Note the auto-generated ID (should be > 3)
3. Run Fix Extension
4. Verify custom item still exists unchanged

---

## 🎓 What You Learned

✅ Extension directory structure  
✅ Service provider registration  
✅ **Seeders with fixed IDs** (CRITICAL!)  
✅ Migrations  
✅ Routes and views  
✅ Fix Extension compatibility  

---

## 🚀 Next Steps

1. **Read:** [SEEDERS-BEST-PRACTICES.md](SEEDERS-BEST-PRACTICES.md) - Deep dive into seeder patterns
2. **Explore:** [EXTENSION-STRUCTURE.md](EXTENSION-STRUCTURE.md) - Complete structure reference
3. **Study:** `/EXTENSIONS/bithoven-extension-tickets/` - Real-world complex example
4. **Build:** Create your own extension!

---

## 💡 Tips

- Always use fixed IDs in base seeders
- Keep DatabaseSeeder for essential data only
- Test both Fix Extension and Fresh Install
- Document ID ranges in seeder comments
- Follow naming conventions (bithoven-extension-{name})

---

**Congratulations!** 🎉 You've created your first Bithoven extension!

For help, see [AI-AGENT-INSTRUCTIONS.md](../COPILOT/AI-AGENT-INSTRUCTIONS.md)
