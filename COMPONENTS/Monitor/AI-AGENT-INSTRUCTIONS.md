# 📡 Monitor Component - AI Agent Instructions

**How to integrate and use Monitor in BITHOVEN applications**

---

## 🎯 When to Use Monitor

### ✅ Use Monitor When:

- **Long-running operations** (>2 seconds) that need user feedback
- **Installation/uninstallation** of extensions or packages
- **Database migrations** or seeders running interactively
- **API batch processing** (multiple requests)
- **File uploads/downloads** with progress tracking
- **System operations** where transparency is important
- **Debugging** complex workflows
- **Multi-step wizards** that need step confirmation

### ❌ Don't Use Monitor When:

- Simple CRUD operations (list, create, update, delete)
- Standard form validations (use Laravel validation)
- Quick AJAX calls (<1 second)
- Operations that should be silent (background jobs)
- Cases where error/success toasts are sufficient

---

## 🚀 Quick Integration Pattern

### Step 1: Add Component to View

```blade
{{-- In your Blade view (modal, page, section) --}}
<x-monitor-panel 
    id="my-operation-monitor"
    title="Operation Progress"
    height="400px"
    :enableRecording="true"
    :enableDownload="true"
    :enableCopy="true"
/>
```

**Naming Convention:**
- Use descriptive IDs: `{feature}-{action}-monitor`
- Examples: `extension-install-monitor`, `migration-run-monitor`, `import-csv-monitor`

### Step 2: Add JavaScript Logging

```javascript
// Clear before starting
Monitor.clear('my-operation-monitor');

// Log operation start
Monitor.info('my-operation-monitor', '═══════════════');
Monitor.info('my-operation-monitor', '🚀 Starting Operation');
Monitor.info('my-operation-monitor', '═══════════════');

// Log progress
Monitor.debug('my-operation-monitor', 'Step 1: Validating data...');
Monitor.success('my-operation-monitor', '✓ Validation passed');

Monitor.debug('my-operation-monitor', 'Step 2: Processing...');
Monitor.warning('my-operation-monitor', '⚠ Found 2 duplicates (skipping)');

// Log completion
Monitor.success('my-operation-monitor', '═══════════════');
Monitor.success('my-operation-monitor', '✓ Operation Complete');
Monitor.success('my-operation-monitor', '═══════════════');
```

### Step 3: Handle Errors

```javascript
try {
    Monitor.info('my-operation-monitor', 'Starting process...');
    
    const response = await fetch('/api/my-endpoint', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(data)
    });
    
    if (!response.ok) {
        throw new Error(`HTTP ${response.status}: ${response.statusText}`);
    }
    
    const result = await response.json();
    Monitor.success('my-operation-monitor', '✓ Process complete');
    
} catch (error) {
    Monitor.error('my-operation-monitor', '✗ Error: ' + error.message);
    Monitor.error('my-operation-monitor', 'Please try again or contact support');
}
```

---

## 📝 Code Templates

### Template 1: Extension Installation

```blade
{{-- View: resources/views/app/extensions/install.blade.php --}}
<div class="modal" id="installModal">
    <div class="modal-dialog modal-lg">
        <div class="modal-content">
            <div class="modal-header">
                <h5>Install Extension</h5>
            </div>
            <div class="modal-body">
                {{-- Installation form --}}
                <form id="installForm">
                    <input type="text" name="name" placeholder="Extension name">
                    <button type="submit">Install</button>
                </form>
                
                {{-- Monitor --}}
                <x-monitor-panel 
                    id="extension-install-monitor"
                    title="Installation Monitor"
                    height="400px"
                    :enableRecording="true"
                    :enableDownload="true"
                />
            </div>
        </div>
    </div>
</div>

@push('scripts')
<script>
$('#installForm').on('submit', function(e) {
    e.preventDefault();
    
    const extensionName = $('input[name="name"]').val();
    
    // Clear monitor
    Monitor.clear('extension-install-monitor');
    Monitor.info('extension-install-monitor', '═══════════════');
    Monitor.info('extension-install-monitor', '📦 Installing Extension');
    Monitor.info('extension-install-monitor', '═══════════════');
    Monitor.debug('extension-install-monitor', `Extension: ${extensionName}`);
    
    $.ajax({
        url: '/app/extensions/install',
        type: 'POST',
        data: { name: extensionName },
        beforeSend: function() {
            Monitor.info('extension-install-monitor', '⏳ Sending request...');
        },
        success: function(response) {
            if (response.success) {
                Monitor.success('extension-install-monitor', '✓ Installation successful');
                Monitor.info('extension-install-monitor', `Version: ${response.version}`);
                
                // Show success toast
                Swal.fire({
                    icon: 'success',
                    title: 'Installed!',
                    text: response.message
                });
            } else {
                Monitor.error('extension-install-monitor', '✗ Installation failed');
                Monitor.error('extension-install-monitor', response.message);
            }
        },
        error: function(xhr) {
            Monitor.error('extension-install-monitor', '✗ Request failed');
            Monitor.error('extension-install-monitor', xhr.responseJSON?.message || 'Unknown error');
        }
    });
});
</script>
@endpush
```

