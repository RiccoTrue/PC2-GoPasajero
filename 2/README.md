# 9.1.5. Módulo 5 / Integrante 5

## Módulo de Gestión de Reportes

## Caso de Uso CU01: Reporte de Ventas del día

| **Código Requisito** | **RF01** |
| -- | -- |
| **Código Interfaz** | P01 |

### Imagen Interfaz

<p align="center">
  <img src="frigorifica.jpg" alt="Logo Camal San Pedro" width="100%">
</p>

**Eventos:**

- **Carga inicial de formulario:**

   Al cargar la pantalla, se obtienen los datos necesarios para los campos desplegables y selectores.
   
```sql
   -- Query 1.1: Obtener contenedores disponibles para asignar
   SELECT 
       c.id_contenedor,
       c.codigo,
       c.peso,
       c.capacidad,
       c.dimensiones,
       tc.nombre AS tipo_contenedor,
       ec.nombre AS estado
   FROM monitoreo.Contenedor c
   JOIN monitoreo.TipoContenedor tc ON c.id_tipo_contenedor = tc.id_tipo_contenedor
   JOIN monitoreo.EstadoContenedor ec ON c.id_estado_contenedor = ec.id_estado_contenedor
   WHERE ec.nombre = 'Disponible'
   ORDER BY c.codigo;
   
   -- Query 1.2: Obtener vehículos disponibles
   SELECT 
       v.id_vehiculo,
       v.placa,
       v.capacidad_ton,
       tv.nombre AS tipo_vehiculo,
       ev.nombre AS estado,
       a.ubicacion
   FROM monitoreo.Vehiculo v
   JOIN monitoreo.TipoVehiculo tv ON v.id_tipo_vehiculo = tv.id_tipo_vehiculo
   JOIN monitoreo.EstadoVehiculo ev ON v.id_estado_vehiculo = ev.id_estado_vehiculo
   JOIN monitoreo.Activo a ON v.id_activo = a.id_activo
   WHERE ev.nombre = 'Disponible'
   ORDER BY v.placa;
   
   -- Query 1.3: Obtener buques disponibles
   SELECT 
       b.id_buque,
       b.matricula,
       b.nombre,
       b.capacidad AS capacidad_teu,
       ee.nombre AS estado,
       b.ubicacion_actual
   FROM monitoreo.Buque b
   JOIN monitoreo.EstadoEmbarcacion ee ON b.id_estado_embarcacion = ee.id_estado_embarcacion
   WHERE ee.nombre IN ('Disponible', 'En operación')
   ORDER BY b.nombre;
   
   -- Query 1.4: Obtener operador actual (del usuario logueado)
   SELECT 
       o.id_operador,
       e.codigo AS codigo_empleado,
       e.nombre,
       e.apellido,
       o.turno,
       o.zona_monitoreo
   FROM monitoreo.Operador o
   JOIN monitoreo.Empleado e ON o.id_empleado = e.id_empleado
   JOIN monitoreo.Usuario u ON u.id_empleado = e.id_empleado
   WHERE u.id_usuario = :id_usuario_actual; -- :id_usuario_actual es un placeholder.
```

- **Botón Guardar:**

   Se crea una nueva operación de monitoreo con todos sus datos asociados (contenedores, vehículo/buque, operador).

```sql
    -- Query 1.5: Crear nueva operación de monitoreo (INSERT)

    -- Paso 1: Insertar en Operacion
    INSERT INTO monitoreo.Operacion (
        id_operacion, 
        codigo, 
        fecha_inicio, 
        fecha_fin, 
        id_estado_operacion
    )
    VALUES (
        gen_random_uuid(),
        'OP-2024-07-A5B2',
        '2024-11-04 10:30:00',
        NULL,
        (SELECT id_estado_operacion FROM monitoreo.EstadoOperacion WHERE nombre = 'En curso')
    )
    RETURNING id_operacion;

    -- Paso 2: Insertar en OperacionMonitoreo
    INSERT INTO monitoreo.OperacionMonitoreo (
        id_operacion_monitoreo,
        id_operacion
    )
    VALUES (
        gen_random_uuid(),
        '123e4567-e89b-12d3-a456-426614174000' -- Colocar ID del Paso 1
    )
    RETURNING id_operacion_monitoreo;

    -- Paso 3: Asociar operador
    INSERT INTO monitoreo.OperadorOperacionMonitoreo (
        id_operador_operacion_monitoreo,
        id_operador,
        id_operacion_monitoreo,
        fecha_realizacion
    )
    VALUES (
        gen_random_uuid(),
        '6e7a21b6-f972-451a-89a3-0be35bd3f382', -- ID del operador logueado (En este ejemplo ya está colocado)
        '456e7890-e89b-12d3-a456-426614174001', -- Colocar ID del Paso 2
        CURRENT_DATE
    );

    -- Paso 4: Asociar vehículo (si aplica - cuando selecciona "Vehículo")
    INSERT INTO monitoreo.OperacionMonitoreoVehiculo (
        id_operacion_monitoreo_vehiculo,
        id_operacion_monitoreo,
        id_vehiculo,
        fecha_operacion
    )
    VALUES (
        gen_random_uuid(),
        '456e7890-e89b-12d3-a456-426614174001', -- Colocar ID del Paso 2
        '76e6f740-2266-4187-8b03-623125267d10', -- ID del vehículo "Volvo FH16 - Placa: ABC-123" (En este ejemplo ya está colocado) 
        CURRENT_DATE
    );

    -- Paso 5: Asociar buque (si aplica - cuando selecciona "Buque")
    INSERT INTO monitoreo.OperacionMonitoreoBuque (
        id_operacion_monitoreo_buque,
        id_operacion_monitoreo,
        id_buque,
        fecha_operacion
    )
    VALUES (
        gen_random_uuid(),
        '456e7890-e89b-12d3-a456-426614174001', -- ID del Paso 2
        '6067208f-2f0f-4322-b9ee-f1d9331a8ffb', -- ID del buque seleccionado (En este ejemplo ya está colocado)
        CURRENT_DATE
    );
```

- **Botón Cancelar:**

   Redirige al usuario a la pantalla anterior o limpia el formulario sin guardar cambios.

---

| **Código Requisito** | **RF01** |
| -- | -- |
| **Código Interfaz** | P02 |
| **Imagen Interfaz** | ![PANTALLA_2](/assets/images/3/3.5/P02.PNG) |

**Eventos:**

- **Carga inicial de tabla:**

   Al cargar la pantalla, se obtienen todas las operaciones de monitoreo activas con sus datos relacionados.

