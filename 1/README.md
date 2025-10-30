
# GoPasajero — Análisis de Requerimientos (SRS Breve)
> Proyecto: Plataforma web de gestión de reservas y venta de pasajes interprovinciales
> 
---

## 1) Catálogo de Requerimientos Funcionales

| Requerimiento | Nombre                                                                 |
|---------------|------------------------------------------------------------------------|
| R-101         | Registro de usuario                                                    |
| R-102         | Generación de token y envío de correo de verificación                  |
| R-103         | Verificación de cuenta (activar usuario)                               |
| R-104         | Reenvío de correo de verificación                                      |
| R-105         | Inicio/cierre de sesión (solo usuarios activos)                        |
| R-106         | Recuperación de contraseña                                             |
| R-107         | Gestión de perfil de usuario                                           |
| R-108         | Roles y permisos (admin / usuario)                                     |
| R-109         | CRUD de empresas de transporte                                         |
| R-110         | CRUD de terminales                                                     |
| R-111         | CRUD de rutas (origen–destino–distancia/tiempo)                        |
| R-112         | CRUD de buses (placa única, modelo, seat-map)                          |
| R-113         | CRUD de viajes (ruta, bus, fecha/hora, precio, estado)                 |
| R-114         | Búsqueda de viajes por origen–destino–fecha                            |
| R-115         | Listado y filtros (empresa, precio, horario)                           |
| R-116         | Visualización de mapa de asientos (libre/ocupado/seleccionado)         |
| R-117         | Bloqueo temporal de asiento durante checkout                           |
| R-118         | Reserva y compra simulada (estados: reservado/confirmado/cancelado)    |
| R-119         | Generación de ticket/comprobante (código único, HTML/PDF)              |
| R-120         | Mis tickets (historial con filtros)                                    |
| R-121         | Panel administrador: publicar/pausar viajes                            |
| R-122         | Reportes: ventas por fecha/ruta/empresa                                |
| R-123         | Exportación de reportes (CSV/PDF)                                      |
| R-124         | Auditoría de acciones (login, verificación, reservas, CRUD)            |
| R-125         | Notificaciones (toast/email simulado)                                   |

---

## 2) Catálogo de Requerimientos No Funcionales

| Requerimiento | Nombre                                                                 |
|---------------|------------------------------------------------------------------------|
| RNF-201       | Hash de contraseñas (bcrypt ≥ 10)                                      |
| RNF-202       | Autenticación con JWT (expiración ≤ 24h)                               |
| RNF-203       | Verificación de correo con token único (TTL 15–30 min)                 |
| RNF-204       | Rate limiting en registro y reenvío de verificación                    |
| RNF-205       | Validación anti XSS/SQLi, CORS controlado                              |
| RNF-206       | Búsqueda p95 ≤ 300 ms en dev local (datos semilla)                     |
| RNF-207       | Confirmación p95 ≤ 800 ms (lock + persistencia)                        |
| RNF-208       | Paginación e índices (origen, destino, fecha/hora)                     |
| RNF-209       | Unicidad transaccional (viaje, asiento)                                |
| RNF-210       | Lock de asiento con TTL 2–5 min + limpieza automática                  |
| RNF-211       | Idempotencia en confirmación (idempotency-key)                         |
| RNF-212       | Arquitectura por capas (Controller–Service–Repository) + DTOs          |
| RNF-213       | Respuesta estándar `{ok,data,error{code,message}}`                     |
| RNF-214       | Lint/format (ESLint/Prettier) + convención de commits                  |
| RNF-215       | Logging en JSON con correlationId por request                          |
| RNF-216       | Endpoint `/health` + métricas (búsquedas, conversión, cancelación)     |
| RNF-217       | Auditoría inmutable (INSERT-only, timestamp, meta JSON)                |
| RNF-218       | Docker Compose (DB+API) + `.env.example`                               |
| RNF-219       | Migraciones versionadas + seeds reproducibles                          |
| RNF-220       | Responsive (mobile-first), contraste AA, foco visible                  |
| RNF-221       | Estados de carga/empty/error y validación inline                       |
| RNF-222       | API versionada (`/api/v1`)                                             |
| RNF-223       | Adaptadores intercambiables (pagos, notificaciones, PDF)               |
| RNF-224       | Preparado para i18n/multimoneda (PEN por defecto)                      |

---

## 4) Casos de Uso 

> Formato: **ID, Actor(es), Descripción, Precondiciones, Flujo Principal, Requerimientos Especiales, Frecuencia de Uso**

