# BITHOVEN Extension Development Guide

**Version:** 2.0.0  
**Last Updated:** 17 de noviembre de 2025  
**Status:** STABLE

---

## 🎯 Overview

Esta guía te llevará paso a paso desde la creación de una extensión desde cero hasta su publicación en GitHub, lista para ser instalada en cualquier instancia de BITHOVEN.

**Tiempo estimado:** 30-60 minutos para tu primera extensión

**Prerequisites:**
- PHP 8.1+ y Laravel 11
- Composer 2.0+
- Git & GitHub account
- MySQL 8.0+ o MariaDB 10.5+
- BITHOVEN CPANEL instalado

---

## 🚀 Quick Start: Your First Extension

### Step 1: Create Extension Directory

```bash
# Ir a directorio de extensiones
cd /path/to/BITHOVEN/EXTENSIONS

# Crear estructura base
mkdir bithoven-extension-tasks
cd bithoven-extension-tasks

# Crear directorios estándar
mkdir -p {src,database/{migrations,seeders},resources/views,routes,config,docs,tests}
```

### Step 2: Initialize Composer

```bash
composer init
```

**Responder con:**
```
Package name: bithoven/tasks
Description: Task management system
Author: Your Name <you@example.com>
Minimum Stability: stable
Package Type: library
License: MIT
```

**Editar `composer.json` generado:**
```json
{
  "name": "bithoven/tasks",
  "description": "Task management system for BITHOVEN",
  "type": "library",
  "license": "MIT",
  "authors": [
    {
      "name": "Your Name",
      "email": "you@example.com"
    }
  ],
  "require": {
    "php": "^8.1",
    "illuminate/support": "^11.0",
    "illuminate/database": "^11.0"
  },
  "autoload": {
    "psr-4": {
      "Bithoven\\Tasks\\": "src/",
      "Bithoven\\Tasks\\Database\\Seeders\\": "database/seeders/"
    }
  },
  "extra": {
    "laravel": {
      "providers": [
        "Bithoven\\Tasks\\TasksServiceProvider"
      ]
    }
  },
  "minimum-stability": "stable"
}
```

### Step 3: Create extension.json

**⚠️ CRITICAL:** Este archivo define TODO sobre tu extensión.

```json
{
  "name": "Task Manager",
  "slug": "tasks",
  "version": "1.0.0",
  "description": "Simple task management system",
  "category": "Productivity",
  "icon": "ki-task",
  "featured": false,
  "tags": ["tasks", "productivity", "todo"],
  "migrations": {
    "required": true,
    "count": 1,
    "can_skip": false,
    "details": [
      "2025_01_01_000001_create_tasks_table"
    ],
    "description": "Creates tasks table for task management"
  },
  "seeders": {
    "core": [],
    "demo": ["TasksDemoSeeder"]
  },
  "permissions": [
    "view-tasks",
    "create-tasks",
    "edit-tasks",
    "delete-tasks"
  ],
  "system_tables": ["users"],
  "dependencies": {
    "bithoven/core": "^1.4.0"
  },
  "changelog": {
    "v1.0.0": {
      "date": "2025-01-01",
      "changes": ["Initial release", "Basic CRUD operations"],
      "migration_notes": "Creates tasks table with user relationship",
      "breaking_changes": false,
      "components": ["code", "database"]
    }
  }
}
```

**Naming Conventions:**
- `slug`: Lowercase, no spaces, usado para prefijo de tablas
- Tablas: `tasks_tabla` (excepto tabla principal que usa solo el slug)
- Permissions: `{action}-{resource}` (view-tasks, create-tasks)

### Step 4: Create Service Provider

