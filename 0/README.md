# Ricco Didier Rashuaman Sapallanay

# 🚌 GoPasajero - Sistema de Venta de Pasajes de Buses

Sistema integral de venta de pasajes de buses interprovinciales que conecta empresas de transporte con pasajeros, ofreciendo una plataforma moderna y eficiente para la gestión de viajes, rutas, buses y ventas.


---

## 🛠️ Tecnologías Utilizadas

### **Frontend**

#### **Next.js 14 (App Router)**
**¿Por qué?** Framework React de última generación que ofrece renderizado del lado del servidor (SSR), generación de sitios estáticos (SSG) y rutas de API en una sola solución.

**Ventajas:**
- Mejora significativa en SEO gracias al renderizado del servidor
- Enrutamiento basado en el sistema de archivos (file-based routing)
- Optimización automática de imágenes y fuentes
- Code splitting automático para cargas más rápidas
- App Router ofrece layouts anidados y carga de datos del servidor

#### **React 18**
**¿Por qué?** Librería líder para construir interfaces de usuario interactivas y reactivas.

**Ventajas:**
- Componentes reutilizables y modulares
- Virtual DOM para actualizaciones eficientes
- Hooks modernos (useState, useEffect, custom hooks)
- Ecosistema robusto y amplia comunidad

#### **TypeScript 5.x**
**¿Por qué?** Superconjunto de JavaScript que añade tipado estático.

**Ventajas:**
- Detecta errores en tiempo de desarrollo antes de ejecutar el código
- Autocompletado inteligente y mejor experiencia de desarrollo
- Refactoring más seguro y mantenible
- Documentación implícita del código a través de tipos
- Reduce bugs en producción significativamente

#### **Tailwind CSS 3.3**
**¿Por qué?** Framework CSS utility-first que acelera el desarrollo de interfaces.

**Ventajas:**
- No necesitas salir del HTML para estilizar componentes
- Sistema de diseño consistente con clases predefinidas
- Tamaño final del CSS optimizado (elimina estilos no utilizados)
- Responsive design simplificado con breakpoints intuitivos
- Personalización total mediante `tailwind.config.ts`

#### **Lucide React**
**¿Por qué?** Librería de iconos moderna, ligera y open-source.

**Ventajas:**
- Iconos vectoriales escalables sin pérdida de calidad
- Más de 1000 iconos consistentes en estilo
- Componentes React nativos (no SVG externos)
- Personalización fácil de tamaño, color y stroke

---

### **Backend**

#### **Node.js con Express.js**
**¿Por qué?** Runtime de JavaScript del lado del servidor con framework web minimalista.

**Ventajas:**
- JavaScript en todo el stack (frontend y backend)
- Alto rendimiento con I/O no bloqueante
- Express es simple, flexible y ampliamente adoptado
- Ecosistema npm inmenso con miles de paquetes

#### **TypeScript**
**¿Por qué?** Mismo beneficio que en frontend: seguridad de tipos.

**Ventajas:**
- APIs más seguras y predecibles
- Contratos claros entre servicios (interfaces)
- Mejor integración con ORMs como Prisma

#### **Prisma ORM 5.22**
**¿Por qué?** ORM de nueva generación para Node.js y TypeScript.

**Ventajas:**
- Schema declarativo y legible (`schema.prisma`)
- Migraciones automáticas de base de datos
- Cliente TypeScript generado con autocompletado total
- Soporte para múltiples bases de datos (PostgreSQL, MySQL, SQLite)
- Prisma Studio para inspección visual de datos
- Queries tipadas que previenen errores SQL

#### **PostgreSQL (Supabase)**
**¿Por qué?** Base de datos relacional robusta y open-source, hospedada en Supabase.

**Ventajas PostgreSQL:**
- ACID compliant (transacciones seguras)
- Soporte para JSON, arrays, y tipos avanzados
- Escalabilidad y rendimiento probado
- Extensiones poderosas (PostGIS para geolocalización)

