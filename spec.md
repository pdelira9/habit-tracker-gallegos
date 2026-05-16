# Habit Tracker — Spec v1.1

---

## Objetivo

Aplicación web de seguimiento de hábitos que permite a cualquier persona crear hábitos, registrar su cumplimiento diario y recibir notificaciones push puntuales para no olvidar realizarlos.

---

## Scope

### Qué SÍ entra en este proyecto

-Registro e inicio de sesión con email y contraseña
-Creación, edición y eliminación de hábitos
-Hábitos con:
---nombre
---descripción
---frecuencia diaria
---frecuencia semanal con número objetivo de check-ins
-Check-in diario por hábito
-Desmarcar un check-in únicamente durante el día actual
-Vista de progreso con racha actual
-Recordatorios push configurables por hábito
-Cancelación automática de recordatorios después de un check-in exitoso
-Reactivación de recordatorios si el check-in del día es desmarcado

# Restricciones funcionales
Todos los cálculos de fecha y hora se realizan usando UTC.
La semana se define de lunes 00:00 UTC a domingo 23:59:59 UTC.
Los hábitos semanales requieren N check-ins libres dentro de la semana; no existen días fijos obligatorios.
La primera semana evaluable de un hábito semanal comienza en la semana donde ocurre el primer check-in del hábito.
# Restricciones de campos
-Nombre del hábito
Obligatorio
Mínimo: 1 carácter
Máximo: 100 caracteres
-Descripción del hábito
Opcional
Máximo: 500 caracteres
-Frecuencia semanal
Mínimo: 1 check-in por semana
Máximo: 7 check-ins por semana
# Push notifications
Las notificaciones push se implementan mediante Web Push API usando Service Workers.
La aplicación requiere HTTPS para funcionamiento de push notifications.
Cada dispositivo registrado guarda su propio token push.
Las notificaciones se envían a todos los dispositivos activos registrados del usuario.

### Qué NO entra en este proyecto

- Recordatorios por email o SMS
- Hábitos con días específicos de la semana asignados
- Registro retroactivo de check-ins de días anteriores
- Historial o estadísticas de hábitos eliminados
- Recálculo del historial al editar la frecuencia de un hábito
- Compartir hábitos o progreso con otros usuarios
- Planes de pago o suscripciones

---

## Criterios de aceptación

### Autenticación

1. Dado que el usuario no tiene cuenta, cuando completa el formulario de registro con email y contraseña válidos y lo envía entonces se crea su cuenta y queda autenticado en la app.

2. Dado que el usuario tiene cuenta, cuando ingresa su email y contraseña correctos y los envía, entonces accede a su panel de hábitos.

3. Dado que el usuario está autenticado, cuando ejecuta la acción de cerrar sesión, entonces la sesión termina y es redirigido a la pantalla de inicio de sesión.

4. Dado que un usuario no autenticado intenta acceder a una ruta protegida, cuando la app evalúa su sesión, entonces es redirigido a la pantalla de inicio de sesión.

### CRUD de hábitos

5. Dado que el usuario está autenticado, cuando crea un hábito con nombre válido, descripción válida y frecuencia válida, entonces el hábito aparece en su lista de hábitos activos.

6. Dado que el usuario intenta crear un hábito sin nombre, cuando intenta guardar el formulario, entonces la app muestra un mensaje de validación indicando que el nombre es obligatorio y el hábito no se crea.

7. Dado que el usuario tiene un hábito creado, cuando modifica nombre, descripción o frecuencia y guarda los cambios, entonces la lista de hábitos refleja inmediatamente los nuevos valores.

8. Dado que el usuario modifica la frecuencia de un hábito, cuando consulta check-ins anteriores a la fecha de edición, entonces el historial previo permanece sin cambios y la nueva frecuencia aplica únicamente desde la fecha de edición en adelante.

9. Eliminar hábito

Dado que el usuario tiene un hábito creado, cuando lo elimina y confirma la acción, entonces:
-el hábito desaparece de la lista
-todos sus check-ins asociados se eliminan permanentemente

### Check-in

10. Dado que el usuario tiene un hábito sin check-in en la fecha UTC actual, cuando marca el check-in del día, entonces:
-el hábito cambia visualmente al estado “Completado”
-se muestra un indicador visible de completado
-el check-in queda registrado para la fecha UTC actual
-el check-in cuenta como uno de los N requeridos dentro de la semana UTC actual

