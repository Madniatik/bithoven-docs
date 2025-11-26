# 📡 Monitor Component v3.0

**Real-time logging and recording system for BITHOVEN applications**

---

## 🎯 Overview

Monitor is a comprehensive logging component that provides:
- ✅ Real-time display of logs with color-coded types
- ✅ Client-side logging (no server required)
- ✅ Optional server-side recording (persistent storage)
- ✅ Export capabilities (download/copy)
- ✅ Reusable Blade component
- ✅ Global JavaScript API

**Use Cases:**
- Extension installation progress
- Long-running operations tracking
- Debug logging
- API request/response monitoring
- System events display

---

## 🚀 Quick Start

### 1. Basic Usage (Client-side only)

```blade
{{-- Minimal setup - no recording --}}
<x-monitor-panel id="my-monitor" />

<script>
    // Log messages
    Monitor.info('my-monitor', 'Starting process...');
    Monitor.success('my-monitor', 'Process completed!');
    Monitor.error('my-monitor', 'Something went wrong');
</script>
```

### 2. Full Featured Setup

```blade
{{-- With all features enabled --}}
<x-monitor-panel 
    id="installation-monitor"
    title="Installation Progress"
    height="400px"
    :enableRecording="true"
    :enableDownload="true"
    :enableCopy="true"
    :enableClear="true"
    recordingEndpoint="{{ route('api.monitor.logs', ['id' => 'installation-monitor']) }}"
    :recordingMetadata="['extension' => 'tickets', 'version' => '1.2.1']"
/>

<script>
    Monitor.clear('installation-monitor');
    Monitor.info('installation-monitor', '📦 Starting installation');
    Monitor.debug('installation-monitor', 'Path: /vendor/bithoven/tickets');
    Monitor.success('installation-monitor', '✓ Installation complete');
</script>
```

---

## 📦 Component Properties

### Required

| Property | Type | Description |
|----------|------|-------------|
| `id` | `string` | Unique monitor identifier (required) |

### Optional - Display

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `title` | `string` | `"📡 Monitor"` | Card header title |
| `height` | `string` | `"300px"` | Minimum height |
| `maxHeight` | `string` | `"600px"` | Maximum height (scroll) |
| `class` | `string` | `""` | Additional CSS classes |

### Optional - Features

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `enableRecording` | `boolean` | `false` | Enable server recording |
| `enableDownload` | `boolean` | `true` | Show download button |
| `enableCopy` | `boolean` | `true` | Show copy button |
| `enableClear` | `boolean` | `true` | Show clear button |

### Optional - Recording

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `recordingEndpoint` | `string` | `route('api.monitor.logs')` | Server endpoint |
| `recordingMetadata` | `array` | `[]` | Custom metadata |

---

## 🔧 JavaScript API

### Core Logging

```javascript
// Basic logging
Monitor.log(monitorId, message, type, timestamp);

// Convenience methods
Monitor.success(monitorId, message);
Monitor.error(monitorId, message);
Monitor.warning(monitorId, message);
Monitor.info(monitorId, message);
Monitor.debug(monitorId, message);

// Clear logs
Monitor.clear(monitorId);

// Get all logs
const logs = Monitor.getLogs(monitorId);
```

### Recording Control

```javascript
// Toggle recording
MonitorRecording.toggle(monitorId, enabled, endpoint, metadata);

// Manual start/stop
MonitorRecording.start(monitorId, endpoint, metadata);
MonitorRecording.stop(monitorId);

// Check status
const isRecording = MonitorRecording.isEnabled(monitorId);
```

### Export Functions

```javascript
// Copy to clipboard
await MonitorExport.copy(monitorId);

// Download as file
MonitorExport.download(monitorId);
```

---

## 🎨 Log Types & Styling

| Type | Icon | Color | Use Case |
|------|------|-------|----------|
| `success` | ✓ | Green | Successful operations |
| `error` | ✗ | Red | Errors and failures |
| `warning` | ⚠ | Yellow | Warnings |
| `info` | ℹ | Blue | General information |
| `debug` | ▸ | Gray | Debug details |

---

## 💾 Recording System

### How It Works

1. **Client Batching:** Logs collected in 2-second batches
2. **AJAX Transmission:** Sent to server endpoint
3. **JSON Storage:** Saved to `storage/app/monitors/{monitor-id}/{session-id}.json`
4. **Session Tracking:** Each recording session has unique ID

