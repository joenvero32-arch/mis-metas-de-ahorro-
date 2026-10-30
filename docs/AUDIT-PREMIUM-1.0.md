# Auditoría técnica — Mis Metas de Ahorro Premium 1.0

Fecha: 2026-10-30
Alcance: repositorio `joenvero32-arch/mis-metas-de-ahorro-` y proyecto Supabase independiente `MIS METAS DE AHORRO`.
Rama de trabajo: `premium-1.0-audit`.
Estado: auditoría de solo lectura. No se modificaron datos, tablas, políticas ni la rama `main`.

## 1. Integridad de los datos

Consulta de solo lectura a las tablas existentes:

- Metas: 1
- Aportes: 3
- Total de aportes completados: COP $459.000
- Objetivo de la meta existente: COP $10.000.000
- Fecha objetivo: 2026-12-31

Movimientos conciliados:
- COP $159.000 — nota: “Ahorro diario recomendado”
- COP $200.000 — nota: “Prueba de ahorro”
- COP $100.000 — sin nota

**Saldo oficial confirmado por el propietario: COP $459.000.** No volver a registrar estos movimientos ni modificar sus importes durante las mejoras.

## 2. Repositorio y aplicación

- Aplicación web de una sola página en `index.html` (aprox. 64 KB).
- Archivos revisados: `index.html`, `manifest.webmanifest`, `sw.js`, `README.md`, `logo-mis-metas.svg`.
- Último commit observado en `main`: `f94177b6176b7c2966fe0b2d3c5c15a2be680048` (“Format COP amount inputs with Colombian thousands separators”).
- Rama de trabajo creada desde `main`: `premium-1.0-audit`.
- El sitio usa Supabase JS v2 desde CDN y una publishable key en el cliente; no se observó una service_role key en el fragmento revisado.

### Funciones ya presentes

- Registro, inicio de sesión y recuperación de contraseña.
- Crear metas y aportes.
- Resumen global, progreso, gráficas configurables y calendario diario.
- Cuota recomendada y estrategias de ahorro.
- Temas visuales y selectores propios.
- Configuración de recordatorios.

### Funciones que faltan o están incompletas

- No hay edición de metas.
- No hay vista de historial financiero con detalle por movimiento.
- No hay edición/corrección controlada de aportes.
- No hay exportación PDF ni Excel.
- No hay registro de auditoría de acciones.
- No se encontró una rutina de respaldo desde la aplicación.
- No hay integración de Push API ni registro de Service Worker en el código actual.
- Los recordatorios usan `setTimeout` y Notification API; el propio texto de la interfaz indica que solo funcionan con la aplicación abierta.
- El calendario registra nuevos aportes; no ofrece historial completo de movimientos.

## 3. Supabase

Proyecto independiente: `thbpburglgxtltesougc`, estado reportado `ACTIVE_HEALTHY`.

Tablas encontradas:
- `savings_goals`
- `savings_contributions`
- `savings_reminder_settings`
- `savings_push_subscriptions`

Las cuatro tablas tienen RLS habilitado y políticas para limitar filas al usuario autenticado. La relación entre aportes y metas tiene `ON DELETE CASCADE`: eliminar una meta también elimina sus aportes asociados. La interfaz actual lo confirma explícitamente; antes de añadir edición/archivado debe protegerse el historial.

No se encontraron Edge Functions ni buckets de Storage. La lista de migraciones disponible devolvió cero migraciones registradas.

### Hallazgos de seguridad

- El asesor de Supabase reporta `auth_leaked_password_protection`: protección de contraseñas filtradas desactivada.
- Se observaron privilegios SQL concedidos a los roles `anon` y `authenticated` en las cuatro tablas, incluidos privilegios amplios. Las políticas RLS restringen las filas al usuario autenticado, pero conviene aplicar mínimo privilegio y revisar expresamente permisos que RLS no filtra, como TRUNCATE, antes de producción.
- RLS está habilitado, pero `FORCE ROW LEVEL SECURITY` no está habilitado. Revisar el modelo de roles y el uso del propietario de tablas antes de cambiarlo.
- No se pudo verificar la configuración completa de Auth, redirects, MFA, SMTP ni el estado real de backups desde las herramientas disponibles.
- El asesor de rendimiento señala un índice no utilizado en `savings_push_subscriptions`; no eliminarlo sin medir uso y revisar la futura implementación push.

## 4. Riesgos funcionales que deben abordarse

1. La eliminación de una meta puede borrar en cascada su historial de aportes. Priorizar archivado/desactivación y confirmación reforzada; no borrar movimientos financieros por defecto.
2. Los recordatorios actuales no son notificaciones en segundo plano.
3. El Service Worker existente usa caché versionada y elimina otras cachés del mismo origen al activarse; revisar su alcance antes de registrarlo.
4. La tabla de aportes tiene identificadores y marcas de tiempo útiles para construir historial, pero la consulta actual de la interfaz no los selecciona.
5. La ausencia de migraciones registradas dificulta reproducir cambios de esquema; establecer migraciones versionadas antes de ampliar el backend.

## 5. Plan recomendado de implementación

1. Protección de datos: respaldo verificable y política de no eliminación de movimientos.
2. Administración de metas: editar nombre, objetivo, fecha, categoría y estado sin tocar aportes.
3. Historial: consulta por meta, fecha y rango; correcciones auditadas y reversos sin borrar físicamente.
4. Informes: PDF y XLSX con conciliación de totales.
5. Notificaciones push: Service Worker, suscripciones y envío seguro desde backend; nunca exponer secretos en el navegador.
6. Seguridad: mínimo privilegio, Auth endurecido, auditoría, respaldos y restauración probada.
7. Diseño Premium: mejoras visuales y accesibilidad sin alterar reglas financieras.

## 6. Controles para cada publicación

- Trabajar primero en `premium-1.0-audit`, no directamente en `main`.
- Revisar diff y ejecutar validaciones estáticas antes de fusionar.
- Probar autenticación, sincronización, creación de meta y aporte, saldos, calendario y móvil.
- Conciliar el total de aportes antes y después de cada cambio: debe seguir siendo COP $459.000 hasta que el propietario registre nuevos movimientos.
- No declarar una publicación terminada sin verificar el commit publicado y el comportamiento del sitio.

## 7. Limitaciones de esta auditoría

La apertura directa del sitio publicado no pudo verificarse con la herramienta web disponible. Tampoco fue posible leer toda la configuración de Supabase Auth ni comprobar restauración real de backups. Esos puntos quedan marcados como pendientes de verificación, no como aprobados.
