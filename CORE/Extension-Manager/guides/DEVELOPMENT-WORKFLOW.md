# Development Workflow for BITHOVEN Extensions

**Version:** 2.1.0  
**Last Updated:** 18 de noviembre de 2025  
**Status:** STABLE

---

## 🎯 Overview

Esta guía describe el flujo de trabajo completo desde la idea inicial hasta la publicación de una extensión en producción.

---

## 📋 Phase 1: Planning

### 1.1 Define Requirements

**Questions:**
- ¿Qué problema resuelve?
- ¿Qué features necesita?
- ¿Necesita permisos especiales?
- ¿Qué tablas de sistema usa?

### 1.2 Design Database

**Sketch tables:**
```
tasks
├── id
├── user_id (FK → users)
├── title
├── status
└── timestamps

tasks_categories
├── id
├── name
└── timestamps
```

### 1.3 Plan Routes

```
GET    /tasks           → index
GET    /tasks/create    → create
POST   /tasks           → store
GET    /tasks/{id}/edit → edit
PUT    /tasks/{id}      → update
DELETE /tasks/{id}      → destroy
```

---

## 🚀 Phase 2: Setup

### 2.1 Create Directory

```bash
cd /path/to/BITHOVEN/EXTENSIONS
mkdir bithoven-extension-tasks
cd bithoven-extension-tasks
```

### 2.2 Initialize Composer

```bash
composer init
```

### 2.3 Create Structure

```bash
mkdir -p {src,database/{migrations,seeders},resources/views,routes,config,tests}
```

### 2.4 Initialize Git

```bash
git init
git add .
git commit -m "feat: initial project structure"
```

---

## 💻 Phase 3: Development

### 3.1 Create Core Files

**Order:**
1. `extension.json` - Metadata
2. `composer.json` - Package definition
3. `src/{Slug}ServiceProvider.php` - Entry point
4. `config/{slug}.php` - Configuration

### 3.2 Database

**Order:**
1. Create migrations (main tables first)
2. Create models
3. Create seeders (separate core/demo)
4. Test migrations

```bash
# Test
php artisan migrate
php artisan migrate:rollback
php artisan migrate:fresh
```

### 3.3 Controllers & Routes

**Order:**
1. Create controller methods
2. Define routes
3. Test each route

```bash
# Test
php artisan route:list | grep tasks
```

### 3.4 Views

**Order:**
1. Create index (list)
2. Create create/edit forms
3. Create partials
4. Style with Metronic

### 3.5 Business Logic

**Optional but recommended:**
1. Create services
2. Create policies
3. Create events/listeners

---

## 🧪 Phase 4: Testing (Local)

### 4.1 Install Locally

**Option A: CLI Installation**

```bash
cd /path/to/CPANEL
php artisan bithoven:extension:install-local tasks ../EXTENSIONS/bithoven-extension-tasks
```

**Option B: UI Installation**

1. Ir a http://localhost:8000/app/extensions/marketplace
2. Click "Install Local"
3. Ingresar path: `../EXTENSIONS/bithoven-extension-tasks`
4. Opciones:
   - ✅ Run migrations
   - ✅ Enable after install
   - ☐ Enable dev-mode (opcional)

**What happens:**
- Composer installs package from local path
- Files copied to `vendor/bithoven/tasks`
- Repository added to `composer.json`
- Extension registered in configs
- Migrations executed (if selected)
- **Core seeders executed automatically** (from `extension.json` → `seeders.core`) ✨
- Extension enabled (if selected)

> **Note (Nov 2025):** A bug existed where core seeders were NOT executed during UI local installation. This was fixed in CPANEL v1.7.0+. If you're on an older version, core data (permissions, configuration) won't be installed automatically and you'll need to run seeders manually.

### 4.2 Enable Development Mode

**Development mode allows live editing without reinstalling.**

**CLI:**
```bash
php artisan bithoven:extension:dev-mode tasks --enable --path=../EXTENSIONS/bithoven-extension-tasks
```

**UI:**
1. Ir a http://localhost:8000/app/extensions/tasks
2. Sección "Development Tools"
3. Click "Enable Dev-Mode"
4. Ingresar path local

**What happens:**
- `vendor/bithoven/tasks` → renamed to `tasks.repo` (backup)
- Symlink created: `tasks → ../EXTENSIONS/bithoven-extension-tasks`
- Now edits in your repo are immediately reflected
- Works with **ANY** installed extension (VCS or Composer)

**Benefits:**
- ✅ Edit code in real-time
- ✅ No need to reinstall after changes
- ✅ Original composer files preserved in `.repo`
- ✅ Easy toggle on/off