### Template 2: Batch Import

```blade
{{-- View: resources/views/app/imports/csv.blade.php --}}
<x-monitor-panel 
    id="csv-import-monitor"
    title="Import Progress"
    height="500px"
    maxHeight="700px"
    :enableRecording="true"
    :enableCopy="true"
/>

@push('scripts')
<script>
async function importCSV(file) {
    Monitor.clear('csv-import-monitor');
    Monitor.info('csv-import-monitor', '📊 Starting CSV Import');
    Monitor.debug('csv-import-monitor', `File: ${file.name}`);
    Monitor.debug('csv-import-monitor', `Size: ${(file.size / 1024).toFixed(2)} KB`);
    
    const formData = new FormData();
    formData.append('csv_file', file);
    
    try {
        Monitor.info('csv-import-monitor', '⏳ Uploading file...');
        
        const response = await fetch('/api/import/csv', {
            method: 'POST',
            body: formData
        });
        
        const result = await response.json();
        
        if (result.success) {
            Monitor.success('csv-import-monitor', '✓ Upload complete');
            Monitor.info('csv-import-monitor', `Total rows: ${result.total_rows}`);
            Monitor.info('csv-import-monitor', `Imported: ${result.imported}`);
            
            if (result.skipped > 0) {
                Monitor.warning('csv-import-monitor', `⚠ Skipped: ${result.skipped} duplicates`);
            }
            
            if (result.errors.length > 0) {
                Monitor.error('csv-import-monitor', `Errors found:`);
                result.errors.forEach(error => {
                    Monitor.error('csv-import-monitor', `  - Row ${error.row}: ${error.message}`);
                });
            }
            
            Monitor.success('csv-import-monitor', '✓ Import complete');
        } else {
            Monitor.error('csv-import-monitor', '✗ Import failed: ' + result.message);
        }
        
    } catch (error) {
        Monitor.error('csv-import-monitor', '✗ Network error: ' + error.message);
    }
}
</script>
@endpush
```

### Template 3: Multi-Step Wizard

```blade
{{-- View: resources/views/app/wizard.blade.php --}}
<x-monitor-panel 
    id="wizard-monitor"
    title="Setup Progress"
    :enableClear="false"
/>

@push('scripts')
<script>
const steps = [
    { name: 'Database Setup', handler: setupDatabase },
    { name: 'Migration', handler: runMigrations },
    { name: 'Seed Data', handler: seedData },
    { name: 'Configuration', handler: configure }
];

async function runWizard() {
    Monitor.clear('wizard-monitor');
    Monitor.info('wizard-monitor', '🧙‍♂️ Starting Setup Wizard');
    Monitor.info('wizard-monitor', `Total steps: ${steps.length}`);
    
    for (let i = 0; i < steps.length; i++) {
        const step = steps[i];
        const stepNum = i + 1;
        
        Monitor.info('wizard-monitor', '');
        Monitor.info('wizard-monitor', `[${stepNum}/${steps.length}] ${step.name}`);
        Monitor.debug('wizard-monitor', '━━━━━━━━━━━━━━━━━━━━');
        
        try {
            await step.handler();
            Monitor.success('wizard-monitor', `✓ ${step.name} complete`);
        } catch (error) {
            Monitor.error('wizard-monitor', `✗ ${step.name} failed`);
            Monitor.error('wizard-monitor', error.message);
            Monitor.error('wizard-monitor', '');
            Monitor.error('wizard-monitor', '❌ Wizard aborted');
            return;
        }
    }
    
    Monitor.info('wizard-monitor', '');
    Monitor.success('wizard-monitor', '═══════════════════════');
    Monitor.success('wizard-monitor', '✓ Setup Complete!');
    Monitor.success('wizard-monitor', '═══════════════════════');
}
</script>
@endpush
```

---

## 🎨 Best Practices

### 1. Clear Before Starting

**❌ Don't:**
```javascript
// Logs pile up from previous runs
Monitor.info('my-monitor', 'Starting...');
```

**✅ Do:**
```javascript
Monitor.clear('my-monitor');
Monitor.info('my-monitor', 'Starting...');
```

### 2. Use Visual Separators

