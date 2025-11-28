# Extension Migration Requirements System

**Versión:** 1.0.0  
**Fecha:** 14 de noviembre de 2025  
**Implementado en:** v1.5.1

---

## 📋 Resumen

Sistema automático de validación de migraciones requeridas para actualizaciones de extensiones. Protege contra updates que rompan la extensión por falta de cambios en la base de datos.

---

## 🎯 Características

### ✅ Detección Automática
El sistema **detecta automáticamente** si una extensión tiene migraciones pendientes al actualizar:

```php
// Auto-detección basada en nombres de archivos
protected function areMigrationsRequired(array $migrations): bool
{
    // Patterns que indican migraciones REQUERIDAS (schema changes)
    $requiredPatterns = [
        'create_.*_table',      // Nuevas tablas
        'add_.*_to_.*_table',   // Nuevas columnas
        'modify_.*_table',      // Cambios de schema
        'alter_.*_table',       // Alteraciones
        'drop_.*_column',       // Eliminación de columnas
        'rename_.*_table',      // Renombrado de tablas
    ];
    
    // Patterns OPCIONALES (data-only)
    $optionalPatterns = [
        'seed_',                // Seeders
        'update_.*_data',       // Updates de datos
        'populate_',            // Población de datos
    ];
}
```

### 🎨 UI Dinámica

**3 Estados del Modal:**

#### 1️⃣ Migraciones REQUERIDAS (Checkbox Disabled)
```html
<input type="checkbox" id="updateMigrate" checked disabled />
<label>Database migrations (REQUIRED)</label>

<div class="alert alert-warning">
    ⚠️ Required Update: This version includes critical database schema changes.
    3 migration(s) will be executed automatically.
    
    🛡️ A full backup will be created before updating. You can rollback if needed.
</div>
```

**Usuario NO puede desmarcar** → Migrations ejecutadas obligatoriamente

---

#### 2️⃣ Migraciones OPCIONALES (Checkbox Enabled)
```html
<input type="checkbox" id="updateMigrate" checked />
<label>Run database migrations (Optional - 2 available)</label>

<div class="alert alert-info">
    Note: No critical database changes. You can skip migrations and run them later.
</div>
```

**Usuario puede desmarcar** → Flexibilidad para updates menores

---

#### 3️⃣ SIN Migraciones (Sin Checkbox)
```html
<div class="alert alert-light-primary">
    Code-Only Update: This version has no database changes. Only code will be updated.
</div>
```

**Sin opción de migrations** → UI más limpia

---

## 📦 Archivo `extension.json` (Opcional)

Las extensiones pueden declarar explícitamente sus requisitos:

```json
{
  "name": "tickets",
  "version": "1.2.0",
  "migrations": {
    "required": true,
    "count": 3,
    "can_skip": false,
    "details": [
      "2025_11_14_000001_add_priority_to_ticket_categories_table",
      "2025_11_14_000002_add_internal_notes_to_tickets_table",
      "2025_11_14_000003_create_ticket_escalations_table"
    ],
    "description": "Adds priority system, internal notes, and escalation workflow"
  },
  "breaking_changes": false,
  "min_version": "1.0.0"
}
```

**Ubicación:** `vendor/bithoven/{extension-name}/extension.json`

---

## 🔒 Validación Backend

Doble capa de seguridad:

### 1. Frontend
```javascript
// Checkbox disabled si es requerido
if (data.migrations.required) {
    migrationCheckbox.disabled = true;
    migrationCheckbox.checked = true;
}
```

### 2. Backend
```php
// Validación crítica antes de actualizar
$migrationInfo = $this->detectPendingMigrations($request->name);

if ($migrationInfo['required'] && !$request->boolean('migrate', true)) {
    return response()->json([
        'success' => false,
        'message' => 'This update requires database migrations. Cannot proceed without migrations.',
        'migrations_required' => true,
        'migrations_count' => $migrationInfo['count']
    ], 400);
}
```

---

## 🚀 Flujo de Actualización