```sql
   -- Query 2.1: Listar operaciones de monitoreo activas
   SELECT 
       o.id_operacion,
       o.codigo,
       o.fecha_inicio,
       o.fecha_fin,
       eo.nombre AS estado,
       STRING_AGG(DISTINCT e.nombre || ' ' || e.apellido, ', ') AS operadores,
       STRING_AGG(DISTINCT v.placa, ', ') AS vehiculos,
       STRING_AGG(DISTINCT b.nombre, ', ') AS buques,
       COUNT(DISTINCT COALESCE(cv.id_contenedor, cb.id_contenedor)) AS total_contenedores
   FROM monitoreo.Operacion o
   JOIN monitoreo.EstadoOperacion eo ON o.id_estado_operacion = eo.id_estado_operacion
   JOIN monitoreo.OperacionMonitoreo om ON om.id_operacion = o.id_operacion
   LEFT JOIN monitoreo.OperadorOperacionMonitoreo oom ON oom.id_operacion_monitoreo = om.id_operacion_monitoreo
   LEFT JOIN monitoreo.Operador op ON op.id_operador = oom.id_operador
   LEFT JOIN monitoreo.Empleado e ON e.id_empleado = op.id_empleado
   LEFT JOIN monitoreo.OperacionMonitoreoVehiculo omv ON omv.id_operacion_monitoreo = om.id_operacion_monitoreo
   LEFT JOIN monitoreo.Vehiculo v ON v.id_vehiculo = omv.id_vehiculo
   LEFT JOIN monitoreo.OperacionMonitoreoBuque omb ON omb.id_operacion_monitoreo = om.id_operacion_monitoreo
   LEFT JOIN monitoreo.Buque b ON b.id_buque = omb.id_buque
   LEFT JOIN monitoreo.ContenedorVehiculo cv ON cv.id_vehiculo = v.id_vehiculo
   LEFT JOIN monitoreo.ContenedorBuque cb ON cb.id_buque = b.id_buque
   WHERE eo.nombre IN ('En curso', 'Pendiente') -- Nota si quiere ver todas las operaciones añadir 'Completada'
   GROUP BY o.id_operacion, o.codigo, o.fecha_inicio, o.fecha_fin, eo.nombre
   ORDER BY o.fecha_inicio DESC;
```

- **Botón Finalizar operación (icono task_alt):**

   Se marca la operación como completada y se registra la fecha/hora de finalización.

```sql
   -- Query 2.2: Finalizar operación
   UPDATE monitoreo.Operacion
   SET 
       fecha_fin = CURRENT_TIMESTAMP,
       id_estado_operacion = (SELECT id_estado_operacion FROM monitoreo.EstadoOperacion WHERE nombre = 'Completada')
   WHERE id_operacion = '123e4567-e89b-12d3-a456-426614174000';
```

- **Botón Crear Nueva Operación:**

   Redirige a la pantalla P01 (Crear Nueva Operación).

- **Botón Ver detalle (icono visibility):**

   Redirige a la pantalla de detalle de la operación seleccionada.

- **Filtros (Estado, Rango de fechas, Operador):**

   Permiten filtrar las operaciones mostradas en la tabla según criterios específicos.

- **Botón Limpiar Filtros:**

   Remueve todos los filtros aplicados y muestra todas las operaciones activas.

---

### Caso de Uso CU02: Monitoreo GPS

| **Código Requisito** | **RF02** |
| -- | -- |
| **Código Interfaz** | P03 |
| **Imagen Interfaz** | ![PANTALLA_3](/assets/images/3/3.5/P03.PNG) |

**Eventos:**

- **Carga inicial del dashboard:**

   Al cargar la pantalla, se obtienen los KPIs principales del sistema de monitoreo.

```sql
   -- Query 3.1: KPIs del dashboard
   
   -- KPI 1: Operaciones activas
   SELECT COUNT(*) AS operaciones_activas
   FROM monitoreo.Operacion o
   JOIN monitoreo.EstadoOperacion eo ON o.id_estado_operacion = eo.id_estado_operacion
   WHERE eo.nombre = 'En curso';
   
   -- KPI 2: Contenedores en tránsito
   SELECT COUNT(*) AS contenedores_transito
   FROM monitoreo.Contenedor c
   JOIN monitoreo.EstadoContenedor ec ON c.id_estado_contenedor = ec.id_estado_contenedor
   WHERE ec.nombre = 'En tránsito';
   
   -- KPI 3: Alertas pendientes (notificaciones críticas sin resolver)
   SELECT COUNT(*) AS alertas_pendientes
   FROM monitoreo.Notificacion n
   JOIN monitoreo.TipoNotificacion tn ON n.id_tipo_notificacion = tn.id_tipo_notificacion
   WHERE tn.nombre IN ('Alerta', 'Crítica')
       AND DATE(n.fecha_hora) >= CURRENT_DATE - INTERVAL '1 day';
   
   -- KPI 4: Entregas programadas hoy
   SELECT COUNT(*) AS entregas_hoy
   FROM monitoreo.Entrega e
   WHERE e.fecha_entrega = CURRENT_DATE;
```

- **Carga del gráfico "Operaciones por estado":**

   Se obtienen los datos para el gráfico de barras que muestra la distribución de operaciones.

```sql
   -- Query 3.2: Operaciones por estado (para gráfico de barras)
   SELECT 
       eo.nombre AS estado,
       COUNT(*) AS cantidad
   FROM monitoreo.Operacion o
   JOIN monitoreo.EstadoOperacion eo ON o.id_estado_operacion = eo.id_estado_operacion
   GROUP BY eo.nombre
   ORDER BY cantidad DESC;
```

- **Carga del gráfico "Notificaciones por día":**

   Se obtienen los datos para el gráfico de líneas con las notificaciones de la última semana.
```sql
   -- Query 3.3: Notificaciones por día (última semana)
   SELECT 
       DATE(n.fecha_hora) AS fecha,
       tn.nombre AS tipo,
       COUNT(*) AS cantidad
   FROM monitoreo.Notificacion n
   JOIN monitoreo.TipoNotificacion tn ON n.id_tipo_notificacion = tn.id_tipo_notificacion
   WHERE n.fecha_hora >= CURRENT_DATE - INTERVAL '7 days'
   GROUP BY DATE(n.fecha_hora), tn.nombre
   ORDER BY fecha DESC, cantidad DESC;
```

- **Carga del panel "Alertas recientes":**

   Se obtienen las últimas alertas críticas del sistema.
