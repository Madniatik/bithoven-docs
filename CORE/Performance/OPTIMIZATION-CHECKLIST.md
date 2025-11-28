# Performance Optimization Checklist

**Versión:** v1.8.0  
**Fecha:** 28 de noviembre de 2025

---

## 🎯 Quick Wins (0 esfuerzo, alto impacto)

### Producción Básica
- [ ] `php artisan optimize` ejecutado
- [ ] `php artisan route:cache` ejecutado
- [ ] `php artisan view:cache` ejecutado
- [ ] `php artisan config:cache` ejecutado
- [ ] `.env` → `APP_DEBUG=false`
- [ ] `.env` → `APP_ENV=production`

**Impacto:** +20% performance global sin cambiar código

---

## 🗄️ Cache Layer

### Application Cache
- [x] Extension Manager: Extensiones instaladas (24h)
- [x] Extension Manager: Marketplace GitHub (1h)
- [x] Notifications: Por usuario (5min)
- [x] Theme: Assets compilados (forever)
- [ ] Dashboard: Estadísticas (10min)
- [ ] Roles: Lista completa (1h)
- [ ] Permissions: Lista completa (1h)
- [ ] Access Reports: Stats por período (10min)
- [ ] User: Contador activos (10min)

### Framework Cache
- [ ] Route cache activo en producción
- [ ] View cache activo en producción
- [ ] Config cache activo en producción
- [ ] Autoload optimization (`composer dump-autoload -o`)

### Cache Driver
- [ ] Development: `CACHE_DRIVER=file`
- [ ] Production: `CACHE_DRIVER=redis` (recomendado)
- [ ] Redis configurado correctamente
- [ ] Redis password set (si aplica)

---

## 🔗 Eager Loading

### DataTables (CRÍTICO)
- [x] UsersDataTable: `with(['roles', 'permissions'])`
- [x] AccessReportsDataTable: `with('user')`
- [x] NotificationsDataTable: `with('user')`
- [ ] SecurityLogDataTable: `with('user')` (si aplica)
- [ ] PermissionsDataTable: `with('roles')` (si existe)
- [ ] RolesDataTable: `with('permissions')` (si existe)

### Controllers
- [ ] DashboardController: Eager load stats relations
- [ ] UserManagementController::show(): `with(['roles', 'permissions'])`
- [ ] AccessReportController: `with('user')` en queries
- [ ] NotificationController: `with('user')` en queries

### Livewire Components
- [x] NotificationDropdown: OK (cached)
- [ ] Otros components: Auditar

### API Endpoints
- [ ] `/api/users`: Eager loading
- [ ] `/api/notifications`: Eager loading
- [ ] `/api/extensions`: Eager loading (si existe)

---

## 📊 Query Optimization

### Indexes Database
- [ ] `users.email` indexed (unique)
- [ ] `users.is_active` indexed
- [ ] `notifications.user_id` indexed
- [ ] `notifications.read_at` indexed
- [ ] `access_reports.user_id` indexed
- [ ] `access_reports.accessed_at` indexed
- [ ] `security_logs.user_id` indexed
- [ ] `security_logs.created_at` indexed

### Query Efficiency
- [ ] No `SELECT *` en queries críticas
- [ ] `limit()` aplicado donde sea posible
- [ ] `chunk()` usado para datasets grandes
- [ ] Subqueries optimizadas

---

## 🖼️ Asset Optimization

### JavaScript
- [ ] Vite build en producción (`npm run build`)
- [ ] Minificación activa
- [ ] Tree-shaking configurado
- [ ] Code splitting aplicado (si aplica)

### CSS
- [ ] Metronic assets minificados
- [ ] CSS unused eliminado (PurgeCSS)
- [ ] Critical CSS inline (opcional)

### Images
- [ ] Imágenes optimizadas (WebP/AVIF)
- [ ] Lazy loading de imágenes
- [ ] Responsive images (`srcset`)

---

## 🔍 Monitoring & Debugging

### Development
- [ ] Laravel Debugbar instalado (`--dev`)
- [ ] Query logging activo en local
- [ ] Slow query detection (>100ms)
- [ ] N+1 detection enabled

### Production
- [ ] Laravel Telescope instalado (opcional)
- [ ] APM tool configurado (New Relic, Datadog)
- [ ] Error tracking (Sentry, Bugsnag)
- [ ] Performance baselines documentados

---

## 🧪 Testing

### Performance Tests
- [ ] Tests para N+1 prevention
- [ ] Load testing (Artillery, k6)
- [ ] Page load time targets defined
- [ ] API response time targets defined

### Automated Checks
- [ ] PHPStan analiza performance
- [ ] CI/CD valida cache keys
- [ ] Pre-commit hook verifica N+1

---

## 📈 Metrics & Goals

### Current Baselines (Pre-Optimization)
```
Dashboard: 500ms
User List: 200ms
Notifications: 100ms
Access Reports: 300ms
Extension Manager: 2000ms
```

### Target Goals (Post-Optimization)
```
Dashboard: < 50ms   (10x improvement)
User List: < 30ms   (6.6x improvement)
Notifications: < 10ms (10x improvement)
Access Reports: < 50ms (6x improvement)
Extension Manager: < 20ms (100x improvement)
```

### Success Criteria
- [ ] 90% de páginas cargan en < 100ms
- [ ] 0 N+1 queries en producción
- [ ] Cache hit rate > 80%
- [ ] Page load time < 200ms (total)

---

## 🚀 Deployment Checklist

### Pre-Deploy
- [ ] `composer install --optimize-autoloader --no-dev`
- [ ] `npm run build`
- [ ] Tests passing (113/113)
- [ ] PHPStan 0 errors

### Deploy
- [ ] `php artisan migrate --force`
- [ ] `php artisan optimize`
- [ ] `php artisan queue:restart` (si usa queues)
- [ ] Cache warming script ejecutado

### Post-Deploy
- [ ] Verificar cache hit rate
- [ ] Verificar tiempos de respuesta
- [ ] Verificar error logs
- [ ] Smoke tests en producción

---

## ⚠️ Common Pitfalls

### Cache
- [ ] ✅ NO usar `env()` fuera de config/*
- [ ] ✅ Invalidar cache cuando cambian datos
- [ ] ✅ TTL apropiado por tipo de dato
- [ ] ✅ Cache driver correcto (Redis en prod)

### Eager Loading
- [ ] ✅ NO over-eager loading (cargar todo)
- [ ] ✅ NO olvidar relaciones anidadas
- [ ] ✅ NO lazy load en loops

### Queries
- [ ] ✅ NO queries en loops (N+1)
- [ ] ✅ NO `SELECT *` sin necesidad
- [ ] ✅ NO missing indexes en columnas filtradas

---

## 📝 Next Steps

### Immediate (Hoy)
1. Ejecutar `php artisan optimize` en producción
2. Instalar Laravel Debugbar en development
3. Auditar 3 páginas principales con Debugbar

### Short-term (Esta Semana)
1. Implementar cache en Dashboard
2. Implementar cache en Roles/Permissions
3. Auditoría completa de N+1 queries
4. Agregar indexes faltantes

### Long-term (Próximo Sprint)
1. Implementar Redis en producción
2. Setup de monitoring (Telescope/APM)
3. Performance testing suite
4. Documentation update

---

## 🔗 Referencias

- **Cache Strategy:** `CACHE-STRATEGY.md`
- **Lazy Loading:** `LAZY-LOADING-GUIDE.md`
- **Performance README:** `README.md`
- **Laravel Optimization:** https://laravel.com/docs/11.x/optimization