### **Caso de uso #1: Registro de usuario**
| **ID** | R-101 |
|---|---|
| **Actor(es)** | Usuario (pasajero) |
| **Descripción** | Registrar una cuenta con DNI, nombre, correo, teléfono y contraseña. |
| **Precondiciones** | Usuario no registrado; email y DNI no existen. |
| **Flujo Principal** | 1) Usuario abre “Crear cuenta”.<br>2) Completa formulario.<br>3) El sistema valida y crea usuario con estado `pending`.<br>4) Genera token y dispara envío de verificación (ver R-102). |
| **Requerimientos Especiales** | Validación de formato DNI/email; unicidad de email y DNI. |
| **Frecuencia de Uso** | Alto (primera interacción). |

### **Caso de uso #2: Generación y envío de verificación**
| **ID** | R-102 |
|---|---|
| **Actor(es)** | Sistema de correo (SMTP), Usuario |
| **Descripción** | Enviar email con enlace de verificación (`/verify?token=...`). |
| **Precondiciones** | Usuario en estado `pending`; token generado. |
| **Flujo Principal** | 1) Tras registro, se genera token (TTL 15–30 min).<br>2) Se envía correo con enlace.<br>3) Usuario recibe el correo. |
| **Requerimientos Especiales** | Token único, expiración (RNF-203), rate limit (RNF-204). |
| **Frecuencia de Uso** | Alto (en nuevos registros y reenvíos). |

### **Caso de uso #3: Verificación de cuenta**
| **ID** | R-103 |
|---|---|
| **Actor(es)** | Usuario (vía enlace de correo) |
| **Descripción** | Activar la cuenta al visitar el enlace con token válido. |
| **Precondiciones** | Token válido y no expirado; usuario `pending`. |
| **Flujo Principal** | 1) Usuario abre enlace.<br>2) Sistema valida token.<br>3) Cambia estado a `active` y borra token.<br>4) Muestra pantalla de éxito. |
| **Requerimientos Especiales** | Seguridad de token; feedback claro si expiró/invalidado. |
| **Frecuencia de Uso** | Por registro. |

### **Caso de uso #4: Reenvío de verificación**
| **ID** | R-104 |
|---|---|
| **Actor(es)** | Usuario |
| **Descripción** | Solicitar un nuevo correo de verificación. |
| **Precondiciones** | Usuario `pending`. |
| **Flujo Principal** | 1) Usuario hace clic en “Reenviar”.<br>2) Sistema invalida token anterior.<br>3) Genera nuevo token y envía correo. |
| **Requerimientos Especiales** | Rate limit para evitar abuso (RNF-204). |
| **Frecuencia de Uso** | Ocasional. |

### **Caso de uso #5: Inicio/cierre de sesión**
| **ID** | R-105 |
|---|---|
| **Actor(es)** | Usuario |
| **Descripción** | Iniciar sesión con email/contraseña (solo `active`) y cerrar sesión. |
| **Precondiciones** | Usuario `active`; credenciales válidas. |
| **Flujo Principal** | 1) Usuario ingresa email/contraseña.<br>2) Sistema valida y emite JWT.<br>3) Usuario accede a funciones protegidas.<br>4) Cierre de sesión invalida token en cliente. |
| **Requerimientos Especiales** | JWT con expiración (RNF-202). |
| **Frecuencia de Uso** | Diario. |

### **Caso de uso #6: Recuperación de contraseña**
| **ID** | R-106 |
|---|---|
| **Actor(es)** | Usuario, Sistema de correo |
| **Descripción** | Restablecer contraseña mediante correo/token temporal. |
| **Precondiciones** | Usuario registrado. |
| **Flujo Principal** | 1) Usuario solicita “Olvidé mi contraseña”.<br>2) Se envía enlace con token.<br>3) Usuario define nueva contraseña. |
| **Requerimientos Especiales** | Token con TTL; reglas de contraseña. |
| **Frecuencia de Uso** | Ocasional. |

### **Caso de uso #7: Gestión de perfil**
| **ID** | R-107 |
|---|---|
| **Actor(es)** | Usuario |
| **Descripción** | Editar datos básicos del perfil. |
| **Precondiciones** | Usuario autenticado. |
| **Flujo Principal** | 1) Abre “Mi perfil”.<br>2) Edita campos.<br>3) Sistema valida y guarda. |
| **Requerimientos Especiales** | Historial mínimo de cambios (opcional). |
| **Frecuencia de Uso** | Bajo/medio. |