**Ventajas Supabase:**
- Hosting gratuito con PostgreSQL managed
- Backups automáticos
- Autenticación integrada (aunque usamos JWT custom)
- Dashboard para gestión visual

#### **JWT (JSON Web Tokens)**
**¿Por qué?** Estándar para autenticación stateless.

**Ventajas:**
- No requiere almacenar sesiones en el servidor
- Escalable horizontalmente (sin sesiones compartidas)
- Incluye información del usuario (userId, role, companyId)
- Firma criptográfica previene falsificación
- Expira automáticamente para mayor seguridad

#### **bcryptjs**
**¿Por qué?** Librería para hasheo seguro de contraseñas.

**Ventajas:**
- Salt automático previene rainbow table attacks
- Computacionalmente costoso para prevenir fuerza bruta
- Estándar de la industria para almacenar contraseñas

#### **Zod 3.22**
**¿Por qué?** Validación de schemas TypeScript-first.

**Ventajas:**
- Validación de datos de entrada en APIs
- Previene inyecciones y datos malformados
- Mensajes de error personalizables
- Inferencia de tipos TypeScript automática

---

## 🏗️ Arquitectura del Sistema

### **Estructura de Carpetas**

```
GoPasajero/
├── backend/                    # API REST con Express + TypeScript
│   ├── prisma/                # Schema y migraciones de BD
│   ├── src/
│   │   ├── modules/           # Módulos por funcionalidad
│   │   │   ├── auth/          # Autenticación (login, register)
│   │   │   ├── empresa/       # Gestión de empresas
│   │   │   ├── tickets/       # Venta de pasajes
│   │   │   ├── search/        # Búsqueda de viajes
│   │   │   └── chat/          # Chatbot de soporte
│   │   ├── middlewares/       # Auth, error handling
│   │   └── libs/              # Database, logger, mailer
│   └── scripts/               # Scripts de inicialización
│
├── frontend/                   # Next.js 14 App Router
│   ├── app/
│   │   ├── (shared)/          # Componentes y estilos compartidos
│   │   ├── admin/             # Panel superadmin
│   │   ├── empresa/           # Panel operadores de empresas
│   │   ├── home/              # Landing page
│   │   ├── search/            # Búsqueda y resultados
│   │   ├── checkout/          # Proceso de compra
│   │   └── auth/              # Login y registro
│   └── public/                # Imágenes y assets estáticos
│
└── infra/                      # Docker para desarrollo
```

### **Modelos de Datos Principales**

#### **User**
Usuarios del sistema (pasajeros, operadores, admins)
- Relación con `Company` para operadores
- Roles: `user`, `operator`, `admin`, `superadmin`

#### **Company**
Empresas de buses (Cruz del Sur, Oltursa, etc.)
- Tiene múltiples operadores (usuarios tipo `operator`)
- Relación 1-N con buses, rutas, viajes

#### **Bus**
Vehículos de las empresas
- Capacidad, tipo (cama, semicama), placa
- Asientos configurables

#### **Terminal**
Paraderos y terminales de buses
- Ubicación (ciudad, dirección, coordenadas)

#### **Route**
Rutas entre terminales (ej: Lima → Arequipa)
- Precio base y duración estimada

#### **Trip**
Viajes programados en rutas específicas
- Fecha/hora de salida, bus asignado
- Asientos disponibles vs vendidos

#### **Ticket**
Pasajes comprados por usuarios
- Estado: pending, confirmed, cancelled
- Información del pasajero

---

## ✨ Características Principales

### **🔐 Sistema de Autenticación Multi-Rol**

```typescript
// 4 tipos de roles con permisos diferenciados:

user         → Compra pasajes, ve historial
operator     → Gestiona buses/viajes de SU empresa
admin        → Gestiona terminales, rutas, reportes globales
superadmin   → Control total, crea empresas y operadores
```

