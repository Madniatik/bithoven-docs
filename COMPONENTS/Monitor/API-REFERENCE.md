# 📡 Monitor Component - API Reference

**Complete method signatures and parameters**

---

## JavaScript API

### Monitor Object

#### `Monitor.log(monitorId, message, type, timestamp)`

Core logging method - all other methods use this internally.

**Parameters:**
- `monitorId` (string, required) - Monitor container ID
- `message` (string, required) - Log message to display
- `type` (string, optional) - Log type: `'success'`, `'error'`, `'warning'`, `'info'`, `'debug'` (default: `'info'`)
- `timestamp` (string|null, optional) - Custom timestamp (default: current time)

**Returns:** `void`

**Example:**
```javascript
Monitor.log('my-monitor', 'Processing...', 'info');
Monitor.log('my-monitor', 'Custom time', 'debug', '12:30:45.123');
```

---

#### `Monitor.success(monitorId, message)`

Shorthand for success logs.

**Parameters:**
- `monitorId` (string, required)
- `message` (string, required)

**Returns:** `void`

**Example:**
```javascript
Monitor.success('my-monitor', '✓ Operation completed');
```

---

#### `Monitor.error(monitorId, message)`

Shorthand for error logs.

**Parameters:**
- `monitorId` (string, required)
- `message` (string, required)

**Returns:** `void`

**Example:**
```javascript
Monitor.error('my-monitor', '✗ Connection failed');
```

---

#### `Monitor.warning(monitorId, message)`

Shorthand for warning logs.

**Parameters:**
- `monitorId` (string, required)
- `message` (string, required)

**Returns:** `void`

**Example:**
```javascript
Monitor.warning('my-monitor', '⚠ Deprecated method used');
```

---

#### `Monitor.info(monitorId, message)`

Shorthand for info logs.

**Parameters:**
- `monitorId` (string, required)
- `message` (string, required)

**Returns:** `void`

**Example:**
```javascript
Monitor.info('my-monitor', 'ℹ Starting process');
```

---

#### `Monitor.debug(monitorId, message)`

Shorthand for debug logs.

**Parameters:**
- `monitorId` (string, required)
- `message` (string, required)

**Returns:** `void`

**Example:**
```javascript
Monitor.debug('my-monitor', '▸ Variable value: 123');
```

---

#### `Monitor.clear(monitorId)`

Clear all logs and reset monitor to "Waiting for events..." state.

**Parameters:**
- `monitorId` (string, required)

**Returns:** `void`

**Example:**
```javascript
Monitor.clear('my-monitor');
```

---

#### `Monitor.getLogs(monitorId)`

Retrieve all current logs from monitor.

**Parameters:**
- `monitorId` (string, required)

**Returns:** `Array<Object>`

```javascript
[
  {
    timestamp: "2025-11-18T22:00:01.123Z",
    type: "info",
    message: "Starting process"
  },
  {
    timestamp: "2025-11-18T22:00:02.456Z",
    type: "success",
    message: "✓ Process complete"
  }
]
```

**Example:**
```javascript
const logs = Monitor.getLogs('my-monitor');
console.log(`Total logs: ${logs.length}`);
```

---

### MonitorRecording Object

#### `MonitorRecording.toggle(monitorId, enabled, endpoint, metadata)`

Toggle recording on/off.

**Parameters:**
- `monitorId` (string, required) - Monitor container ID
- `enabled` (boolean, required) - `true` to start, `false` to stop
- `endpoint` (string, required if enabled) - Server endpoint URL
- `metadata` (object, optional) - Custom metadata to include in session (default: `{}`)

**Returns:** `void`

**Example:**
```javascript
// Start recording
MonitorRecording.toggle('my-monitor', true, '/api/monitor/logs', {
    user_id: 123,
    action: 'install'
});

// Stop recording
MonitorRecording.toggle('my-monitor', false);
```

---

#### `MonitorRecording.start(monitorId, endpoint, metadata)`

Manually start recording session.

**Parameters:**
- `monitorId` (string, required)
- `endpoint` (string, required) - Server endpoint
- `metadata` (object, optional) - Session metadata (default: `{}`)

**Returns:** `void`

**Example:**
```javascript
MonitorRecording.start('my-monitor', '/api/monitor/logs', {
    extension: 'tickets',
    version: '1.2.1'
});
```

---

#### `MonitorRecording.stop(monitorId)`

Manually stop recording session.

**Parameters:**
- `monitorId` (string, required)

**Returns:** `void`

**Example:**
```javascript
MonitorRecording.stop('my-monitor');
```