### **Caso de uso #8: Roles y permisos**
| **ID** | R-108 |
|---|---|
| **Actor(es)** | Administrador |
| **Descripción** | Gestionar acceso por rol a módulos (admin/usuario). |
| **Precondiciones** | Admin autenticado. |
| **Flujo Principal** | 1) Admin asigna rol.<br>2) Sistema aplica middleware de autorización. |
| **Requerimientos Especiales** | Matriz de permisos documentada. |
| **Frecuencia de Uso** | Bajo. |

### **Caso de uso #9: CRUD de empresas**
| **ID** | R-109 |
|---|---|
| **Actor(es)** | Administrador |
| **Descripción** | Alta/baja/modificación/consulta de empresas. |
| **Precondiciones** | Admin autenticado. |
| **Flujo Principal** | 1) Crear/editar empresa (RUC, nombre, etc.).<br>2) Guardar y auditar. |
| **Requerimientos Especiales** | RUC único; auditoría (R-124). |
| **Frecuencia de Uso** | Medio. |

### **Caso de uso #10: CRUD de terminales**
| **ID** | R-110 |
|---|---|
| **Actor(es)** | Administrador |
| **Descripción** | Gestionar terminales (nombre, ciudad). |
| **Precondiciones** | Empresa existente. |
| **Flujo Principal** | 1) Crear/editar terminal.<br>2) Asociar empresa.<br>3) Guardar. |
| **Requerimientos Especiales** | Integridad referencial. |
| **Frecuencia de Uso** | Medio. |

### **Caso de uso #11: CRUD de rutas**
| **ID** | R-111 |
|---|---|
| **Actor(es)** | Administrador |
| **Descripción** | Definir rutas (origen–destino–distancia/tiempo). |
| **Precondiciones** | Terminales existentes. |
| **Flujo Principal** | 1) Crear ruta.<br>2) Asignar terminales origen/destino.<br>3) Guardar. |
| **Requerimientos Especiales** | Índices en (origen,destino). |
| **Frecuencia de Uso** | Medio. |

### **Caso de uso #12: CRUD de buses**
| **ID** | R-112 |
|---|---|
| **Actor(es)** | Administrador |
| **Descripción** | Registrar buses (placa, modelo, seat-map). |
| **Precondiciones** | Empresa existente. |
| **Flujo Principal** | 1) Crear bus (placa única).<br>2) Cargar seat-map.<br>3) Guardar. |
| **Requerimientos Especiales** | Unicidad de placa. |
| **Frecuencia de Uso** | Medio. |

### **Caso de uso #13: CRUD de viajes**
| **ID** | R-113 |
|---|---|
| **Actor(es)** | Administrador |
| **Descripción** | Publicar/editar viajes con precios y estado. |
| **Precondiciones** | Ruta y bus existentes. |
| **Flujo Principal** | 1) Definir horario, precio base, estado.<br>2) Guardar/publicar/pausar. |
| **Requerimientos Especiales** | Validación de solapes por bus/hora. |
| **Frecuencia de Uso** | Diario. |

### **Caso de uso #14: Búsqueda de viajes**
| **ID** | R-114 |
|---|---|
| **Actor(es)** | Usuario |
| **Descripción** | Buscar viajes por origen–destino–fecha. |
| **Precondiciones** | Viajes publicados. |
| **Flujo Principal** | 1) Usuario ingresa filtros.<br>2) Sistema consulta y pagina resultados. |
| **Requerimientos Especiales** | p95 ≤ 300 ms (RNF-206), índices (RNF-208). |
| **Frecuencia de Uso** | Alta. |

### **Caso de uso #15: Listado y filtros**
| **ID** | R-115 |
|---|---|
| **Actor(es)** | Usuario |
| **Descripción** | Ver resultados con empresa, precio y horario; ordenar/filtrar. |
| **Precondiciones** | R-114 completado. |
| **Flujo Principal** | 1) Mostrar tarjetas.<br>2) Filtrar/ordenar.<br>3) Seleccionar viaje. |
| **Requerimientos Especiales** | UI responsive (RNF-220). |
| **Frecuencia de Uso** | Alta. |

### **Caso de uso #16: Mapa de asientos**
| **ID** | R-116 |
|---|---|
| **Actor(es)** | Usuario |
| **Descripción** | Visualizar asientos disponibles/ocupados/seleccionados. |
| **Precondiciones** | Viaje seleccionado; seat-map cargado. |
| **Flujo Principal** | 1) Renderizar seat-map.<br>2) Marcar ocupados.<br>3) Usuario elige asiento. |
| **Requerimientos Especiales** | Accesibilidad AA; estados de feedback (RNF-221). |
| **Frecuencia de Uso** | Alta. |