**Disable dev-mode:**
```bash
php artisan bithoven:extension:dev-mode tasks --disable
```
- Symlink removed
- `tasks.repo` → restored to `tasks`
- Back to composer-installed version

### 4.3 Test Features

**Checklist:**
- [ ] All routes work
- [ ] CRUD operations functional
- [ ] Permissions enforced
- [ ] Views render correctly
- [ ] No console errors
- [ ] Database constraints work

### 4.4 Test Uninstall

```bash
php artisan bithoven:extension:uninstall tasks
```

**Verify:**
- [ ] Tables dropped (or data retained if intended)
- [ ] Permissions removed
- [ ] Menu items removed
- [ ] No orphaned data
- [ ] Vendor files removed
- [ ] Repository removed from composer.json
- [ ] Extension removed from configs

### 4.5 Reinstall & Reseed

```bash
php artisan bithoven:extension:install-local tasks ../EXTENSIONS/bithoven-extension-tasks
php artisan db:seed --class="Bithoven\Tasks\Database\Seeders\TasksDemoSeeder"
```

---

## 🔄 Development Mode Workflow

### Recommended Development Flow

**1. Initial Setup:**
```bash
# Install extension locally
php artisan bithoven:extension:install-local tasks ../EXTENSIONS/bithoven-extension-tasks

# Enable dev-mode for live editing
php artisan bithoven:extension:dev-mode tasks --enable --path=../EXTENSIONS/bithoven-extension-tasks
```

**2. Development:**
- Edit files in `/EXTENSIONS/bithoven-extension-tasks/`
- Changes reflect immediately (no reinstall needed)
- Test in browser
- Git commit as you go

**3. Testing Full Cycle:**
```bash
# Disable dev-mode to test with composer-installed version
php artisan bithoven:extension:dev-mode tasks --disable

# Uninstall
php artisan bithoven:extension:uninstall tasks

# Reinstall fresh
php artisan bithoven:extension:install-local tasks ../EXTENSIONS/bithoven-extension-tasks
```

**4. Before Publishing:**
- Disable dev-mode
- Test complete install/uninstall cycle
- Verify all migrations work
- Push to GitHub

### Dev-Mode Status Check

**CLI:**
```bash
php artisan bithoven:extension:dev-mode tasks --status
```

**Output example:**
```
Development Mode Status for 'tasks'
┌────────────┬─────────────────────────────────────────┐
│ Property   │ Value                                   │
├────────────┼─────────────────────────────────────────┤
│ Enabled    │ Yes                                     │
│ Local Path │ ../EXTENSIONS/bithoven-extension-tasks  │
│ Activated  │ 2025-11-18 03:58:24                     │
│ By User    │ Admin (ID: 1)                           │
└────────────┴─────────────────────────────────────────┘
```

---

## 📝 Phase 5: Documentation

### 5.1 Create README.md

```markdown
# BITHOVEN Tasks Extension

## Features
- Create/edit/delete tasks
- Task priorities
- Due dates

## Installation
Via BITHOVEN Admin UI or CLI

## Usage
Visit /tasks after installation

## Requirements
- BITHOVEN Core ^1.4.0
- PHP ^8.1

## License
MIT
```

### 5.2 Create CHANGELOG.md

```markdown
# Changelog

## [1.0.0] - 2025-01-01

### Added
- Initial release
- Basic CRUD operations
- Task priorities
- Due dates
```

### 5.3 Update extension.json

**Add changelog:**
```json
{
  "changelog": {
    "v1.0.0": {
      "date": "2025-01-01",
      "changes": ["Initial release"],
      "migration_notes": "Creates tasks table",
      "breaking_changes": false,
      "components": ["code", "database"]
    }
  }
}
```

---

## 🚢 Phase 6: Publishing

### 6.1 Create GitHub Repository

```bash
gh repo create bithoven-extension-tasks --public --source=. --remote=origin
```

### 6.2 Push Code

```bash
git add .
git commit -m "feat: complete Tasks extension v1.0.0"
git push -u origin main
```

### 6.3 Create Release

```bash
git tag v1.0.0
git push --tags
```

**Or via GitHub UI:**
1. Go to Releases
2. Click "Draft a new release"
3. Tag: `v1.0.0`
4. Title: `Tasks Extension v1.0.0`
5. Description: Copy from CHANGELOG
6. Publish

---

## 🔄 Phase 7: Maintenance

### 7.1 Monitor Issues

**GitHub Issues:**
- Respond to bug reports
- Consider feature requests
- Label and prioritize

### 7.2 Update Dependencies

```bash
composer update
```

