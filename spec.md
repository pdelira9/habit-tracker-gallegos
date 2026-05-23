# Habit Tracker - Spec v0

## Objetivo

Crear una aplicación web para que una persona pueda crear hábitos, registrar su cumplimiento diario, ver progreso básico y recibir notificaciones push puntuales para no olvidarlos.

## Scope

### Sí entra

- Registro e inicio de sesión con email y contraseña (Supabase Auth). Cada usuario ve solo sus hábitos.
- Hábitos personales con nombre (obligatorio), descripción (opcional) y frecuencia diaria o semanal; si es semanal, meta de cumplimientos por semana (1–7).
- Creación, edición y archivado de hábitos (desactivar = archivar; el historial de check-ins se conserva; no se permiten nuevos check-ins en hábitos archivados).
- Registro diario: toggle cumplido / no cumplido solo para el día actual (se puede desmarcar el mismo día).
- Vista principal (`/`) con hábitos activos y estado del día (pendiente / cumplido).
- Progreso básico: racha actual por hábito (visible en la lista principal y en detalle del hábito).
- Recordatorios push: hora opcional por hábito; un aviso si a esa hora el hábito no está cumplido hoy (requiere permiso del navegador).

### No entra

- Analíticas avanzadas o reportes detallados.
- Pagos, planes comerciales o suscripciones.
- Colaboración entre usuarios o hábitos compartidos.
- Diseño visual premium o sistema de diseño completo.
- Automatizaciones complejas de recordatorios (múltiples ventanas, reglas condicionales, reintentos).
- Check-in de días pasados o futuros.
- OAuth, SMS, email como canal de recordatorio.

## Decisiones de producto

- **Día actual:** la fecha del check-in es `DATE` según la zona horaria del navegador del usuario.
- **Racha diaria:** días consecutivos calendario con check-in "cumplido", terminando hoy; un día sin cumplir rompe la racha.
- **Racha semanal:** semanas consecutivas (lunes–domingo, TZ del navegador) con al menos `target_per_week` días cumplidos; una semana por debajo de la meta rompe la racha.
- **Hábito nuevo:** racha mostrada = 0 hasta el primer cumplimiento.
- **Archivado:** reversible; el hábito desaparece de `/` y no acepta toggles; el historial previo permanece para consulta de racha en detalle.

## Criterios de aceptación

- Un usuario puede registrarse con email y contraseña, iniciar sesión y cerrar sesión; con credenciales inválidas ve un error y no entra.
- Un usuario autenticado puede crear y editar hábitos con nombre, descripción opcional y frecuencia diaria o semanal (con meta semanal si aplica).
- Un usuario autenticado puede archivar un hábito activo y este deja de aparecer en `/` sin borrar su historial previo.
- Un usuario puede marcar y desmarcar el cumplimiento de un hábito activo solo en el día actual; el estado persiste al recargar la página.
- Un usuario en `/` ve cada hábito activo como pendiente o cumplido para hoy y ve la racha actual de cada uno.
- Un usuario en el detalle de un hábito ve la racha actual y puede distinguir hábitos archivados (solo lectura de progreso, sin toggle).
- Un usuario que configuró hora de recordatorio en un hábito activo recibe un push a esa hora local si el hábito no está cumplido hoy; si ya cumplió o denegó permisos, no se envía push (o se informa en UI que los recordatorios están desactivados).
- Un usuario A no ve ni puede operar hábitos de un usuario B.

## No-goals

- Construir un producto comercial completo.
- Crear un sistema avanzado de estadísticas o predicción.
- Garantizar soporte responsive perfecto en todos los dispositivos.
- Alcanzar cobertura completa de tests automatizados.
- Integrar recordatorios por email, SMS u otros canales externos.
