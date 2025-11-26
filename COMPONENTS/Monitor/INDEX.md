# 📡 Monitor Component Documentation

**Complete documentation for BITHOVEN Monitor Protocol v3.0**

---

## 📚 Documentation Index

### Quick Start

- **[README.md](./README.md)** - Main documentation, quick start, component reference
- **[AI-AGENT-INSTRUCTIONS.md](./AI-AGENT-INSTRUCTIONS.md)** - Integration guide for AI assistants

### Complete Reference

- **[API-REFERENCE.md](./API-REFERENCE.md)** - Complete API specifications (JavaScript, Blade, Backend)
- **[EXAMPLES.md](./EXAMPLES.md)** - Real-world integration examples and patterns
- **[CONFIGURATION.md](./CONFIGURATION.md)** - Configuration files, environment variables, deployment

---

## 🎯 What is Monitor Component?

Monitor is a **core BITHOVEN component** that provides:

- ✅ **Real-time logging** - Client-side display with color-coded log types
- ✅ **Server recording** - Persistent storage with session management
- ✅ **Export capabilities** - Download/copy logs in TXT, JSON, CSV
- ✅ **Reusable component** - Single Blade component, global JavaScript API
- ✅ **Zero-config** - Works out-of-the-box with sensible defaults

**Use Cases:**
- Extension installation/uninstallation tracking
- Long-running operations feedback
- Debug logging and error reporting
- User transparency during migrations
- Real-time process monitoring

---

## 🚀 Quick Links

### For Developers

