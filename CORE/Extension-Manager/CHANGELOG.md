# Changelog - Bithoven Extensions Documentation

All notable changes to the extensions system will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

---

## [2.0.0] - 2025-11-28

### Changed - MAJOR REFACTORING COMPLETED ✅
- **ExtensionManagerController Refactoring v2.0.0**
  - **Reduced complexity:** 2,090 lines → 450 lines (78% reduction)
  - **Methods extracted:** 39 methods → 9 methods (77% reduction)
  - **PHPStan errors:** 100 errors → 0 errors (100% fixed)
  - **Test coverage:** 107/113 tests → 113/113 tests (100% coverage)
  - **Architecture:** Monolithic controller → Action-based architecture
  - **Duration:** ~8 hours across 7 phases (25-28 Nov 2025)

### Added
- **12 Action Classes** (Domain-Driven Design)
  - Backup Domain: CreateBackupAction, RestoreBackupAction, DeleteBackupAction
  - Configuration Domain: UpdateConfigurationAction, ResetConfigurationAction
  - Installation Domain: InstallExtensionAction, UninstallExtensionAction
  - Update Domain: UpdateExtensionAction, RollbackUpdateAction, CheckUpdatesAction
  - Marketplace: SearchMarketplaceAction, RefreshMarketplaceAction

- **6 Validation Traits**
  - ValidatesBackups, ValidatesConfiguration, ValidatesExtensions
  - ValidatesInstallation, ValidatesMarketplace, ValidatesUpdate

- **4 Response Helpers**
  - BackupResponses, ConfigurationResponses, InstallationResponses, UpdateResponses

### Improved
- **Type Safety:** Strict types in all new classes
- **Error Handling:** Consistent exception handling across all actions
- **Code Organization:** Clear separation of concerns (Actions, Validation, Responses)
- **Maintainability:** Single Responsibility Principle applied
- **Testability:** Each action is independently testable

### Documentation
- **Refactoring Reports:** Moved to `reports/refactoring.v1/`
  - REFACTORING-STATUS.md - Complete overview
  - FASE-2-COMPLETE.md through FASE-7-COMPLETE.md
  - BASELINE-METRICS.md - Before/After comparison
  - EXTENSION-MANAGER-v2.0.0-REFACTORING.md - Technical details

### Breaking Changes
- None - All public APIs maintained backward compatibility
- Internal architecture completely redesigned but external interfaces unchanged

---

## [1.4.0] - 2025-11-27

### Fixed
- **CRITICAL: Extension Permissions Protocol v2.0 Implementation**
  - **Tickets Extension:** v1.2.2 → v1.2.3
    - Added TicketsPermissionsSeeder (135 líneas) - 8 permisos con alias/description
    - Added TicketsUninstallSeeder (75 líneas) - Limpieza completa en desinstalación
    - Fixed: Permisos se crean con alias/description (no NULL)
    - Fixed: Roles se asignan automáticamente (5 roles: super-admin, master-developer, administrator, support, user)
    - Fixed: Permisos se eliminan correctamente en desinstalación
  - **LLM Manager Extension:** v1.0.2 → v1.0.3
    - Added LLMUninstallSeeder (75 líneas)
    - Code sanitation: Removed ~190 líneas hooks muertos (registerExtensionHooks, installPermissions, uninstallPermissions)
  - **CPANEL Core:**
    - Added ExtensionSeederManager::runUninstall() (65 líneas)
    - Updated ExtensionUninstaller to execute uninstall seeders

### Removed
- **Code Sanitation: 380 líneas total**
  - Removed registerExtensionHooks() from LLMServiceProvider and TicketsServiceProvider
  - Removed installPermissions() methods (reemplazados por seeders)
  - Removed uninstallPermissions() methods (reemplazados por UninstallSeeder)
  - **Reason:** ExtensionManager NO implementa hooks estáticos - nunca existieron

### Changed
- **Extension Permissions Protocol v2.0:**
  - Clarification: Seeder-based approach es el ÚNICO mecanismo válido
  - Hooks estáticos (registerInstallHook/registerUninstallHook) NO existen en ExtensionManager
  - Auto-detección por ExtensionSeederManager funciona perfectamente
  - createExtensionPermissions() es solo fallback si no hay seeder

### Documentation
- **README.md** - Updated to v1.4.0
  - Added PERMISSIONS-PROTOCOL-v2.md to Core Concepts (ranked #1)
  - Added llm-manager to Available Extensions list
  - Updated Common Pitfalls with permissions anti-patterns
  - Updated Most Important Documents ranking
  - Added Permissions topic to By Topic section
- **PERMISSIONS-PROTOCOL-v2.md** - Already up to date (updated 2025-11-26)

### Lessons Learned
- **Marketplace Cache:** Requiere clearCache() después de push a GitHub para reflejar versiones actualizadas
- **Installation Flow:** Seeders ejecutan primero, createExtensionPermissions() solo es fallback
- **Version Tracking:** UI puede mostrar versiones incorrectas si cache no se limpia

---

## [1.3.1] - 2025-11-26

### Fixed
- **CRITICAL: Core seeders not executed during local installation via UI**
  - **Problem:** `ExtensionManagerController::installLocal()` had custom installation logic that only ran migrations, skipping seeders completely
  - **Impact:** Extensions installed via "Install from Local Path" UI had missing permissions, configuration, and other core data
  - **Root Cause:** Controller bypassed `ExtensionInstaller` and directly called `ExtensionMigrationManager::run()` without calling `ExtensionSeederManager::runBase()`
  - **Solution:** Added `runBase()` call after migrations in `installLocal()` method
  - **Verified:** llm-manager v1.0.1 - 12/12 permissions installed successfully with correct `extensions:llm-manager:*` prefix
  - **Scope:** Only affected local installations via UI. VCS installations and CLI installs were not affected (they use `ExtensionInstaller::install()`)

### Changed
- **ExtensionManagerController::installLocal()** - Now executes core seeders after migrations
  - Step 5.5: `ExtensionSeederManager::runBase($name)` added
  - Reads `seeders.core` from `extension.json`
  - Executes in subprocess for fresh autoloader context

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