```sql
   -- Query 3.4: Últimas alertas críticas
   SELECT 
       n.codigo,
       tn.nombre AS tipo,
       n.fecha_hora,
       n.valor,
       s.nombre AS sensor,
       c.codigo AS contenedor,
       ts.nombre AS tipo_sensor
   FROM monitoreo.Notificacion n
   JOIN monitoreo.TipoNotificacion tn ON n.id_tipo_notificacion = tn.id_tipo_notificacion
   JOIN monitoreo.Sensor s ON n.id_sensor = s.id_sensor
   JOIN monitoreo.TipoSensor ts ON s.id_tipo_sensor = ts.id_tipo_sensor
   JOIN monitoreo.Contenedor c ON s.id_contenedor = c.id_contenedor
   WHERE tn.nombre IN ('Alerta', 'Crítica')
   ORDER BY n.fecha_hora DESC
   LIMIT 10;
```

- **Botón "Ver mapa completo":**

   Redirige a la pantalla de mapa con visualización completa de ubicaciones GPS.

- **Campo de búsqueda (header):**

   Permite buscar operaciones o contenedores específicos en el sistema.

- **Botón de notificaciones:**

   Muestra un panel con las notificaciones pendientes del usuario.

---

### Caso de Uso CU03: Monitoreo de sensores IoT

| **Código Requisito** | **RF03** |
| -- | -- |
| **Código Interfaz** | P05 |
| **Imagen Interfaz** | ![PANTALLA_5](/assets/images/3/3.5/P05.PNG) |

**Eventos:**

- **Carga inicial de la pantalla:**

   Al cargar la vista detallada del contenedor, se obtiene toda la información del contenedor seleccionado.

```sql
   -- Query 5.1: Información detallada del contenedor
   SELECT 
       c.id_contenedor,
       c.codigo,
       c.peso,
       c.capacidad,
       c.dimensiones,
       tc.nombre AS tipo_contenedor,
       tc.costo,
       ec.nombre AS estado
   FROM monitoreo.Contenedor c
   JOIN monitoreo.TipoContenedor tc ON c.id_tipo_contenedor = tc.id_tipo_contenedor
   JOIN monitoreo.EstadoContenedor ec ON c.id_estado_contenedor = ec.id_estado_contenedor
   WHERE c.id_contenedor = '1ab37ca7-67e2-4644-8b45-35e5ffd480f7';
```

- **Carga de sensores activos:**

   Se obtienen todos los sensores del contenedor con sus últimas lecturas para mostrar en las tarjetas de métricas.

```sql
   -- Query 5.2: Sensores del contenedor con última lectura
   WITH UltimasNotificaciones AS (
       SELECT 
           id_sensor,
           valor,
           fecha_hora,
           ROW_NUMBER() OVER (PARTITION BY id_sensor ORDER BY fecha_hora DESC) AS rn
       FROM monitoreo.Notificacion
   )
   SELECT 
       s.id_sensor,
       s.codigo,
       s.nombre,
       ts.nombre AS tipo_sensor,
       rs.nombre AS rol_sensor,
       un.valor AS valor_actual,
       un.fecha_hora AS ultima_lectura
   FROM monitoreo.Sensor s
   JOIN monitoreo.TipoSensor ts ON s.id_tipo_sensor = ts.id_tipo_sensor
   JOIN monitoreo.RolSensor rs ON s.id_rol_sensor = rs.id_rol_sensor
   LEFT JOIN UltimasNotificaciones un ON un.id_sensor = s.id_sensor AND un.rn = 1
   WHERE s.id_contenedor = '1ab37ca7-67e2-4644-8b45-35e5ffd480f7'
   ORDER BY ts.nombre;
```

- **Carga de gráficos de sensores:**

   Para cada sensor, se obtiene el historial de las últimas 24 horas para renderizar los mini-gráficos.

```sql
   -- Query 5.3: Historial de lecturas de un sensor (últimas 24 horas)
   SELECT 
       n.fecha_hora,
       n.valor,
       tn.nombre AS tipo_notificacion
   FROM monitoreo.Notificacion n
   JOIN monitoreo.TipoNotificacion tn ON n.id_tipo_notificacion = tn.id_tipo_notificacion
   WHERE n.id_sensor = '9b93ce7b-d582-4cd0-944e-5adc64377975'
       AND n.fecha_hora >= CURRENT_TIMESTAMP - INTERVAL '24 hours'
   ORDER BY n.fecha_hora ASC;
```

- **Carga de documentación:**

   Se obtienen todos los documentos asociados al contenedor.

```sql
   -- Query 5.4: Documentación del contenedor
   SELECT 
       d.id_documentacion,
       d.codigo,
       d.nombre,
       d.fecha_emision,
       td.nombre AS tipo_documento
   FROM monitoreo.Documentacion d
   JOIN monitoreo.TipoDocumento td ON d.id_tipo_documento = td.id_tipo_documento
   JOIN monitoreo.DocumentacionContenedor dc ON dc.id_documentacion = d.id_documentacion
   WHERE dc.id_contenedor = '1ab37ca7-67e2-4644-8b45-35e5ffd480f7'
   ORDER BY d.fecha_emision DESC;
```

- **Carga de última posición GPS:**

   Se obtiene la ubicación más reciente del contenedor para mostrar en el mapa.

```sql
   -- Query 5.5: Última posición del contenedor
   SELECT 
       pc.latitud,
       pc.longitud,
       pc.fecha_hora
   FROM monitoreo.PosicionContenedor pc
   WHERE pc.id_contenedor = '1ab37ca7-67e2-4644-8b45-35e5ffd480f7'
   ORDER BY pc.fecha_hora DESC
   LIMIT 1;
```

- **Botón Ver (visibility) en documentos:**

   Abre una vista previa del documento seleccionado.

- **Botón Descargar (download) en documentos:**

   Descarga el documento seleccionado en formato PDF.

- **Navegación breadcrumb:**

   Permite regresar a la lista de contenedores haciendo click en "Contenedores".

---

| **Código Requisito** | **RF03** |
| -- | -- |
| **Código Interfaz** | P06 |
| **Imagen Interfaz** | ![PANTALLA_6](/assets/images/3/3.5/P06.PNG) |

**Eventos:**

- **Carga inicial del panel de sensores:**

   Al cargar la pantalla, se obtiene el resumen de todos los sensores del contenedor con su estado actual y última lectura.

