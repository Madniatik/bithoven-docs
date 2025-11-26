# Extension Permissions Protocol v2.0

**Última actualización:** 26 de noviembre de 2025  
**Versión:** 2.0.0  
**Estado:** Implementado y funcional

---

## 📋 Resumen

Sistema automático de detección y carga de permisos de extensiones con alias y description.  
**Cero hardcodeo** - las extensiones se autoregistran automáticamente.

---

## 🎯 Objetivo

Permitir que las extensiones definan permisos con alias (nombre amigable) y description (descripción detallada) sin necesidad de modificar el proyecto principal.

---

## 🏗️ Arquitectura

### Componente Principal
**ExtensionInstaller::createExtensionPermissions()**
- Ubicación: `app/Services/Extensions/Installation/ExtensionInstaller.php`
- Responsabilidad: Detectar y cargar permisos automáticamente durante instalación

### Flujo de Detección (2 Fases)

#### FASE 1: Detección de Data Class (Recomendado - v2.0)
```
1. Buscar: vendor/bithoven/{extension}/database/seeders/data/{ExtensionName}Permissions.php
2. Si existe → Cargar con alias y description
3. Usar método estático all() para obtener permisos
```

#### FASE 2: Fallback a ServiceProvider (Legacy)
```
1. Si no hay Data Class → Leer ServiceProvider
2. Extraer array $permissions con regex
3. Crear permisos SIN alias ni description
```

---

## 📝 Convención de Nomenclatura

### Estructura de Archivos
```
vendor/bithoven/{extension-slug}/
└── database/
    └── seeders/
        └── data/
            └── {ExtensionName}Permissions.php
```

### Ejemplos de Conversión
| Extension Slug | Clase Esperada |
|---------------|----------------|
| `tickets` | `TicketsPermissions` |
| `llm-manager` | `LLMManagerPermissions` o `LLMPermissions` |
| `user-analytics` | `UserAnalyticsPermissions` |
| `api-gateway` | `APIGatewayPermissions` |

**Regla:** `slug-name` → `StudlyCase` → `{StudlyCase}Permissions`

---

## 🔧 Implementación para Desarrolladores de Extensiones

### Paso 1: Crear Estructura de Datos

**Ubicación:** `database/seeders/data/{ExtensionName}Permissions.php`

```php
<?php

namespace Bithoven\{ExtensionName}\Database\Seeders\Data;

class {ExtensionName}Permissions
{
    /**
     * Get all extension permissions with alias and descriptions
     * 
     * @return array
     */
    public static function all(): array
    {
        return [
            [
                'name' => 'extensions:{slug}:scope:action',
                'alias' => 'Nombre Amigable en Español',
                'description' => 'Descripción detallada de qué permite hacer este permiso. Incluye casos de uso y alcance.'
            ],
            // ... más permisos
        ];
    }

    /**
     * Get permissions grouped by scope (opcional pero recomendado)
     * 
     * @return array
     */
    public static function byScope(): array
    {
        $all = self::all();
        $grouped = [];

        foreach ($all as $permission) {
            $parts = explode(':', $permission['name']);
            $scope = $parts[2] ?? 'other';
            
            if (!isset($grouped[$scope])) {
                $grouped[$scope] = [];
            }
            
            $grouped[$scope][] = $permission;
        }

        return $grouped;
    }

    /**
     * Get permission names only (opcional pero recomendado)
     * 
     * @return array
     */
    public static function names(): array
    {
        return array_column(self::all(), 'name');
    }
}
```

### Paso 2: Definir Permisos en composer.json (Opcional pero Recomendado)

```json
{
    "extra": {
        "bithoven": {
            "extension": true,
            "permissions": [
                "extensions:myext:base:view",
                "extensions:myext:base:create",
                "extensions:myext:admin:manage"
            ]
        }
    }
}
```

**Nota:** Esta lista es para documentación y validación. El ExtensionInstaller usará la Data Class automáticamente.

---

## 🚀 Proceso Automático

### Durante la Instalación
```bash
php artisan bithoven:extension:install my-extension
```

**Flujo interno:**
1. ExtensionInstaller detecta `MyExtensionPermissions.php`
2. Carga permisos con `MyExtensionPermissions::all()`
3. Crea permisos usando `Permission::updateOrCreate()`:
   - name: `extensions:myext:scope:action`
   - alias: `Nombre Amigable`
   - description: `Descripción detallada`
4. Asigna permisos a rol super-admin
5. Limpia cache de permisos

### Durante la Desinstalación
```bash
php artisan bithoven:extension:uninstall my-extension
```

**Flujo interno:**
1. Elimina permisos que empiecen con `extensions:myext:`
2. Limpia asignaciones de roles
3. Remueve de cache

---

## 📊 Ejemplos Reales

### Ejemplo 1: Tickets Extension

**Archivo:** `vendor/bithoven/tickets/database/seeders/data/TicketsPermissions.php`

