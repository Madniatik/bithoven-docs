# Bithoven Extensions - Complete Documentation

**Version:** 1.3.0  
**Last Updated:** 18 de noviembre de 2025  
**Maintainer:** Madniatik

---

## 📚 Table of Contents

### Getting Started
- [Quick Start Guide](guides/QUICK-START.md) - Start developing your first extension
- [Development Mode](guides/DEVELOPMENT-MODE.md) - Live editing with symlinks (NEW!)
- [Extension Architecture](guides/ARCHITECTURE.md) - Understanding the extension system
- [Development Workflow](guides/DEVELOPMENT-WORKFLOW.md) - Best practices and workflow

### Core Concepts
- [Configuration System](guides/CONFIGURATION-SYSTEM.md) - Settings, repos, auth
- [Extension Structure](guides/EXTENSION-STRUCTURE.md) - File organization and structure
- [Database Conventions](guides/DATABASE-CONVENTIONS.md) - Table naming, indexes, foreign keys
- [Seeders Best Practices](guides/SEEDERS-BEST-PRACTICES.md) - **CRITICAL:** How to write seeders properly
- [Migrations Guidelines](guides/MIGRATIONS-GUIDELINES.md) - Database migrations for extensions
- [Service Providers](guides/SERVICE-PROVIDERS.md) - Extension service providers

### Advanced Topics
- [Fix Extension System](guides/FIX-EXTENSION-SYSTEM.md) - How Fix Extension works
- [Fresh Install System](guides/FRESH-INSTALL-SYSTEM.md) - Complete reinstallation process
- [Backup & Recovery](guides/BACKUP-RECOVERY.md) - Automatic backups and restoration
- [Extension Manager API](guides/EXTENSION-MANAGER-API.md) - Complete API reference

### Templates & Examples
- [Extension Template](templates/extension-template/) - Complete starter template
- [Seeder Templates](templates/seeders/) - Pre-configured seeder templates
- [Migration Templates](templates/migrations/) - Migration boilerplate
- [Real Examples](examples/) - Working examples from Tickets and Dummy extensions

### For AI Agents
- [COPILOT Instructions](COPILOT/) - Specific instructions for AI coding assistants

---

## 🎯 Quick Links

### Most Important Documents
1. **[SEEDERS-BEST-PRACTICES.md](guides/SEEDERS-BEST-PRACTICES.md)** - Read this FIRST! Critical for Fix Extension compatibility
2. **[QUICK-START.md](guides/QUICK-START.md)** - Get started in 5 minutes
3. **[COPILOT/AI-AGENT-INSTRUCTIONS.md](COPILOT/AI-AGENT-INSTRUCTIONS.md)** - For AI assistants working on extensions

### By Topic
- **Creating Extensions:** [QUICK-START.md](guides/QUICK-START.md) → [EXTENSION-STRUCTURE.md](guides/EXTENSION-STRUCTURE.md)
- **Database:** [DATABASE-CONVENTIONS.md](guides/DATABASE-CONVENTIONS.md) → [SEEDERS-BEST-PRACTICES.md](guides/SEEDERS-BEST-PRACTICES.md) → [MIGRATIONS-GUIDELINES.md](guides/MIGRATIONS-GUIDELINES.md)
- **Troubleshooting:** [FIX-EXTENSION-SYSTEM.md](guides/FIX-EXTENSION-SYSTEM.md) → [BACKUP-RECOVERY.md](guides/BACKUP-RECOVERY.md)

---

## 📦 Extension Ecosystem

### Available Extensions
- **bithoven-extension-tickets** - Complete ticket management system
- **bithoven-extension-dummy** - Minimal example extension for learning

### Extension Registry
Extensions are managed through:
- **ExtensionManager Service** - Core management API
- **Admin UI** - Web interface at `/admin/extensions`
- **CLI Commands** - Artisan commands (`php artisan bithoven:extension:*`)

---

## 🛠️ Development Tools

