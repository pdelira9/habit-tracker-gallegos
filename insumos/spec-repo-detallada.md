# Spec ÔÇö Habit Tracker

## Objetivo

Una app web (PWA instalable) para personas que quieren sostener h├íbitos diarios o semanales, registrar hecho/no-hecho del d├¡a y ver su racha, con un plan gratuito acotado y un plan premium de pago que desbloquea estad├¡sticas y mayor capacidad. Sin elementos sociales, sin gamificaci├│n agresiva, sin ranking.

## Scope

### S├¡ entra

- **Auth:** signup con email + contrase├▒a, login, logout, recuperaci├│n de contrase├▒a por email (todo v├¡a Supabase Auth). Sin OAuth, sin verificaci├│n de email obligatoria.
- **H├íbitos (CRUD + archivar/desarchivar):** cada h├íbito tiene `nombre` (1ÔÇô60 chars, trim, libre), `descripci├│n` (0ÔÇô280 chars, opcional), `frecuencia` (`diaria` o `semanal`), y si es semanal un `target_per_week` (1ÔÇô7). Archivar es soft-delete reversible v├¡a `archived_at`. Nombre ├║nico por usuario entre h├íbitos activos.
- **Registro diario:** toggle hecho/no-hecho del d├¡a actual por h├íbito, en la pantalla principal `/`. Estado binario, ├║nico por (h├íbito, d├¡a). La fecha se calcula en el cliente con la TZ del navegador y se guarda como `DATE`.
- **Vista de progreso:** racha actual y franja visual de los ├║ltimos 14 d├¡as (incluyendo hoy) por h├íbito, en `/habito/[id]`.
- **Estad├¡sticas (premium):** vista `/estadisticas` con % de cumplimiento de los ├║ltimos 30 d├¡as y mejor racha hist├│rica por h├íbito (incluye h├íbitos archivados, etiquetados como tales).
- **Recordatorios por email (premium y free):** cada h├íbito puede tener una hora opcional de recordatorio. Se env├¡a email a esa hora si el h├íbito no est├í marcado "hecho" del d├¡a.
- **Planes y pago:** Free hasta 3 h├íbitos activos sin acceso a `/estadisticas`. Premium hasta 30 h├íbitos activos con acceso a `/estadisticas`. Suscripci├│n mensual gestionada con Stripe Checkout + webhooks. P├ígina `/cuenta` muestra plan vigente y permite cancelar o reactivar.
- **Gamificaci├│n suave:** modal de celebraci├│n al cruzar racha de 7 y 30 d├¡as por primera vez en un h├íbito.
- **Compartir:** bot├│n "compartir racha" en `/habito/[id]` que invoca Web Share API con texto plano del logro.
- **Onboarding m├¡nimo:** una sola pantalla `/onboarding` post-signup con CTA "Crear tu primer h├íbito".
- **PWA:** manifest + service worker. La pantalla del d├¡a (`/`) es accesible offline en modo read-only mostrando el ├║ltimo estado sincronizado.
- **Persistencia:** Supabase Postgres con Row Level Security. Migraciones en `supabase/migrations/`. Dev y prod son dos proyectos Supabase distintos.
- **Stack:** Next.js 15 App Router, TypeScript estricto, Tailwind, Client Components + cliente Supabase + SWR.

### No entra

- OAuth (Google/Apple/GitHub) ni verificaci├│n de email obligatoria.
- Color, ├¡cono, categor├¡as, notas por d├¡a, reordenamiento de h├íbitos.
- Hard-delete de h├íbitos.
- Check-in de d├¡as pasados o futuros.
- Selector de zona horaria configurable.
- Estados intermedios (parcial, saltado).
- H├íbitos compartidos entre usuarios, feed social, ranking, comentarios.
- Badges, niveles, puntos, retos.
- Notificaciones push, SMS, ni mensajes motivacionales programados (solo emails de recordatorio configurados por h├íbito).
- Importar/exportar datos.
- Modo enfoque, heatmap anual.
- App nativa iOS/Android.
- Modo offline con escritura (solo read-only).
- Tests automatizados, optimizaci├│n de Core Web Vitals, WCAG AA completo, i18n.

## Criterios de aceptaci├│n

### Auth

