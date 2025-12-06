# Development Mode Guide

**Version:** 2.0.0  
**Last Updated:** 6 de diciembre de 2025  
**Status:** STABLE

---

## 🎯 Overview

Development Mode permite editar extensiones en tiempo real sin necesidad de reinstalar. Funciona creando symlinks desde `vendor/bithoven/{extension}` y `public/vendor/bithoven/{extension}` hacia tu repositorio local, permitiendo que los cambios se reflejen inmediatamente.

**Características principales:**
- ✅ Funciona con **CUALQUIER** extensión instalada (VCS o Composer)
- ✅ **Double symlink:** vendor/ + public/ (NEW v2.0.0)
- ✅ Preserva archivos originales en backup `.repo`
- ✅ Toggle fácil on/off
- ✅ Tracking de activación (usuario + timestamp)
- ✅ Disponible via CLI y UI
- ✅ **Assets públicos en sync automático** (NEW v2.0.0)

---

## 🚀 Quick Start

### Activar Dev-Mode

**CLI:**
```bash
php artisan bithoven:extension:dev-mode tickets --enable --path=../EXTENSIONS/bithoven-extension-tickets
```

**UI:**
1. Ir a http://localhost:8000/app/extensions/tickets
2. Sección "Development Tools"
3. Click "Enable Dev-Mode"
4. Ingresar path: `../EXTENSIONS/bithoven-extension-tickets`

### Desactivar Dev-Mode

**CLI:**
```bash
php artisan bithoven:extension:dev-mode tickets --disable
```

**UI:**
1. Click "Disable Dev-Mode" en la misma página

### Ver Estado

**CLI:**
```bash
php artisan bithoven:extension:dev-mode tickets --status
```

---

## 🔧 How It Works

### Proceso de Activación

**BEFORE Dev-Mode:**
```
vendor/bithoven/tickets/
├── src/
├── database/
├── resources/
└── ... (archivos de composer)

public/vendor/bithoven/tickets/
└── ... (assets publicados)
```

**Activación ejecuta:**
1. `mv vendor/bithoven/tickets vendor/bithoven/tickets.repo`
2. `ln -s /absolute/path/to/EXTENSIONS/bithoven-extension-tickets vendor/bithoven/tickets`
3. `mv public/vendor/bithoven/tickets public/vendor/bithoven/tickets.repo` (NEW v2.0.0)
4. `ln -s /absolute/path/to/EXTENSIONS/bithoven-extension-tickets/public public/vendor/bithoven/tickets` (NEW v2.0.0)
5. Guarda config en `extension-settings.json`:
   ```json
   {
     "development_mode": {
       "enabled": true,
       "local_path": "../EXTENSIONS/bithoven-extension-tickets",
       "activated_at": "2025-12-06T20:36:24+00:00",
       "activated_by": 1
     }
   }
   ```

**AFTER Dev-Mode:**
```
vendor/bithoven/
├── tickets → /path/to/EXTENSIONS/bithoven-extension-tickets (symlink)
└── tickets.repo/ (backup de archivos originales)

public/vendor/bithoven/
├── tickets → /path/to/EXTENSIONS/bithoven-extension-tickets/public (symlink) ✨ NEW
└── tickets.repo/ (backup de assets originales) ✨ NEW
```

### Proceso de Desactivación

1. `rm vendor/bithoven/tickets` (elimina symlink)
2. `mv vendor/bithoven/tickets.repo vendor/bithoven/tickets` (restaura backup)
3. `rm public/vendor/bithoven/tickets` (elimina public symlink) ✨ NEW
4. `mv public/vendor/bithoven/tickets.repo public/vendor/bithoven/tickets` (restaura public backup) ✨ NEW
5. Actualiza config: `enabled: false`, agrega `deactivated_at`

---

## ✨ Public Assets Sync (v2.0.0)

### Automatic Sync

Cuando dev-mode está activo, los assets públicos (JS, CSS, imágenes) se sincronizan automáticamente:

**Sin dev-mode (manual):**
```bash
# Editar asset
vim dev-path/public/js/app.js

# Publicar manualmente
php artisan vendor:publish --force --tag=tickets-assets
```