### Session Structure

```json
{
  "session_id": "session-1700000000000",
  "monitor_id": "installation-monitor",
  "metadata": {
    "extension": "tickets",
    "version": "1.2.1"
  },
  "started_at": "2025-11-18T22:00:00+00:00",
  "updated_at": "2025-11-18T22:05:00+00:00",
  "total_logs": 42,
  "logs": [
    {
      "timestamp": "2025-11-18T22:00:01+00:00",
      "type": "info",
      "message": "Starting installation"
    }
  ]
}
```

### Storage Location

```
storage/
└── app/
    └── monitors/
        ├── installation-monitor/
        │   ├── session-1700000000000.json
        │   └── session-1700000001000.json
        └── llm-generation-monitor/
            └── session-1700000002000.json
```

---

## 🔌 Backend Setup

### 1. Controller (Already Included)

`app/Http/Controllers/Api/MonitorController.php`

### 2. Service (Already Included)

`app/Services/MonitorService.php`

### 3. Routes (Already Configured)

```php
// routes/api.php
Route::prefix('monitor')->name('api.monitor.')->group(function () {
    Route::post('/logs', [MonitorController::class, 'saveLogs'])
        ->name('logs');
    Route::get('/session/{sessionId}', [MonitorController::class, 'getSession'])
        ->name('session');
    Route::get('/sessions/{monitorId}', [MonitorController::class, 'listSessions'])
        ->name('sessions');
    Route::get('/export/{sessionId}', [MonitorController::class, 'exportSession'])
        ->name('export');
    Route::get('/download/{sessionId}', [MonitorController::class, 'downloadSession'])
        ->name('download');
});
```

### 4. Interface (Optional)

```php
// app/Contracts/MonitorServiceInterface.php
interface MonitorServiceInterface
{
    public function saveLogs(string $sessionId, string $monitorId, array $metadata, array $logs): bool;
    public function getSession(string $sessionId): ?array;
    public function getSessions(string $monitorId, int $limit = 50): array;
    public function cleanOldSessions(int $days = 30): int;
    public function exportSession(string $sessionId, string $format = 'txt'): ?string;
}
```

---

## 📝 Real-World Examples

### Example 1: Extension Installation

```blade
{{-- resources/views/app/extension-manager/partials/modals/install-local.blade.php --}}
<x-monitor-panel 
    id="extension-install-monitor"
    title="Installation Monitor"
    height="400px"
    :enableRecording="true"
    :enableDownload="true"
    :enableCopy="true"
    :enableClear="true"
    recordingEndpoint="{{ route('api.monitor.logs', ['id' => 'extension-install-monitor']) }}"
/>

<script>
function installExtension(slug, path) {
    Monitor.clear('extension-install-monitor');
    Monitor.info('extension-install-monitor', '═══════════════');
    Monitor.info('extension-install-monitor', '📦 Starting Installation');
    Monitor.info('extension-install-monitor', '═══════════════');
    Monitor.debug('extension-install-monitor', `Extension: ${slug}`);
    Monitor.debug('extension-install-monitor', `Path: ${path}`);
    
    $.ajax({
        url: '/app/extensions/install-local',
        type: 'POST',
        data: { name: slug, path: path },
        success: function(response) {
            if (response.success) {
                Monitor.success('extension-install-monitor', '✓ Installation complete');
            } else {
                Monitor.error('extension-install-monitor', '✗ ' + response.message);
            }
        },
        error: function(xhr) {
            Monitor.error('extension-install-monitor', '✗ Request failed');
            Monitor.error('extension-install-monitor', xhr.responseJSON?.message || 'Unknown error');
        }
    });
}
</script>
```

### Example 2: LLM Generation (from bithoven-extension-llm-manager)

```blade
<x-monitor-panel 
    id="llm-generation-monitor"
    title="🤖 Generation Monitor"
    height="250px"
/>

<script>
async function generateWithLLM(prompt) {
    Monitor.clear('llm-generation-monitor');
    Monitor.info('llm-generation-monitor', `🚀 Sending prompt: "${prompt}"`);
    
    try {
        const response = await fetch('/api/llm/generate', {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify({ prompt })
        });
        
        const data = await response.json();
        
        if (data.success) {
            Monitor.success('llm-generation-monitor', '✓ Generation complete');
            Monitor.debug('llm-generation-monitor', `Tokens: ${data.tokens_used}`);
        } else {
            Monitor.error('llm-generation-monitor', '✗ ' + data.error);
        }
    } catch (error) {
        Monitor.error('llm-generation-monitor', '✗ Network error: ' + error.message);
    }
}
</script>
```