1. Dado un visitante sin sesi├│n, cuando llena email y contrase├▒a v├ílidos en `/signup`, entonces queda autenticado, se le crea cuenta y aterriza en `/onboarding`.
2. Dado un visitante intentando `/signup` con un email ya registrado, cuando env├¡a el form, entonces ve el mensaje "Ese email ya tiene cuenta" y la cuenta no se duplica.
3. Dado un visitante sin sesi├│n, cuando ingresa credenciales v├ílidas en `/login`, entonces queda autenticado y aterriza en `/`.
4. Dado un visitante en `/login` que clickea "┬┐Olvidaste tu contrase├▒a?", cuando ingresa un email y env├¡a, entonces recibe un email con link de reset que al usarse permite definir nueva contrase├▒a en `/reset` y autenticarse.
5. Dado un usuario autenticado en cualquier ruta, cuando clickea "Cerrar sesi├│n" en el header, entonces se cierra la sesi├│n y se le redirige a `/login`.
6. Dado un usuario con sesi├│n expirada, cuando intenta una acci├│n autenticada, entonces se le redirige a `/login` con un toast "Tu sesi├│n expir├│, ingresa de nuevo".

### Onboarding

7. Dado un usuario que acaba de hacer signup, cuando aterriza en `/onboarding`, entonces ve una sola pantalla con texto introductorio y CTA "Crear tu primer h├íbito" que lo lleva al formulario de creaci├│n.
8. Dado un usuario que ya complet├│ signup en una sesi├│n previa, cuando hace login, entonces aterriza en `/` directamente, no en `/onboarding`.

### H├íbitos

9. Dado un usuario plan Free con 0ÔÇô2 h├íbitos activos, cuando crea un h├íbito con nombre "Leer" y frecuencia "diaria", entonces el h├íbito aparece en `/` sin recargar.
10. Dado un usuario plan Free con 3 h├íbitos activos, cuando intenta crear un cuarto, entonces ve un modal "Alcanzaste el l├¡mite de 3 h├íbitos. Sube a Premium para crear m├ís" con CTA a `/cuenta`.
11. Dado un usuario plan Premium con 0ÔÇô29 h├íbitos activos, cuando crea uno m├ís, entonces se crea exitosamente.
12. Dado un usuario plan Premium con 30 h├íbitos activos, cuando intenta crear otro, entonces ve "Alcanzaste el l├¡mite de 30 h├íbitos activos" y no se crea.
13. Dado un usuario con un h├íbito existente, cuando edita su nombre, descripci├│n, frecuencia o target_per_week dentro de los l├¡mites de validaci├│n, entonces los cambios se persisten y se reflejan al recargar.
14. Dado un usuario que intenta crear o renombrar un h├íbito con el mismo nombre que otro h├íbito activo suyo, cuando env├¡a el form, entonces ve "Ya tienes un h├íbito activo con ese nombre" y la operaci├│n se rechaza.
15. Dado un usuario que archiva un h├íbito, cuando vuelve a `/`, entonces el h├íbito ya no aparece en la lista del d├¡a; sigue visible en `/archivados`.
16. Dado un h├íbito archivado en `/archivados`, cuando el usuario clickea "Desarchivar", entonces vuelve a aparecer en `/` y acepta toggles nuevos.
17. Dado un usuario que intenta marcar hecho/no-hecho un h├íbito archivado v├¡a URL directa o API, cuando lo intenta, entonces la operaci├│n se rechaza con error 400 "H├íbito archivado".

### Registro diario y racha

18. Dado un h├íbito activo en `/`, cuando el usuario hace toggle a "hecho", entonces el estado se persiste y se mantiene "hecho" al recargar la p├ígina en Ôëñ5 segundos y al iniciar sesi├│n desde otro dispositivo.
19. Dado un h├íbito diario reci├®n creado sin ning├║n check-in, entonces la racha mostrada es 0 con la etiqueta "Empieza hoy".
20. Dado un h├íbito diario con check-in "hecho" en cada uno de los ├║ltimos N d├¡as consecutivos terminando hoy, entonces la racha mostrada es N. Si existe un d├¡a sin "hecho" entre hoy y el ├║ltimo "hecho", la racha es 0.
21. Dado un h├íbito semanal con `target_per_week = T`, entonces la racha cuenta el n├║mero de semanas consecutivas (terminando en la semana actual o anterior) en las que el usuario alcanz├│ ÔëÑT check-ins "hecho". Una semana sin ÔëÑT rompe la racha.
22. Dado un h├íbito con historial, cuando el usuario entra a `/habito/[id]`, entonces ve una franja de 14 celdas incluyendo hoy: celda verde = hecho, roja = no-hecho, gris vac├¡a = anterior a `created_at`.
23. Dado un h├íbito que cruza por primera vez racha de 7 d├¡as, cuando el toggle del d├¡a 7 se completa, entonces aparece un modal de celebraci├│n "┬íRacha de 7!" descartable. Lo mismo para racha 30.

### Estad├¡sticas y plan