**Con dev-mode (automático):**
```bash
# Editar asset
vim dev-path/public/js/app.js

# ✅ Cambio visible INMEDIATAMENTE en public/vendor/bithoven/tickets/
# NO requiere vendor:publish
```

### Graceful Degradation

Si la extensión **no tiene carpeta `public/`**, dev-mode funciona normalmente:

```bash
php artisan bithoven:extension:dev-mode dummy --enable --path=../EXTENSIONS/dummy
# ✅ Vendor symlink creado
# ℹ️  Public symlink skipped (no public/ folder)
# ✅ Dev-mode activo sin errores
```

### Verification

```bash
# Ver ambos symlinks
ls -la vendor/bithoven/tickets
ls -la public/vendor/bithoven/tickets

# Ver targets
readlink vendor/bithoven/tickets
readlink public/vendor/bithoven/tickets

# Verificar backups
ls -la vendor/bithoven/tickets.repo
ls -la public/vendor/bithoven/tickets.repo
```

---

## 💡 Use Cases

### Caso 1: Desarrollador trabajando en nueva feature

```bash
# Setup inicial
php artisan bithoven:extension:install tickets  # Desde GitHub
php artisan bithoven:extension:dev-mode tickets --enable --path=../EXTENSIONS/bithoven-extension-tickets

# Desarrollo
# Editas archivos en EXTENSIONS/bithoven-extension-tickets/
# Cambios se ven inmediatamente en http://localhost:8000

# Testing
php artisan bithoven:extension:dev-mode tickets --disable
php artisan bithoven:extension:uninstall tickets
php artisan bithoven:extension:install tickets  # Reinstalar para verificar

# Publicar
git push origin main
```

### Caso 2: Extension instalada via Composer, necesitas editar

```bash
# Extension ya instalada como composer package
php artisan bithoven:extension:list
# tickets | 1.2.1 | Active | composer

# Activar dev-mode para editar
php artisan bithoven:extension:dev-mode tickets --enable --path=../EXTENSIONS/bithoven-extension-tickets

# Ahora puedes editar en EXTENSIONS/ y ver cambios inmediatos
```

### Caso 3: Testing de install/uninstall completo

```bash
# Deshabilitar dev-mode primero
php artisan bithoven:extension:dev-mode tickets --disable

# Ahora usas la versión de composer
php artisan bithoven:extension:uninstall tickets

# Reinstalar para test
php artisan bithoven:extension:install tickets
```

---

## 🎨 UI Features

### Development Tools Card

Disponible en `http://localhost:8000/app/extensions/{extension}`

**Muestra:**
- Current status (enabled/disabled)
- Local path (si está habilitado)
- Activation timestamp
- User who activated

**Acciones:**
- Enable Dev-Mode (si deshabilitado)
- Disable Dev-Mode (si habilitado)

### DEV MODE Badge

En la página overview (`http://localhost:8000/app/extensions`), las extensiones en dev-mode muestran un badge amarillo "DEV MODE" debajo del status Active/Inactive.

---

## ⚙️ Configuration

### extension-settings.json

Cuando dev-mode está activo:

```json
{
  "extensions": {
    "tickets": {
      "type": "vcs",
      "url": "https://github.com/Madniatik/bithoven-extension-tickets.git",
      "development_mode": {
        "enabled": true,
        "local_path": "../EXTENSIONS/bithoven-extension-tickets",
        "activated_at": "2025-11-18T03:58:24+00:00",
        "activated_by": 1
      }
    }
  }
}
```

Cuando se desactiva:

```json
{
  "development_mode": {
    "enabled": false,
    "local_path": "../EXTENSIONS/bithoven-extension-tickets",
    "deactivated_at": "2025-11-18T04:05:12+00:00",
    "deactivated_by": 1
  }
}
```

---

## 🔍 Troubleshooting

### Symlink no funciona

**Problema:** Cambios en código no se reflejan