```sql
   -- Query 6.1: Resumen de sensores con estado
   WITH UltimasLecturas AS (
       SELECT 
           n.id_sensor,
           n.valor,
           n.fecha_hora,
           tn.nombre AS tipo_notificacion,
           ROW_NUMBER() OVER (PARTITION BY n.id_sensor ORDER BY n.fecha_hora DESC) AS rn
       FROM monitoreo.Notificacion n
       JOIN monitoreo.TipoNotificacion tn ON n.id_tipo_notificacion = tn.id_tipo_notificacion
   )
   SELECT 
       s.id_sensor,
       s.codigo,
       s.nombre,
       ts.nombre AS tipo_sensor,
       ul.valor AS valor_actual,
       ul.fecha_hora AS ultima_actualizacion,
       ul.tipo_notificacion,
       CASE 
           WHEN ul.tipo_notificacion = 'Crítica' THEN 'rojo'
           WHEN ul.tipo_notificacion IN ('Alerta', 'Advertencia') THEN 'amarillo'
           ELSE 'verde'
       END AS indicador_estado
   FROM monitoreo.Sensor s
   JOIN monitoreo.TipoSensor ts ON s.id_tipo_sensor = ts.id_tipo_sensor
   LEFT JOIN UltimasLecturas ul ON ul.id_sensor = s.id_sensor AND ul.rn = 1
   WHERE s.id_contenedor = '1ab37ca7-67e2-4644-8b45-35e5ffd480f7'
   ORDER BY ts.nombre;
```

- **Carga de mini gráficos (sparklines):**

   Para cada tarjeta de sensor, se obtienen las últimas 10 lecturas para renderizar el gráfico de tendencia.
```sql
   -- Query 6.2: Datos para mini gráfico sparkline (últimas 10 lecturas)
   SELECT 
       n.fecha_hora,
       n.valor
   FROM monitoreo.Notificacion n
   WHERE n.id_sensor = '85ca6bb2-5277-4810-a5e9-976afe810bac'
   ORDER BY n.fecha_hora DESC
   LIMIT 10;
```

- **Botón "Ver Historial" en cada tarjeta:**

   Redirige a una vista detallada con el historial completo de ese sensor específico.

- **Navegación breadcrumb:**

   Permite regresar a la lista de contenedores haciendo click en "Contenedores".

- **Actualización automática (simulada):**

   En producción, esta pantalla tendría auto-refresh cada 30-60 segundos para mostrar datos en tiempo real.

---

### Caso de Uso CU04: Generación automática de notificaciones

| **Código Requisito** | **RF04** |
| -- | -- |
| **Código Interfaz** | P07 |
| **Imagen Interfaz** | ![PANTALLA_7](/assets/images/3/3.5/P07.PNG) |

**Eventos:**

- **Carga inicial del panel de notificaciones:**

   Al cargar la pantalla, se obtienen las notificaciones más recientes del sistema con sus detalles.

```sql
   -- Query 7.1: Listar notificaciones recientes
   SELECT 
       n.id_notificacion,
       n.codigo,
       tn.nombre AS tipo_notificacion,
       n.fecha_hora,
       n.valor,
       s.nombre AS sensor,
       ts.nombre AS tipo_sensor,
       c.codigo AS contenedor,
       r.codigo AS reporte
   FROM monitoreo.Notificacion n
   JOIN monitoreo.TipoNotificacion tn ON n.id_tipo_notificacion = tn.id_tipo_notificacion
   JOIN monitoreo.Sensor s ON n.id_sensor = s.id_sensor
   JOIN monitoreo.TipoSensor ts ON s.id_tipo_sensor = ts.id_tipo_sensor
   JOIN monitoreo.Contenedor c ON s.id_contenedor = c.id_contenedor
   JOIN monitoreo.Reporte r ON n.id_reporte = r.id_reporte
   WHERE 
       (NULL IS NULL OR tn.nombre = NULL)
       AND (NULL IS NULL OR c.id_contenedor = NULL)
       AND n.fecha_hora BETWEEN CURRENT_DATE - INTERVAL '30 days' AND CURRENT_TIMESTAMP
   ORDER BY n.fecha_hora DESC
   LIMIT 50 OFFSET 0;
```

- **Aplicar filtro por tipo:**

   Al seleccionar un tipo de notificación específico, se filtra la lista.
```sql
   -- Query 7.1 con filtro de tipo
   SELECT 
       n.id_notificacion,
       n.codigo,
       tn.nombre AS tipo_notificacion,
       n.fecha_hora,
       n.valor,
       s.nombre AS sensor,
       ts.nombre AS tipo_sensor,
       c.codigo AS contenedor,
       r.codigo AS reporte
   FROM monitoreo.Notificacion n
   JOIN monitoreo.TipoNotificacion tn ON n.id_tipo_notificacion = tn.id_tipo_notificacion
   JOIN monitoreo.Sensor s ON n.id_sensor = s.id_sensor
   JOIN monitoreo.TipoSensor ts ON s.id_tipo_sensor = ts.id_tipo_sensor
   JOIN monitoreo.Contenedor c ON s.id_contenedor = c.id_contenedor
   JOIN monitoreo.Reporte r ON n.id_reporte = r.id_reporte
   WHERE 
       ('Alerta' IS NULL OR tn.nombre = 'Alerta')
       AND (NULL IS NULL OR c.id_contenedor = NULL)
   ORDER BY n.fecha_hora DESC
   LIMIT 50 OFFSET 0;
```

- **Aplicar filtro por rango de fechas:**

   Al seleccionar un rango de fechas, se filtra la lista por período.
```sql
   -- Query 7.1 con filtro de fechas
   SELECT 
       n.id_notificacion,
       n.codigo,
       tn.nombre AS tipo_notificacion,
       n.fecha_hora,
       n.valor,
       s.nombre AS sensor,
       ts.nombre AS tipo_sensor,
       c.codigo AS contenedor,
       r.codigo AS reporte
   FROM monitoreo.Notificacion n
   JOIN monitoreo.TipoNotificacion tn ON n.id_tipo_notificacion = tn.id_tipo_notificacion
   JOIN monitoreo.Sensor s ON n.id_sensor = s.id_sensor
   JOIN monitoreo.TipoSensor ts ON s.id_tipo_sensor = ts.id_tipo_sensor
   JOIN monitoreo.Contenedor c ON s.id_contenedor = c.id_contenedor
   JOIN monitoreo.Reporte r ON n.id_reporte = r.id_reporte
   WHERE 
       (NULL IS NULL OR tn.nombre = NULL)
       AND (NULL IS NULL OR c.id_contenedor = NULL)
       AND n.fecha_hora BETWEEN '2024-11-01 00:00:00' AND '2024-11-04 23:59:59'
   ORDER BY n.fecha_hora DESC
   LIMIT 50 OFFSET 0;
   -- Muestra notificaciones del rango seleccionado
```