24. Dado un usuario Free que entra a `/estadisticas`, entonces ve una pantalla "Estad├¡sticas es premium" con CTA a `/cuenta` para suscribirse.
25. Dado un usuario Premium en `/estadisticas`, entonces ve, por cada h├íbito activo y archivado: nombre, % cumplimiento ├║ltimos 30 d├¡as (definido abajo), mejor racha hist├│rica.
26. Para un h├íbito diario, el % cumplimiento = `d├¡as_hechos_en_ventana / d├¡as_activos_en_ventana`. "D├¡as activos en ventana" = d├¡as entre `MAX(created_at, hoy ÔêÆ 29)` y `MIN(archived_at ÔêÆ 1 d├¡a, hoy)`, ambos inclusive.
27. Para un h├íbito semanal con target T, el % cumplimiento = `semanas_con_ÔëÑT_hechos / semanas_activas_en_ventana`, con la misma regla de acotaci├│n por `created_at` y `archived_at`.
28. Para un h├íbito archivado en `/estadisticas`, se muestra la etiqueta "Archivado" junto al nombre y el % se calcula solo sobre su per├¡odo activo dentro de la ventana.
29. Dado un usuario Free en `/cuenta` que clickea "Activar Premium", cuando completa el flujo de Stripe Checkout con tarjeta v├ílida, entonces tras el redirect vuelve a `/cuenta` y ve plan = Premium en Ôëñ10 segundos (tras webhook).
30. Dado un usuario Premium en `/cuenta` que cancela, cuando confirma, entonces el plan se marca "Premium hasta DD/MM/YYYY" y al expirar el periodo se convierte en Free; si ten├¡a >3 h├íbitos activos, el exceso queda en read-only (visible pero sin poder toggle) hasta que archive o reduzca a 3 o reactive el plan.

### Recordatorios

31. Dado un usuario que edita un h├íbito y le pone hora de recordatorio "08:00", cuando llega esa hora local y el h├íbito no est├í marcado "hecho" del d├¡a, entonces recibe un email con asunto "Recordatorio: [nombre del h├íbito]" y link a la app. Si ya est├í "hecho", no se env├¡a email.
32. Dado un h├íbito sin hora de recordatorio configurada, entonces nunca se env├¡an emails para ese h├íbito.

### Aislamiento entre usuarios (UI)

33. Dado un usuario A y un usuario B con h├íbitos creados, cuando A inicia sesi├│n y abre `/`, `/estadisticas`, `/archivados` o cualquier `/habito/[id]`, entonces solo ve h├íbitos creados por A; intentar abrir `/habito/[id]` con un id de B muestra 404.

### Compartir y PWA

34. Dado un usuario en `/habito/[id]` con racha ÔëÑ1, cuando clickea "Compartir racha", entonces se invoca Web Share API con texto "Llevo N d├¡as con [nombre del h├íbito]" (si el navegador no soporta Web Share, el bot├│n no aparece).
35. Dado un usuario que visit├│ `/` al menos una vez con sesi├│n activa, cuando pierde conexi├│n y entra a `/`, entonces ve la lista de h├íbitos del d├¡a y su ├║ltimo estado sincronizado en modo read-only, con banner "Sin conexi├│n".
36. Dado un usuario en navegador compatible, cuando entra a la app por primera vez, entonces puede instalarla como PWA usando el prompt nativo del navegador.

### Errores

37. Dado un toggle, creaci├│n o edici├│n que falla por red o servidor, cuando el usuario lo intenta, entonces aparece un toast no-bloqueante "No se pudo guardar, intenta de nuevo" y la UI no queda en estado inconsistente.

## No-goals

- OAuth, verificaci├│n de email obligatoria, recuperaci├│n por SMS.
- Hard-delete, color, ├¡cono, descripci├│n >280 chars, reordenamiento.
- Check-in de d├¡as pasados o futuros, estados intermedios, TZ configurable.
- M├ís de 30 h├íbitos activos por usuario.
- H├íbitos compartidos entre usuarios, feed social, ranking, comentarios, perfiles p├║blicos.
- Badges, niveles, puntos, retos, recompensas materiales.
- Notificaciones push, SMS, mensajes motivacionales programados.
- Importar/exportar datos en CSV/JSON.
- Modo enfoque, heatmap anual, categor├¡as, filtros, etiquetas, notas por d├¡a.
- App nativa iOS/Android.
- Modo offline con escritura.
- Multi-idioma (la app es solo en espa├▒ol).
- Tests automatizados (unit, integraci├│n, e2e).
- Optimizaci├│n avanzada de Core Web Vitals, SSR/ISR estrat├®gico.
- Accesibilidad WCAG AA completa, navegaci├│n por teclado avanzada.
- M├ís de un plan de pago (solo Free y Premium mensual; no anual, no equipos, no familia).

## Pruebas t├®cnicas fuera de QA manual

Estas pruebas requieren acceso a la base de datos o a herramientas de desarrollo; no las realiza el QA de UI:

- **RLS aislamiento:** autenticarse como usuario A v├¡a el cliente Supabase y ejecutar `select * from habits` y `select * from checkins`; debe devolver solo filas de A. Repetir con `service_role` desde un script para verificar que las policies existen y son `using (auth.uid() = user_id)`.
- **Webhook de Stripe:** simular eventos `checkout.session.completed`, `customer.subscription.deleted`, `customer.subscription.updated` con la CLI de Stripe contra el endpoint local y verificar que la tabla `subscriptions` se actualiza correctamente.
- **Unicidad y constraints:** intentar insertar dos checkins para `(habit_id, date)` y dos h├íbitos activos con el mismo `(user_id, name)`; ambos deben fallar con error de constraint.
- **Job de recordatorios:** verificar que el cron (Supabase Edge Function o equivalente) que dispara emails se ejecuta a la cadencia esperada y que respeta la hora local del usuario.

## Decisiones tomadas en entrevista

### Ronda 1 ÔÇö Datos
- Frecuencia semanal = N veces por semana; columna `habits.target_per_week INT` (1ÔÇô7).
- Check-in = `checkins(habit_id, date DATE, done BOOL)`, UNIQUE(habit_id, date); toggle = UPSERT.
- Archivado = `habits.archived_at TIMESTAMPTZ NULL`; reversible.
- Unicidad: `UNIQUE(user_id, name) WHERE archived_at IS NULL` + `UNIQUE(habit_id, date)`.
- Mejor racha persistida en `habits.best_streak INT`, actualizada por la action de toggle.
- Checkins de h├íbitos archivados se conservan; no se aceptan toggles nuevos; archivado es reversible.
- Fecha = `DATE` calculada en el cliente con la TZ del navegador.

### Ronda 2 ÔÇö QA
- Sincron├¡a multi-dispositivo: Ôëñ5s al recargar, sin caching agresivo.
- Racha inicial = 0 con etiqueta UI "Empieza hoy" para h├íbitos sin checkins.
- Franja: 14 celdas incluyendo hoy; d├¡as previos a `created_at` = gris vac├¡o.
- % cumplimiento 30d definido por h├íbito diario (d├¡as hechos / d├¡as activos en ventana) y semanal (semanas con ÔëÑtarget / semanas activas en ventana), acotado por `created_at` y `archived_at`.
- Archivados en stats: siempre visibles, etiquetados, denominador acotado por `archived_at`.
- Aislamiento de usuarios reformulado como criterio de UI observable; RLS profunda va a "Pruebas t├®cnicas fuera de QA manual".
- Signup, logout y reset password con criterios expl├¡citos.

### Ronda 3 ÔÇö Developer
- Client Components + cliente Supabase + SWR (no RSC + Server Actions).
- Rutas: `/`, `/login`, `/signup`, `/reset`, `/onboarding`, `/estadisticas`, `/habito/[id]`, `/archivados`, `/cuenta`.
- Errores: toast no-bloqueante + reintento manual; sesi├│n expirada redirige a `/login` con toast.
- Validaci├│n: nombre 1ÔÇô60 (trim, libre), descripci├│n 0ÔÇô280, target 1ÔÇô7, m├íx 30 h├íbitos activos; cliente y servidor.
- Init: `create-next-app@latest --ts --tailwind --app --src-dir` + Supabase CLI; migraciones en `supabase/migrations/`; dev y prod = dos proyectos Supabase.
- Env: `NEXT_PUBLIC_SUPABASE_URL`, `NEXT_PUBLIC_SUPABASE_ANON_KEY` (m├ís las claves de Stripe necesarias para Checkout y webhooks); sin seeds.

### Ronda 4 ÔÇö Scope vs brief (brief modificado)
- PWA m├¡nima (instalable + offline read-only) ÔÇö agregada al brief.
- Stripe + plan premium ÔÇö agregado al brief.
- Paywall: estad├¡sticas premium + l├¡mite 3 h├íbitos free / 30 premium ÔÇö agregado al brief.
- Gamificaci├│n suave (celebraci├│n rachas 7 y 30) ÔÇö agregada al brief.
- Recordatorios por email como segunda extensi├│n ÔÇö agregada al brief (regla "m├íx 1 extensi├│n" levantada).
- Compartir nativo (Web Share API + OG tags) ÔÇö agregada al brief.
- Onboarding m├¡nimo (1 pantalla) ÔÇö declarado permitido en el brief.
- Animaciones ÔÇö restricci├│n levantada en el brief.
- Se mantienen como no-goals: app nativa, push, h├íbitos compartidos, ranking social, categor├¡as, notas, modo enfoque, exportar, offline con escritura, i18n, WCAG AA, tests automatizados.