**¿Por qué esta arquitectura?**
- Separación de responsabilidades
- Operadores solo ven datos de su empresa (filtrado por `companyId`)
- Admins tienen vista global sin modificar empresas
- Superadmin es el único que crea nuevas empresas

### **🎨 Componentes UI Modernos**

#### **ConfirmDialog**
Reemplaza los `confirm()` nativos del navegador
- 4 tipos visuales: danger, warning, info, success
- Animaciones suaves (fade-in, scale-in, shake)
- Mensajes descriptivos personalizables

```tsx
const { showConfirm } = useConfirmDialog();

await showConfirm({
  type: 'danger',
  title: '¿Eliminar empresa?',
  message: 'Esta acción no se puede deshacer',
  confirmText: 'Sí, eliminar',
  cancelText: 'Cancelar'
});
```

#### **Toast Notifications**
Notificaciones no intrusivas para feedback
- Tipos: success, error, info, warning
- Auto-desaparición configurable
- Posición personalizable

### **🔍 Búsqueda Avanzada**
- Filtros por origen, destino, fecha
- Resultados en tiempo real
- Muestra disponibilidad de asientos
- Ordenamiento por precio, horario, empresa

### **💳 Checkout Completo**
- Selección de asientos visual
- Datos del pasajero (nombre, DNI, teléfono)
- Resumen de compra
- Estados de pago (integrable con pasarelas)

### **📊 Dashboards Diferenciados**

**Panel Operador (`/empresa`):**
- Gestión de buses de la empresa
- Programación de viajes
- Configuración de rutas propias
- Reportes de ventas por empresa
- Reclamos de usuarios

**Panel Admin (`/admin`):**
- Gestión de todas las empresas
- CRUD de terminales globales
- Auditoría del sistema
- Business Intelligence (BI)
- Reportes consolidados
- Gestión de integraciones

---

## 🚀 Instalación

### **Prerrequisitos**

- Node.js 18+ 
- PostgreSQL 14+ (o cuenta Supabase)
- npm o yarn

### **Configuración Backend**

```bash
cd backend
npm install

# Configurar variables de entorno
# Crear archivo .env con:
DATABASE_URL="postgresql://user:pass@host:5432/gopasajero"
JWT_SECRET="tu-secreto-super-seguro-aqui"
PORT=3001

# Ejecutar migraciones
npx prisma migrate dev

# Crear superadmin inicial
npm run create-superadmin

# Seed de empresas de prueba (opcional)
npx ts-node scripts/seed-companies.ts

# Iniciar servidor
npm run dev
```

### **Configuración Frontend**

```bash
cd frontend
npm install

# Configurar variables de entorno
# Crear archivo .env.local con:
NEXT_PUBLIC_API_URL=http://localhost:3001/api

# Iniciar aplicación
npm run dev
```

La aplicación estará disponible en:
- **Frontend:** http://localhost:3000
- **Backend API:** http://localhost:3001/api

---

## 📜 Scripts Disponibles

### **Backend**

```bash
npm run dev          # Desarrollo con ts-node-dev (hot reload)
npm run build        # Compilar TypeScript a JavaScript
npm run start        # Producción (requiere build previo)
npm run test         # Ejecutar tests con Jest

# Scripts especiales
npm run create-superadmin              # Crear cuenta superadmin
npx ts-node scripts/seed-companies.ts  # Seed de empresas
```

### **Frontend**

```bash
npm run dev          # Desarrollo (http://localhost:3000)
npm run build        # Build de producción optimizado
npm run start        # Servidor de producción
npm run lint         # ESLint para verificar código
```

### **Prisma**

```bash
npx prisma migrate dev       # Crear y aplicar migración
npx prisma generate          # Regenerar Prisma Client
npx prisma studio            # Abrir GUI para ver datos
npx prisma db push           # Sincronizar schema sin migración
```

---

## 👥 Roles y Permisos