- **Aplicar filtro por contenedor:**

   Al seleccionar un contenedor específico, se filtran solo sus notificaciones.
```sql
   -- Query 7.1 con filtro de contenedor
   SELECT 
       n.id_notificacion,
       n.codigo,
       tn.nombre AS tipo_notificacion,
       n.fecha_hora,
       n.valor,
       s.nombre AS sensor,
       ts.nombre AS tipo_sensor,
       c.codigo AS contenedor,
       r.codigo AS reporte
   FROM monitoreo.Notificacion n
   JOIN monitoreo.TipoNotificacion tn ON n.id_tipo_notificacion = tn.id_tipo_notificacion
   JOIN monitoreo.Sensor s ON n.id_sensor = s.id_sensor
   JOIN monitoreo.TipoSensor ts ON s.id_tipo_sensor = ts.id_tipo_sensor
   JOIN monitoreo.Contenedor c ON s.id_contenedor = c.id_contenedor
   JOIN monitoreo.Reporte r ON n.id_reporte = r.id_reporte
   WHERE 
       (NULL IS NULL OR tn.nombre = NULL)
       AND ('1ab37ca7-67e2-4644-8b45-35e5ffd480f7' IS NULL OR c.id_contenedor = '1ab37ca7-67e2-4644-8b45-35e5ffd480f7')
   ORDER BY n.fecha_hora DESC
   LIMIT 50 OFFSET 0;
```

- **Carga de estadísticas (opcional - para dashboard interno):**

   Obtiene resumen de notificaciones por tipo en los últimos 7 días.
```sql
   -- Query 7.2: Estadísticas de notificaciones
   SELECT 
       tn.nombre AS tipo,
       COUNT(*) AS cantidad,
       MIN(n.fecha_hora) AS primera,
       MAX(n.fecha_hora) AS ultima
   FROM monitoreo.Notificacion n
   JOIN monitoreo.TipoNotificacion tn ON n.id_tipo_notificacion = tn.id_tipo_notificacion
   WHERE n.fecha_hora >= CURRENT_DATE - INTERVAL '7 days'
   GROUP BY tn.nombre
   ORDER BY cantidad DESC;
```

- **Botón "Ver Detalle":**

   Redirige a la vista detallada del contenedor asociado a la notificación (P05).

- **Checkbox de selección:**

   Permite seleccionar múltiples notificaciones para acciones masivas.

- **Botón "Marcar todas como leídas" (done_all):**

   Actualiza el estado de las notificaciones seleccionadas a "leídas".

- **Botón "Eliminar seleccionadas" (delete):**

   Elimina las notificaciones seleccionadas del sistema.

- **Botón "Nueva Regla de Alerta":**

   Abre un formulario para configurar nuevas reglas automáticas de notificación.

- **Paginación:**

   Permite navegar entre páginas de resultados (50 notificaciones por página).

---

### Caso de Uso CU05: Registro de incidencias

| **Código Requisito** | **RF05** |
| -- | -- |
| **Código Interfaz** | P08 |
| **Imagen Interfaz** | ![PANTALLA_8](/assets/images/3/3.5/P08.PNG) |

**Eventos:**

- **Carga inicial del panel de incidencias:**

   Al cargar la pantalla, se obtiene el historial de incidencias registradas en el sistema.

```sql
   -- Query 8.1: Listar incidencias
   SELECT 
       i.id_incidencia,
       i.codigo,
       ti.nombre AS tipo_incidencia,
       i.descripcion,
       i.grado_severidad,
       i.fecha_hora,
       ei.nombre AS estado,
       o.codigo AS operacion,
       u.correo_electronico AS usuario_registro,
       e.nombre || ' ' || e.apellido AS nombre_completo
   FROM monitoreo.Incidencia i
   JOIN monitoreo.TipoIncidencia ti ON i.id_tipo_incidencia = ti.id_tipo_incidencia
   JOIN monitoreo.EstadoIncidencia ei ON i.id_estado_incidencia = ei.id_estado_incidencia
   JOIN monitoreo.Operacion o ON i.id_operacion = o.id_operacion
   JOIN monitoreo.Usuario u ON i.id_usuario = u.id_usuario
   JOIN monitoreo.Empleado e ON u.id_empleado = e.id_empleado
   ORDER BY i.fecha_hora DESC
   LIMIT 100;
```

- **Carga del selector "Tipo de Incidencia":**

   Se obtienen todos los tipos de incidencia disponibles para el formulario de registro.
```sql
   -- Query 8.2: Obtener tipos de incidencia
   SELECT 
       id_tipo_incidencia,
       nombre
   FROM monitoreo.TipoIncidencia
   ORDER BY nombre;
```

- **Carga del campo "Operación Relacionada":**

   Se obtienen las operaciones activas disponibles para vincular con la nueva incidencia.
```sql
   -- Query 8.3: Obtener operaciones activas para vincular
   SELECT 
       o.id_operacion,
       o.codigo,
       o.fecha_inicio,
       eo.nombre AS estado
   FROM monitoreo.Operacion o
   JOIN monitoreo.EstadoOperacion eo ON o.id_estado_operacion = eo.id_estado_operacion
   WHERE eo.nombre = 'En curso'
   ORDER BY o.fecha_inicio DESC;
```

- **Botón "Registrar Incidencia":**

   Se crea una nueva incidencia en el sistema con los datos del formulario.
```sql
   -- Query 8.4: Registrar nueva incidencia
   INSERT INTO monitoreo.Incidencia (
       id_incidencia,
       codigo,
       id_tipo_incidencia,
       descripcion,
       grado_severidad,
       fecha_hora,
       id_estado_incidencia,
       id_operacion,
       id_usuario
   )
   VALUES (
       gen_random_uuid(),
       'INC-006',
       '09948553-9461-4e81-9e6b-826543db3187', -- ID tipo: Daño en Contenedor
       'Contenedor presenta abolladura en puerta lateral izquierda que impide cierre hermético',
       4,
       CURRENT_TIMESTAMP,
       (SELECT id_estado_incidencia FROM monitoreo.EstadoIncidencia WHERE nombre = 'Reportada'),
       '49141564-a44e-435e-b3ee-c2d8b5c7a117', -- ID operación
       '694ea01c-f274-4acd-ac07-44b5be3dc2d1' -- ID usuario actual
   )
   RETURNING id_incidencia;
   -- Retorna el UUID de la nueva incidencia creada
```

- **Botón Editar (edit) en tabla:**

   Permite modificar los detalles de una incidencia existente y actualizar su estado.
