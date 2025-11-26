# Changelog - Bithoven Extensions Documentation

All notable changes to the extensions system will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

---

## [1.3.0] - 2025-11-18

### Added
- **DEVELOPMENT-MODE.md** - Complete guide for Development Mode system
  - Live editing with symlinks
  - Works with ANY extension type (VCS/Composer)
  - CLI and UI usage
  - Troubleshooting guide
  - Best practices

### Changed
- **DEVELOPMENT-WORKFLOW.md** - Updated to v2.1.0
  - Added local installation instructions (CLI + UI)
  - Added Development Mode workflow section
  - Updated CLI commands reference
  - Added dev-mode status check examples
  
- **README.md** - Updated to v1.3.0
  - Added Development Mode to Getting Started
  - Added local installation commands
  - Added dev-mode commands
  - Added marketplace refresh feature
  
- **INDEX.md** - Updated to v1.3.0
  - Added DEVELOPMENT-MODE.md to Development section
  - Updated "I want to create a new extension" workflow
  - Updated document count (15 guides + 3 root = 18 total)

### Features Implemented
- Local installation via UI (modal with path input)
- Development mode enable/disable via UI
- DEV MODE badge in extension overview
- Marketplace cache refresh button
- Composer require bug fixes
- VCS restriction removed from dev-mode

### Statistics
- **Total Guides:** 15 (was 14 - +7% increase)
- **Total Documentation:** 18 files (was 17 - +6% increase)
- **New Content:** ~400 lines of dev-mode documentation

---

## [1.2.0] - 2025-11-17

### Added
- **DATABASE-CONVENTIONS.md** - Comprehensive database conventions guide (730 lines)
  - Table naming rules with `{slug}_` prefix requirement
  - Foreign key naming conventions
  - Index best practices
  - Migration patterns and examples
  
- **DEVELOPER-GUIDE.md** - Complete step-by-step extension creation guide
- **EXTENSION-STRUCTURE.md** - Standard extension directory structure
- **MIGRATIONS-GUIDELINES.md** - Migration naming and ordering guidelines
- **SERVICE-PROVIDERS.md** - Service provider patterns and examples
- **DEVELOPMENT-WORKFLOW.md** - Complete development lifecycle guide
- **ARCHITECTURE.md** - System architecture overview
- **FRESH-INSTALL-SYSTEM.md** - Fresh install process documentation
- **BACKUP-RECOVERY.md** - Backup strategies and recovery procedures
- **EXTENSION-MANAGER-API.md** - Complete API reference

### Changed
- **INDEX.md** - Updated to reflect all newly created guides (14 guides total)
- **README.md** - Added DATABASE-CONVENTIONS.md to Core Concepts section
- **Documentation status** - Changed icons from 📝 (planned) to ✅ (complete) for all new guides
- **Version bump** - Updated to v1.2.0 across all documentation files
- **Document counts** - Now 17 total documents (14 guides + 3 root docs), was 8
- **Last updated dates** - Updated to November 17, 2025

### Statistics
- **Total Guides:** 14 (was 5 - +180% increase)
- **Total Documentation:** 17 files (was 8 - +112% increase)
- **New Content:** ~5,000 lines of comprehensive documentation
- **Coverage Areas:** Development, Database, System Operations, Architecture, API Reference

---

## [1.0.1] - 2025-11-15

### Changed
- **Documentation consolidation** - Centralized all extension docs in `/EXTENSIONS/DOCUMENTATION/`
- **Copilot instructions reorganization** - Created separate `.github/copilot-instructions-extensions.md`
- **Token optimization** - Extension docs now load only when needed (not in daily workflow)

### Removed
- **Scattered documentation** - Archived 5 old files to `.github/docs/archive/`
  - `EXTENSION-DEVELOPMENT.md`
  - `EXTENSION-SYSTEM-COMPLETE.md`
  - `QUICK-GUIDE-EXTENSIONS.md`
  - `EXTENSION-RELEASE-PROTOCOL.md`
  - `app/Core/documentation/guides/CREATING-EXTENSIONS.md`

---

## [1.0.0] - 2025-11-15