**Solución:**
```bash
# Verificar symlink
ls -la vendor/bithoven/tickets

# Debe mostrar:
# lrwxr-xr-x ... tickets -> /absolute/path/to/EXTENSIONS/bithoven-extension-tickets

# Si no es symlink, desactivar y reactivar
php artisan bithoven:extension:dev-mode tickets --disable
php artisan bithoven:extension:dev-mode tickets --enable --path=../EXTENSIONS/bithoven-extension-tickets
```

### Path incorrecto

**Problema:** Error al activar dev-mode

**Verificar path:**
```bash
# Path debe existir
ls -la ../EXTENSIONS/bithoven-extension-tickets

# Debe contener composer.json
cat ../EXTENSIONS/bithoven-extension-tickets/composer.json
```

### Backup .repo existe antes de activar

**Problema:** Ya existe `tickets.repo`

**Solución:**
```bash
# Eliminar backup antiguo
rm -rf vendor/bithoven/tickets.repo

# Reactivar dev-mode
php artisan bithoven:extension:dev-mode tickets --enable --path=../EXTENSIONS/bithoven-extension-tickets
```

### Extension not installed

**Problema:** "Extension is not installed"

**Solución:**
```bash
# Instalar primero
php artisan bithoven:extension:install tickets

# O instalar local
php artisan bithoven:extension:install-local tickets ../EXTENSIONS/bithoven-extension-tickets

# Luego activar dev-mode
php artisan bithoven:extension:dev-mode tickets --enable --path=../EXTENSIONS/bithoven-extension-tickets
```

---

## 📋 Best Practices

### 1. Disable before uninstall

```bash
# ✅ Correcto
php artisan bithoven:extension:dev-mode tickets --disable
php artisan bithoven:extension:uninstall tickets

# ❌ Evitar (puede dejar symlink huérfano)
php artisan bithoven:extension:uninstall tickets  # con dev-mode activo
```

### 2. Use relative paths

```bash
# ✅ Recomendado (portable)
--path=../EXTENSIONS/bithoven-extension-tickets

# ⚠️ Evitar (no portable)
--path=/Users/madniatik/CODE/LARAVEL/BITHOVEN/EXTENSIONS/bithoven-extension-tickets
```

### 3. Test both modes

```bash
# Desarrollo con dev-mode
php artisan bithoven:extension:dev-mode tickets --enable

# Testing sin dev-mode (simula install real)
php artisan bithoven:extension:dev-mode tickets --disable
```

### 4. Commit often

Cuando trabajas con dev-mode, los cambios son instantáneos. Commitea frecuentemente:

```bash
git add .
git commit -m "feat: add new feature"
```

---

## 🔗 Related Documentation

- **[Development Workflow](./DEVELOPMENT-WORKFLOW.md)** - Flujo completo de desarrollo
- **[Quick Start](./QUICK-START.md)** - Inicio rápido
- **[Extension Manager API](./EXTENSION-MANAGER-API.md)** - API de gestión

---

## 📝 Changelog

### v2.0.0 - 2025-12-06

**Added:**
- Public assets symlink durante dev-mode
- Backup `.repo` para assets públicos
- Sincronización automática de cambios en `{extension}/public/`
- Validación de public symlinks en `validate()`
- Graceful degradation para extensiones sin `public/`

**Changed:**
- `ExtensionDevelopmentService::enable()` ahora crea doble symlink (vendor + public)
- `ExtensionDevelopmentService::disable()` restaura ambos backups
- `ExtensionUninstaller::disableDevelopmentMode()` limpia symlinks públicos
- `getInfo()` incluye información de public symlink

**Developer Experience:**
- ✅ Editar JS/CSS en extensión → Cambios inmediatos en app
- ✅ No más `php artisan vendor:publish --force` manual
- ✅ Assets siempre sincronizados durante desarrollo

### v1.0.0 - 2025-11-18

**Added:**
- Initial release
- Dev-mode funciona con ANY extension type (VCS/Composer)
- UI integration (enable/disable buttons)
- CLI commands
- Status tracking (user + timestamp)

**Breaking Changes:**
- Removed VCS-only restriction
- No longer separate `local` and `local-composer` install types

---

**Remember:** Dev-mode es para DESARROLLO. Siempre deshabilitar antes de push a production!