- [Component Props](./README.md#-component-properties) - All 11 props with examples
- [JavaScript API](./README.md#-javascript-api) - 15 methods across 3 objects
- [Integration Pattern](./AI-AGENT-INSTRUCTIONS.md#-quick-integration-pattern) - 3-step setup guide
- [Common Patterns](./EXAMPLES.md#common-patterns) - Copy-paste examples

### For System Admins

- [Configuration](./CONFIGURATION.md#-configuration-files) - config/monitor.php setup
- [Scheduled Tasks](./CONFIGURATION.md#-scheduled-tasks) - Cleanup automation
- [Security](./CONFIGURATION.md#-security-best-practices) - Authentication, rate limiting
- [Performance](./CONFIGURATION.md#-performance-optimization) - Batch config, storage

### For AI Agents

- [When to Use](./AI-AGENT-INSTRUCTIONS.md#-when-to-use-monitor) - Decision guide
- [Integration Checklist](./AI-AGENT-INSTRUCTIONS.md#-integration-checklist) - Complete checklist
- [Code Templates](./AI-AGENT-INSTRUCTIONS.md#-code-templates) - Ready-to-use templates
- [Common Issues](./AI-AGENT-INSTRUCTIONS.md#-common-integration-issues) - Troubleshooting

---

## 📦 Files Location

### Frontend

```
CPANEL/resources/
├── js/custom/monitor/
│   └── monitor.js              # JavaScript Protocol (3 objects, 15 methods)
└── views/components/
    └── monitor-panel.blade.php # Blade Component (11 props)
```

### Backend

```
CPANEL/app/
├── Contracts/
│   └── MonitorServiceInterface.php   # Service interface
├── Services/
│   └── MonitorService.php            # Service implementation
├── Http/Controllers/Api/
│   └── MonitorController.php         # API controller (5 endpoints)
└── Console/Commands/
    └── CleanMonitorSessions.php      # Cleanup command
```

### Configuration

```
CPANEL/
├── config/
│   └── monitor.php                   # Main configuration
└── routes/
    └── api.php                       # API routes (/api/monitor/*)
```

### Storage

```
CPANEL/storage/app/
├── monitors/
│   └── {monitor-id}/
│       └── session-{timestamp}.json  # Session data
└── exports/
    └── session-{timestamp}.{format}  # Exported files
```

---

## 🎨 Architecture Overview

```
┌─────────────────────────────────────────────────┐
│              APPLICATION                        │
│  ┌──────────────────────────────────────────┐  │
│  │  <x-monitor-panel                        │  │
│  │      id="my-monitor"                     │  │
│  │      :enableRecording="true"             │  │
│  │  />                                       │  │
│  └──────────────────────────────────────────┘  │
│                     ▼                           │
└─────────────────────────────────────────────────┘
                      │
                      │ JavaScript API
                      ▼
┌─────────────────────────────────────────────────┐
│           FRONTEND (monitor.js)                 │
│  ┌──────────────────────────────────────────┐  │
│  │  Monitor.log()                           │  │
│  │  MonitorRecording.toggle()               │  │
│  │  MonitorExport.download()                │  │
│  └──────────────────────────────────────────┘  │
│                     ▼                           │
│               (2-second batches)                │
│                     ▼                           │
└─────────────────────────────────────────────────┘
                      │
                      │ AJAX POST
                      ▼
┌─────────────────────────────────────────────────┐
│           BACKEND (Laravel API)                 │
│  ┌──────────────────────────────────────────┐  │
│  │  MonitorController                       │  │
│  │    → MonitorService                      │  │
│  │       → JSON Storage                     │  │
│  └──────────────────────────────────────────┘  │
└─────────────────────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────┐
│         STORAGE (storage/app/monitors/)         │
│  {monitor-id}/session-{timestamp}.json          │
└─────────────────────────────────────────────────┘
```

---

## 📖 Reading Guide

### If you're new to Monitor

1. Start with **[README.md](./README.md)** - Overview and Quick Start
2. Try the **[Simple Example](./README.md#1-basic-usage-client-side-only)**
3. Read **[Component Properties](./README.md#-component-properties)**
4. Check **[Common Patterns](./EXAMPLES.md#common-patterns)**

### If you're integrating Monitor

1. Read **[AI-AGENT-INSTRUCTIONS.md](./AI-AGENT-INSTRUCTIONS.md)** - Full integration guide
2. Follow **[Quick Integration Pattern](./AI-AGENT-INSTRUCTIONS.md#-quick-integration-pattern)**
3. Use **[Code Templates](./AI-AGENT-INSTRUCTIONS.md#-code-templates)**
4. Review **[Integration Checklist](./AI-AGENT-INSTRUCTIONS.md#-integration-checklist)**

### If you're configuring Monitor

1. Review **[CONFIGURATION.md](./CONFIGURATION.md)** - Complete config reference
2. Setup **[config/monitor.php](./CONFIGURATION.md#configmonitorphp)**
3. Configure **[Scheduled Tasks](./CONFIGURATION.md#-scheduled-tasks)**
4. Apply **[Security Best Practices](./CONFIGURATION.md#-security-best-practices)**

### If you need API details

1. Check **[API-REFERENCE.md](./API-REFERENCE.md)** - Complete specifications
2. Review **[JavaScript API](./API-REFERENCE.md#javascript-api)** - All methods
3. Check **[Backend API](./API-REFERENCE.md#backend-api)** - All endpoints
4. See **[Data Structures](./API-REFERENCE.md#data-structures)** - JSON formats

---

## 🔗 Related Systems

- **Extension Manager** - Uses Monitor for installation tracking
- **Migration System** - Uses Monitor for migration progress
- **LLM Manager** - Uses simplified Monitor for test feedback
- **Import/Export** - Uses Monitor for batch processing

---

## 📝 Version History

### v3.0.0 (18 de noviembre de 2025)
- ✅ Complete protocol implementation
- ✅ Backend service with interface
- ✅ API endpoints for recording and export
- ✅ Session management with metadata
- ✅ Auto-cleanup of old sessions
- ✅ Export to TXT, JSON, CSV
- ✅ Recording with batch processing
- ✅ Complete documentation (4 files, centralized)

### v2.0.0 (Historical)
- Inline Monitor in extensions
- No backend
- No recording

### v1.0.0 (Initial)
- Basic implementation
- Frontend only

---

## 📍 Documentation Location

**⚠️ IMPORTANT:** All Monitor documentation is now centralized in:

```
/DOCS/COMPONENTS/Monitor/
├── INDEX.md                      # This file
├── README.md                     # Main documentation
├── API-REFERENCE.md              # Complete API specs
├── EXAMPLES.md                   # Real-world examples
├── CONFIGURATION.md              # Config & deployment
└── AI-AGENT-INSTRUCTIONS.md      # Integration guide
```

**Do NOT create Monitor documentation in:**
- ❌ `/CPANEL/docs/core/` (old location, removed)
- ❌ `/CPANEL/.github/` (reserved for Copilot instructions)
- ❌ Extension READMEs (link to central docs instead)

**When referencing Monitor from other docs:**

```markdown
See [Monitor Component Documentation](/DOCS/COMPONENTS/Monitor/)
```

---

## 🆘 Support

### Quick Help

- **"How do I add Monitor to my page?"** → [README Quick Start](./README.md#-quick-start)
- **"What props can I use?"** → [Component Properties](./README.md#-component-properties)
- **"How do I log messages?"** → [JavaScript API](./README.md#-javascript-api)
- **"Monitor not showing logs?"** → [Troubleshooting](./AI-AGENT-INSTRUCTIONS.md#-common-integration-issues)

### Full Documentation

For complete information, read all 5 documentation files in order:
1. INDEX.md (this file)
2. README.md
3. AI-AGENT-INSTRUCTIONS.md
4. API-REFERENCE.md
5. EXAMPLES.md
6. CONFIGURATION.md

---

**Version:** 3.0.0  
**Last Updated:** 18 de noviembre de 2025  
**Status:** ✅ Production Ready  
**Maintainer:** Bithoven Development Team