```sql
   -- Query 8.5: Actualizar estado de incidencia
   UPDATE monitoreo.Incidencia
   SET id_estado_incidencia = (
       SELECT id_estado_incidencia 
       FROM monitoreo.EstadoIncidencia 
       WHERE nombre = 'En Proceso'
   )
   WHERE id_incidencia = '123e4567-e89b-12d3-a456-426614174000';
   -- Actualiza el estado de INC-001 de "Nuevo" a "En Proceso"
```

- **Botón Ver (visibility) en tabla:**

   Muestra una vista detallada de la incidencia seleccionada con todo su historial.

- **Control deslizante "Nivel de Severidad":**

   Permite seleccionar el grado de severidad de 1 (Bajo) a 5 (Crítico) para la nueva incidencia.

- **Campo de búsqueda en header:**

   Permite buscar incidencias por código, tipo o descripción.

---

### Caso de Uso CU06: Generar reportes de monitoreo

| **Código Requisito** | **RF06** |
| -- | -- |
| **Código Interfaz** | P09 |
| **Imagen Interfaz** | ![PANTALLA_9](/assets/images/3/3.5/P09.PNG) |

**Eventos:**

- **Carga inicial del historial de reportes:**

   Al cargar la pantalla, se obtiene la lista de reportes generados con sus estadísticas básicas.

```sql
   -- Query 9.1: Listar reportes existentes
   SELECT 
       r.id_reporte,
       r.codigo,
       r.fecha_reporte,
       r.detalle,
       COUNT(DISTINCT n.id_notificacion) AS total_notificaciones,
       COUNT(DISTINCT ir.id_incidencia) AS total_incidencias
   FROM monitoreo.Reporte r
   LEFT JOIN monitoreo.Notificacion n ON n.id_reporte = r.id_reporte
   LEFT JOIN monitoreo.IncidenciaReporte ir ON ir.id_reporte = r.id_reporte
   GROUP BY r.id_reporte, r.codigo, r.fecha_reporte, r.detalle
   ORDER BY r.fecha_reporte DESC
   LIMIT 50;
```

- **Click en fila de reporte (seleccionar reporte):**

   Al seleccionar un reporte de la tabla, se obtiene su información completa para mostrar en el panel de detalles.

```sql
   -- Query 9.2: Detalle completo de un reporte
   
   -- Información del reporte
   SELECT 
       r.id_reporte,
       r.codigo,
       r.fecha_reporte,
       r.detalle
   FROM monitoreo.Reporte r
   WHERE r.id_reporte = '03481fbb-ab33-4e8d-a433-f743d4d20bc0';
   
   -- Notificaciones del reporte
   SELECT 
       n.codigo,
       tn.nombre AS tipo,
       n.fecha_hora,
       n.valor,
       s.nombre AS sensor,
       ts.nombre AS tipo_sensor,
       c.codigo AS contenedor
   FROM monitoreo.Notificacion n
   JOIN monitoreo.TipoNotificacion tn ON n.id_tipo_notificacion = tn.id_tipo_notificacion
   JOIN monitoreo.Sensor s ON n.id_sensor = s.id_sensor
   JOIN monitoreo.TipoSensor ts ON s.id_tipo_sensor = ts.id_tipo_sensor
   JOIN monitoreo.Contenedor c ON s.id_contenedor = c.id_contenedor
   WHERE n.id_reporte = '03481fbb-ab33-4e8d-a433-f743d4d20bc0'
   ORDER BY n.fecha_hora;
   
   -- Incidencias del reporte
   SELECT 
       i.codigo,
       ti.nombre AS tipo,
       i.descripcion,
       i.grado_severidad,
       i.fecha_hora,
       ei.nombre AS estado
   FROM monitoreo.Incidencia i
   JOIN monitoreo.IncidenciaReporte ir ON ir.id_incidencia = i.id_incidencia
   JOIN monitoreo.TipoIncidencia ti ON i.id_tipo_incidencia = ti.id_tipo_incidencia
   JOIN monitoreo.EstadoIncidencia ei ON i.id_estado_incidencia = ei.id_estado_incidencia
   WHERE ir.id_reporte = '03481fbb-ab33-4e8d-a433-f743d4d20bc0'
   ORDER BY i.fecha_hora;
```

- **Botón "Generar Reporte":**

   Se crea un nuevo reporte basado en el rango de fechas y operación seleccionados.
```sql
   -- Query 9.3: Crear nuevo reporte
   INSERT INTO monitoreo.Reporte (
       id_reporte,
       codigo,
       fecha_reporte,
       detalle
   )
   VALUES (
       gen_random_uuid(),
       'RPT-2023-12-004',
       CURRENT_DATE,
       'Reporte de monitoreo del 10 al 16 de diciembre 2023 para operación OP-567-FCL. Contenedor MSCU-123456-7 en ruta Shanghái - Long Beach.'
   )
   RETURNING id_reporte;
   -- Retorna el UUID del nuevo reporte creado
```

- **Asociar incidencias al reporte (automático al generar):**

   Al crear un reporte, se vinculan automáticamente las incidencias del período seleccionado.

```sql
   -- Query 9.4: Asociar incidencia a reporte
   INSERT INTO monitoreo.IncidenciaReporte (
       id_incidencia_reporte,
       id_incidencia,
       id_reporte
   )
   VALUES (
       gen_random_uuid(),
       'bab9391d-ad51-476a-af46-2fc3e7da3ae0', -- ID incidencia del período
       'd7a78fe3-7c76-41e6-9eff-72867708c165' -- ID del reporte recién creado
   );
```

- **Carga de estadísticas del reporte:**

   Se obtienen las métricas agregadas para mostrar en el panel de detalles.
```sql
   -- Query 9.5: Estadísticas para el reporte
   SELECT 
       COUNT(DISTINCT n.id_notificacion) AS total_notificaciones,
       COUNT(DISTINCT CASE WHEN tn.nombre = 'Crítica' THEN n.id_notificacion END) AS notificaciones_criticas,
       COUNT(DISTINCT ir.id_incidencia) AS total_incidencias,
       AVG(i.grado_severidad) AS severidad_promedio
   FROM monitoreo.Reporte r
   LEFT JOIN monitoreo.Notificacion n ON n.id_reporte = r.id_reporte
   LEFT JOIN monitoreo.TipoNotificacion tn ON n.id_tipo_notificacion = tn.id_tipo_notificacion
   LEFT JOIN monitoreo.IncidenciaReporte ir ON ir.id_reporte = r.id_reporte
   LEFT JOIN monitoreo.Incidencia i ON i.id_incidencia = ir.id_incidencia
   WHERE r.id_reporte = '03481fbb-ab33-4e8d-a433-f743d4d20bc0'
   GROUP BY r.id_reporte;
```

