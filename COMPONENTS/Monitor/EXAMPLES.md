# 📡 Monitor Component - Examples & Use Cases

**Real-world integration examples for BITHOVEN Monitor Protocol v3.0**

---

## 📋 Table of Contents

- [Extension Integration](#extension-integration)
- [Common Patterns](#common-patterns)
- [AJAX Integration](#ajax-integration)
- [Livewire Integration](#livewire-integration)
- [Real-time SSE](#real-time-sse)
- [Multi-step Wizard](#multi-step-wizard)
- [Batch Processing](#batch-processing)
- [Error Handling](#error-handling)

---

## Extension Integration

### Complete Extension Setup

```blade
{{-- resources/views/admin/models/show.blade.php --}}

<x-default-layout>
    @section('title', 'Model Details')
    
    <div class="row">
        <div class="col-md-8">
            {{-- Main content --}}
            <div class="card">
                <div class="card-header">
                    <h3>Model: {{ $model->name }}</h3>
                </div>
                <div class="card-body">
                    <button type="button" class="btn btn-primary" onclick="testModel()">
                        Test Connection
                    </button>
                </div>
            </div>
        </div>
        
        <div class="col-md-4">
            {{-- Monitor Panel --}}
            <x-monitor-panel 
                id="llm-model-{{ $model->id }}"
                title="📡 Test Activity"
                height="400px"
                :enableRecording="true"
                :enableDownload="true"
                :enableCopy="true"
                :recordingMetadata="[
                    'extension' => 'llm-manager',
                    'model_id' => $model->id,
                    'user_id' => auth()->id(),
                    'context' => 'connection-test'
                ]"
            />
        </div>
    </div>
    
    @push('scripts')
    <script>
        function testModel() {
            const monitorId = 'llm-model-{{ $model->id }}';
            
            // Clear previous logs
            Monitor.clear(monitorId);
            
            // Start test
            Monitor.info(monitorId, '━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━');
            Monitor.info(monitorId, '🧪 Iniciando Test de Conexión');
            Monitor.info(monitorId, '━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━');
            Monitor.debug(monitorId, 'Model ID: {{ $model->id }}');
            Monitor.debug(monitorId, 'Endpoint: {{ $model->endpoint }}');
            
            fetch('/api/llm/test', {
                method: 'POST',
                headers: { 
                    'Content-Type': 'application/json',
                    'X-CSRF-TOKEN': '{{ csrf_token() }}'
                },
                body: JSON.stringify({ 
                    model_id: {{ $model->id }} 
                })
            })
            .then(response => {
                if (!response.ok) {
                    throw new Error(`HTTP ${response.status}: ${response.statusText}`);
                }
                return response.json();
            })
            .then(data => {
                if (data.success) {
                    Monitor.info(monitorId, '');
                    Monitor.info(monitorId, '📊 METADATA:');
                    Monitor.debug(monitorId, `   URL: ${data.metadata.url}`);
                    Monitor.debug(monitorId, `   Method: ${data.metadata.method}`);
                    Monitor.debug(monitorId, `   HTTP Code: ${data.metadata.http_code}`);
                    Monitor.debug(monitorId, `   Time: ${data.metadata.total_time_ms}ms`);
                    Monitor.info(monitorId, '');
                    Monitor.success(monitorId, '✓ Test exitoso');
                    Monitor.info(monitorId, '━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━');
                    
                    // Show success toast
                    Swal.fire({
                        icon: 'success',
                        title: 'Connection OK',
                        text: `Response time: ${data.metadata.total_time_ms}ms`,
                        timer: 3000
                    });
                } else {
                    Monitor.error(monitorId, `✗ ${data.message}`);
                    Monitor.info(monitorId, '━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━');
                }
            })
            .catch(error => {
                Monitor.error(monitorId, '✗ Error de conexión');
                Monitor.error(monitorId, error.message);
                Monitor.info(monitorId, '━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━');
            });
        }
    </script>
    @endpush
</x-default-layout>
```

---

## Common Patterns

### Pattern 1: Simple Monitor

Minimal setup for quick feedback.

```blade
<x-monitor-panel 
    id="simple-monitor"
    title="Simple Monitor"
/>

@push('scripts')
<script>
    Monitor.clear('simple-monitor');
    Monitor.info('simple-monitor', 'Hello World');
    Monitor.success('simple-monitor', '✓ Operation complete');
</script>
@endpush
```

---

### Pattern 2: Monitor with Recording

Persistent logging to server.

```blade
<x-monitor-panel 
    id="recording-monitor"
    title="Recording Monitor"
    :enableRecording="true"
    recordingEndpoint="{{ route('api.monitor.logs') }}"
    :recordingMetadata="[
        'extension' => 'my-extension',
        'action' => 'install',
        'user_id' => auth()->id()
    ]"
/>

@push('scripts')
<script>
    // Logs will be automatically saved to server when recording is enabled
    Monitor.info('recording-monitor', 'This log is saved to server');
    Monitor.success('recording-monitor', '✓ Persistent record');
</script>
@endpush
```

---

### Pattern 3: Installation/Uninstallation Tracking

```blade
<x-monitor-panel 
    id="extension-install-monitor"
    title="Installation Progress"
    height="500px"
    :enableRecording="true"
    :enableDownload="true"
    :recordingMetadata="[
        'extension' => $extension->name,
        'version' => $extension->version,
        'action' => 'install'
    ]"
/>

@push('scripts')
<script>
function installExtension(extensionSlug, localPath) {
    const mid = 'extension-install-monitor';
    
    Monitor.clear(mid);
    Monitor.info(mid, '═══════════════════════════════════');
    Monitor.info(mid, '📦 Extension Installation');
    Monitor.info(mid, '═══════════════════════════════════');
    Monitor.debug(mid, `Extension: ${extensionSlug}`);
    Monitor.debug(mid, `Path: ${localPath}`);
    Monitor.info(mid, '');
    
    $.ajax({
        url: '/app/extensions/install-local',
        type: 'POST',
        data: {
            name: extensionSlug,
            path: localPath,
            _token: '{{ csrf_token() }}'
        },
        beforeSend: function() {
            Monitor.info(mid, '⏳ Sending installation request...');
        },
        success: function(response) {
            Monitor.info(mid, '');
            
            if (response.success) {
                Monitor.success(mid, '✓ Installation successful');
                Monitor.info(mid, `Version: ${response.version}`);
                Monitor.info(mid, `Files: ${response.files_count}`);
                Monitor.info(mid, '');
                Monitor.info(mid, '═══════════════════════════════════');
                
                // Reload page after 2 seconds
                setTimeout(() => {
                    window.location.reload();
                }, 2000);
            } else {
                Monitor.error(mid, '✗ Installation failed');
                Monitor.error(mid, response.message);
                
                if (response.errors) {
                    Monitor.error(mid, '');
                    Monitor.error(mid, 'Errors:');
                    response.errors.forEach(error => {
                        Monitor.error(mid, `  - ${error}`);
                    });
                }
                
                Monitor.info(mid, '═══════════════════════════════════');
            }
        },
        error: function(xhr) {
            Monitor.error(mid, '✗ Request failed');
            Monitor.error(mid, xhr.responseJSON?.message || 'Unknown error');
            Monitor.debug(mid, `HTTP ${xhr.status}: ${xhr.statusText}`);
            Monitor.info(mid, '═══════════════════════════════════');
        }
    });
}
</script>
@endpush
```

---

## AJAX Integration

### Laravel Backend with Frontend Logging

```javascript
// Frontend
function performAction() {
    const mid = 'ajax-monitor';
    
    Monitor.clear(mid);
    Monitor.info(mid, 'Starting action...');
    
    fetch('/api/action', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ data: 'value' })
    })
    .then(response => response.json())
    .then(data => {
        // Backend can send logs to display
        if (data.logs) {
            data.logs.forEach(log => {
                Monitor[log.type](mid, log.message);
            });
        }
        
        Monitor.success(mid, `✓ Action completed: ${data.result}`);
    })
    .catch(error => {
        Monitor.error(mid, `✗ Failed: ${error.message}`);
    });
}
```

```php
// Backend Controller
public function action(Request $request)
{
    $logs = [];
    
    $logs[] = ['type' => 'info', 'message' => 'Validating data...'];
    
    // Validation
    $validated = $request->validate([...]);
    $logs[] = ['type' => 'success', 'message' => '✓ Validation passed'];
    
    // Processing
    $logs[] = ['type' => 'debug', 'message' => 'Processing items...'];
    foreach ($items as $item) {
        // Process
        $logs[] = ['type' => 'debug', 'message' => "  - Item {$item->id} processed"];
    }
    
    $logs[] = ['type' => 'success', 'message' => '✓ All items processed'];
    
    return response()->json([
        'success' => true,
        'result' => 'Completed',
        'logs' => $logs
    ]);
}
```

---

## Livewire Integration

### Component with Real-time Updates

```php
// Livewire Component: app/Livewire/InstallExtension.php
namespace App\Livewire;

use Livewire\Component;

class InstallExtension extends Component
{
    public $monitorId = 'extension-install-monitor';
    public $extensionName;
    
    public function install()
    {
        // Emit log events
        $this->dispatch('monitor-log', [
            'id' => $this->monitorId,
            'type' => 'info',
            'message' => '📦 Starting installation'
        ]);
        
        $this->dispatch('monitor-log', [
            'id' => $this->monitorId,
            'type' => 'debug',
            'message' => "Extension: {$this->extensionName}"
        ]);
        
        try {
            // Installation logic
            app(ExtensionManager::class)->install($this->extensionName);
            
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
{{-- View: resources/views/livewire/install-extension.blade.php --}}
<div>
    <div class="mb-5">
        <input type="text" wire:model="extensionName" placeholder="Extension name">
        <button wire:click="install" class="btn btn-primary">Install</button>
    </div>
    
    <x-monitor-panel :id="$monitorId" />
</div>

@push('scripts')
<script>
Livewire.on('monitor-log', (data) => {
    Monitor[data.type](data.id, data.message);
});
</script>
@endpush
```

---

## Real-time SSE

### Server-Sent Events Integration

```javascript
// Frontend - Server-Sent Events
function processWithSSE(operationId) {
    const mid = 'sse-monitor';
    
    Monitor.clear(mid);
    Monitor.info(mid, 'Connecting to server...');
    
    const eventSource = new EventSource(`/api/process/${operationId}/stream`);
    
    eventSource.addEventListener('log', (event) => {
        const data = JSON.parse(event.data);
        Monitor[data.type](mid, data.message);
    });
    
    eventSource.addEventListener('complete', (event) => {
        Monitor.success(mid, '✓ Process complete');
        eventSource.close();
    });
    
    eventSource.addEventListener('error', (event) => {
        Monitor.error(mid, '✗ Connection lost');
        eventSource.close();
    });
}
```

```php
// Backend - SSE Controller
public function stream(Request $request, $operationId)
{
    return response()->stream(function () use ($operationId) {
        // Send initial log
        echo "event: log\n";
        echo 'data: ' . json_encode([
            'type' => 'info', 
            'message' => 'Starting process...'
        ]) . "\n\n";
        ob_flush();
        flush();
        
        // Process with real-time updates
        sleep(1);
        echo "event: log\n";
        echo 'data: ' . json_encode([
            'type' => 'success', 
            'message' => '✓ Step 1 complete'
        ]) . "\n\n";
        ob_flush();
        flush();
        
        sleep(1);
        echo "event: log\n";
        echo 'data: ' . json_encode([
            'type' => 'success', 
            'message' => '✓ Step 2 complete'
        ]) . "\n\n";
        ob_flush();
        flush();
        
        // Complete
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

## Multi-step Wizard

### Sequential Operations Tracking

```blade
<x-monitor-panel 
    id="wizard-monitor"
    title="Setup Progress"
    :enableClear="false"
    :enableRecording="true"
/>

@push('scripts')
<script>
const setupSteps = [
    { name: 'Database Setup', handler: setupDatabase },
    { name: 'Run Migrations', handler: runMigrations },
    { name: 'Seed Data', handler: seedData },
    { name: 'Configure Services', handler: configureServices }
];

async function runSetupWizard() {
    const mid = 'wizard-monitor';
    
    Monitor.clear(mid);
    Monitor.info(mid, '🧙‍♂️ Starting Setup Wizard');
    Monitor.info(mid, `Total steps: ${setupSteps.length}`);
    Monitor.info(mid, '');
    
    for (let i = 0; i < setupSteps.length; i++) {
        const step = setupSteps[i];
        const stepNum = i + 1;
        
        Monitor.info(mid, '━━━━━━━━━━━━━━━━━━━━━━━━━━━━');
        Monitor.info(mid, `[${stepNum}/${setupSteps.length}] ${step.name}`);
        Monitor.info(mid, '━━━━━━━━━━━━━━━━━━━━━━━━━━━━');
        
        try {
            await step.handler(mid);
            Monitor.success(mid, `✓ ${step.name} complete`);
            Monitor.info(mid, '');
        } catch (error) {
            Monitor.error(mid, `✗ ${step.name} failed`);
            Monitor.error(mid, error.message);
            Monitor.info(mid, '');
            Monitor.error(mid, '❌ Wizard aborted');
            return;
        }
    }
    
    Monitor.success(mid, '═══════════════════════════════');
    Monitor.success(mid, '✓ Setup Complete!');
    Monitor.success(mid, '═══════════════════════════════');
}

async function setupDatabase(mid) {
    Monitor.debug(mid, 'Checking database connection...');
    await new Promise(resolve => setTimeout(resolve, 1000));
    Monitor.debug(mid, 'Creating tables...');
    await new Promise(resolve => setTimeout(resolve, 1000));
}

async function runMigrations(mid) {
    Monitor.debug(mid, 'Running migrations...');
    await new Promise(resolve => setTimeout(resolve, 1500));
    Monitor.debug(mid, 'Applied 15 migrations');
}

async function seedData(mid) {
    Monitor.debug(mid, 'Seeding users...');
    await new Promise(resolve => setTimeout(resolve, 800));
    Monitor.debug(mid, 'Seeding roles...');
    await new Promise(resolve => setTimeout(resolve, 600));
}

async function configureServices(mid) {
    Monitor.debug(mid, 'Configuring services...');
    await new Promise(resolve => setTimeout(resolve, 1000));
    Monitor.debug(mid, 'Services configured');
}
</script>
@endpush
```

---

## Batch Processing

### CSV Import Example

```blade
<x-monitor-panel 
    id="csv-import-monitor"
    title="Import Progress"
    height="500px"
    maxHeight="700px"
    :enableRecording="true"
    :enableCopy="true"
    :enableDownload="true"
/>

@push('scripts')
<script>
async function importCSV(file) {
    const mid = 'csv-import-monitor';
    
    Monitor.clear(mid);
    Monitor.info(mid, '📊 Starting CSV Import');
    Monitor.info(mid, '═══════════════════════════════════');
    Monitor.debug(mid, `File: ${file.name}`);
    Monitor.debug(mid, `Size: ${(file.size / 1024).toFixed(2)} KB`);
    Monitor.info(mid, '');
    
    const formData = new FormData();
    formData.append('csv_file', file);
    
    try {
        Monitor.info(mid, '⏳ Uploading file...');
        
        const response = await fetch('/api/import/csv', {
            method: 'POST',
            body: formData
        });
        
        const result = await response.json();
        
        if (result.success) {
            Monitor.success(mid, '✓ Upload complete');
            Monitor.info(mid, '');
            Monitor.info(mid, '📈 STATISTICS:');
            Monitor.info(mid, `   Total rows: ${result.total_rows}`);
            Monitor.success(mid, `   Imported: ${result.imported}`);
            
            if (result.skipped > 0) {
                Monitor.warning(mid, `   Skipped: ${result.skipped} (duplicates)`);
            }
            
            if (result.errors && result.errors.length > 0) {
                Monitor.info(mid, '');
                Monitor.error(mid, 'ERRORS FOUND:');
                result.errors.forEach(error => {
                    Monitor.error(mid, `  • Row ${error.row}: ${error.message}`);
                });
            }
            
            Monitor.info(mid, '');
            Monitor.success(mid, '✓ Import complete');
            Monitor.info(mid, '═══════════════════════════════════');
            
            // Show summary toast
            Swal.fire({
                icon: 'success',
                title: 'Import Complete',
                html: `
                    Imported: ${result.imported}<br>
                    Skipped: ${result.skipped}<br>
                    Errors: ${result.errors.length}
                `
            });
        } else {
            Monitor.error(mid, '✗ Import failed: ' + result.message);
            Monitor.info(mid, '═══════════════════════════════════');
        }
        
    } catch (error) {
        Monitor.error(mid, '✗ Network error: ' + error.message);
        Monitor.info(mid, '═══════════════════════════════════');
    }
}
</script>
@endpush
```

---

## Error Handling

### Comprehensive Error Logging

```javascript
async function complexOperation() {
    const mid = 'error-monitor';
    
    Monitor.clear(mid);
    Monitor.info(mid, 'Starting complex operation...');
    
    try {
        // Step 1
        Monitor.debug(mid, 'Step 1: Validating input...');
        await validateInput();
        Monitor.success(mid, '✓ Step 1 complete');
        
        // Step 2
        Monitor.debug(mid, 'Step 2: Processing data...');
        await processData();
        Monitor.success(mid, '✓ Step 2 complete');
        
        // Step 3
        Monitor.debug(mid, 'Step 3: Saving results...');
        await saveResults();
        Monitor.success(mid, '✓ Step 3 complete');
        
        Monitor.info(mid, '');
        Monitor.success(mid, '✓ Operation complete');
        
    } catch (error) {
        Monitor.error(mid, '');
        Monitor.error(mid, '✗ OPERATION FAILED');
        Monitor.error(mid, `Error: ${error.message}`);
        
        if (error.stack) {
            Monitor.debug(mid, '');
            Monitor.debug(mid, 'Stack trace:');
            error.stack.split('\n').slice(0, 3).forEach(line => {
                Monitor.debug(mid, line.trim());
            });
        }
        
        Monitor.info(mid, '');
        Monitor.warning(mid, '⚠ Please try again or contact support');
        
        // Show error toast
        Swal.fire({
            icon: 'error',
            title: 'Operation Failed',
            text: error.message
        });
    }
}
```

---

## Configuration Examples

### Config File Setup

```php
// config/monitor.php
return [
    'enabled' => env('MONITOR_ENABLED', true),
    'storage_path' => 'monitors',
    
    'cleanup' => [
        'enabled' => true,
        'retention_days' => 30,
        'schedule' => 'daily',
    ],
    
    'recording' => [
        'batch_interval' => 2000,  // 2 seconds
        'max_batch_size' => 100,
        'timeout' => 10000,
    ],
    
    'export' => [
        'enabled' => true,
        'formats' => ['txt', 'json', 'csv'],
        'default_format' => 'txt',
    ],
];
```

### Environment Variables

```env
# .env
MONITOR_ENABLED=true
MONITOR_CLEANUP_ENABLED=true
MONITOR_RETENTION_DAYS=30
```

---

## Migration from Inline Implementation

### Before (Inline)

```blade
{{-- Old inline implementation --}}
<div class="card">
    <div class="card-header">
        <h3>Monitor</h3>
        <button onclick="clearInlineMonitor()">Clear</button>
    </div>
    <div class="card-body">
        <div id="inline-monitor" style="font-family: monospace;"></div>
    </div>
</div>

@push('scripts')
<script>
const InlineMonitor = {
    log(id, msg, type) {
        const container = document.getElementById(id);
        const div = document.createElement('div');
        div.textContent = `[${type}] ${msg}`;
        container.appendChild(div);
    },
    clear(id) {
        document.getElementById(id).innerHTML = '';
    }
};
function clearInlineMonitor() { InlineMonitor.clear('inline-monitor'); }
</script>
@endpush
```

### After (Using Protocol)

```blade
{{-- New protocol implementation --}}
<x-monitor-panel id="inline-monitor" />

@push('scripts')
<script>
    // Use global Monitor API
    Monitor.info('inline-monitor', 'Using Monitor Protocol v3.0');
</script>
@endpush
```

---

**For more details, see:**
- [README.md](./README.md) - Complete documentation
- [API-REFERENCE.md](./API-REFERENCE.md) - API specifications
- [AI-AGENT-INSTRUCTIONS.md](./AI-AGENT-INSTRUCTIONS.md) - Integration guide