### Added
- **Complete documentation structure** for Bithoven Extensions (~10,000 lines)
- **SEEDERS-BEST-PRACTICES.md** - Critical guide for writing seeders with fixed IDs (2,500 lines)
- **QUICK-START.md** - Step-by-step tutorial for first extension (1,800 lines)
- **FIX-EXTENSION-SYSTEM.md** - Complete Fix Extension guide (1,600 lines)
- **AI-AGENT-INSTRUCTIONS.md** - Dedicated guide for AI coding assistants (2,000 lines)
- **Fix Extension System** - Repair mode that preserves user data
- **Fresh Install System** - Complete reinstall mode with automatic backups
- **INDEX.md** - Complete navigation guide for all documentation

### Changed
- **Seeder pattern** - All base seeders now use `updateOrCreate(['id' => ...])` instead of `['name' => ...]`
  - Tickets extension: CategorySeeder (IDs 1-8)
  - Tickets extension: TemplatesResponsesSeeder (Templates IDs 1-10, Responses IDs 1-24)
  - Tickets extension: AutomationRulesSeeder (IDs 1-6)
- **Fresh Install behavior** - Now properly refreshes config cache after uninstall
- **ExtensionManager::uninstall()** - Fixed parameter type (expects array, not boolean)

### Fixed
- **Fix Extension duplicate bug** - Using field-based updateOrCreate created duplicates when users edited those fields
- **Fresh Install "already installed" error** - Config cache not refreshed after uninstall
- **Type error in freshInstall()** - Passing `true` instead of `['remove_data' => true]` to uninstall()

### Removed
- **Load demo data checkboxes** - Removed from Fix Extension and Fresh Install modals
- **Demo data integration** - Base seeders no longer load demo data automatically

---

## [0.9.0] - 2025-11-14 (Pre-Documentation)

### Added
- Extension card redesign with dropdown menus
- Two-mode reinstall system (Fix / Fresh)
- Automatic backup creation before reinstall operations
- Activity logging for all extension operations

### Fixed
- Table name inconsistencies (`canned_responses` → `ticket_canned_responses`)
- Validation rules with explicit table names
- Missing Auth facade imports
- Variable name typos in ExtensionController

---

## Future Releases

### Planned for [1.1.0]
- [ ] Extension generator CLI command
- [ ] Extension scaffolding tool
- [ ] Automated testing suite for extensions
- [ ] Extension marketplace integration

### Planned for [1.2.0]
- [ ] Extension dependency graph visualization
- [ ] Rollback system for failed installations
- [ ] Extension health checks
- [ ] Performance monitoring

---

## Migration Notes

### Upgrading to 1.0.0

**For existing extensions with seeders:**

1. **Update all base seeders** to use fixed IDs:
   ```php
   // OLD (breaks Fix Extension):
   Model::updateOrCreate(['name' => 'Base Item'], [...]);
   
   // NEW (works correctly):
   Model::updateOrCreate(['id' => 1], ['name' => 'Base Item', ...]);
   ```

2. **Define ID ranges** in comments:
   ```php
   /**
    * ID Ranges:
    * - IDs 1-10: Base records (restored by Fix Extension)
    * - IDs > 10: Custom user records (preserved)
    */
   ```

3. **Test Fix Extension**:
   - Edit base records (change names/slugs)
   - Create custom records
   - Run Fix Extension
   - Verify: base restored, custom preserved, no duplicates

4. **Separate demo data**:
   - Move demo data to DemoSeeder.php
   - DatabaseSeeder only for essential data

---

## Breaking Changes

### Version 1.0.0

**Seeder Pattern Change:**
- Extensions using `updateOrCreate(['name' => ...])` or `updateOrCreate(['slug' => ...])` will create duplicates on Fix Extension
- **Action Required:** Update all base seeders to use `updateOrCreate(['id' => ...])`

**Fresh Install Behavior:**
- Now requires array parameter: `uninstall($name, ['remove_data' => true])`
- Direct boolean (`uninstall($name, true)`) no longer supported
- **Action Required:** Update any direct calls to `uninstall()`

---

## Contributors

- **Madniatik** - Initial implementation and documentation
- **Claude (AI Agent)** - Code generation and documentation assistance

---

**Documentation Version:** 1.0.0  
**Last Updated:** 15 de noviembre de 2025