### **Caso de uso #17: Bloqueo de asiento**
| **ID** | R-117 |
|---|---|
| **Actor(es)** | Usuario, Sistema |
| **Descripción** | Bloquear asiento temporalmente durante el checkout. |
| **Precondiciones** | R-116 completado. |
| **Flujo Principal** | 1) Usuario procede a pagar.<br>2) Sistema crea lock (TTL 2–5 min).<br>3) Si expira, libera asiento. |
| **Requerimientos Especiales** | Job de limpieza; consistencia transaccional. |
| **Frecuencia de Uso** | Alta. |

### **Caso de uso #18: Reserva/compra simulada**
| **ID** | R-118 |
|---|---|
| **Actor(es)** | Usuario |
| **Descripción** | Completar compra simulada cambiando estado del ticket. |
| **Precondiciones** | Asiento bloqueado; usuario autenticado. |
| **Flujo Principal** | 1) Confirmar datos de pasajero.<br>2) Sistema verifica lock y disponibilidad.<br>3) Registra ticket `confirmado`.<br>4) Libera lock. |
| **Requerimientos Especiales** | Idempotencia (RNF-211); p95 ≤ 800 ms (RNF-207). |
| **Frecuencia de Uso** | Alta. |

### **Caso de uso #19: Ticket/comprobante**
| **ID** | R-119 |
|---|---|
| **Actor(es)** | Usuario |
| **Descripción** | Generar comprobante con código único y opción descargar/impresión. |
| **Precondiciones** | Ticket `confirmado`. |
| **Flujo Principal** | 1) Generar vista HTML/PDF.<br>2) Mostrar/descargar. |
| **Requerimientos Especiales** | Código único; adaptador PDF intercambiable. |
| **Frecuencia de Uso** | Media. |

### **Caso de uso #20: Mis tickets**
| **ID** | R-120 |
|---|---|
| **Actor(es)** | Usuario |
| **Descripción** | Ver historial y filtrar por estado/fecha. |
| **Precondiciones** | Usuario autenticado. |
| **Flujo Principal** | 1) Listar tickets.<br>2) Filtros y detalles. |
| **Requerimientos Especiales** | Paginación. |
| **Frecuencia de Uso** | Media/alta. |

### **Caso de uso #21: Publicar/pausar viajes**
| **ID** | R-121 |
|---|---|
| **Actor(es)** | Administrador |
| **Descripción** | Cambiar estado de publicación de un viaje. |
| **Precondiciones** | Viaje creado. |
| **Flujo Principal** | 1) Seleccionar viaje.<br>2) Publicar/pausar.<br>3) Auditar. |
| **Requerimientos Especiales** | Auditoría (R-124). |
| **Frecuencia de Uso** | Diario. |

### **Caso de uso #22: Reportes de ventas**
| **ID** | R-122 |
|---|---|
| **Actor(es)** | Administrador |
| **Descripción** | Consultar ventas por fecha/ruta/empresa con gráficos. |
| **Precondiciones** | Tickets registrados. |
| **Flujo Principal** | 1) Seleccionar rango y agrupación.<br>2) Ver tabla/gráfico. |
| **Requerimientos Especiales** | Performance en agregados; exportación (R-123). |
| **Frecuencia de Uso** | Diario/semanal. |

### **Caso de uso #23: Exportación de reportes**
| **ID** | R-123 |
|---|---|
| **Actor(es)** | Administrador |
| **Descripción** | Exportar CSV/PDF. |
| **Precondiciones** | Reporte generado. |
| **Flujo Principal** | 1) Clic en exportar.<br>2) Sistema genera archivo. |
| **Requerimientos Especiales** | Encabezados y formato estándar. |
| **Frecuencia de Uso** | Medio. |

### **Caso de uso #24: Auditoría**
| **ID** | R-124 |
|---|---|
| **Actor(es)** | Sistema, Administrador (consulta) |
| **Descripción** | Registrar y consultar acciones críticas. |
| **Precondiciones** | Eventos generados. |
| **Flujo Principal** | 1) Insert de auditoría en acciones clave.<br>2) Admin consulta por filtros. |
| **Requerimientos Especiales** | Insert-only; correlationId (RNF-215/217). |
| **Frecuencia de Uso** | Medio. |

