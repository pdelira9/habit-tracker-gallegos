








Modelo de datos
1. ¿Los hábitos semanales tienen días específicos asignados o simplemente requieren X check-ins en cualquier día de la semana?

A) Días fijos: "lunes, miércoles, viernes"
B) Frecuencia libre: "3 veces esta semana, cuando quieras"

Mi selección B) Frecuencia libre: "3 veces esta semana, cuando quieras" ya que es mas flexible

2. ¿Un hábito borrado elimina todo su historial de check-ins o lo conserva (aunque ya no aparezca en la lista activa)?

A) Borrado duro: desaparece todo, incluyendo historial
B) Borrado suave: el hábito se archiva y el historial permanece

Mi selección A) Borrado duro: desaparece todo, incluyendo historial porque no es necesario mantener lo que no se necesita


Reglas del check-in
3. ¿Se puede deshacer un check-in del mismo día o es definitivo una vez marcado?

A) Se puede desmarcar mientras sea el mismo día
B) Una vez marcado, es permanente

Mi selección A) Se puede desmarcar mientras sea el mismo día, ya que habrá veces que comenzaras pero por alguna cuestión mayor no terminaras y puedes continuar el mismo día más tarde.

4. ¿Se permite registrar un check-in de un día pasado (retroactivo)?

A) No, solo el día actual
B) Sí, pero con un límite (ej. hasta 24 horas atrás)
C) Sí, cualquier día pasado sin restricción

Mi selección A) No, solo el día actual, para mejor control de metas y objetivos.


Cálculo de la racha
5. Para un hábito diario, ¿qué rompe la racha: fallar cualquier día del calendario, o fallar cualquier día en que el usuario abrió la app?

A) El calendario manda: sin check-in ayer = racha rota
B) Solo rompe si el usuario estuvo activo y no lo hizo

Mi selección A) El calendario manda: sin check-in ayer = racha rota, para mejor historial.

6. ¿La racha se mide en días consecutivos de check-in o en porcentaje de cumplimiento sobre una ventana de tiempo?

A) Racha clásica: días consecutivos sin fallo
B) Racha flexible: cumplir al menos X% en los últimos 7 días

Mi selección A) Racha clásica: días consecutivos sin fallo, que sea un dato claro y medible.


Edición de hábitos
7. Si el usuario edita la frecuencia de un hábito (ej. pasa de diario a semanal), ¿el historial anterior se recalcula o queda congelado como estaba?

A) Recalcular todo: el historial se reinterpreta con la nueva regla
B) Congelar: el historial previo es inmutable, la nueva regla aplica desde hoy

Mi selección B) Congelar: el historial previo es inmutable, la nueva regla aplica desde hoy, esto para tener datos claros de su progreso.


Extensión: Recordatorios push
8. ¿El recordatorio se configura por hábito individual o es un recordatorio global para todos los hábitos pendientes del día?

A) Por hábito: cada uno tiene su hora específica (ej. agua a las 9am, caminar cada 30 min)
B) Global: una sola notificación al día que lista los hábitos pendientes

Mi selección A) Por hábito: cada uno tiene su hora específica (ej. agua a las 9am, caminar cada 30 min), porque habrá halgun haábito con seguimiento frecuente y otro solo una vez al día.

9. Para el caso de "caminar cada 30 minutos", ¿los recordatorios repetitivos tienen un horario de silencio o funcionan 24/7 mientras el hábito esté activo?

A) El usuario define ventana activa: "solo entre 8am y 8pm"
B) Se disparan siempre, sin restricción de horario

Mi selección B) Se disparan siempre, sin restricción de horario, porque puede ser en la oficina, trabajo, escuela.

10. Si el usuario ya hizo el check-in del día, ¿los recordatorios de ese hábito se cancelan automáticamente o siguen sonando independientemente?

A) Se cancelan en cuanto hay check-in registrado
B) Siguen el horario programado sin importar el check-in

Mi selección A) Se cancelan en cuanto hay check-in registrado porque ya se completó.





Crítica de spec — Habit Tracker v1.1