---

#### `MonitorRecording.isEnabled(monitorId)`

Check if recording is active.

**Parameters:**
- `monitorId` (string, required)

**Returns:** `boolean`

**Example:**
```javascript
if (MonitorRecording.isEnabled('my-monitor')) {
    console.log('Recording is active');
}
```

---

#### `MonitorRecording.queue(monitorId, log)`

Manually add log to recording queue (internal use).

**Parameters:**
- `monitorId` (string, required)
- `log` (object, required) - Log object with `{ timestamp, type, message }`

**Returns:** `void`

**Example:**
```javascript
MonitorRecording.queue('my-monitor', {
    timestamp: new Date().toISOString(),
    type: 'info',
    message: 'Custom log'
});
```

---

#### `MonitorRecording.flush(monitorId)`

Force flush queued logs to server (internal use).

**Parameters:**
- `monitorId` (string, required)

**Returns:** `Promise<void>`

**Example:**
```javascript
await MonitorRecording.flush('my-monitor');
```

---

### MonitorExport Object

#### `MonitorExport.copy(monitorId)`

Copy all logs to clipboard.

**Parameters:**
- `monitorId` (string, required)

**Returns:** `Promise<void>`

**Example:**
```javascript
await MonitorExport.copy('my-monitor');
// Shows SweetAlert2 notification on success/failure
```

---

#### `MonitorExport.download(monitorId)`

Download logs as text file.

**Parameters:**
- `monitorId` (string, required)

**Returns:** `void`

**Example:**
```javascript
MonitorExport.download('my-monitor');
// Downloads file: monitor-my-monitor-1700000000000.txt
```

---

## Blade Component

### `<x-monitor-panel>`

#### Required Props

| Prop | Type | Description |
|------|------|-------------|
| `id` | `string` | Unique monitor identifier |

#### Optional Props - Display

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `title` | `string` | `"📡 Monitor"` | Card header title |
| `height` | `string` | `"300px"` | Minimum container height |
| `maxHeight` | `string` | `"600px"` | Maximum height (scroll after) |
| `class` | `string` | `""` | Additional CSS classes |

#### Optional Props - Features

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `enableRecording` | `boolean` | `false` | Enable server recording toggle |
| `enableDownload` | `boolean` | `true` | Show download button |
| `enableCopy` | `boolean` | `true` | Show copy-to-clipboard button |
| `enableClear` | `boolean` | `true` | Show clear logs button |

#### Optional Props - Recording

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `recordingEndpoint` | `string` | `route('api.monitor.logs')` | Server endpoint for recording |
| `recordingMetadata` | `array` | `[]` | Custom metadata for session |

#### Example

```blade
<x-monitor-panel 
    id="my-monitor"
    title="Custom Title"
    height="500px"
    maxHeight="800px"
    class="shadow-sm"
    :enableRecording="true"
    :enableDownload="true"
    :enableCopy="true"
    :enableClear="true"
    recordingEndpoint="{{ route('api.monitor.logs', ['id' => 'my-monitor']) }}"
    :recordingMetadata="['user_id' => auth()->id(), 'action' => 'install']"
/>
```

---

## Backend API

### MonitorController

Base URL: `/api/monitor`

#### `POST /api/monitor/logs`

Save batch of logs to session.

**Request Body:**
```json
{
  "session_id": "session-1700000000000",
  "monitor_id": "my-monitor",
  "metadata": {
    "user_id": 123,
    "action": "install"
  },
  "logs": [
    {
      "timestamp": "2025-11-18T22:00:01.123Z",
      "type": "info",
      "message": "Starting process"
    }
  ]
}
```

**Response (200):**
```json
{
  "success": true,
  "message": "Logs saved successfully",
  "saved_logs": 1
}
```

**Validation Rules:**
- `session_id`: required, string
- `monitor_id`: required, string
- `metadata`: optional, array
- `logs`: required, array
- `logs.*.timestamp`: required, string
- `logs.*.type`: required, string, in: success,error,warning,info,debug
- `logs.*.message`: required, string

---

#### `GET /api/monitor/session/{sessionId}`

Retrieve session data.

**Response (200):**
```json
{
  "success": true,
  "session": {
    "session_id": "session-1700000000000",
    "monitor_id": "my-monitor",
    "metadata": {},
    "started_at": "2025-11-18T22:00:00+00:00",
    "updated_at": "2025-11-18T22:05:00+00:00",
    "total_logs": 42,
    "logs": [...]
  }
}
```