**`src/TasksServiceProvider.php`:**
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
            __DIR__.'/../config/tasks.php', 'tasks'
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
        
        // Publish config
        $this->publishes([
            __DIR__.'/../config/tasks.php' => config_path('tasks.php'),
        ], 'tasks-config');
        
        // Publish views (optional)
        $this->publishes([
            __DIR__.'/../resources/views' => resource_path('views/vendor/tasks'),
        ], 'tasks-views');
    }
}
```

### Step 5: Create Migration

```bash
# Crear archivo de migración
touch database/migrations/2025_01_01_000001_create_tasks_table.php
```

**Contenido:**
```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    /**
     * Run the migrations.
     */
    public function up(): void
    {
        Schema::create('tasks', function (Blueprint $table) {
            $table->id();
            $table->foreignId('user_id')->constrained()->onDelete('cascade');
            $table->string('title');
            $table->text('description')->nullable();
            $table->enum('status', ['pending', 'in_progress', 'completed'])->default('pending');
            $table->enum('priority', ['low', 'medium', 'high'])->default('medium');
            $table->timestamp('due_date')->nullable();
            $table->timestamp('completed_at')->nullable();
            $table->timestamps();
            
            // Indexes para performance
            $table->index(['user_id', 'status']);
            $table->index('due_date');
        });
    }

    /**
     * Reverse the migrations.
     */
    public function down(): void
    {
        Schema::dropIfExists('tasks');
    }
};
```

**⚠️ Naming Convention:**
- Tabla principal: `tasks` (slug sin prefijo)
- Tablas relacionadas: `tasks_comments`, `tasks_attachments`, etc.

### Step 6: Create Model

**`src/Models/Task.php`:**
```php
<?php

namespace Bithoven\Tasks\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\BelongsTo;
use App\Models\User;

class Task extends Model
{
    /**
     * The table associated with the model.
     */
    protected $table = 'tasks';

    /**
     * The attributes that are mass assignable.
     */
    protected $fillable = [
        'user_id',
        'title',
        'description',
        'status',
        'priority',
        'due_date',
        'completed_at',
    ];

    /**
     * The attributes that should be cast.
     */
    protected $casts = [
        'due_date' => 'datetime',
        'completed_at' => 'datetime',
    ];

    /**
     * Get the user that owns the task.
     */
    public function user(): BelongsTo
    {
        return $this->belongsTo(User::class);
    }

    /**
     * Scope: pending tasks
     */
    public function scopePending($query)
    {
        return $query->where('status', 'pending');
    }

    /**
     * Scope: completed tasks
     */
    public function scopeCompleted($query)
    {
        return $query->where('status', 'completed');
    }

    /**
     * Scope: overdue tasks
     */
    public function scopeOverdue($query)
    {
        return $query->where('due_date', '<', now())
            ->whereNotIn('status', ['completed']);
    }
}
```

### Step 7: Create Controller

**`src/Http/Controllers/TaskController.php`:**
```php
<?php

namespace Bithoven\Tasks\Http\Controllers;

use App\Http\Controllers\Controller;
use Bithoven\Tasks\Models\Task;
use Illuminate\Http\Request;

class TaskController extends Controller
{
    /**
     * Display a listing of tasks.
     */
    public function index()
    {
        $tasks = Task::where('user_id', auth()->id())
            ->latest()
            ->paginate(20);
        
        return view('tasks::index', compact('tasks'));
    }

    /**
     * Show the form for creating a new task.
     */
    public function create()
    {
        return view('tasks::create');
    }

    /**
     * Store a newly created task.
     */
    public function store(Request $request)
    {
        $validated = $request->validate([
            'title' => 'required|max:255',
            'description' => 'nullable',
            'priority' => 'required|in:low,medium,high',
            'due_date' => 'nullable|date|after:today',
        ]);

        $validated['user_id'] = auth()->id();
        $validated['status'] = 'pending';

        Task::create($validated);

        return redirect()->route('tasks.index')
            ->with('success', 'Task created successfully');
    }

    /**
     * Show the form for editing the task.
     */
    public function edit(Task $task)
    {
        // Authorization
        if ($task->user_id !== auth()->id()) {
            abort(403, 'Unauthorized action.');
        }
        
        return view('tasks::edit', compact('task'));
    }

    /**
     * Update the specified task.
     */
    public function update(Request $request, Task $task)
    {
        // Authorization
        if ($task->user_id !== auth()->id()) {
            abort(403, 'Unauthorized action.');
        }

        $validated = $request->validate([
            'title' => 'required|max:255',
            'description' => 'nullable',
            'status' => 'required|in:pending,in_progress,completed',
            'priority' => 'required|in:low,medium,high',
            'due_date' => 'nullable|date',
        ]);

        // Auto-complete timestamp
        if ($validated['status'] === 'completed' && !$task->completed_at) {
            $validated['completed_at'] = now();
        } elseif ($validated['status'] !== 'completed') {
            $validated['completed_at'] = null;
        }

        $task->update($validated);

        return redirect()->route('tasks.index')
            ->with('success', 'Task updated successfully');
    }