### 7.3 Test with New BITHOVEN Versions

**When BITHOVEN updates:**
```bash
# Update CPANEL
cd /path/to/CPANEL
git pull
composer update

# Test extension
php artisan bithoven:extension:uninstall tasks
php artisan bithoven:extension:install tasks
```

---

## 🔧 Development Best Practices

### Use Branches

```bash
# Feature
git checkout -b feature/add-categories
# ... develop
git commit -m "feat: add task categories"
git push origin feature/add-categories
# Create PR

# Bugfix
git checkout -b fix/validation-error
# ... fix
git commit -m "fix: validation error on empty title"
git push origin fix/validation-error
```

### Commit Convention

```
feat: add new feature
fix: bug fix
docs: documentation changes
style: formatting changes
refactor: code restructuring
test: add tests
chore: maintenance
```

### Version Bumping

**Semantic Versioning:**
- `1.0.0` → `1.0.1` - Patch (bug fixes)
- `1.0.0` → `1.1.0` - Minor (new features, backwards compatible)
- `1.0.0` → `2.0.0` - Major (breaking changes)

---

## 🐛 Debugging Workflow

### Issue: Routes not working

```bash
# Check routes loaded
php artisan route:list | grep tasks

# Clear cache
php artisan route:clear
php artisan optimize:clear
```

### Issue: Views not found

```bash
# Check view namespace
php artisan tinker
>>> view()->exists('tasks::index')

# Clear view cache
php artisan view:clear
```

### Issue: Migrations failing

```bash
# Check migration status
php artisan migrate:status

# Rollback specific
php artisan migrate:rollback --step=1

# Fresh start
php artisan migrate:fresh
```

### Issue: Permissions not working

```bash
# Check permission exists
php artisan tinker
>>> Permission::where('name', 'view-tasks')->exists()

# Assign to role
>>> $role = Role::find(1);
>>> $role->givePermissionTo('view-tasks');
```

---

## 📊 Performance Optimization

### Database

**Add indexes:**
```php
$table->index('status');
$table->index(['user_id', 'status']);
```

**Eager loading:**
```php
// ❌ N+1 problem
$tasks = Task::all();
foreach ($tasks as $task) {
    echo $task->user->name;
}

// ✅ Eager load
$tasks = Task::with('user')->get();
```

### Caching

```php
use Illuminate\Support\Facades\Cache;

public function index()
{
    $tasks = Cache::remember('user.tasks.' . auth()->id(), 60, function () {
        return Task::where('user_id', auth()->id())->get();
    });
    
    return view('tasks::index', compact('tasks'));
}
```

---

## 🔗 Useful Commands

### Extension Management

```bash
# List extensions
php artisan bithoven:extension:list

# Install from GitHub (VCS)
php artisan bithoven:extension:install {slug}

# Install from local path
php artisan bithoven:extension:install-local {slug} {path}

# Enable/disable
php artisan bithoven:extension:enable {slug}
php artisan bithoven:extension:disable {slug}

# Development mode
php artisan bithoven:extension:dev-mode {slug} --enable --path={path}
php artisan bithoven:extension:dev-mode {slug} --disable
php artisan bithoven:extension:dev-mode {slug} --status

# Uninstall
php artisan bithoven:extension:uninstall {slug}
```

### Development

```bash
# Clear caches
php artisan optimize:clear

# Run migrations
php artisan migrate

# Seed
php artisan db:seed --class="Bithoven\Tasks\Database\Seeders\TasksDemoSeeder"

# Tinker
php artisan tinker
```

### Git

```bash
# Status
git status --short

# Commit
git add .
git commit -m "feat: description"

# Push
git push origin main

# Tag
git tag v1.0.0
git push --tags
```

---

## 📋 Checklist: Before Release

- [ ] All features working
- [ ] Tests passing
- [ ] No console errors
- [ ] README.md complete
- [ ] CHANGELOG.md updated
- [ ] LICENSE file present
- [ ] extension.json complete
- [ ] Migrations reversible
- [ ] Seeders idempotent
- [ ] No hardcoded paths
- [ ] Proper namespacing
- [ ] GitHub repository created
- [ ] Version tagged
- [ ] Release notes published

---

## 🔗 Related Documentation

- **[Developer Guide](../DEVELOPER-GUIDE.md)** - Step-by-step creation
- **[Architecture](./ARCHITECTURE.md)** - System overview
- **[Testing Guide](./TESTING-GUIDE.md)** - Testing strategies
- **[Quick Start](./QUICK-START.md)** - Fast start

---

**Remember:** Good workflow = Better code. Take time to plan, develop systematically, test thoroughly!