### CLI Commands
```bash
# List all extensions
php artisan bithoven:extension:list

# Install from GitHub (VCS)
php artisan bithoven:extension:install {name}

# Install from local path
php artisan bithoven:extension:install-local {name} {path}

# Development Mode (live editing)
php artisan bithoven:extension:dev-mode {name} --enable --path={path}
php artisan bithoven:extension:dev-mode {name} --disable
php artisan bithoven:extension:dev-mode {name} --status

# Enable/Disable extension
php artisan bithoven:extension:enable {name}
php artisan bithoven:extension:disable {name}

# Uninstall extension
php artisan bithoven:extension:uninstall {name}
```

### Admin UI Features
- **Install/Uninstall** - Manage extension lifecycle
- **Install Local** - Install from local development path (NEW!)
- **Development Mode** - Enable live editing with symlinks (NEW!)
- **Enable/Disable** - Control extension activation
- **Fix Extension** - Repair corrupted configuration (preserves data)
- **Fresh Install** - Complete reinstall from scratch (deletes data)
- **Backup Management** - Automatic backups before destructive operations
- **Update Management** - GitHub integration for updates
- **Marketplace Refresh** - Clear cache and reload from GitHub (NEW!)

---

## ⚠️ Critical Information

### Before You Start
1. **Read [SEEDERS-BEST-PRACTICES.md](guides/SEEDERS-BEST-PRACTICES.md)** - Failure to follow seeder patterns will break Fix Extension
2. **Use fixed IDs for base records** - Essential for Fix Extension to work correctly
3. **Separate demo data** - Keep DemoSeeder separate from DatabaseSeeder
4. **Test Fix Extension** - Always test that Fix Extension restores edited records properly

### Common Pitfalls
❌ Using `updateOrCreate(['name' => ...])` for base records → Creates duplicates  
❌ Mixing demo data with base seeders → Pollutes essential data  
❌ Not defining ID ranges → Custom records get overwritten  
❌ Forgetting to backup → Data loss on Fresh Install  

✅ Use `updateOrCreate(['id' => ...])` for base records  
✅ Keep DemoSeeder separate  
✅ Document ID ranges (1-N base, >N custom)  
✅ Test both Fix and Fresh Install thoroughly  

---

## 📖 Documentation Standards

### For Extension Developers
Each extension should include:
- `README.md` - Overview and installation
- `CHANGELOG.md` - Version history
- `docs/` - Detailed documentation
- `extension.json` - Extension metadata

### For Contributors
When updating this documentation:
- Keep examples working and tested
- Update version numbers and dates
- Follow existing formatting
- Add to CHANGELOG.md

---

## 🔗 Related Resources

### Main Project Documentation
- **Copilot Instructions:** `/Users/madniatik/CODE/LARAVEL/BITHOVEN/CPANEL/.github/copilot-instructions.md`
- **Core System:** `/Users/madniatik/CODE/LARAVEL/BITHOVEN/CPANEL/.github/copilot-core/CORE-SYSTEM.md`
- **Extension System (CPANEL):** `/Users/madniatik/CODE/LARAVEL/BITHOVEN/CPANEL/.github/copilot-core/EXTENSION-DEVELOPMENT.md`

### External Links
- **Laravel Documentation:** https://laravel.com/docs
- **Composer Documentation:** https://getcomposer.org/doc/
- **GitHub API:** https://docs.github.com/en/rest

---

## 📝 Changelog

See [CHANGELOG.md](CHANGELOG.md) for version history and updates.

---

## 🤝 Contributing

Contributions are welcome! Please:
1. Follow existing patterns and conventions
2. Update documentation for any changes
3. Test thoroughly (both Fix and Fresh Install)
4. Update CHANGELOG.md

---

## 📄 License

This documentation is part of the Bithoven project. See individual extension licenses for details.

---

**Need Help?**
- Check the guides in `guides/`
- Look at working examples in `examples/`
- Review AI agent instructions in `COPILOT/`