    /**
     * Remove the specified task.
     */
    public function destroy(Task $task)
    {
        // Authorization
        if ($task->user_id !== auth()->id()) {
            abort(403, 'Unauthorized action.');
        }
        
        $task->delete();

        return redirect()->route('tasks.index')
            ->with('success', 'Task deleted successfully');
    }
}
```

### Step 8: Create Routes

**`routes/web.php`:**
```php
<?php

use Illuminate\Support\Facades\Route;
use Bithoven\Tasks\Http\Controllers\TaskController;

/*
|--------------------------------------------------------------------------
| Extension Routes
|--------------------------------------------------------------------------
*/

Route::middleware(['web', 'auth'])->group(function () {
    Route::prefix('tasks')->name('tasks.')->group(function () {
        Route::get('/', [TaskController::class, 'index'])->name('index');
        Route::get('/create', [TaskController::class, 'create'])->name('create');
        Route::post('/', [TaskController::class, 'store'])->name('store');
        Route::get('/{task}/edit', [TaskController::class, 'edit'])->name('edit');
        Route::put('/{task}', [TaskController::class, 'update'])->name('update');
        Route::delete('/{task}', [TaskController::class, 'destroy'])->name('destroy');
    });
});
```

**Best Practices:**
- Siempre usar middleware `['web', 'auth']`
- Prefijo de rutas = slug de extensión
- Nombrar rutas con namespace: `tasks.index`, `tasks.create`

### Step 9: Create Demo Seeder

**`database/seeders/TasksDemoSeeder.php`:**
```php
<?php

namespace Bithoven\Tasks\Database\Seeders;

use Illuminate\Database\Seeder;
use Bithoven\Tasks\Models\Task;
use App\Models\User;

