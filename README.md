# Cobro - Sistema de Trazabilidad de Créditos

**Gestioné** el desarrollo completo de un sistema web de trazabilidad de créditos para cobradores, **automatizando** el cálculo de cuotas, pagos, mora y caja diaria, **reduciendo** el manejo manual de libros de cobranza en un 90% mediante una plataforma digital con roles jerárquicos, multi-moneda, multilenguaje y reportes en PDF.

## Demo en vivo

**[https://cobros-sage.vercel.app](https://cobros-sage.vercel.app)**

> El backend puede tardar ~30s en responder la primera vez (Render free tier duerme tras inactividad).

## Tabla de contenidos

- [Descripcion](#descripcion)
- [Tecnologias](#tecnologias)
- [Funcionalidades](#funcionalidades)
- [Modelo de datos](#modelo-de-datos)
- [Arquitectura](#arquitectura)
- [Seguridad](#seguridad)
- [Testing y CI/CD](#testing-y-cicd)
- [Despliegue](#despliegue)

## Descripcion

Sistema web completo para gestionar creditos de clientes con trazabilidad de pagos, caja diaria, control de mora y suscripcion mensual. Diseñado para cobradores que necesitan llevar control de sus prestamos desde el movil.

### Roles de usuario
- **ADMIN**: Control absoluto del sistema, crea usuarios y gestiona suscripciones
- **ADMIN_COBRADOR**: Administra sus propios cobradores, con capital y moneda configurable
- **INDEPENDIENTE**: Trabaja por su cuenta sin cobradores subordinados
- **COBRADOR**: Gestiona sus clientes, créditos y pagos

## Tecnologias

### Backend
- **NestJS** (TypeScript) - API REST modular
- **Prisma ORM 7** - PostgreSQL con adapter @prisma/adapter-pg
- **JWT + Passport** - Autenticación con refresh token (access 1h, refresh 7d)
- **@nestjs/throttler** - Rate limiting en autenticación
- **Helmet** - Headers de seguridad HTTP
- **Swagger** - Documentación API (solo en desarrollo)
- **PDFKit** - Generación de reportes PDF con diseño profesional
- **@nestjs/schedule** - Cron jobs (zona horaria America/Bogota)
- **bcrypt** - Hash de contraseñas

### Frontend
- **React 19 + Vite** (TypeScript)
- **Tailwind CSS v4** - Estilos responsive con dark mode
- **React Router** - Navegación SPA
- **Recharts** - Gráficos del dashboard
- **Axios** - Cliente HTTP con interceptor JWT y refresh automático
- **Lucide Icons** - Iconografía
- **xlsx** - Exportación a Excel
- **i18n** - Sistema de traducciones propio (español, portugués, inglés)

### Base de datos y DevOps
- **PostgreSQL 16** (Neon serverless en producción)
- **Docker** - Dockerización completa para VPS
- **GitHub Actions** - CI/CD con tests automáticos
- **Render + Vercel + Neon** - Despliegue gratuito

## Funcionalidades

### Autenticación y seguridad
- Login con documento o email + contraseña
- JWT con doble token (access 1h + refresh 7d)
- Renovación automática de token (interceptor axios)
- Rate limiting en login (5/min), register (5/min) y refresh (30/min)
- Bloqueo de cuenta tras 5 intentos fallidos (5 min)
- Sanitización global de inputs (anti-XSS)
- CORS estricto en producción
- Validación de JWT_SECRET al arranque
- Helmet (X-Frame-Options, CSP, HSTS)
- Política de contraseñas robusta (min 8, mayúscula, minúscula, número)
- Errores 500 con mensaje genérico
- Swagger deshabilitado en producción
- trust proxy para IP real del cliente

### Clientes
- CRUD completo con búsqueda y resaltado
- Restricción única: un cobrador no duplica documento
- Clientes compartibles entre cobradores
- Capitalización automática de nombres
- Agrupación por cobrador (admin)
- Diálogo de confirmación antes de crear/editar/eliminar
- Exportar a Excel
- Paginación

### Créditos
- Cálculo con interés simple: `total = monto * (1 + interés/100)`
- Periodicidades: DIARIO, SEMANAL, QUINCENAL, MENSUAL
- Créditos diarios excluyen domingos (días hábiles)
- Mora dinámica con días de atraso
- Clientes en mora resaltados en rojo
- Cuota mínima recalculada según saldo pendiente
- Pago parcial no avanza cuota hasta completar
- Tabla de cuotas con estados (pagada/parcial/pendiente/cubierta)
- Ticket de venta imprimible
- Diálogo para crear crédito después de crear cliente
- Filtros por periodicidad
- Exportar a Excel
- Layout responsive (cards en móvil, tabla en desktop)

### Pagos
- Pago sugerido automático (abono pendiente o cuota mínima)
- Botón "Pagar saldo total" con un clic
- Pagos parciales permitidos en el mismo día
- Pago total cierra crédito automáticamente
- Filtros por método de pago y fecha
- Búsqueda por cliente o ID de crédito
- Admin puede editar/eliminar pagos (revierte crédito y caja)
- Prevención de envío duplicado en formularios
- Layout responsive (cards en móvil)

### Caja diaria
- Cobrador debe abrir caja antes de crear créditos o pagos
- Pagos y préstamos se registran automáticamente
- Movimientos manuales (entradas/salidas)
- Cierre automático a medianoche (cron America/Bogota)
- Capital del admin cobrador se actualiza al abrir/cerrar
- Admin puede ver, editar y eliminar movimientos
- Eliminar movimiento revierte crédito/pago
- Historial con filtros por fecha (hoy, semana, fecha específica)
- Exportar a Excel

### Dashboard
- Filtros por periodo (hoy, 7 días, 30 días, todo)
- Tarjetas: capital, saldo pendiente, recaudado, clientes, créditos activos
- Resumen financiero: total prestado, recaudado, créditos pagados
- Gráfico de línea: recaudo por día
- Gráfico de torta: métodos de pago
- Gráfico de barras: top 5 deudores
- Gráfico de evolución del saldo
- Lista: últimos pagos
- Multi-moneda (COP, USD, BRL)

### Reportes PDF
- Estado de cuenta con diseño profesional (header morado, tarjetas, tablas con colores)
- Reporte de morosidad con días de atraso y cobrador
- Generados en memoria con PDFKit

### Multi-moneda
- COP (Peso Colombiano), USD (Dólar), BRL (Real Brasileño)
- formatCurrency adaptativo según moneda del usuario
- Capital configurable por admin cobrador
- Soporte de decimales para BRL/USD

### Multilenguaje (i18n)
- Español (por defecto), Portugués, Inglés
- Selector en la página de perfil
- Persistencia en localStorage
- 180+ claves de traducción

### Modo oscuro
- Toggle en el sidebar
- Detecta preferencia del sistema operativo
- Persistencia en localStorage
- Variables CSS para ambos temas

### Notificaciones in-app
- Campana en el sidebar con badge de no leídas
- Alertas automáticas: créditos en mora, vence pronto (3 días), caja sin abrir, pagos hoy
- Panel desplegable con iconos por tipo
- Navegación al hacer clic
- Persistencia de leídas en localStorage

### Auditoría
- Registro automático de cambios en clientes, créditos, pagos y cajas
- Usuario responsable en cada cambio
- Página dedicada (solo admin)
- Layout responsive

### UI/UX
- Modo responsive completo (sidebar colapsable, cards en móvil)
- Skeleton loaders
- Diálogos de confirmación
- Notificaciones toast
- Búsqueda con resaltado de coincidencias
- Título de pestaña dinámico
- Prevención de envío duplicado en formularios

## Modelo de datos

```prisma
model Usuario {
  id, nombre, documento, email, password, rol, estado,
  capital, moneda, adminCobradorId, intentosFallidos, bloqueadoHasta
}

model Cliente { id, documento, nombre, apellido, telefono, email, direccion, cobradorId }
  @@unique([cobradorId, documento])

model Credito { id, monto, interes, plazo, periodicidad, cuota, saldo, estado, fechaInicio, fechaVence }

model Pago { id, creditoId, cuotaNumero, monto, metodo, estado, fechaPago }

model Caja { id, cobradorId, montoInicial, montoFinal, estado, fecha, cerradaEn }

model MovimientoCaja { id, cajaId, tipo, monto, descripcion, pagoId }

model Auditoria { id, entidad, entidadId, accion, descripcion, usuarioId }
```

## Arquitectura

### Backend (NestJS)
```
src/
├── auth/             # JWT, guards, refresh token, rate limiting
├── caja/             # Caja diaria, movimientos, cron cierre
├── common/           # Filters, pipes, interceptors, helpers
├── prisma/           # PrismaService con adapter PostgreSQL
├── usuarios/         # CRUD admin + PerfilService
├── suscripciones/    # Planes y renovación
├── clientes/         # CRUD con filtros por rol
├── creditos/         # CRUD, interés simple, mora.helper
├── pagos/            # CRUD transaccional, reversión
├── reportes/         # PDF con PDFKit
├── cron/             # Cron jobs America/Bogota
└── auditoria/        # Registro de cambios
```

### Frontend (React + Vite)
```
frontend/src/
├── pages/            # 11 páginas (login, dashboard, clientes, etc.)
├── components/       # UI, tabla-cuotas, pagination, skeleton, creditos/
├── hooks/            # useCreditos, usePagos, useCaja, useDashboard
├── context/          # Auth, Toast, Theme, Language, Notificaciones
├── services/         # Axios con interceptor JWT
└── lib/              # finance.ts, utils.ts, export.ts, translations.ts
```

## Seguridad

- JWT con refresh token rotativo
- Rate limiting en auth (5/min login, 5/min register, 30/min refresh)
- Bloqueo de cuenta tras 5 intentos fallidos
- Sanitización anti-XSS global (SanitizePipe)
- Helmet (headers de seguridad HTTP)
- CORS estricto en producción
- Validación de JWT_SECRET al arranque (min 20 chars en producción)
- Errores 500 con mensaje genérico
- Swagger deshabilitado en producción
- trust proxy para IP real del cliente
- Política de contraseñas robusta (min 8, mayúscula, minúscula, número)

## Testing y CI/CD

### Backend (92 tests con Jest)
- Unitarios: creditos, pagos, caja, mora, guards
- Seguridad: SQL injection, JWT, authorization, input validation
- E2E: flujo completo de crédito y pago

### Frontend (49 tests con Vitest, 97% coverage)
- Utilidades: formatCurrency, formatDate, cn
- Componentes: Button, Input, Badge, Dialog, Card
- Export, Login, ConfirmDialog, Pagination, TablaCuotas

### CI/CD (GitHub Actions)
- Tests backend (Jest + PostgreSQL service)
- Tests frontend (Vitest)
- Build backend y frontend
- Se ejecuta en cada push/PR a main

## Despliegue

### Producción actual (gratuito)
- **BD**: Neon (PostgreSQL serverless)
- **Backend**: Render (Web Service)
- **Frontend**: Vercel
- **Monitoreo**: UptimeRobot (keep-alive)

### Docker (VPS)
```bash
cp .env.production.example .env.production
docker compose -f docker-compose.prod.yml up -d --build
```

## Capturas

### Login
![Login](https://raw.githubusercontent.com/Camilo2051/Cobros-Portafolio/main/screenshots/login.png)

### Dashboard
![Dashboard](https://raw.githubusercontent.com/Camilo2051/Cobros-Portafolio/main/screenshots/dashboard.png)

### Clientes
![Clientes](https://raw.githubusercontent.com/Camilo2051/Cobros-Portafolio/main/screenshots/clientes.png)

### Creditos
![Creditos](https://raw.githubusercontent.com/Camilo2051/Cobros-Portafolio/main/screenshots/creditos.png)

### Detalle de Credito
![Detalle de Credito](https://raw.githubusercontent.com/Camilo2051/Cobros-Portafolio/main/screenshots/detalles-credito.png)

### Pagos
![Pagos](https://raw.githubusercontent.com/Camilo2051/Cobros-Portafolio/main/screenshots/pagos.png)

## Licencia

Copyright (c) 2026 Camilo Garzón. Todos los derechos reservados.

Uso educativo y de portafolio únicamente. Prohibido uso comercial, copia o distribución sin autorización.


Uso educativo y de portafolio únicamente. Prohibido uso comercial, copia o distribución sin autorización.

Para licencia comercial: camiloardila2573@gmail.com