**❌ Don't:**
```javascript
Monitor.info('my-monitor', 'Starting installation');
Monitor.debug('my-monitor', 'Path: /vendor/...');
```

**✅ Do:**
```javascript
Monitor.info('my-monitor', '═══════════════');
Monitor.info('my-monitor', '📦 Installation');
Monitor.info('my-monitor', '═══════════════');
Monitor.debug('my-monitor', 'Path: /vendor/...');
```

### 3. Log Meaningful Context

**❌ Don't:**
```javascript
Monitor.error('my-monitor', 'Failed');
```

**✅ Do:**
```javascript
Monitor.error('my-monitor', '✗ Database connection failed');
Monitor.error('my-monitor', `Host: ${config.host}:${config.port}`);
Monitor.error('my-monitor', `Error: ${error.message}`);
Monitor.info('my-monitor', 'Check database credentials in .env');
```

### 4. Use Appropriate Log Types

**❌ Don't:**
```javascript
Monitor.info('my-monitor', 'Warning: duplicate found');
Monitor.info('my-monitor', 'Error: connection failed');
```

**✅ Do:**
```javascript
Monitor.warning('my-monitor', '⚠ Duplicate found (skipping)');
Monitor.error('my-monitor', '✗ Connection failed');
```

### 5. Enable Recording for Critical Operations

**❌ Don't:**
```blade
{{-- No recording for installation (data lost on page close) --}}
<x-monitor-panel id="install-monitor" />
```

**✅ Do:**
```blade
{{-- Recording enabled - session saved to server --}}
<x-monitor-panel 
    id="install-monitor"
    :enableRecording="true"
    :recordingMetadata="['extension' => $extension->name]"
/>
```

### 6. Include Timestamps for Benchmarks

```javascript
const startTime = Date.now();

Monitor.info('my-monitor', 'Starting heavy operation...');

// ... operation ...

const duration = ((Date.now() - startTime) / 1000).toFixed(2);
Monitor.success('my-monitor', `✓ Complete in ${duration}s`);
```

---

## 🔧 Integration Patterns

### Pattern 1: Laravel + AJAX

```javascript
// Frontend
Monitor.clear('my-monitor');
Monitor.info('my-monitor', 'Sending request...');

$.ajax({
    url: '/api/process',
    type: 'POST',
    data: { items: [...] },
    success: function(response) {
        if (response.success) {
            Monitor.success('my-monitor', `✓ ${response.message}`);
            
            // Log details from backend
            if (response.logs) {
                response.logs.forEach(log => {
                    Monitor[log.type]('my-monitor', log.message);
                });
            }
        }
    }
});
```

```php
// Backend Controller
public function process(Request $request)
{
    $logs = [];
    
    $logs[] = ['type' => 'info', 'message' => 'Validating data...'];
    
    // ... validation ...
    
    $logs[] = ['type' => 'success', 'message' => '✓ Validation passed'];
    
    foreach ($request->items as $item) {
        $logs[] = ['type' => 'debug', 'message' => "Processing: {$item->name}"];
        // ... process ...
    }
    
    return response()->json([
        'success' => true,
        'message' => 'Processing complete',
        'logs' => $logs
    ]);
}
```

### Pattern 2: Livewire Integration

```php
// Livewire Component
class InstallExtension extends Component
{
    public $monitorId = 'extension-install-monitor';
    
    public function install($extensionName)
    {
        // Emit events to frontend
        $this->dispatch('monitor-log', [
            'id' => $this->monitorId,
            'type' => 'info',
            'message' => '📦 Installing extension'
        ]);
        
        try {
            // ... installation logic ...
            
            $this->dispatch('monitor-log', [
                'id' => $this->monitorId,
                'type' => 'success',
                'message' => '✓ Installation complete'
            ]);
            
        } catch (\Exception $e) {
            $this->dispatch('monitor-log', [
                'id' => $this->monitorId,
                'type' => 'error',
                'message' => '✗ ' . $e->getMessage()
            ]);
        }
    }
    
    public function render()
    {
        return view('livewire.install-extension');
    }
}
```

```blade
{{-- View --}}
<div>
    <x-monitor-panel :id="$monitorId" />
    
    <button wire:click="install('tickets')">Install</button>
</div>

@push('scripts')
<script>
Livewire.on('monitor-log', (data) => {
    Monitor[data.type](data.id, data.message);
});
</script>
@endpush
```

### Pattern 3: Real-time Server Logs (SSE)

