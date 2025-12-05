# 💡 Debug Console Examples

**Real-world integration examples and common patterns**

---

## 📖 Table of Contents

1. [Extension Registration](#-extension-registration)
2. [Basic Logging](#-basic-logging)
3. [Advanced Patterns](#-advanced-patterns)
4. [Performance Monitoring](#-performance-monitoring)
5. [Error Handling](#-error-handling)
6. [Common Use Cases](#-common-use-cases)

---

## 🔌 Extension Registration

### Example 1: LLM Manager Extension (Complete)

**File 1:** `config/llm-manager.php`

```php
<?php

return [
    // ... existing config ...
    
    'debug_console' => [
        'enabled' => env('LLM_DEBUG_CONSOLE', false),
        'level' => env('LLM_DEBUG_LEVEL', 'info'),
    ],
];
```

---

**File 2:** `resources/views/partials/debug-console-registration.blade.php`

```blade
@push('debug-console-extensions')
<script>
    window.DEBUG_CONSOLE_CONFIG = window.DEBUG_CONSOLE_CONFIG || {};
    window.DEBUG_CONSOLE_CONFIG['llm-manager'] = {
        enabled: {{ config('llm-manager.debug_console.enabled', false) ? 'true' : 'false' }},
        level: '{{ config('llm-manager.debug_console.level', 'info') }}'
    };
    
    document.addEventListener('DOMContentLoaded', function() {
        if (window.DebugConsole) {
            window.MonitorLogger = window.DebugConsole.create('llm-manager');
            
            @if(config('llm-manager.debug_console.enabled', false))
                MonitorLogger.info('LLM Manager Debug Console registered');
            @endif
        }
    });
</script>
@endpush
```

---

**File 3:** `src/LLMServiceProvider.php`

```php
<?php

namespace Bithoven\LLMManager;

use Illuminate\Support\ServiceProvider;
use Illuminate\Support\Facades\View;

class LLMServiceProvider extends ServiceProvider
{
    public function register()
    {
        // ... existing registrations ...
        
        // Register Debug Console for extension
        View::composer('*', function ($view) {
            static $registered = false;
            if (!$registered) {
                $registered = true;
                $view->with('__llmDebugConsoleRegistration', 
                    view('llm-manager::partials.debug-console-registration')->render()
                );
            }
        });
    }
}
```

---

**File 4:** `.env`

```env
LLM_DEBUG_CONSOLE=true
LLM_DEBUG_LEVEL=debug
```

---

### Example 2: Tickets Extension (Complete)

**File 1:** `config/tickets.php`

```php
<?php

return [
    // ... existing config ...
    
    'debug_console' => [
        'enabled' => env('TICKETS_DEBUG_CONSOLE', false),
        'level' => env('TICKETS_DEBUG_LEVEL', 'info'),
    ],
];
```

---

**File 2:** `resources/views/partials/debug-console-registration.blade.php`

```blade
@push('debug-console-extensions')
<script>
    window.DEBUG_CONSOLE_CONFIG = window.DEBUG_CONSOLE_CONFIG || {};
    window.DEBUG_CONSOLE_CONFIG['tickets'] = {
        enabled: {{ config('tickets.debug_console.enabled', false) ? 'true' : 'false' }},
        level: '{{ config('tickets.debug_console.level', 'info') }}'
    };
    
    document.addEventListener('DOMContentLoaded', function() {
        if (window.DebugConsole) {
            window.TicketsLogger = window.DebugConsole.create('tickets');
            
            @if(config('tickets.debug_console.enabled', false))
                TicketsLogger.info('Tickets System Debug Console registered');
            @endif
        }
    });
</script>
@endpush
```

---

**File 3:** `src/TicketsServiceProvider.php`

```php
<?php

namespace Bithoven\Tickets;

use Illuminate\Support\ServiceProvider;
use Illuminate\Support\Facades\View;

class TicketsServiceProvider extends ServiceProvider
{
    public function register()
    {
        // ... existing registrations ...
        
        // Register Debug Console for extension
        View::composer('*', function ($view) {
            static $registered = false;
            if (!$registered) {
                $registered = true;
                $view->with('__ticketsDebugConsoleRegistration', 
                    view('tickets::partials.debug-console-registration')->render()
                );
            }
        });
    }
}
```

---

**File 4:** `.env`

```env
TICKETS_DEBUG_CONSOLE=true
TICKETS_DEBUG_LEVEL=info
```

---

## 📝 Basic Logging

### Example 3: Simple Logging

```javascript
// Info logging (normal operations)
AppLogger.info('Application started');
MonitorLogger.info('Chat session initialized');
TicketsLogger.info('Ticket list loaded');

// Debug logging (development details)
AppLogger.debug('Configuration:', config);
MonitorLogger.debug('Message sent:', messageData);
TicketsLogger.debug('Form data:', formData);

// Warning logging (non-critical issues)
AppLogger.warn('Cache miss for key:', cacheKey);
MonitorLogger.warn('API rate limit approaching');
TicketsLogger.warn('Deprecated method used');

// Error logging (critical failures)
AppLogger.error('Database connection failed');
MonitorLogger.error('API request failed:', error);
TicketsLogger.error('Validation failed:', errors);
```

---

### Example 4: Conditional Logging

```javascript
// Safe access (undefined if not registered)
window.MonitorLogger?.info('Extension loaded');

// Explicit check
if (window.TicketsLogger) {
    TicketsLogger.debug('Ticket details:', ticket);
}

// Function wrapper
function logTicket(action, data) {
    if (window.TicketsLogger) {
        TicketsLogger.info(`Ticket ${action}:`, data);
    }
}

logTicket('created', {id: 123, title: 'Bug report'});
```

---

## 🚀 Advanced Patterns

### Example 5: Grouped Logging

```javascript
// Simple group
MonitorLogger.group('Processing Chat Messages');
messages.forEach(msg => {
    MonitorLogger.debug('Message:', msg.id, msg.content);
});
MonitorLogger.groupEnd();

// Nested groups
TicketsLogger.group('Batch Processing 100 Tickets');

for (let i = 0; i < 100; i += 10) {
    TicketsLogger.group(`Batch ${i}-${i+10}`);
    
    for (let j = i; j < i + 10; j++) {
        TicketsLogger.debug(`Processing ticket ${j}`);
        processTicket(j);
    }
    
    TicketsLogger.groupEnd();
}

TicketsLogger.groupEnd();
TicketsLogger.info('All tickets processed');
```

---

### Example 6: Table Display

```javascript
// Display array of objects
const users = [
    {id: 1, name: 'John', email: 'john@example.com', role: 'Admin'},
    {id: 2, name: 'Jane', email: 'jane@example.com', role: 'User'},
    {id: 3, name: 'Bob', email: 'bob@example.com', role: 'User'}
];

AppLogger.table(users);

// Display filtered data
const tickets = await fetchTickets();
TicketsLogger.info(`Fetched ${tickets.length} tickets`);
TicketsLogger.table(tickets.map(t => ({
    id: t.id,
    title: t.title,
    status: t.status,
    priority: t.priority
})));

// Display object properties
const config = {
    apiUrl: 'https://api.example.com',
    timeout: 30000,
    retries: 3
};

MonitorLogger.table(config);
```

---

## ⚡ Performance Monitoring

### Example 7: Timing Operations

```javascript
// Single operation
MonitorLogger.time('Fetch Chat History');
const history = await fetch('/api/chat/history').then(r => r.json());
MonitorLogger.timeEnd('Fetch Chat History');
// Output: llm-manager Fetch Chat History: 234.56ms

// Multiple operations
AppLogger.time('Total Load Time');

AppLogger.time('Fetch Users');
const users = await fetchUsers();
AppLogger.timeEnd('Fetch Users');

AppLogger.time('Fetch Settings');
const settings = await fetchSettings();
AppLogger.timeEnd('Fetch Settings');

AppLogger.timeEnd('Total Load Time');
```

---

### Example 8: Performance Analysis

```javascript
// With detailed logging
async function loadDashboard() {
    MonitorLogger.group('Dashboard Load Performance');
    
    MonitorLogger.time('Total Dashboard Load');
    
    // Step 1
    MonitorLogger.time('Fetch User Data');
    const user = await fetchUser();
    MonitorLogger.timeEnd('Fetch User Data');
    MonitorLogger.debug('User loaded:', user.name);
    
    // Step 2
    MonitorLogger.time('Fetch Stats');
    const stats = await fetchStats(user.id);
    MonitorLogger.timeEnd('Fetch Stats');
    MonitorLogger.debug('Stats:', stats);
    
    // Step 3
    MonitorLogger.time('Render Charts');
    renderCharts(stats);
    MonitorLogger.timeEnd('Render Charts');
    
    MonitorLogger.timeEnd('Total Dashboard Load');
    MonitorLogger.groupEnd();
}

// Output:
// [llm-manager] [GROUP] 14:30:45 Dashboard Load Performance
//   llm-manager Fetch User Data: 123ms
//   [llm-manager] [DEBUG] 14:30:45 User loaded: John Doe
//   llm-manager Fetch Stats: 456ms
//   [llm-manager] [DEBUG] 14:30:45 Stats: {...}
//   llm-manager Render Charts: 78ms
//   llm-manager Total Dashboard Load: 657ms
```

---

## 🐛 Error Handling

### Example 9: Try-Catch with Logging

```javascript
async function createTicket(data) {
    TicketsLogger.info('Creating ticket:', data.title);
    TicketsLogger.time('Create Ticket');
    
    try {
        const response = await fetch('/api/tickets', {
            method: 'POST',
            body: JSON.stringify(data)
        });
        
        if (!response.ok) {
            throw new Error(`HTTP ${response.status}: ${response.statusText}`);
        }
        
        const ticket = await response.json();
        
        TicketsLogger.timeEnd('Create Ticket');
        TicketsLogger.info('Ticket created:', ticket.id);
        
        return ticket;
        
    } catch (error) {
        TicketsLogger.timeEnd('Create Ticket');
        TicketsLogger.error('Failed to create ticket:', error);
        TicketsLogger.debug('Request data:', data);
        
        throw error;
    }
}
```

---

### Example 10: Validation Logging

```javascript
function validateTicketForm(form) {
    TicketsLogger.group('Ticket Form Validation');
    
    const errors = {};
    
    // Title validation
    if (!form.title || form.title.length < 3) {
        errors.title = 'Title must be at least 3 characters';
        TicketsLogger.warn('Invalid title:', form.title);
    } else {
        TicketsLogger.debug('Title valid:', form.title);
    }
    
    // Description validation
    if (!form.description) {
        errors.description = 'Description is required';
        TicketsLogger.warn('Missing description');
    } else {
        TicketsLogger.debug('Description valid');
    }
    
    // Priority validation
    if (!['low', 'medium', 'high'].includes(form.priority)) {
        errors.priority = 'Invalid priority';
        TicketsLogger.warn('Invalid priority:', form.priority);
    } else {
        TicketsLogger.debug('Priority valid:', form.priority);
    }
    
    TicketsLogger.groupEnd();
    
    if (Object.keys(errors).length > 0) {
        TicketsLogger.error('Validation failed:', errors);
        return {valid: false, errors};
    }
    
    TicketsLogger.info('Validation passed');
    return {valid: true};
}
```

---

## 💼 Common Use Cases

### Example 11: Chat Application

```javascript
// Initialize chat
MonitorLogger.info('Chat workspace initialized');
MonitorLogger.debug('Configuration:', chatConfig);

// Send message
async function sendMessage(content) {
    MonitorLogger.group('Send Message');
    MonitorLogger.debug('Content:', content);
    
    MonitorLogger.time('Send Message API');
    
    try {
        const response = await fetch('/api/chat/send', {
            method: 'POST',
            body: JSON.stringify({content})
        });
        
        const data = await response.json();
        
        MonitorLogger.timeEnd('Send Message API');
        MonitorLogger.info('Message sent:', data.id);
        MonitorLogger.debug('Response:', data);
        
        MonitorLogger.groupEnd();
        return data;
        
    } catch (error) {
        MonitorLogger.timeEnd('Send Message API');
        MonitorLogger.error('Send failed:', error);
        MonitorLogger.groupEnd();
        throw error;
    }
}

// Receive streaming response
function handleStreamingResponse(reader) {
    MonitorLogger.info('Streaming response started');
    MonitorLogger.time('Stream Complete');
    
    let chunks = 0;
    
    reader.on('data', chunk => {
        chunks++;
        MonitorLogger.debug(`Chunk ${chunks}:`, chunk.length, 'bytes');
    });
    
    reader.on('end', () => {
        MonitorLogger.timeEnd('Stream Complete');
        MonitorLogger.info('Streaming complete:', chunks, 'chunks received');
    });
    
    reader.on('error', error => {
        MonitorLogger.timeEnd('Stream Complete');
        MonitorLogger.error('Streaming failed:', error);
    });
}
```

---

### Example 12: Ticket Management

```javascript
// Load tickets list
async function loadTickets(filters = {}) {
    TicketsLogger.group('Load Tickets');
    TicketsLogger.debug('Filters:', filters);
    
    TicketsLogger.time('Fetch Tickets');
    
    const response = await fetch('/api/tickets?' + new URLSearchParams(filters));
    const tickets = await response.json();
    
    TicketsLogger.timeEnd('Fetch Tickets');
    TicketsLogger.info(`Loaded ${tickets.length} tickets`);
    
    TicketsLogger.table(tickets.map(t => ({
        id: t.id,
        title: t.title,
        status: t.status,
        priority: t.priority,
        created: t.created_at
    })));
    
    TicketsLogger.groupEnd();
    
    return tickets;
}

// Update ticket status
async function updateTicketStatus(ticketId, newStatus) {
    TicketsLogger.info(`Updating ticket ${ticketId} to ${newStatus}`);
    
    TicketsLogger.time(`Update Ticket ${ticketId}`);
    
    try {
        const response = await fetch(`/api/tickets/${ticketId}/status`, {
            method: 'PATCH',
            body: JSON.stringify({status: newStatus})
        });
        
        const ticket = await response.json();
        
        TicketsLogger.timeEnd(`Update Ticket ${ticketId}`);
        TicketsLogger.info('Ticket updated successfully');
        TicketsLogger.debug('Updated ticket:', ticket);
        
        return ticket;
        
    } catch (error) {
        TicketsLogger.timeEnd(`Update Ticket ${ticketId}`);
        TicketsLogger.error('Update failed:', error);
        throw error;
    }
}
```

---

### Example 13: Extension Installation

```javascript
// Monitor extension installation process
async function installExtension(extensionName) {
    AppLogger.group(`Installing Extension: ${extensionName}`);
    AppLogger.time('Total Installation Time');
    
    try {
        // Step 1: Validate
        AppLogger.info('Step 1/5: Validating extension...');
        AppLogger.time('Validation');
        await validateExtension(extensionName);
        AppLogger.timeEnd('Validation');
        AppLogger.info('✓ Validation passed');
        
        // Step 2: Download
        AppLogger.info('Step 2/5: Downloading extension...');
        AppLogger.time('Download');
        const package = await downloadExtension(extensionName);
        AppLogger.timeEnd('Download');
        AppLogger.info('✓ Downloaded:', package.size, 'bytes');
        
        // Step 3: Extract
        AppLogger.info('Step 3/5: Extracting files...');
        AppLogger.time('Extract');
        await extractExtension(package);
        AppLogger.timeEnd('Extract');
        AppLogger.info('✓ Files extracted');
        
        // Step 4: Run migrations
        AppLogger.info('Step 4/5: Running migrations...');
        AppLogger.time('Migrations');
        const migrations = await runMigrations(extensionName);
        AppLogger.timeEnd('Migrations');
        AppLogger.info('✓ Migrations complete:', migrations.length, 'ran');
        
        // Step 5: Activate
        AppLogger.info('Step 5/5: Activating extension...');
        AppLogger.time('Activation');
        await activateExtension(extensionName);
        AppLogger.timeEnd('Activation');
        AppLogger.info('✓ Extension activated');
        
        AppLogger.timeEnd('Total Installation Time');
        AppLogger.info(`✓ ${extensionName} installed successfully`);
        AppLogger.groupEnd();
        
    } catch (error) {
        AppLogger.timeEnd('Total Installation Time');
        AppLogger.error('Installation failed:', error);
        AppLogger.debug('Extension:', extensionName);
        AppLogger.groupEnd();
        throw error;
    }
}
```

---

### Example 14: DataTable Integration

```javascript
// Monitor DataTable initialization
$(document).ready(function() {
    AppLogger.info('Initializing tickets DataTable');
    AppLogger.time('DataTable Init');
    
    const table = $('#tickets-table').DataTable({
        processing: true,
        serverSide: true,
        ajax: '/api/tickets/data',
        
        // Log AJAX requests
        ajax: function(data, callback, settings) {
            AppLogger.debug('DataTable AJAX request:', {
                draw: data.draw,
                start: data.start,
                length: data.length
            });
            
            AppLogger.time('DataTable AJAX');
            
            $.ajax('/api/tickets/data', {
                data: data,
                success: function(response) {
                    AppLogger.timeEnd('DataTable AJAX');
                    AppLogger.info('DataTable data loaded:', response.recordsTotal, 'records');
                    AppLogger.debug('Response:', response);
                    callback(response);
                },
                error: function(error) {
                    AppLogger.timeEnd('DataTable AJAX');
                    AppLogger.error('DataTable AJAX failed:', error);
                }
            });
        }
    });
    
    AppLogger.timeEnd('DataTable Init');
    AppLogger.info('DataTable initialized');
});
```

---

## 📖 Related Documentation

- **[README.md](./README.md)** - Main documentation, API overview
- **[AI-AGENT-INSTRUCTIONS.md](./AI-AGENT-INSTRUCTIONS.md)** - Integration guide
- **[API-REFERENCE.md](./API-REFERENCE.md)** - Complete API specifications
- **[CONFIGURATION.md](./CONFIGURATION.md)** - Deployment and configuration

---

**Last Updated:** 5 de diciembre de 2025  
**Version:** 1.0.0  
**Examples:** Production-Ready ✅
