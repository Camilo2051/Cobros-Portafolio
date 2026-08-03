# Cobro - Sistema de Trazabilidad de Créditos

**Gestioné** el desarrollo completo de un sistema web de trazabilidad de créditos para cobradores, **automatizando** el cálculo de cuotas, pagos, mora y caja diaria, **reduciendo** el manejo manual de libros de cobranza en un 90% mediante una plataforma digital con roles diferenciados, multi-moneda y reportes en PDF.

## 🚀 Demo en vivo

**[https://cobros-sage.vercel.app](https://cobros-sage.vercel.app)**

### Credenciales de prueba
- **Demo**: `demo@cobro.com` / `demo-camilo2051`

> Nota: El backend puede tardar ~30s en responder la primera vez (Render free tier duerme tras inactividad).

## 📋 Tabla de contenidos

- [Descripción](#-descripción)
- [Tecnologías](#-tecnologías)
- [Funcionalidades](#-funcionalidades)
- [Modelo de datos](#-modelo-de-datos)
- [Arquitectura](#-arquitectura)
- [Seguridad](#-seguridad)
- [Testing y CI/CD](#-testing-y-cicd)
- [Despliegue](#-despliegue)
- [Capturas](#-capturas)

## 📖 Descripción

Sistema web completo para gestionar créditos de clientes con trazabilidad de pagos, caja diaria, control de mora y suscripción mensual. Diseñado para cobradores que necesitan llevar controle de sus préstamos desde el móvil.

### Roles de usuario
- **ADMIN**: Control absoluto del sistema, crea usuarios y gestiona suscripciones
- **ADMIN_COBRADOR**: Administra sus propios cobradores, con capital y moneda configurable
- **INDEPENDIENTE**: Trabaja por su cuenta sin cobradores subordinados
- **COBRADOR**: Gestiona sus clientes, créditos y pagos

## 🛠 Tecnologías

### Backend
- **NestJS** (TypeScript) - API REST modular
- **Prisma ORM 7** - PostgreSQL con adapter @prisma/adapter-pg
- **JWT + Passport** - Autenticación con refresh token (access 1h, refresh 7d)
- **@nestjs/throttler** - Rate limiting global
- **Helmet** - Headers de seguridad
- **Swagger** - Documentación API
- **PDFKit** - Generación de reportes PDF en memoria
- **@nestjs/schedule** - Cron jobs (zona horaria America/Bogota)
- **bcrypt** - Hash de contraseñas (salt rounds 10)

### Frontend
- **React 19 + Vite** (TypeScript)
- **Tailwind CSS v4** - Estilos responsive
- **React Router** - Navegación SPA
- **Recharts** - Gráficos del dashboard
- **Axios** - Cliente HTTP con interceptor JWT y refresh automático
- **Lucide Icons** - Iconografía
- **xlsx** - Exportación a Excel

### Base de datos y DevOps
- **PostgreSQL 16** (Neon serverless en producción)
- **Docker** - Dockerización completa para VPS
- **GitHub Actions** - CI/CD con tests automáticos
- **Render + Vercel + Neon** - Despliegue gratuito

## ✨ Funcionalidades

### Autenticación y seguridad
- Login con documento o email + contraseña
- JWT con doble token (access 1h + refresh 7d)
- Renovación automática de token (interceptor axios)
- Rate limiting global (1000/min) y en auth (5/min)
- Bloqueo de cuenta tras 5 intentos fallidos (5 min)
- Sanitización global de inputs (anti-XSS)
- CORS configurable y estricto en producción
- Validación de JWT_SECRET al arranque
- Helmet (X-Frame-Options, CSP, HSTS)

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
- Mora dinámica con días de atraso (sin recargo automático)
- Clientes en mora resaltados en rojo
- Cuota mínima recalculada según saldo pendiente
- Pago parcial no avanza cuota hasta completar
- Tabla de cuotas con estados (pagada/parcial/pendiente/cubierta)
- Ticket de venta imprimible
- Diálogo para crear crédito después de crear cliente
- Preguntar renovar crédito al completar pago total
- Filtros por periodicidad
- Exportar a Excel

### Pagos
- Pago sugerido automático (abono pendiente o cuota mínima)
- Botón "Pagar saldo total" con un clic
- Pagos parciales permitidos en el mismo día
- Pago total cierra crédito automáticamente
- Filtros por método de pago
- Búsqueda por cliente o ID de crédito
- Admin puede editar/eliminar pagos (revierte crédito y caja)
- Prevención de envío duplicado en formularios

### Caja diaria
- Cobrador debe abrir caja antes de crear créditos o pagos
- Pagos y préstamos se registran automáticamente
- Movimientos manuales (entradas/salidas)
- Cierre automático a medianoche (cron America/Bogota)
- Admin puede ver, editar y eliminar movimientos
- Capital del admin cobrador se actualiza al abrir/cerrar
- Historial con exportación a Excel

### Dashboard
- Filtros por periodo (hoy, 7 días, 30 días, todo)
- Tarjetas: clientes, créditos activos, saldo pendiente, recaudo
- Gráfico de línea: recaudo por día
- Gráfico de torta: métodos de pago
- Gráfico de barras: top 5 deudores
- Gráfico de evolución del saldo
- Lista: últimos 5 pagos
- Multi-moneda (COP, USD, BRL)

### Reportes PDF
- Estado de cuenta con título personalizado (nombre del cliente)
- Reporte de morosidad con días de atraso y cobrador
- Generados en memoria con PDFKit

### Multi-moneda
- COP (Peso Colombiano), USD (Dólar), BRL (Real Brasileño)
- formatCurrency adaptativo según moneda del usuario
- Capital configurable por admin cobrador

### Auditoría
- Registro automático de cambios en clientes, créditos, pagos y cajas
-Usuario responsable en cada cambio
- Página dedicada (solo admin)

### Perfil de usuario
- Editar nombre
- Cambiar contraseña (valida actual)
- Configurar capital y moneda (setup inicial)

### UI/UX
- Modo responsive completo (sidebar colapsable, cards en móvil)
- Skeleton loaders
- Diálogos de confirmación
- Notificaciones toast
- Búsqueda con resaltado de coincidencias
- Título de pestaña dinámico

## 📊 Modelo de datos

```prisma
model Usuario {
  id               String         @id @default(uuid())
  nombre           String
  documento        String         @unique
  email            String         @unique
  password         String
  rol              RolUsuario     @default(COBRADOR)
  estado           EstadoUsuario  @default(INACTIVO)
  capital          Float?
  moneda           Moneda?
  adminCobradorId  String?
  intentosFallidos Int            @default(0)
  bloqueadoHasta   DateTime?
  // ...relaciones
}

model Cliente {
  id         String     @id @default(uuid())
  documento  String
  nombre     String
  apellido   String
  telefono   String?
  email      String?
  direccion  String?
  cobradorId String?
  // ...
  @@unique([cobradorId, documento])
}

model Credito {
  id            String          @id @default(uuid())
  monto         Float
  interes       Float
  plazo         Int
  periodicidad  Periodicidad    @default(MENSUAL)
  cuota         Float
  saldo         Float
  estado        EstadoCredito   @default(APROBADO)
  fechaInicio   DateTime        @default(now())
  fechaVence    DateTime
  // ...
}

model Pago {
  id           String      @id @default(uuid())
  creditoId    String
  cuotaNumero  Int
  monto        Float
  metodo       MetodoPago
  estado       EstadoPago  @default(PAGADO)
  fechaPago    DateTime    @default(now())
  // ...
}

model Caja {
  id           String          @id @default(uuid())
  cobradorId   String
  montoInicial Float
  montoFinal   Float?
  estado       EstadoCaja      @default(ABIERTA)
  fecha        DateTime        @default(now())
  cerradaEn    DateTime?
  // ...
}

model MovimientoCaja {
  id          String          @id @default(uuid())
  cajaId      String
  tipo        TipoMovimiento
  monto       Float
  descripcion String?
  pagoId      String?
  // ...
}

model Auditoria {
  id          String   @id @default(uuid())
  entidad     String
  entidadId   String
  accion      String
  descripcion String
  usuarioId   String?
  // ...
}
```

## 🏗 Arquitectura

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
├── components/       # UI, tabla-cuotas, pagination, skeleton
├── hooks/            # useCreditos, usePagos, useCaja, useDashboard
├── context/          # Auth y toast
├── services/         # Axios con interceptor JWT
└── lib/              # finance.ts, utils.ts, export.ts
```

## 🔒 Seguridad

- JWT con refresh token rotativo
- Rate limiting global + auth (5/min)
- Bloqueo de cuenta tras 5 intentos fallidos
- Sanitización anti-XSS global
- Helmet (headers de seguridad)
- CORS estricto en producción
- Validación JWT_SECRET al arranque
- Errores 500 con mensaje genérico
- Swagger deshabilitado en producción
- trust proxy para IP real del cliente
- Política de contraseñas (min 8, mayúscula, minúscula, número)

## 🧪 Testing y CI/CD

### Backend (92 tests con Jest)
- Unitarios: creditos, pagos, caja, guards
- Seguridad: SQL injection, JWT, authorization, input validation
- E2E: flujo completo de crédito y pago

### Frontend (50 tests con Vitest, 97% coverage)
- Utilidades: formatCurrency, formatDate, cn
- Componentes: Button, Input, Badge, Dialog, Card
- Export: exportToExcel
- Login: renderizado, inputs, éxito/fallo
- ConfirmDialog y Pagination

### CI/CD (GitHub Actions)
- Tests backend (Jest + PostgreSQL service)
- Tests frontend (Vitest)
- Build backend y frontend
- Se ejecuta en cada push/PR a main

## 🚀 Despliegue

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

## 📸 Capturas

> Las capturas se agregarán próximamente en la carpeta `/screenshots/`

- Login
- Dashboard
- Lista de clientes (agrupados por cobrador)
- Lista de créditos (con mora)
- Detalle de crédito con ticket de venta
- Caja diaria
- Página de cobradores

## 📄 Licencia

Copyright (c) 2026 Camilo Garzón. Todos los derechos reservados.

Uso educativo y de portafolio únicamente. Prohibido uso comercial, copia o distribución sin autorización.

Para licencia comercial: camiloardila2573@gmail.com