```javascript
// Frontend - Server-Sent Events
function processWithSSE(operationId) {
    Monitor.clear('sse-monitor');
    Monitor.info('sse-monitor', 'Connecting to server...');
    
    const eventSource = new EventSource(`/api/process/${operationId}/stream`);
    
    eventSource.addEventListener('log', (event) => {
        const data = JSON.parse(event.data);
        Monitor[data.type]('sse-monitor', data.message);
    });
    
    eventSource.addEventListener('complete', (event) => {
        Monitor.success('sse-monitor', '✓ Process complete');
        eventSource.close();
    });
    
    eventSource.addEventListener('error', (event) => {
        Monitor.error('sse-monitor', '✗ Connection lost');
        eventSource.close();
    });
}
```

```php
// Backend - SSE Controller
public function stream(Request $request, $operationId)
{
    return response()->stream(function () use ($operationId) {
        echo "event: log\n";
        echo 'data: ' . json_encode(['type' => 'info', 'message' => 'Starting...']) . "\n\n";
        ob_flush();
        flush();
        
        // ... process with real-time updates ...
        
        sleep(1);
        echo "event: log\n";
        echo 'data: ' . json_encode(['type' => 'success', 'message' => 'Step 1 complete']) . "\n\n";
        ob_flush();
        flush();
        
        // ...
        
        echo "event: complete\n";
        echo "data: {}\n\n";
        ob_flush();
        flush();
        
    }, 200, [
        'Content-Type' => 'text/event-stream',
        'Cache-Control' => 'no-cache',
        'X-Accel-Buffering' => 'no',
    ]);
}
```

---

## 🐛 Common Integration Issues

### Issue 1: Monitor Not Showing Logs

**Problem:**
```javascript
Monitor.info('wrong-id', 'Test message'); // Wrong ID
```

**Solution:**
```javascript
// Use exact ID from Blade component
Monitor.info('extension-install-monitor', 'Test message');
```

---

### Issue 2: Recording Not Saving

**Problem:**
```blade
{{-- Endpoint missing --}}
<x-monitor-panel 
    id="my-monitor"
    :enableRecording="true"
/>
```

**Solution:**
```blade
{{-- Provide endpoint --}}
<x-monitor-panel 
    id="my-monitor"
    :enableRecording="true"
    recordingEndpoint="{{ route('api.monitor.logs') }}"
/>
```

---

### Issue 3: SweetAlert2 Not Showing

**Problem:** Export/Copy buttons not showing notifications.

**Solution:** Ensure SweetAlert2 is loaded:
```blade
@push('scripts')
<script src="https://cdn.jsdelivr.net/npm/sweetalert2@11"></script>
@endpush
```

---

### Issue 4: Logs Lost on Page Reload

**Problem:** User refreshes page, loses all logs.

**Solution:** Enable recording to save logs server-side:
```blade
<x-monitor-panel 
    id="my-monitor"
    :enableRecording="true"
/>
```

Then retrieve later:
```javascript
// Fetch previous session
const response = await fetch('/api/monitor/sessions/my-monitor');
const data = await response.json();
console.log(data.sessions); // List of all sessions
```

---

## 📊 Integration Checklist

When integrating Monitor into a new feature:

- [ ] Component added to view with unique `id`
- [ ] Recording enabled if operation is critical
- [ ] Recording endpoint configured (if recording enabled)
- [ ] Recording metadata includes context (user_id, feature, etc.)
- [ ] `Monitor.clear()` called before operation starts
- [ ] Visual separators used for readability
- [ ] Appropriate log types used (success, error, warning, info, debug)
- [ ] Error handling includes Monitor.error() logs
- [ ] Success messages confirm completion
- [ ] Download/Copy buttons enabled if logs might be large
- [ ] Tested in modal (if applicable) - monitor visible
- [ ] Tested error scenarios - errors logged properly
- [ ] SweetAlert2 loaded (if using export features)

---

## 🚀 Quick Reference

### Minimal Setup
```blade
<x-monitor-panel id="my-monitor" />
```

### Full Setup
```blade
<x-monitor-panel 
    id="my-monitor"
    title="Custom Title"
    :enableRecording="true"
    :enableDownload="true"
    :enableCopy="true"
/>
```

### Log Patterns
```javascript
// Start
Monitor.clear('my-monitor');
Monitor.info('my-monitor', '═══════════════');
Monitor.info('my-monitor', '🚀 Operation');
Monitor.info('my-monitor', '═══════════════');

// Progress
Monitor.debug('my-monitor', 'Step 1...');
Monitor.success('my-monitor', '✓ Step 1 done');

// Warnings
Monitor.warning('my-monitor', '⚠ Duplicate found');

// Errors
Monitor.error('my-monitor', '✗ Failed');
Monitor.error('my-monitor', error.message);

// Complete
Monitor.success('my-monitor', '✓ Complete');
```

---

**Questions?** Review [README.md](./README.md) or [API-REFERENCE.md](./API-REFERENCE.md)