class TasksDemoSeeder extends Seeder
{
    /**
     * Run the database seeds.
     */
    public function run(): void
    {
        // Get first 5 users for demo
        $users = User::limit(5)->get();
        
        if ($users->isEmpty()) {
            $this->command->warn('No users found. Please create users first.');
            return;
        }

        $statuses = ['pending', 'in_progress', 'completed'];
        $priorities = ['low', 'medium', 'high'];
        
        foreach ($users as $user) {
            // Create 10 random tasks per user
            foreach (range(1, 10) as $i) {
                Task::create([
                    'user_id' => $user->id,
                    'title' => "Task #{$i} for {$user->name}",
                    'description' => "This is a demo task description for task #{$i}",
                    'status' => $statuses[array_rand($statuses)],
                    'priority' => $priorities[array_rand($priorities)],
                    'due_date' => now()->addDays(rand(1, 30)),
                    'completed_at' => rand(0, 1) ? now()->subDays(rand(1, 10)) : null,
                ]);
            }
        }

        $this->command->info('✅ Created 50 demo tasks for 5 users');
    }
}
```

**⚠️ CRITICAL - Seeder Best Practices:**
- **NEVER** usar `updateOrCreate(['name' => ...])` - crea duplicados
- **ALWAYS** usar `updateOrCreate(['id' => ...])` para registros base
- Separar `core` (esenciales) de `demo` (datos de prueba)
- Documentar rangos de IDs: base (1-100), custom (>100)

Ver: [SEEDERS-BEST-PRACTICES.md](guides/SEEDERS-BEST-PRACTICES.md)

### Step 10: Create Views (Metronic Layout)

**`resources/views/index.blade.php`:**
```blade
<x-default-layout>
    @section('title', 'My Tasks')
    
    @section('breadcrumbs')
        {{ Breadcrumbs::render('tasks.index') }}
    @endsection

    {{-- Card Header --}}
    <div class="card">
        <div class="card-header border-0 pt-6">
            <h3 class="card-title align-items-start flex-column">
                <span class="card-label fw-bold fs-3 mb-1">My Tasks</span>
                <span class="text-muted mt-1 fw-semibold fs-7">{{ $tasks->total() }} total tasks</span>
            </h3>
            <div class="card-toolbar">
                <a href="{{ route('tasks.create') }}" class="btn btn-primary">
                    <i class="ki-duotone ki-plus fs-2"></i>
                    New Task
                </a>
            </div>
        </div>

        {{-- Card Body --}}
        <div class="card-body py-4">
            @if($tasks->isEmpty())
                <div class="text-center py-10">
                    <i class="ki-duotone ki-information-5 fs-5x text-primary mb-5">
                        <span class="path1"></span>
                        <span class="path2"></span>
                        <span class="path3"></span>
                    </i>
                    <p class="text-gray-600 fs-5 fw-semibold mb-5">No tasks yet</p>
                    <a href="{{ route('tasks.create') }}" class="btn btn-primary">Create Your First Task</a>
                </div>
            @else
                <table class="table align-middle table-row-dashed fs-6 gy-5">
                    <thead>
                        <tr class="text-start text-gray-400 fw-bold fs-7 text-uppercase gs-0">
                            <th class="min-w-125px">Title</th>
                            <th class="min-w-100px">Status</th>
                            <th class="min-w-100px">Priority</th>
                            <th class="min-w-125px">Due Date</th>
                            <th class="text-end min-w-100px">Actions</th>
                        </tr>
                    </thead>
                    <tbody class="text-gray-600 fw-semibold">
                        @foreach($tasks as $task)
                            <tr>
                                <td>
                                    <div class="d-flex flex-column">
                                        <span class="text-gray-800 fw-bold text-hover-primary mb-1">
                                            {{ $task->title }}
                                        </span>
                                        @if($task->description)
                                            <span class="text-muted fs-7">
                                                {{ Str::limit($task->description, 50) }}
                                            </span>
                                        @endif
                                    </div>
                                </td>
                                <td>
                                    @php
                                        $statusBadges = [
                                            'pending' => 'badge-light-warning',
                                            'in_progress' => 'badge-light-info',
                                            'completed' => 'badge-light-success'
                                        ];
                                    @endphp
                                    <span class="badge {{ $statusBadges[$task->status] }}">
                                        {{ ucfirst(str_replace('_', ' ', $task->status)) }}
                                    </span>
                                </td>
                                <td>
                                    @php
                                        $priorityBadges = [
                                            'low' => 'badge-light-primary',
                                            'medium' => 'badge-light-warning',
                                            'high' => 'badge-light-danger'
                                        ];
                                    @endphp
                                    <span class="badge {{ $priorityBadges[$task->priority] }}">
                                        {{ ucfirst($task->priority) }}
                                    </span>
                                </td>
                                <td>
                                    @if($task->due_date)
                                        <span class="{{ $task->due_date->isPast() && $task->status !== 'completed' ? 'text-danger' : '' }}">
                                            {{ $task->due_date->format('M d, Y') }}
                                        </span>
                                    @else
                                        <span class="text-muted">-</span>
                                    @endif
                                </td>
                                <td class="text-end">
                                    <a href="{{ route('tasks.edit', $task) }}" class="btn btn-sm btn-light btn-active-light-primary">
                                        Edit
                                    </a>
                                    <form action="{{ route('tasks.destroy', $task) }}" method="POST" class="d-inline">
                                        @csrf
                                        @method('DELETE')
                                        <button type="submit" class="btn btn-sm btn-light btn-active-light-danger" onclick="return confirm('Delete this task?')">
                                            Delete
                                        </button>
                                    </form>
                                </td>
                            </tr>
                        @endforeach
                    </tbody>
                </table>

                {{-- Pagination --}}
                <div class="d-flex justify-content-between align-items-center mt-5">
                    <div class="text-muted">
                        Showing {{ $tasks->firstItem() }} to {{ $tasks->lastItem() }} of {{ $tasks->total() }} tasks
                    </div>
                    {{ $tasks->links() }}
                </div>
            @endif
        </div>
    </div>
</x-default-layout>
```

**⚠️ Layout Best Practices:**
- **ALWAYS** usar `<x-default-layout>` component
- **NEVER** usar `@extends('layouts._default')`
- Usar `@section('title')` y `@section('breadcrumbs')`
- Contenido directo (NO usar `@section('content')`)
- Scripts con `@push('scripts')`

### Step 11: Create Config File

**`config/tasks.php`:**
```php
<?php

