# Bithoven Documentation

**Versión:** 1.0.0  
**Última Actualización:** 26 de noviembre de 2025

---

## 📖 Descripción

Este repositorio contiene la documentación completa del proyecto **Bithoven**, un sistema modular Laravel con arquitectura de extensiones basado en el tema Metronic 8.3.2.

La documentación está organizada en dos categorías principales:
- **CORE**: Documentación del sistema central
- **COMPONENTS**: Documentación de componentes reutilizables

---

## 📂 Estructura del Repositorio

```
DOCS/
├── CORE/                      # Sistema Central
│   ├── Extension-Manager/     # Sistema de gestión de extensiones
│   └── User-Management/       # Sistema de usuarios y permisos
│
└── COMPONENTS/                # Componentes
    └── Monitor/               # Sistema de monitoreo
```

---

## 🎯 CORE - Sistema Central

### Extension Manager

Sistema completo de gestión de extensiones con marketplace integrado.

**Documentación:**
- [README Principal](CORE/Extension-Manager/README.md) - Guía completa del sistema
- [Developer Guide](CORE/Extension-Manager/DEVELOPER-GUIDE.md) - Guía para desarrolladores
- [CHANGELOG](CORE/Extension-Manager/CHANGELOG.md) - Historial de cambios
- [Audit Report](CORE/Extension-Manager/AUDIT-REPORT-2025-11-19.md) - Auditoría del sistema

**Características:**
- ✅ Instalación/desinstalación de extensiones
- ✅ Sistema de dependencias
- ✅ Marketplace con GitHub API
- ✅ Gestión de permisos
- ✅ Migraciones automáticas
- ✅ CLI commands

### User Management

Sistema de gestión de usuarios, roles y permisos (Spatie Permission).

**Documentación:**
- [Permissions](CORE/User-Management/Permissions/) - Sistema de permisos
- [Roles](CORE/User-Management/Roles/) - Sistema de roles

---

## 🧩 COMPONENTS - Componentes Reutilizables

### Monitor Component

Sistema de monitoreo en tiempo real para aplicaciones web.

**Documentación:**
- [README](COMPONENTS/Monitor/README.md) - Introducción y características
- [Configuration](COMPONENTS/Monitor/CONFIGURATION.md) - Guía de configuración
- [API Reference](COMPONENTS/Monitor/API-REFERENCE.md) - Referencia de API
- [Examples](COMPONENTS/Monitor/EXAMPLES.md) - Ejemplos de uso
- [AI Agent Instructions](COMPONENTS/Monitor/AI-AGENT-INSTRUCTIONS.md) - Instrucciones para AI

**Características:**
- ✅ Monitoreo de recursos (CPU, RAM, Disco)
- ✅ Métricas del sistema
- ✅ Integración con Laravel
- ✅ Dashboard en tiempo real
- ✅ Alertas configurables

---

## 🚀 Uso de la Documentación

### Para Desarrolladores

1. **Extension Development**: Consulta [Extension Manager Developer Guide](CORE/Extension-Manager/DEVELOPER-GUIDE.md)
2. **Component Integration**: Revisa [Components README](COMPONENTS/README.md)
3. **Permissions Setup**: Consulta [User Management](CORE/User-Management/)

### Para Administradores

1. **Extension Management**: [Extension Manager README](CORE/Extension-Manager/README.md)
2. **System Monitoring**: [Monitor Component](COMPONENTS/Monitor/README.md)

---

## 📋 Convenciones de Documentación

Toda la documentación sigue estos estándares:

- **Formato**: Markdown (.md)
- **Estructura**: Títulos jerárquicos con emojis
- **Ejemplos**: Código con sintaxis highlight
- **Versionado**: Fecha de actualización y versión en cada documento
- **Referencias**: Links internos relativos

---

## 🔗 Proyecto Principal

Este repositorio es parte del proyecto **Bithoven CPANEL**:
- **Repositorio Principal**: [bithoven-cpanel](https://github.com/Madniatik/bithoven-cpanel)
- **Tecnologías**: Laravel 11, Metronic 8.3.2, Livewire 3.5
- **Licencia**: MIT

---

## 📝 Contribuciones

Esta documentación se mantiene sincronizada con el proyecto principal. Para contribuir:

1. Fork del repositorio
2. Crea una rama: `git checkout -b docs/nueva-documentacion`
3. Commit: `git commit -m 'docs: añadir documentación de X'`
4. Push: `git push origin docs/nueva-documentacion`
5. Abre un Pull Request

---

## 📞 Contacto

**Proyecto**: Bithoven CPANEL  
**Mantenedor**: Madniatik  
**Última Revisión**: 26 de noviembre de 2025

---

**🎉 Documentación completa y actualizada** - Consulta el índice para navegar por los diferentes módulos.
