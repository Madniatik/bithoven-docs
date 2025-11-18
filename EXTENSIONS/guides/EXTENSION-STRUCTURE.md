# Extension Structure Guide

**Version:** 2.0.0  
**Last Updated:** 17 de noviembre de 2025  
**Status:** STABLE

---

## 📁 Standard Directory Structure

\`\`\`
bithoven-extension-{slug}/
├── composer.json
├── extension.json
├── README.md
├── CHANGELOG.md
├── LICENSE
├── config/{slug}.php
├── database/
│   ├── migrations/
│   └── seeders/
├── resources/
│   ├── lang/ (optional)
│   └── views/
├── routes/
│   ├── web.php
│   └── api.php (optional)
├── src/
│   ├── {Slug}ServiceProvider.php
│   ├── Http/Controllers/
│   ├── Models/
│   ├── DataTables/ (optional)
│   ├── Policies/ (optional)
│   └── Services/ (optional)
├── tests/ (optional)
│   ├── Feature/
│   └── Unit/
└── docs/ (optional)
\`\`\`

---

## 📄 Required Files

### composer.json
Package definition, dependencies, and autoloading.

### extension.json
Extension metadata for BITHOVEN system.

### README.md
User and developer documentation.

### src/{Slug}ServiceProvider.php
Bootstrap extension (routes, views, migrations).

---

## 🔧 Namespace Convention

Pattern: `Bithoven\{PascalCaseSlug}\{Directory}\{Class}`

Examples:
- `Bithoven\Tasks\TasksServiceProvider`
- `Bithoven\Tasks\Models\Task`
- `Bithoven\Tasks\Http\Controllers\TaskController`

---

## 🎨 View Namespace

Pattern: `{slug}::{view}`

\`\`\`php
return view('tasks::index', compact('tasks'));
\`\`\`

---

## 🔗 Related Documentation

- [Developer Guide](../DEVELOPER-GUIDE.md)
- [Database Conventions](./DATABASE-CONVENTIONS.md)
- [Seeders Best Practices](./SEEDERS-BEST-PRACTICES.md)