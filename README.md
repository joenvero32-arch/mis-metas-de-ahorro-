# MIS METAS DE AHORRO

Aplicación independiente para administrar metas y aportes de ahorro personales.

## Estado

- Repositorio independiente de Santo Pecado 17-29 y Parqueadero Santo Cristo.
- Backend Supabase independiente: `thbpburglgxtltesougc`.
- Tablas preparadas con Row Level Security (RLS).
- Publicación web, autenticación y sincronización entre dispositivos pendientes de configuración y pruebas.

## Próximos pasos

1. Incorporar los archivos de la aplicación.
2. Revisar configuración de Supabase y políticas RLS.
3. Configurar GitHub Pages desde `main`.
4. Añadir la URL publicada a los redirects permitidos de Supabase Auth.
5. Probar registro, inicio de sesión y sincronización.

## Seguridad

No subir claves secretas ni credenciales de servidor. En el cliente solo debe usarse una publishable key. Nunca incluir una clave `service_role`.

Las notificaciones push requieren configuración adicional y no están activas todavía.