Perspectiva 1 — Senior en code review
Problema 1 — CA #10: "medianoche según la zona horaria del servidor" es una decisión técnica implícita disfrazada de criterio.
El spec dice que el check-in es desmarcable "antes de que termine el día (medianoche según la zona horaria del servidor)". Si el servidor está en UTC y el usuario en UTC-6 (Ciudad de México), el "día" del servidor termina a las 6pm hora local. Eso rompe la promesa funcional. El spec no dice si el día se calcula en zona del servidor, zona del usuario, o UTC fijo — y las tres producen comportamientos distintos.
ARREGLAR UTC general

Problema 2 — CA #14: La semana no tiene fecha de inicio definida.
"Semana consecutiva cumplida" no especifica si la semana empieza el lunes o el domingo. Eso afecta directamente el cálculo de racha: un usuario que hace check-in el domingo puede pertenecer a semanas distintas dependiendo de la convención. El spec lo deja implícito.
ARREGLAR LUN-DOM

Problema 3 — CA #17: No hay comportamiento definido para dispositivos múltiples.
"Se envía una notificación push al dispositivo del usuario" — si el usuario tiene la app en teléfono y tablet, ¿se envía a todos los dispositivos registrados, solo al último activo, o a uno cualquiera? El spec no lo define y eso cambia el modelo de datos de tokens push.
ARREGLAR a todos


Problema 4 — CA #18 y #19: No hay comportamiento definido si el check-in falla en el servidor.
CA #18 dice que los recordatorios se cancelan "cuando el check-in queda guardado". CA #19 asume que el desmarcar también es una operación exitosa. Pero si hay error de red y el check-in no se persiste, ¿el cliente cancela los recordatorios de todas formas? El spec no tiene criterio para el estado de fallo.
ARREGLAR

Problema 5 — Scope: "notificaciones push" no define el mecanismo, y el mecanismo importa para el alcance.
Push nativa (Web Push API / service worker) requiere HTTPS, permiso explícito, y no funciona en iOS Safari en condiciones específicas. Push vía servicio tercero (OneSignal, etc.) tiene dependencias externas. El spec asume que "push" es trivial, pero en Next.js + Vercel la implementación varía radicalmente. Esto no es un detalle técnico menor — afecta si la extensión es buildeable en 3 horas.
ARREGLAR

Perspectiva 2 — QA que va a probar la app
Problema 6 — CA #9: "muestra visualmente que fue completado" no es verificable.
¿Qué significa "visualmente"? ¿Un ícono de palomita? ¿Un color de fondo? ¿Un texto "Completado"? Un caso de prueba no puede afirmar pasa/falla sin saber qué buscar en pantalla. Es el único criterio visual que no tiene comportamiento observable concreto.
ARREGLAR

Problema 7 — CA #12: "sin contar el día de hoy si todavía no se ha marcado" introduce una condición no testeada en sentido contrario.
El criterio dice que si hoy NO está marcado, no cuenta. Pero no define qué muestra la racha si hoy SÍ está marcado. ¿Cuenta el día de hoy en la racha actual? Un QA no puede escribir el caso de prueba "usuario con check-in hoy: racha = X" porque el spec no lo especifica.
ARREGLAR, SI ESTA MARCADO HOY, HOY YA CUENTA PARA LA RACHA


Problema 8 — CA #16: "el sistema queda registrado" no tiene resultado observable verificable.
El criterio termina con que el sistema "queda registrado". Un QA no puede verificar el estado interno del scheduler. El criterio necesita un resultado observable: ¿aparece algo en la UI confirmando que el recordatorio fue guardado? ¿El usuario ve la hora configurada reflejada en algún lugar?
ARREGLAR, Un aviso de hábito creado

Problema 9 — CA #6: "los cambios se reflejan de inmediato en la lista" mezcla dos assertions en un solo criterio.
El criterio tiene dos resultados: (a) cambios reflejados en la lista, y (b) historial previo inmutable con nueva regla desde hoy. Un QA necesita dos casos de prueba separados — uno que verifica la UI y otro que verifica el comportamiento histórico. Mezclados, uno puede pasar y el otro fallar sin que el criterio lo capture.
ARREGLAR , separando las peubeas, independientes