11. Dado que el usuario registró un check-in hoy, cuando lo desmarca antes de las 23:59:59 UTC del mismo día, entonces el hábito vuelve al estado “Pendiente”.

12. Dado que ya comenzó un nuevo día UTC, cuando el usuario intenta registrar un check-in para una fecha anterior, entonces la app no ofrece esa opción ni permite guardar check-ins retroactivos.

13. Dado que ocurre un error de red o persistencia al registrar un check-in, cuando el servidor responde con error, entonces:
el hábito permanece en estado pendiente
los recordatorios activos no se cancelan
la app muestra un mensaje de error al usuario

14. Dado que ocurre un error al desmarcar un check-in, cuando el servidor no confirma la operación, entonces:
-el hábito permanece marcado como completado
-los recordatorios no se reactivan
-la app muestra un mensaje de error

### Vista de progreso y racha

15. Dado que el usuario abre la vista de progreso de un hábito diario y todavía no realizó check-in hoy, cuando consulta la racha actual, entonces la racha muestra únicamente los días consecutivos completados hasta ayer.

16. Dado que el usuario ya registró check-in hoy en un hábito diario, cuando consulta la racha actual, entonces el día actual sí se incluye dentro de la racha consecutiva.

17. Dado que un hábito diario tenía racha activa y no hubo check-in el día UTC anterior, cuando el usuario consulta la racha, entonces la racha muestra cero días consecutivos.

18. Dado que un hábito semanal requiere N check-ins por semana, cuando:
-el usuario completa al menos N check-ins en la semana actual 
-y también cumplió todas las semanas evaluables anteriores consecutivas desde su primer check-in
Entonces la app muestra el número de semanas consecutivas cumplidas.
-cuando el usuario consulta su racha, entonces la racha mostrada es cero semanas

19. Dado que un hábito semanal no alcanzó el mínimo requerido de check-ins en una semana completa UTC (lunes a domingo), cuando el usuario consulta la racha posteriormente, entonces la racha semanal se reinicia desde cero.

20. Dado que el usuario consulta la vista de progreso, cuando visualiza sus hábitos, entonces:
-los hábitos diarios muestran racha en días
-los hábitos semanales muestran racha en semanas

### Recordatorios push

21. Dado que el usuario crea o edita un hábito, cuando configura una hora fija o una frecuencia repetitiva de recordatorios y guarda la configuración, entonces:
-la configuración queda almacenada
-el hábito muestra visualmente la hora o frecuencia configurada
-la app muestra un mensaje confirmando que el recordatorio fue guardado correctamente

22. Dado que llega la hora configurada de un recordatorio y el hábito no tiene check-in hoy, cuando el sistema evalúa el hábito, entonces se envía una notificación push a todos los dispositivos activos registrados del usuario.

23. Dado que un hábito tiene recordatorios activos, cuando el check-in queda guardado exitosamente en el servidor, entonces todos los recordatorios restantes de ese hábito para el día actual se cancelan.

24. Dado que el usuario desmarca exitosamente un check-in del día actual, cuando existen recordatorios repetitivos configurados,  entonces:
-se reactivan únicamente los recordatorios futuros cuya hora aún no ocurrió
-no se recuperan ni reenvían recordatorios ya vencidos

25. Dado que un hábito tiene recordatorios repetitivos configurados, cuando llega la hora programada y no existe check-in del día, entonces la notificación puede enviarse a cualquier hora del día sin restricciones de silencio.

26. Dado que el usuario no otorgó permisos de notificaciones, cuando intenta activar recordatorios push, entonces la app solicita permiso de notificaciones antes de guardar la configuración.

27. Dado que el usuario rechaza el permiso de notificaciones del navegador o sistema operativo, cuando la app intenta guardar un recordatorio push, entonces:
-el recordatorio no se guarda
-la app muestra un mensaje de error indicando que los permisos son necesarios
---

## No-goals

1. Recuperación de hábitos eliminados.
2. Registro o edición retroactiva de check-ins.
3. Notificaciones por email, SMS o canales alternativos.
4. Reinterpretación histórica al cambiar frecuencia.
5. Hábitos con días específicos de la semana.
6. Funcionalidad social, rankings o colaboración multiusuario.