return [
    /*
    |--------------------------------------------------------------------------
    | Task Manager Configuration
    |--------------------------------------------------------------------------
    */

    // Default task priority
    'default_priority' => env('TASKS_DEFAULT_PRIORITY', 'medium'),

    // Default pagination
    'per_page' => env('TASKS_PER_PAGE', 20),

    // Task statuses
    'statuses' => [
        'pending' => 'Pending',
        'in_progress' => 'In Progress',
        'completed' => 'Completed',
    ],

    // Task priorities
    'priorities' => [
        'low' => 'Low',
        'medium' => 'Medium',
        'high' => 'High',
    ],

    // Reminder settings
    'reminders' => [
        'enabled' => env('TASKS_REMINDERS_ENABLED', true),
        'days_before' => env('TASKS_REMINDER_DAYS', 1),
    ],
];
```

### Step 12: Create README.md

```markdown
# BITHOVEN Tasks Extension

Simple task management system for BITHOVEN platform.

## Features

- ✅ Create, edit, delete tasks
- ✅ Task priorities (low, medium, high)
- ✅ Task statuses (pending, in progress, completed)
- ✅ Due dates
- ✅ User-specific tasks
- ✅ Metronic UI integration

## Installation

### Via BITHOVEN Admin UI
1. Go to Admin → Extensions
2. Search for "tasks"
3. Click "Install"

### Via CLI (Local Development)
```bash
php artisan bithoven:extension:install tasks --local --path=../EXTENSIONS/bithoven-extension-tasks
```

### Via CLI (GitHub)
```bash
php artisan bithoven:extension:install tasks
```

## Usage

After installation, visit `/tasks` to start managing your tasks.

## Requirements

- BITHOVEN Core ^1.4.0
- PHP ^8.1
- Laravel ^11.0

## License

MIT
```

### Step 13: Initialize Git & Push to GitHub

```bash
# Initialize git
git init
git add .
git commit -m "feat: initial Tasks extension setup

- Basic CRUD operations
- Metronic UI integration
- User-specific tasks
- Priority and status management"

# Create GitHub repo and push
gh repo create bithoven-extension-tasks --public --source=. --remote=origin
git push -u origin main

# Create first release
git tag v1.0.0
git push --tags
```

### Step 14: Test Installation (Local Mode)

```bash
# En directorio CPANEL
cd /path/to/BITHOVEN/CPANEL

# Install en modo local
php artisan bithoven:extension:install tasks \
  --local \
  --path=../EXTENSIONS/bithoven-extension-tasks

# Verify installation
php artisan bithoven:extension:list

# Run demo seeder
php artisan db:seed --class="Bithoven\Tasks\Database\Seeders\TasksDemoSeeder"

# Test in browser
open http://localhost:8000/tasks
```

**Expected Output:**
```
+-------------+---------+----------+-------+
| Extension   | Version | Status   | Type  |
+-------------+---------+----------+-------+
| tasks       | 1.0.0   | ● Active | local |
+-------------+---------+----------+-------+
```

### Step 15: Test Installation (GitHub Mode)

```bash
# Uninstall local version
php artisan bithoven:extension:uninstall tasks

# Install from GitHub
php artisan bithoven:extension:install tasks

# Should download from GitHub and install
```

---

## 🔧 Advanced Features

### Add DataTables

**Install Yajra DataTables:**
```bash
# In extension directory
composer require yajra/laravel-datatables-oracle
```

**Create DataTable class:**
```bash
# In CPANEL (will be moved to extension)
php artisan datatables:make TasksDataTable
mv app/DataTables/TasksDataTable.php ../EXTENSIONS/bithoven-extension-tasks/src/DataTables/
```

**`src/DataTables/TasksDataTable.php`:**
```php
<?php

namespace Bithoven\Tasks\DataTables;

use Bithoven\Tasks\Models\Task;
use Yajra\DataTables\Html\Column;
use Yajra\DataTables\Services\DataTable;