**Response (404):**
```json
{
  "success": false,
  "message": "Session not found"
}
```

---

#### `GET /api/monitor/sessions/{monitorId}?limit=50`

List all sessions for a monitor.

**Query Parameters:**
- `limit` (optional, int, default: 50) - Max sessions to return

**Response (200):**
```json
{
  "success": true,
  "monitor_id": "my-monitor",
  "total": 3,
  "sessions": [
    {
      "session_id": "session-1700000002000",
      "started_at": "2025-11-18T23:00:00+00:00",
      "total_logs": 15
    },
    {
      "session_id": "session-1700000001000",
      "started_at": "2025-11-18T22:30:00+00:00",
      "total_logs": 28
    }
  ]
}
```

---

#### `GET /api/monitor/export/{sessionId}?format=txt`

Export session to file.

**Query Parameters:**
- `format` (optional, string, default: `txt`) - Export format: `txt`, `json`, `csv`

**Response (200):**
```json
{
  "success": true,
  "file_path": "/path/to/storage/exports/session-1700000000000.txt",
  "download_url": "/api/monitor/download/session-1700000000000?format=txt"
}
```

**Response (400):**
```json
{
  "success": false,
  "message": "Invalid format. Allowed: txt, json, csv"
}
```

---

#### `GET /api/monitor/download/{sessionId}?format=txt`

Download exported session file.

**Query Parameters:**
- `format` (optional, string, default: `txt`) - File format

**Response (200):** File download (auto-deleted after send)

**Response (404):**
```json
{
  "success": false,
  "message": "File not found"
}
```

---

### MonitorService

#### `saveLogs(string $sessionId, string $monitorId, array $metadata, array $logs): bool`

Save logs batch to storage.

**Parameters:**
- `$sessionId` - Unique session identifier
- `$monitorId` - Monitor identifier
- `$metadata` - Custom session metadata
- `$logs` - Array of log objects

**Returns:** `bool` - Success status

---

#### `getSession(string $sessionId): ?array`

Retrieve session by ID.

**Parameters:**
- `$sessionId` - Session identifier

**Returns:** `array|null` - Session data or null if not found

---

#### `getSessions(string $monitorId, int $limit = 50): array`

Get all sessions for monitor.

**Parameters:**
- `$monitorId` - Monitor identifier
- `$limit` - Maximum sessions (default: 50)

**Returns:** `array` - Array of sessions

---

#### `cleanOldSessions(int $days = 30): int`

Delete sessions older than specified days.

**Parameters:**
- `$days` - Age threshold (default: 30)

**Returns:** `int` - Number of deleted sessions

---

#### `exportSession(string $sessionId, string $format = 'txt'): ?string`

Export session to file.

**Parameters:**
- `$sessionId` - Session identifier
- `$format` - Export format (`txt`, `json`, `csv`)

**Returns:** `string|null` - File path or null on failure

---

## Data Structures

### Log Object

```typescript
interface Log {
    timestamp: string;  // ISO 8601 format
    type: 'success' | 'error' | 'warning' | 'info' | 'debug';
    message: string;
}
```

### Session Object

```typescript
interface Session {
    session_id: string;
    monitor_id: string;
    metadata: Record<string, any>;
    started_at: string;  // ISO 8601
    updated_at: string;  // ISO 8601
    total_logs: number;
    logs: Log[];
}
```

### Recording Session (Internal)

```typescript
interface RecordingSession {
    enabled: boolean;
    sessionId: string;
    endpoint: string;
    metadata: Record<string, any>;
    startedAt: string;  // ISO 8601
}
```

---

## Constants

### Log Types

```javascript
const LOG_TYPES = ['success', 'error', 'warning', 'info', 'debug'];
```

### Icons

```javascript
const ICONS = {
    success: '✓',
    error: '✗',
    warning: '⚠',
    info: 'ℹ',
    debug: '▸'
};
```

### Colors (CSS Classes)

```javascript
const COLORS = {
    success: 'text-success',
    error: 'text-danger',
    warning: 'text-warning',
    info: 'text-info',
    debug: 'text-muted'
};
```

### Batch Interval

```javascript
const BATCH_INTERVAL = 2000; // milliseconds
```

---

## Events

Monitor does not emit custom events. Use callbacks within your code.

**Example:**
```javascript
function processWithCallback(onProgress, onComplete) {
    Monitor.info('my-monitor', 'Starting...');
    onProgress?.(0);
    
    // ... processing ...
    
    Monitor.success('my-monitor', 'Complete!');
    onComplete?.();
}
```