Perspectiva 3 — Implementador sin acceso al autor
Problema 10 — CA #14: "todas las semanas anteriores sin interrupción" no define desde cuándo.
¿Desde que se creó el hábito? ¿Desde el primer check-in? Si el usuario crea un hábito el miércoles y esa semana solo hace 2 check-ins de 3 requeridos, ¿esa semana parcial cuenta como fallo o se omite? El implementador va a tener que inventar esta regla.
ARREGLAR Desde el primer check in, semana es se cumple o no

Problema 11 — Scope: La descripción del hábito no tiene restricciones definidas.
El Scope dice "nombre, descripción, frecuencia". No hay mínimos ni máximos de caracteres para ninguno de los tres campos. El implementador va a poner límites arbitrarios (o no poner ninguno). Si el autor tiene un criterio, no está en el spec.
ARREGLAR

Problema 12 — CA #21: "la app solicita el permiso" no define qué pasa si el usuario rechaza.
Si el usuario ve el diálogo del sistema operativo y presiona "Bloquear", ¿qué hace la app? ¿Guarda el hábito sin recordatorio? ¿Muestra un mensaje de error? ¿Impide crear el recordatorio permanentemente? El implementador no tiene instrucción para este flujo, que es el caso de error más probable.
ARREGLAR muestra un mesaje de error

A. Los 4 elementos obligatorios

A.1 Objetivo en una sola línea
⚠ Cumple parcialmente

“Aplicación web de seguimiento de hábitos que permite a cualquier persona crear hábitos, registrar su cumplimiento diario y recibir notificaciones push puntuales…”

No es una sola línea (es una oración larga con múltiples partes).
Incluye producto y audiencia (“cualquier persona”), eso sí cumple.

A.2 Scope con lista de qué SÍ entra
✓ Cumple

A.3 Criterios de aceptación numerados
✓ Cumple

A.4 No-goals con mínimo 4 ítems
✓ Cumple (tienes 6)

B. Cobertura del núcleo del proyecto

B.1 Autenticación (registro, login, logout)
✓ Cumple

B.2 CRUD de hábitos
✓ Cumple

B.3 Check-in completo (incluye deshacer y comportamiento)
⚠ Cumple parcialmente

Falta cobertura explícita para diferencia diaria vs semanal en check-in.

Ejemplo de lo que sí tienes:

“Check-in diario por hábito”
“Desmarcar un check-in únicamente durante el día actual”

Pero:

No hay criterio que describa cómo se comporta el check-in en hábitos semanales (más allá de contarlos).
Solo se describe comportamiento operativo, no interacción específica distinta por tipo.

B.4 Vista de progreso (qué se muestra)
✓ Cumple

B.5 Cálculo de racha (qué la rompe / mantiene)
✓ Cumple

B.6 Criterios para la extensión elegida
⚠ Cumple parcialmente

Extensión implícita: recordatorios push.

Sí hay criterios (21–27), pero:

No está declarada explícitamente como “extensión”.
Está integrada como parte del core.
C. Verificabilidad

C.1 Cada criterio es verificable (sí/no)
✓ Cumple

C.2 Sin palabras subjetivas
✓ Cumple

C.3 Describe comportamiento observable, no intención interna
✓ Cumple

D. Decisiones tomadas

D.1 Sin frases ambiguas (“ya veremos”)
✓ Cumple

D.2 Sin etiquetas sin resolver
✓ Cumple

D.3 Casos extremos cubiertos
⚠ Cumple parcialmente

Sí cubres varios:

Sin conexión:

“Dado que ocurre un error de red…”

Eliminación con historial:

“todos sus check-ins asociados se eliminan permanentemente”

Datos inválidos:

validación de nombre

Pero faltan algunos casos límite:

Qué pasa si no hay dispositivos push registrados
Qué pasa si el usuario nunca hace su primer check-in en semanal (inicio de evaluación depende de eso)
Qué pasa si frecuencia cambia antes de primer check-in
E. Scope realista

E.1 La extensión es exactamente UNA
⚠ Cumple parcialmente

Hay una extensión clara (push notifications), pero:
No está separada como tal, parece parte del core.

E.2 No hay features fuera del núcleo + extensión
✓ Cumple

E.3 No-goals cubren tentaciones típicas
✓ Cumple

Ejemplos correctos:

“Registro retroactivo de check-ins”
“Compartir hábitos”
“Planes de pago”