### **Caso de uso #25: Notificaciones**
| **ID** | R-125 |
|---|---|
| **Actor(es)** | Usuario, Sistema |
| **Descripción** | Mostrar toasts y envío de correos simulados (registro/verificación/compra/cancelación). |
| **Precondiciones** | Eventos de negocio. |
| **Flujo Principal** | 1) Disparar notificación.<br>2) Registrar resultado (opcional). |
| **Requerimientos Especiales** | Canal abstracto (adaptador); i18n futuro. |
| **Frecuencia de Uso** | Alta. |

---

## 5) Políticas y Criterios de Aceptación (NFR)

- **RNF-201 (bcrypt)**: Todas las contraseñas se almacenan con hash bcrypt (≥ 10 rounds).  
- **RNF-202 (JWT)**: Tokens expiran en ≤ 24h; rutas sensibles exigen JWT válido.  
- **RNF-203 (Verificación correo)**: Enlace válido durante 15–30 min; al validar, estado → `active`.  
- **RNF-204 (Rate limit)**: `/auth/register` y `/auth/resend-verification` limitados (p. ej., 5/min/IP).  
- **RNF-205 (Seguridad entrada/CORS)**: Sanitización; CORS restringido al dominio del frontend (en pruebas `*`).  
- **RNF-206 (Rendimiento búsquedas)**: p95 ≤ 300 ms con semilla de 5k viajes/100k tickets en ambiente local.  
- **RNF-207 (Confirmación)**: p95 ≤ 800 ms incluyendo lock y persistencia.  
- **RNF-208 (Índices/paginación)**: Todos los listados son paginados; índices compuestos en (origen,destino,fechaHora).  
- **RNF-209 (Unicidad asiento)**: UNIQUE (trip_id, seat_id); error si se intenta duplicar.  
- **RNF-210 (Lock TTL)**: Lock expira en 2–5 min; job de limpieza corre cada 1 min.  
- **RNF-211 (Idempotencia)**: Confirmación acepta `Idempotency-Key`; segundas solicitudes retornan el mismo resultado.  
- **RNF-212 (Capas)**: Código separado en Controller/Service/Repository; DTOs validados.  
- **RNF-213 (Respuesta estándar)**: Todas las respuestas siguen `{ok, data?, error?}` con `error.code` documentado.  
- **RNF-214 (Calidad código)**: ESLint/Prettier obligatorios en CI.  
- **RNF-215 (Logging JSON)**: Logs con timestamp y correlationId por request.  
- **RNF-216 (Health/Metrics)**: `/health` responde `{ok:true}`; métrica de búsquedas/ratio de conversión/cancelaciones.  
- **RNF-217 (Auditoría)**: Solo INSERT; guarda user_id, acción, entidad, entidad_id, meta JSON.  
- **RNF-218 (Docker/.env)**: Proyecto levanta con `docker-compose`; `.env.example` documenta variables.  
- **RNF-219 (Migraciones/seeds)**: `migrate deploy` y `db seed` reproducibles; versiones controladas.  
- **RNF-220 (Accesibilidad)**: Contraste AA, foco visible, navegación teclado; mobile-first.  
- **RNF-221 (Feedback UI)**: Formularios con validación inline; estados loading/empty/error definidos.  
- **RNF-222 (Versionado API)**: Endpoints bajo `/api/v1`; cambios breaking generan `/v2`.  
- **RNF-223 (Adaptadores)**: Pagos/Notificaciones/PDF con interfaces intercambiables.  
- **RNF-224 (i18n/moneda)**: Infra preparada para PEN y futura extensión a otros idiomas/monedas.

---

## 6) Supuestos y Restricciones

- **Supuestos**:  
  - Los usuarios finales se auto-registran; admin gestiona catálogos y reportes.  
  - La compra es **simulada** (sin gateway real) en esta práctica.  
  - La base de datos es **PostgreSQL en Supabase** (cloud, SSL).

- **Restricciones**:  
  - Cumplir con práctica: **BD relacional (PostgreSQL/MySQL)**, **API REST**, **Frontend web**.  
  - No exponer credenciales en frontend ni abrir la BD a Internet: acceso solo desde backend.

---

## 7) Trazabilidad (Matriz corta)

| Objetivo | RF/RNF relacionados |
|----------|---------------------|
| Registro y acceso seguro | R-101–R-107, RNF-201…205 |
| Búsqueda y compra | R-114…R-120, RNF-206…211 |
| Administración y reportes | R-121…R-124, RNF-208, RNF-215…217 |
| UX y notificaciones | R-125, RNF-220…221 |
| Despliegue y mantenibilidad | RNF-212…219, RNF-222…224 |

---