```php
public static function all(): array
{
    return [
        [
            'name' => 'extensions:tickets:base:view',
            'alias' => 'Ver Tickets',
            'description' => 'Permite ver la lista de tickets y sus detalles. Incluye acceso al módulo de soporte y visualización de tickets propios y asignados.'
        ],
        [
            'name' => 'extensions:tickets:base:create',
            'alias' => 'Crear Tickets',
            'description' => 'Permite crear nuevos tickets de soporte. Incluye selección de categoría, prioridad, asignación y carga de archivos adjuntos.'
        ],
        // ... 6 permisos más
    ];
}
```

**Total:** 8 permisos con alias y description completos

### Ejemplo 2: LLM Manager Extension

**Archivo:** `vendor/bithoven/llm-manager/database/seeders/data/LLMPermissions.php`

```php
public static function all(): array
{
    return [
        [
            'name' => 'extensions:llm:base:view',
            'alias' => 'Ver LLM Manager',
            'description' => 'Permite acceder al dashboard de LLM Manager y ver configuraciones generales. Incluye visualización de modelos, proveedores y estadísticas básicas.'
        ],
        [
            'name' => 'extensions:llm:models:manage',
            'alias' => 'Gestionar Modelos LLM',
            'description' => 'Permite configurar y gestionar modelos de lenguaje. Incluye activación, desactivación, configuración de parámetros y selección de proveedores.'
        ],
        // ... 10 permisos más
    ];
}
```

**Total:** 12 permisos con alias y description completos

---

## 🔄 Actualización de Extensiones Existentes

### Opción A: Script Temporal (Para sistema actual)

```bash
php artisan db:seed --class=UpdateExtensionPermissionsSeeder
```

Este seeder es temporal y actualiza los permisos de extensiones ya instaladas.

### Opción B: Reinstalación (Recomendado para producción)

```bash
# Desinstalar
php artisan bithoven:extension:uninstall my-extension

# Reinstalar (ahora con alias/description)
php artisan bithoven:extension:install my-extension
```

---

## ✅ Ventajas del Sistema

### 1. Cero Hardcodeo
- No hay que modificar el proyecto principal NUNCA
- Nueva extensión = funciona automáticamente

### 2. Backward Compatible
- Extensiones antiguas sin Data Class siguen funcionando
- Migración gradual sin breaking changes

### 3. Autodocumentado
- Los permisos tienen descripciones claras
- Facilita auditorías y compliance

### 4. Modular y Escalable
- Cada extensión gestiona sus propios permisos
- Sin dependencias entre extensiones

### 5. DX (Developer Experience) Mejorado
- Convención clara y simple
- Menos código repetitivo
- Errores más claros en logs

---

## 🐛 Troubleshooting

### Problema: "Permissions Data Class no encontrada"
**Causa:** Nombre de clase o namespace incorrecto

**Solución:**
1. Verificar que el archivo existe en `database/seeders/data/`
2. Verificar nomenclatura: `{StudlyCase}Permissions.php`
3. Verificar namespace: `Bithoven\{ExtensionName}\Database\Seeders\Data\{ClassName}`

### Problema: "Permisos creados sin alias/description"
**Causa:** Data Class no tiene método `all()` o devuelve formato incorrecto

**Solución:**
1. Verificar que existe `public static function all(): array`
2. Verificar que devuelve array de arrays con keys: `name`, `alias`, `description`

### Problema: "Class not found error durante instalación"
**Causa:** Autoload de Composer no ha cargado la clase

**Solución:**
```bash
composer dump-autoload
```

---

## 📈 Métricas de Éxito

### Proyecto BITHOVEN CPANEL
- ✅ 60 permisos core con alias/description (100%)
- ✅ 12 permisos LLM Manager con alias/description (100%)
- ✅ 8 permisos Tickets preparados (pendiente instalación)
- ✅ 0 líneas hardcodeadas en proyecto principal
- ✅ 100% compatibilidad con extensiones legacy

---

## 🔮 Próximos Pasos

### v2.1.0 (Futuro)
- [ ] Agregar validación de permisos en extension.json
- [ ] CLI command para generar boilerplate de Permissions Data Class
- [ ] Soporte para traducciones de alias/description
- [ ] Dashboard de auditoría de permisos

### v2.2.0 (Futuro)
- [ ] Soporte para permisos jerárquicos
- [ ] Detección automática de permisos obsoletos
- [ ] Migración automática entre versiones de permisos

---

## 📚 Referencias

- **ExtensionInstaller:** `app/Services/Extensions/Installation/ExtensionInstaller.php`
- **Ejemplo Tickets:** `/EXTENSIONS/bithoven-extension-tickets/database/seeders/data/TicketsPermissions.php`
- **Ejemplo LLM:** `/EXTENSIONS/bithoven-extension-llm-manager/database/seeders/data/LLMPermissions.php`
- **Core Permissions:** `database/seeders/data/CorePermissions.php`

---

**Última revisión:** 26 de noviembre de 2025, 11:15  
**Autor:** AI Agent (Claude Sonnet 4.5)  
**Estado:** ✅ Production Ready