```
┌─────────────────────────────────────┐
│ 1. Usuario click "Update"           │
└──────────────┬──────────────────────┘
               │
               ▼
┌─────────────────────────────────────┐
│ 2. Fetch /update-info/{name}        │
│    • Detecta migrations pendientes  │
│    • Determina si son requeridas    │
│    • Cuenta cantidad                │
└──────────────┬──────────────────────┘
               │
               ▼
┌─────────────────────────────────────┐
│ 3. Muestra modal con estado         │
│    • Required → disabled checkbox   │
│    • Optional → enabled checkbox    │
│    • None → sin checkbox            │
└──────────────┬──────────────────────┘
               │
               ▼
┌─────────────────────────────────────┐
│ 4. Usuario confirma update          │
└──────────────┬──────────────────────┘
               │
               ▼
┌─────────────────────────────────────┐
│ 5. Validación backend                │
│    ✓ Verifica migrations required   │
│    ✓ Bloquea si no marcado          │
└──────────────┬──────────────────────┘
               │
               ▼
┌─────────────────────────────────────┐
│ 6. Ejecuta update                   │
│    ✓ Crea backup automático         │
│    ✓ Composer update                │
│    ✓ Ejecuta migrations             │
│    ✓ Auto-rollback si falla         │
└──────────────┬──────────────────────┘
               │
               ▼
        ┌──────┴──────┐
        │             │
   ✅ SUCCESS    ❌ FAILED
        │             │
     Reload    Auto-Rollback
```

---

## 📊 Matriz de Decisión

| Migrations | Count | Required | Checkbox | Usuario Puede | Resultado |
|------------|-------|----------|----------|---------------|-----------|
| ✅ Sí | 3 | ✅ Yes | `disabled` | ❌ NO | Forced migrations |
| ✅ Sí | 2 | ❌ No | `enabled` | ✅ SÍ | Opcional, recomendado |
| ❌ No | 0 | N/A | Hidden | N/A | Sin migrations |

---

## 🛠️ Para Desarrolladores de Extensiones

### Opción 1: Auto-Detección (Recomendado)
Usar convención de nombres en migrations:

```php
// REQUERIDA (schema change)
2025_11_14_000001_create_escalations_table.php
2025_11_14_000002_add_priority_to_categories_table.php

// OPCIONAL (data-only)
2025_11_14_000003_seed_default_categories.php
2025_11_14_000004_update_existing_ticket_data.php
```

### Opción 2: Declarar Explícitamente
Crear `extension.json`:

```json
{
  "migrations": {
    "required": true,
    "count": 3,
    "description": "Critical schema changes"
  }
}
```

---

## 🧪 Testing

### Test Manual
```bash
# 1. Crear migration requerida
php artisan make:migration add_test_column_to_tickets_table

# 2. Verificar detección
GET /admin/extensions/update-info/tickets

# Response:
{
  "migrations": {
    "has_migrations": true,
    "required": true,
    "count": 1,
    "can_skip": false
  }
}

# 3. Intentar update sin migrations
POST /admin/extensions/update
{
  "name": "tickets",
  "migrate": false  // ❌ Bloqueado
}

# Response:
{
  "success": false,
  "message": "This update requires database migrations. Cannot proceed without migrations.",
  "migrations_required": true
}
```

---

## ⚠️ Escenarios de Error

### 1. Usuario Intenta Saltar Migrations Requeridas
```
❌ ERROR 400
"This update requires database migrations. Cannot proceed without migrations."
```

### 2. Migrations Fallan Durante Update
```
✅ AUTO-ROLLBACK
• Restaura database desde backup
• Downgrade Composer a versión anterior
• Rollback migrations parciales
• Extension vuelve a estado anterior
```

### 3. Composer Update Falla
```
✅ AUTO-ROLLBACK
• No ejecuta migrations
• Restaura desde backup
• Extension queda en versión anterior
```

---

## 📈 Beneficios

✅ **Protección contra updates rotos**  
✅ **UX clara y directa**  
✅ **Validación en frontend y backend**  
✅ **Auto-detección inteligente**  
✅ **Metadata opcional para control fino**  
✅ **Rollback automático en errores**  
✅ **Funciona para TODAS las extensiones**

---

## 🔗 Archivos Modificados

1. `app/Http/Controllers/Admin/ExtensionController.php`
   - `getUpdateInfo()` - Nuevo endpoint
   - `detectPendingMigrations()` - Detección automática
   - `areMigrationsRequired()` - Análisis de patterns
   - `update()` - Validación agregada

2. `routes/web.php`
   - Nueva ruta `GET /admin/extensions/update-info/{name}`

3. `resources/views/admin/extensions/index.blade.php`
   - Modal dinámico con 3 estados
   - Checkbox disabled cuando es requerido
   - Mensajes contextuales

4. `DOCS/CORE/Extension-Manager/examples/extension.json` (workspace DOCS)
   - Template de referencia

---

## 📝 Notas para el Futuro

- El sistema funciona **sin** `extension.json` (auto-detección)
- El `extension.json` es **opcional** para control fino
- Patterns de detección son **extensibles**
- Sistema es **backward-compatible** (extensiones sin `extension.json` funcionan)

---

**Implementado por:** Claude (Session 20251113-1813)  
**Fecha:** 14 de noviembre de 2025, 02:50