- **Botón "Descargar PDF":**

   Genera un archivo PDF con el contenido completo del reporte seleccionado.

- **Botón "Exportar Excel":**

   Genera un archivo Excel con las tablas de notificaciones e incidencias del reporte.

- **Pestañas (Resumen General, Notificaciones, Incidencias):**

   Cambian la vista del panel de detalles para mostrar diferentes secciones del reporte.

- **Selector de rango de fechas (calendario):**

   Permite seleccionar el período para el nuevo reporte a generar.

- **Selector "Operación":**

   Permite filtrar el reporte por una operación específica o incluir todas.

---

### Caso de Uso CU07: Visualización en mapa

| **Código Requisito** | **RF07** |
| -- | -- |
| **Código Interfaz** | P04 |
| **Imagen Interfaz** | ![PANTALLA_4](/assets/images/3/3.5/P04.PNG) |

**Eventos:**

- **Carga inicial del mapa:**

   Al cargar la pantalla, se obtienen las últimas posiciones GPS de todos los activos (contenedores, vehículos y buques) para mostrarlos en el mapa.

```sql
   -- Query 4.1: Últimas posiciones de contenedores
   WITH UltimasPosiciones AS (
       SELECT 
           id_contenedor,
           latitud,
           longitud,
           fecha_hora,
           ROW_NUMBER() OVER (PARTITION BY id_contenedor ORDER BY fecha_hora DESC) AS rn
       FROM monitoreo.PosicionContenedor
   )
   SELECT 
       c.id_contenedor,
       c.codigo,
       tc.nombre AS tipo,
       ec.nombre AS estado,
       up.latitud,
       up.longitud,
       up.fecha_hora AS ultima_actualizacion
   FROM monitoreo.Contenedor c
   JOIN monitoreo.TipoContenedor tc ON c.id_tipo_contenedor = tc.id_tipo_contenedor
   JOIN monitoreo.EstadoContenedor ec ON c.id_estado_contenedor = ec.id_estado_contenedor
   JOIN UltimasPosiciones up ON up.id_contenedor = c.id_contenedor
   WHERE up.rn = 1
       AND ec.nombre = 'En tránsito'
   ORDER BY c.codigo;
   
   -- Query 4.2: Últimas posiciones de vehículos
   WITH UltimasPosicionesVehiculo AS (
       SELECT 
           id_vehiculo,
           latitud,
           longitud,
           fecha_hora,
           ROW_NUMBER() OVER (PARTITION BY id_vehiculo ORDER BY fecha_hora DESC) AS rn
       FROM monitoreo.PosicionVehiculo
   )
   SELECT 
       v.id_vehiculo,
       v.placa,
       tv.nombre AS tipo,
       ev.nombre AS estado,
       upv.latitud,
       upv.longitud,
       upv.fecha_hora AS ultima_actualizacion
   FROM monitoreo.Vehiculo v
   JOIN monitoreo.TipoVehiculo tv ON v.id_tipo_vehiculo = tv.id_tipo_vehiculo
   JOIN monitoreo.EstadoVehiculo ev ON v.id_estado_vehiculo = ev.id_estado_vehiculo
   JOIN UltimasPosicionesVehiculo upv ON upv.id_vehiculo = v.id_vehiculo
   WHERE upv.rn = 1
       AND ev.nombre IN ('En ruta', 'Disponible')
   ORDER BY v.placa;
   
   -- Query 4.3: Últimas posiciones de buques
   WITH UltimasPosicionesBuque AS (
       SELECT 
           id_buque,
           latitud,
           longitud,
           fecha_hora,
           ROW_NUMBER() OVER (PARTITION BY id_buque ORDER BY fecha_hora DESC) AS rn
       FROM monitoreo.PosicionBuque
   )
   SELECT 
       b.id_buque,
       b.matricula,
       b.nombre,
       ee.nombre AS estado,
       upb.latitud,
       upb.longitud,
       upb.fecha_hora AS ultima_actualizacion
   FROM monitoreo.Buque b
   JOIN monitoreo.EstadoEmbarcacion ee ON b.id_estado_embarcacion = ee.id_estado_embarcacion
   JOIN UltimasPosicionesBuque upb ON upb.id_buque = b.id_buque
   WHERE upb.rn = 1
       AND ee.nombre = 'En operación'
   ORDER BY b.nombre;
```

- **Click en marcador del mapa:**

   Al hacer click en un marcador, se muestra el panel de información del activo seleccionado con sus datos actuales.

- **Botón "Ver Ruta Histórica":**

   Se obtiene el historial completo de posiciones del activo seleccionado para trazar su recorrido en el mapa.

```sql
   -- Query 4.4: Historial de ruta de un contenedor específico
   SELECT 
       pc.latitud,
       pc.longitud,
       pc.fecha_hora
   FROM monitoreo.PosicionContenedor pc
   WHERE pc.id_contenedor = '1ab37ca7-67e2-4644-8b45-35e5ffd480f7'
   ORDER BY pc.fecha_hora ASC;
```

- **Filtros de tipo de activo (Todos, Contenedor, Vehículo, Buque):**

   Filtra los marcadores visibles en el mapa y la lista del sidebar según el tipo seleccionado.

- **Filtros de estado (Todos, En Tránsito, Detenido, Alerta):**

   Filtra los marcadores visibles en el mapa y la lista del sidebar según el estado seleccionado.

- **Campo de búsqueda en sidebar:**

   Permite buscar activos específicos por código o estado en tiempo real.

- **Click en item de la lista (sidebar):**

   Centra el mapa en el activo seleccionado con animación y muestra su panel de información.

- **Botón de zoom (+/-):**

   Aumenta o disminuye el nivel de zoom del mapa.

- **Botón de cambio de capa (layers):**

   Alterna entre vista de mapa callejero y vista satelital.

- **Botón "my_location":**

   Centra el mapa en la ubicación predeterminada (Lima, Perú: -12.046374, -77.042793).

- **Botón cerrar (X) en panel de información:**

   Oculta el panel de información del activo.

---

### Caso de Uso CU08: Registrar entrega a importador

| **Código Requisito** | **RF08** |
| -- | -- |
| **Código Interfaz** | P10 |
| **Imagen Interfaz** | ![PANTALLA_10](/assets/images/3/3.5/P10.PNG) ![PANTALLA_11](/assets/images/3/3.5/P10_1.PNG) |

**Eventos:**