### Example 3: Simple Progress Tracking

```blade
<x-monitor-panel 
    id="progress-monitor"
    title="Progress Tracker"
    :enableClear="false"
/>

<script>
function processItems(items) {
    Monitor.clear('progress-monitor');
    Monitor.info('progress-monitor', `Processing ${items.length} items...`);
    
    items.forEach((item, index) => {
        setTimeout(() => {
            Monitor.debug('progress-monitor', `[${index + 1}/${items.length}] Processing ${item}`);
            
            if (index === items.length - 1) {
                Monitor.success('progress-monitor', '✓ All items processed');
            }
        }, index * 500);
    });
}
</script>
```

---

## 🛠️ Advanced Configuration

### Custom Metadata

```javascript
MonitorRecording.start('my-monitor', '/api/monitor/logs', {
    user_id: 123,
    extension: 'tickets',
    version: '1.2.1',
    environment: 'production'
});
```

### Custom Batch Interval

Edit `resources/js/custom/monitor/monitor.js`:

```javascript
// Change from default 2000ms to 5000ms
this.intervals[monitorId] = setInterval(() => {
    this.flush(monitorId);
}, 5000); // 5 seconds
```

### Storage Cleanup

```php
// app/Console/Kernel.php
protected function schedule(Schedule $schedule)
{
    // Clean monitor sessions older than 30 days
    $schedule->call(function () {
        app(MonitorService::class)->cleanOldSessions(30);
    })->daily();
}
```

---

## 🚨 Common Pitfalls

### ❌ Missing Monitor ID

```blade
{{-- ERROR: No ID specified --}}
<x-monitor-panel />

{{-- CORRECT --}}
<x-monitor-panel id="my-monitor" />
```

### ❌ Recording Without Endpoint

```blade
{{-- ERROR: Recording enabled but no endpoint --}}
<x-monitor-panel 
    id="my-monitor"
    :enableRecording="true"
/>

{{-- CORRECT --}}
<x-monitor-panel 
    id="my-monitor"
    :enableRecording="true"
    recordingEndpoint="{{ route('api.monitor.logs') }}"
/>
```

### ❌ Multiple Components Same ID

```blade
{{-- ERROR: Duplicate IDs --}}
<x-monitor-panel id="monitor" />
<x-monitor-panel id="monitor" /> {{-- Conflicts! --}}

{{-- CORRECT --}}
<x-monitor-panel id="install-monitor" />
<x-monitor-panel id="update-monitor" />
```

---

## 🔍 Troubleshooting

### Logs Not Appearing

1. **Check monitor exists:** `document.getElementById('my-monitor')`
2. **Console errors:** Open DevTools Console
3. **Script loaded:** Verify `monitor.js` in Network tab

### Recording Not Working

1. **Check endpoint:** Verify route exists: `php artisan route:list | grep monitor`
2. **Check network:** DevTools → Network → Look for POST to `/api/monitor/logs`
3. **Check logs:** `tail -f storage/logs/laravel.log`
4. **Check storage:** `ls -la storage/app/monitors/`

### Export Not Working

1. **Check SweetAlert2:** Verify library is loaded
2. **Browser clipboard:** Check clipboard permissions
3. **Console errors:** Look for JavaScript errors

---

## 📚 API Reference

See dedicated [API Reference](./API-REFERENCE.md) for complete method signatures and parameters.

---

## 🔗 Related Documentation

- **[Examples](./EXAMPLES.md)** - More real-world examples
- **[API Reference](./API-REFERENCE.md)** - Complete API documentation
- **[Migration Guide](./MIGRATION.md)** - Upgrading from older versions

---

## 📄 Version History

- **v3.0.0** (2025-11-18) - Initial stable release
  - Server-side recording
  - Export capabilities
  - Blade component
  - Global JS API

---

**Questions?** Check [EXAMPLES.md](./EXAMPLES.md) or review implementation in:
- Extension Manager: `resources/views/app/extension-manager/partials/modals/install-local.blade.php`
- LLM Manager: `vendor/bithoven/llm-manager/resources/views/components/monitor-panel.blade.php`