class TasksDataTable extends DataTable
{
    public function dataTable($query)
    {
        return datatables()
            ->eloquent($query)
            ->addColumn('action', 'tasks::datatables.actions')
            ->editColumn('status', function (Task $task) {
                $badges = [
                    'pending' => 'warning',
                    'in_progress' => 'info',
                    'completed' => 'success'
                ];
                return '<span class="badge badge-light-'.$badges[$task->status].'">'.ucfirst(str_replace('_', ' ', $task->status)).'</span>';
            })
            ->editColumn('priority', function (Task $task) {
                $badges = [
                    'low' => 'primary',
                    'medium' => 'warning',
                    'high' => 'danger'
                ];
                return '<span class="badge badge-light-'.$badges[$task->priority].'">'.ucfirst($task->priority).'</span>';
            })
            ->editColumn('due_date', function (Task $task) {
                return $task->due_date ? $task->due_date->format('M d, Y') : '-';
            })
            ->rawColumns(['action', 'status', 'priority']);
    }

    public function query(Task $model)
    {
        return $model->newQuery()
            ->where('user_id', auth()->id())
            ->select('tasks.*');
    }

    public function html()
    {
        return $this->builder()
            ->setTableId('tasks-table')
            ->columns($this->getColumns())
            ->minifiedAjax()
            ->orderBy(1, 'desc')
            ->parameters([
                'dom' => 'Bfrtip',
                'buttons' => ['csv', 'excel', 'pdf'],
            ]);
    }

    protected function getColumns()
    {
        return [
            Column::make('id')->title('ID'),
            Column::make('title')->title('Title'),
            Column::make('status')->title('Status'),
            Column::make('priority')->title('Priority'),
            Column::make('due_date')->title('Due Date'),
            Column::computed('action')
                ->exportable(false)
                ->printable(false)
                ->width(100)
                ->addClass('text-end'),
        ];
    }
}
```

**Update Controller:**
```php
public function index(TasksDataTable $dataTable)
{
    return $dataTable->render('tasks::index');
}
```

**Update View:**
```blade
<x-default-layout>
    @section('title', 'My Tasks')
    
    <div class="card">
        <div class="card-header">
            <h3 class="card-title">Tasks</h3>
            <div class="card-toolbar">
                <a href="{{ route('tasks.create') }}" class="btn btn-primary">
                    <i class="ki-duotone ki-plus fs-2"></i>
                    New Task
                </a>
            </div>
        </div>
        <div class="card-body">
            {!! $dataTable->table(['class' => 'table align-middle table-row-dashed fs-6 gy-5']) !!}
        </div>
    </div>

    @push('scripts')
        {!! $dataTable->scripts() !!}
    @endpush
</x-default-layout>
```

### Add Policies

```bash
# In CPANEL
php artisan make:policy TaskPolicy --model=Bithoven\\Tasks\\Models\\Task
mv app/Policies/TaskPolicy.php ../EXTENSIONS/bithoven-extension-tasks/src/Policies/
```

**`src/Policies/TaskPolicy.php`:**
```php
<?php

namespace Bithoven\Tasks\Policies;

use App\Models\User;
use Bithoven\Tasks\Models\Task;

class TaskPolicy
{
    public function viewAny(User $user): bool
    {
        return $user->hasPermissionTo('view-tasks');
    }

    public function view(User $user, Task $task): bool
    {
        return $user->id === $task->user_id;
    }

    public function create(User $user): bool
    {
        return $user->hasPermissionTo('create-tasks');
    }

    public function update(User $user, Task $task): bool
    {
        return $user->id === $task->user_id 
            && $user->hasPermissionTo('edit-tasks');
    }

    public function delete(User $user, Task $task): bool
    {
        return $user->id === $task->user_id 
            && $user->hasPermissionTo('delete-tasks');
    }
}
```

**Register in Service Provider:**
```php
use Illuminate\Support\Facades\Gate;
use Bithoven\Tasks\Models\Task;
use Bithoven\Tasks\Policies\TaskPolicy;

public function boot(): void
{
    // ... existing code
    
    // Register policy
    Gate::policy(Task::class, TaskPolicy::class);
}
```

### Add Events & Listeners

**`src/Events/TaskCreated.php`:**
```php
<?php

namespace Bithoven\Tasks\Events;

use Bithoven\Tasks\Models\Task;
use Illuminate\Foundation\Events\Dispatchable;
use Illuminate\Queue\SerializesModels;

class TaskCreated
{
    use Dispatchable, SerializesModels;

    public function __construct(public Task $task)
    {
    }
}
```

**`src/Listeners/SendTaskCreatedNotification.php`:**
```php
<?php