- **Carga inicial del formulario:**

   Al cargar la pantalla, se obtiene la información del contenedor seleccionado para entrega.

```sql
   -- Query 10.1: Obtener contenedores listos para entrega
   SELECT 
       c.id_contenedor,
       c.codigo,
       tc.nombre AS tipo,
       ec.nombre AS estado,
       o.codigo AS operacion_actual
   FROM monitoreo.Contenedor c
   JOIN monitoreo.TipoContenedor tc ON c.id_tipo_contenedor = tc.id_tipo_contenedor
   JOIN monitoreo.EstadoContenedor ec ON c.id_estado_contenedor = ec.id_estado_contenedor
   LEFT JOIN monitoreo.ContenedorVehiculo cv ON cv.id_contenedor = c.id_contenedor
   LEFT JOIN monitoreo.OperacionMonitoreoVehiculo omv ON omv.id_vehiculo = cv.id_vehiculo
   LEFT JOIN monitoreo.OperacionMonitoreo om ON om.id_operacion_monitoreo = omv.id_operacion_monitoreo
   LEFT JOIN monitoreo.Operacion o ON o.id_operacion = om.id_operacion
   WHERE ec.nombre = 'En tránsito'
   ORDER BY c.codigo;
```

- **Carga del selector "Importador":**

   Se obtienen todos los importadores registrados con sus datos principales.

```sql
   -- Query 10.2: Listar importadores disponibles
   SELECT 
       i.id_importador,
       i.codigo,
       i.ruc,
       i.razon_social,
       STRING_AGG(
           CASE WHEN id_d.principal = TRUE 
           THEN id_d.direccion 
           ELSE NULL END, 
           ''
       ) AS direccion_principal
   FROM monitoreo.Importador i
   LEFT JOIN monitoreo.ImportadorDireccion id_d ON id_d.id_importador = i.id_importador
   GROUP BY i.id_importador, i.codigo, i.ruc, i.razon_social
   ORDER BY i.razon_social;
```

- **Validación de documentación (automática al cargar):**

   Verifica que el contenedor tenga toda la documentación requerida para la entrega.

```sql
   -- Query 10.3: Validar documentación del contenedor
   SELECT 
       d.id_documentacion,
       d.codigo,
       d.nombre,
       td.nombre AS tipo_documento,
       d.fecha_emision,
       CASE 
           WHEN d.id_documentacion IS NOT NULL THEN 'Completo'
           ELSE 'Falta'
       END AS estado_doc
   FROM monitoreo.TipoDocumento td
   LEFT JOIN monitoreo.Documentacion d ON d.id_tipo_documento = td.id_tipo_documento
   LEFT JOIN monitoreo.DocumentacionContenedor dc ON dc.id_documentacion = d.id_documentacion 
       AND dc.id_contenedor = '1ab37ca7-67e2-4644-8b45-35e5ffd480f7'
   WHERE td.nombre IN ('Bill of Lading', 'DUA', 'Certificado sanitario')
   ORDER BY td.nombre;
```

- **Botón "Confirmar Entrega":**

   Registra la entrega del contenedor, actualiza estados y finaliza la operación asociada.

```sql
   -- Query 10.4: Registrar entrega
   
   -- Paso 1: Insertar entrega
   INSERT INTO monitoreo.Entrega (
       id_entrega,
       codigo,
       id_estado_entrega,
       fecha_entrega,
       lugar_entrega,
       id_contenedor,
       id_importador
   )
   VALUES (
       gen_random_uuid(),
       'ENT-2024-001',
       (SELECT id_estado_entrega FROM monitoreo.EstadoEntrega WHERE nombre = 'Completada'),
       '2024-11-04 14:30:00',
       'Av. Industrial 123, Parque Industrial, Manzanillo, Colima, México',
       '123e4567-e89b-12d3-a456-426614174000', -- ID contenedor
       '987fcdeb-51a2-43d7-9876-543210fedcba' -- ID Importadora Global S.A.
   )
   RETURNING id_entrega;
   -- Retorna el UUID de la nueva entrega registrada
   
   -- Paso 2: Actualizar estado del contenedor
   UPDATE monitoreo.Contenedor
   SET 
       id_estado_contenedor = (SELECT id_estado_contenedor FROM monitoreo.EstadoContenedor WHERE nombre = 'Entregado')
   WHERE id_contenedor = '123e4567-e89b-12d3-a456-426614174000';
   -- Marca el contenedor como entregado y no disponible
   
   -- Paso 3: Finalizar operación de monitoreo asociada
   UPDATE monitoreo.Operacion o
   SET 
       fecha_fin = CURRENT_TIMESTAMP,
       id_estado_operacion = (SELECT id_estado_operacion FROM monitoreo.EstadoOperacion WHERE nombre = 'Completada')
   FROM monitoreo.OperacionMonitoreo om
   LEFT JOIN monitoreo.OperacionMonitoreoVehiculo omv ON omv.id_operacion_monitoreo = om.id_operacion_monitoreo
   LEFT JOIN monitoreo.ContenedorVehiculo cv ON cv.id_vehiculo = omv.id_vehiculo
   WHERE o.id_operacion = om.id_operacion
       AND cv.id_contenedor = '123e4567-e89b-12d3-a456-426614174000'
       AND o.fecha_fin IS NULL;
   -- Finaliza la operación OP-2024-07-A5B2 asociada al contenedor
```

- **Visualización del historial de entregas (opcional):**

   Muestra las entregas realizadas previamente.
```sql
   -- Query 10.5: Listar entregas realizadas
   SELECT 
       e.id_entrega,
       e.codigo,
       e.fecha_entrega,
       e.lugar_entrega,
       ee.nombre AS estado,
       c.codigo AS contenedor,
       i.razon_social AS importador,
       i.ruc
   FROM monitoreo.Entrega e
   JOIN monitoreo.EstadoEntrega ee ON e.id_estado_entrega = ee.id_estado_entrega
   JOIN monitoreo.Contenedor c ON e.id_contenedor = c.id_contenedor
   JOIN monitoreo.Importador i ON e.id_importador = i.id_importador
   ORDER BY e.fecha_entrega DESC
   LIMIT 100;
```

- **Zona de carga de archivos:**

   Permite subir evidencias de entrega (fotos, documentos firmados, etc.).

- **Checkbox "Validación Final":**

   Debe marcarse para habilitar el botón "Confirmar Entrega".

- **Botón "Cancelar":**

   Descarta los cambios y regresa a la pantalla anterior sin registrar la entrega.

- **Botón eliminar (delete) en archivos cargados:**

   Elimina un archivo de evidencia de la lista antes de confirmar.

---