| Rol | Acceso | Permisos |
|-----|--------|----------|
| **user** | `/home`, `/search`, `/checkout`, `/tickets` | Buscar viajes, comprar pasajes, ver historial |
| **operator** | `/empresa/*` | Gestionar buses, viajes y rutas de SU empresa únicamente |
| **admin** | `/admin/*` (excepto empresas) | Ver reportes globales, gestionar terminales, auditoría |
| **superadmin** | `/admin/*` (todo) | Control total, crear empresas, asignar operadores |

### **Protección de Rutas**

#### Frontend (Next.js Layout)
```tsx
// app/empresa/layout.tsx
const allowedRoles = ['operator'];
if (!allowedRoles.includes(userRole)) {
  redirect('/auth/login');
}
```

#### Backend (Middleware)
```typescript
// middlewares/auth.ts
export const requireRole = (roles: string[]) => {
  return (req, res, next) => {
    if (!roles.includes(req.user.role)) {
      return res.status(403).json({ error: 'No autorizado' });
    }
    next();
  };
};
```

---

## 📋 Credenciales de Prueba

### **Superadmin**
```
📧 Email:      admin@gopasajero.com
🔑 Contraseña: Admin@2025!
```

### **Operadores de Empresas**
```
Cruz del Sur:  cmendoza@cruzdelsur.com.pe  / CruzDelSur2024!
Oltursa:       metorres@oltursa.pe         / Oltursa2024!
Movil Tours:   rsilva@moviltours.com.pe    / MovilTours2024!
Civa:          aramirez@civa.com.pe        / Civa2024!
```

---

## 🎨 Sistema de Diseño

### **Paleta de Colores**

```css
/* Colores principales */
Primary (Indigo):   #4F46E5  /* Botones, enlaces, acentos */
Secondary (Purple): #7C3AED  /* Gradientes, highlights */
Accent (Pink):      #EC4899  /* Call-to-actions especiales */

/* Estados */
Success (Green):    #10B981  /* Confirmaciones, éxitos */
Warning (Yellow):   #F59E0B  /* Advertencias, atención */
Error (Red):        #EF4444  /* Errores, cancelaciones */
Info (Blue):        #3B82F6  /* Información general */

/* Neutros */
Gray Scale:         #F9FAFB → #111827  /* Fondos, textos, bordes */
```

### **Animaciones Personalizadas**

```css
@keyframes fade-in {
  from { opacity: 0; }
  to { opacity: 1; }
}

@keyframes scale-in {
  from { transform: scale(0.9); opacity: 0; }
  to { transform: scale(1); opacity: 1; }
}

@keyframes shake {
  0%, 100% { transform: translateX(0); }
  25% { transform: translateX(-10px); }
  75% { transform: translateX(10px); }
}
```

---

## 🔒 Seguridad Implementada

- ✅ Contraseñas hasheadas con bcrypt (salt rounds: 10)
- ✅ JWT con expiración (7 días por defecto)
- ✅ Validación de inputs con Zod en todas las APIs
- ✅ Middleware de autenticación en rutas protegidas
- ✅ CORS configurado para dominios permitidos
- ✅ SQL Injection prevenido (Prisma usa prepared statements)
- ✅ XSS prevenido (React escapa HTML por defecto)
- ✅ Roles verificados tanto en frontend como backend

---

## 📈 Mejoras Futuras Sugeridas

- [ ] Integración con pasarelas de pago (Culqi, MercadoPago)
- [ ] Sistema de notificaciones por email/SMS
- [ ] App móvil con React Native
- [ ] Chatbot con IA para soporte
- [ ] Sistema de puntos de fidelidad
- [ ] Mapas de rutas con Google Maps
- [ ] Reportes avanzados con gráficos (Chart.js)
- [ ] Modo offline con Service Workers
- [ ] Tests E2E con Playwright/Cypress
- [ ] CI/CD con GitHub Actions

---

## 📄 Licencia

Este proyecto es privado y confidencial.

---