namespace Bithoven\Tasks\Listeners;

use Bithoven\Tasks\Events\TaskCreated;
use Illuminate\Contracts\Queue\ShouldQueue;

class SendTaskCreatedNotification implements ShouldQueue
{
    public function handle(TaskCreated $event): void
    {
        // Send notification logic
    }
}
```

**Register in Service Provider:**
```php
use Illuminate\Support\Facades\Event;

public function boot(): void
{
    // ... existing code
    
    // Register events
    Event::listen(
        TaskCreated::class,
        [SendTaskCreatedNotification::class, 'handle']
    );
}
```

---

## 📚 Next Steps

### Documentation
1. Create detailed README.md with screenshots
2. Add API documentation (if applicable)
3. Create user guide
4. Update CHANGELOG.md for each version

### Testing
1. Write unit tests for models
2. Write feature tests for controllers
3. Test installation/uninstallation process
4. Test with different BITHOVEN versions

### Publishing
1. Create GitHub releases
2. Write release notes
3. Update version in extension.json
4. Tag releases properly

### Maintenance
1. Monitor GitHub issues
2. Keep dependencies updated
3. Test with new BITHOVEN versions
4. Respond to user feedback

---

## 🔗 Related Documentation

- **[Extension JSON Schema](../CPANEL/docs/extensions/EXTENSION-JSON-SCHEMA.md)** - Complete extension.json reference
- **[Installation Flow](../CPANEL/docs/extensions/EXTENSION-INSTALLATION-FLOW.md)** - How installation works
- **[Seeders Best Practices](./guides/SEEDERS-BEST-PRACTICES.md)** - CRITICAL for Fix Extension
- **[Quick Start Guide](./guides/QUICK-START.md)** - 5-minute quick start
- **[Fix Extension System](./guides/FIX-EXTENSION-SYSTEM.md)** - Understanding Fix Extension

---

## ⚠️ Common Mistakes to Avoid

### ❌ Wrong Table Names
```php
// ❌ WRONG - No prefix
Schema::create('items', function (Blueprint $table) {

// ✅ CORRECT - Use slug as prefix (or just slug for main table)
Schema::create('tasks', function (Blueprint $table) {
Schema::create('tasks_comments', function (Blueprint $table) {
```

### ❌ Wrong Seeder Patterns
```php
// ❌ WRONG - Creates duplicates on re-run
Status::updateOrCreate(['name' => 'Active'], [...]);

// ✅ CORRECT - Idempotent
Status::updateOrCreate(['id' => 1], ['name' => 'Active', ...]);
```

### ❌ Wrong Layout Usage
```blade
{{-- ❌ WRONG --}}
@extends('layouts._default')

{{-- ✅ CORRECT --}}
<x-default-layout>
```

### ❌ Mixing Core and Demo Data
```php
// ❌ WRONG - All in one seeder
public function run(): void
{
    // Base statuses
    Status::create(['name' => 'Active']);
    
    // Demo data
    Task::create(['title' => 'Test task']);
}

// ✅ CORRECT - Separate seeders
// extension.json:
"seeders": {
  "core": ["StatusSeeder"],
  "demo": ["TasksDemoSeeder"]
}
```

---

## 🎯 Checklist: Extension Release

Before publishing your extension, verify:

- [ ] `composer.json` complete with correct namespace
- [ ] `extension.json` with all required fields
- [ ] Migrations use correct table naming (slug prefix)
- [ ] Seeders separated (core vs demo)
- [ ] Views use `<x-default-layout>`
- [ ] Routes use middleware `['web', 'auth']`
- [ ] Permissions listed in extension.json
- [ ] README.md with installation instructions
- [ ] CHANGELOG.md created
- [ ] LICENSE file
- [ ] Git repository initialized
- [ ] GitHub repository created
- [ ] First release tagged (v1.0.0)
- [ ] Tested installation (local mode)
- [ ] Tested installation (GitHub mode)
- [ ] Tested uninstallation
- [ ] Verified no database orphans after uninstall

---

**¡Felicidades!** 🎉 Has creado tu primera extensión BITHOVEN lista para producción.

**Support:** Open issues at [GitHub](https://github.com/Madniatik/bithoven-cpanel/issues)
