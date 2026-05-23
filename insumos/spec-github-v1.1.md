# Habit Tracker ÔÇö Spec v1.1

---

## Objetivo

Aplicaci├│n web de seguimiento de h├íbitos que permite a cualquier persona crear h├íbitos, registrar su cumplimiento diario y recibir notificaciones push puntuales para no olvidar realizarlos.

---

## Scope

### Qu├® S├ì entra en este proyecto

-Registro e inicio de sesi├│n con email y contrase├▒a
-Creaci├│n, edici├│n y eliminaci├│n de h├íbitos
-H├íbitos con:
---nombre
---descripci├│n
---frecuencia diaria
---frecuencia semanal con n├║mero objetivo de check-ins
-Check-in diario por h├íbito
-Desmarcar un check-in ├║nicamente durante el d├¡a actual
-Vista de progreso con racha actual
-Recordatorios push configurables por h├íbito
-Cancelaci├│n autom├ítica de recordatorios despu├®s de un check-in exitoso
-Reactivaci├│n de recordatorios si el check-in del d├¡a es desmarcado

# Restricciones funcionales
Todos los c├ílculos de fecha y hora se realizan usando UTC.
La semana se define de lunes 00:00 UTC a domingo 23:59:59 UTC.
Los h├íbitos semanales requieren N check-ins libres dentro de la semana; no existen d├¡as fijos obligatorios.
La primera semana evaluable de un h├íbito semanal comienza en la semana donde ocurre el primer check-in del h├íbito.
# Restricciones de campos
-Nombre del h├íbito
Obligatorio
M├¡nimo: 1 car├ícter
M├íximo: 100 caracteres
-Descripci├│n del h├íbito
Opcional
M├íximo: 500 caracteres
-Frecuencia semanal
M├¡nimo: 1 check-in por semana
M├íximo: 7 check-ins por semana
# Push notifications
Las notificaciones push se implementan mediante Web Push API usando Service Workers.
La aplicaci├│n requiere HTTPS para funcionamiento de push notifications.
Cada dispositivo registrado guarda su propio token push.
Las notificaciones se env├¡an a todos los dispositivos activos registrados del usuario.

### Qu├® NO entra en este proyecto

- Recordatorios por email o SMS
- H├íbitos con d├¡as espec├¡ficos de la semana asignados
- Registro retroactivo de check-ins de d├¡as anteriores
- Historial o estad├¡sticas de h├íbitos eliminados
- Rec├ílculo del historial al editar la frecuencia de un h├íbito
- Compartir h├íbitos o progreso con otros usuarios
- Planes de pago o suscripciones

---

## Criterios de aceptaci├│n

### Autenticaci├│n

1. Dado que el usuario no tiene cuenta, cuando completa el formulario de registro con email y contrase├▒a v├ílidos y lo env├¡a entonces se crea su cuenta y queda autenticado en la app.

2. Dado que el usuario tiene cuenta, cuando ingresa su email y contrase├▒a correctos y los env├¡a, entonces accede a su panel de h├íbitos.

3. Dado que el usuario est├í autenticado, cuando ejecuta la acci├│n de cerrar sesi├│n, entonces la sesi├│n termina y es redirigido a la pantalla de inicio de sesi├│n.

4. Dado que un usuario no autenticado intenta acceder a una ruta protegida, cuando la app eval├║a su sesi├│n, entonces es redirigido a la pantalla de inicio de sesi├│n.

### CRUD de h├íbitos

5. Dado que el usuario est├í autenticado, cuando crea un h├íbito con nombre v├ílido, descripci├│n v├ílida y frecuencia v├ílida, entonces el h├íbito aparece en su lista de h├íbitos activos.

6. Dado que el usuario intenta crear un h├íbito sin nombre, cuando intenta guardar el formulario, entonces la app muestra un mensaje de validaci├│n indicando que el nombre es obligatorio y el h├íbito no se crea.

7. Dado que el usuario tiene un h├íbito creado, cuando modifica nombre, descripci├│n o frecuencia y guarda los cambios, entonces la lista de h├íbitos refleja inmediatamente los nuevos valores.

8. Dado que el usuario modifica la frecuencia de un h├íbito, cuando consulta check-ins anteriores a la fecha de edici├│n, entonces el historial previo permanece sin cambios y la nueva frecuencia aplica ├║nicamente desde la fecha de edici├│n en adelante.

9. Eliminar h├íbito

Dado que el usuario tiene un h├íbito creado, cuando lo elimina y confirma la acci├│n, entonces:
-el h├íbito desaparece de la lista
-todos sus check-ins asociados se eliminan permanentemente

### Check-in

10. Dado que el usuario tiene un h├íbito sin check-in en la fecha UTC actual, cuando marca el check-in del d├¡a, entonces:
-el h├íbito cambia visualmente al estado ÔÇ£CompletadoÔÇØ
-se muestra un indicador visible de completado
-el check-in queda registrado para la fecha UTC actual
-el check-in cuenta como uno de los N requeridos dentro de la semana UTC actual

11. Dado que el usuario registr├│ un check-in hoy, cuando lo desmarca antes de las 23:59:59 UTC del mismo d├¡a, entonces el h├íbito vuelve al estado ÔÇ£PendienteÔÇØ.

12. Dado que ya comenz├│ un nuevo d├¡a UTC, cuando el usuario intenta registrar un check-in para una fecha anterior, entonces la app no ofrece esa opci├│n ni permite guardar check-ins retroactivos.

13. Dado que ocurre un error de red o persistencia al registrar un check-in, cuando el servidor responde con error, entonces:
el h├íbito permanece en estado pendiente
los recordatorios activos no se cancelan
la app muestra un mensaje de error al usuario

14. Dado que ocurre un error al desmarcar un check-in, cuando el servidor no confirma la operaci├│n, entonces:
-el h├íbito permanece marcado como completado
-los recordatorios no se reactivan
-la app muestra un mensaje de error

### Vista de progreso y racha

15. Dado que el usuario abre la vista de progreso de un h├íbito diario y todav├¡a no realiz├│ check-in hoy, cuando consulta la racha actual, entonces la racha muestra ├║nicamente los d├¡as consecutivos completados hasta ayer.

16. Dado que el usuario ya registr├│ check-in hoy en un h├íbito diario, cuando consulta la racha actual, entonces el d├¡a actual s├¡ se incluye dentro de la racha consecutiva.

17. Dado que un h├íbito diario ten├¡a racha activa y no hubo check-in el d├¡a UTC anterior, cuando el usuario consulta la racha, entonces la racha muestra cero d├¡as consecutivos.

18. Dado que un h├íbito semanal requiere N check-ins por semana, cuando:
-el usuario completa al menos N check-ins en la semana actual 
-y tambi├®n cumpli├│ todas las semanas evaluables anteriores consecutivas desde su primer check-in
Entonces la app muestra el n├║mero de semanas consecutivas cumplidas.
-cuando el usuario consulta su racha, entonces la racha mostrada es cero semanas

19. Dado que un h├íbito semanal no alcanz├│ el m├¡nimo requerido de check-ins en una semana completa UTC (lunes a domingo), cuando el usuario consulta la racha posteriormente, entonces la racha semanal se reinicia desde cero.

20. Dado que el usuario consulta la vista de progreso, cuando visualiza sus h├íbitos, entonces:
-los h├íbitos diarios muestran racha en d├¡as
-los h├íbitos semanales muestran racha en semanas

### Recordatorios push

21. Dado que el usuario crea o edita un h├íbito, cuando configura una hora fija o una frecuencia repetitiva de recordatorios y guarda la configuraci├│n, entonces:
-la configuraci├│n queda almacenada
-el h├íbito muestra visualmente la hora o frecuencia configurada
-la app muestra un mensaje confirmando que el recordatorio fue guardado correctamente

22. Dado que llega la hora configurada de un recordatorio y el h├íbito no tiene check-in hoy, cuando el sistema eval├║a el h├íbito, entonces se env├¡a una notificaci├│n push a todos los dispositivos activos registrados del usuario.

23. Dado que un h├íbito tiene recordatorios activos, cuando el check-in queda guardado exitosamente en el servidor, entonces todos los recordatorios restantes de ese h├íbito para el d├¡a actual se cancelan.

24. Dado que el usuario desmarca exitosamente un check-in del d├¡a actual, cuando existen recordatorios repetitivos configurados,  entonces:
-se reactivan ├║nicamente los recordatorios futuros cuya hora a├║n no ocurri├│
-no se recuperan ni reenv├¡an recordatorios ya vencidos

25. Dado que un h├íbito tiene recordatorios repetitivos configurados, cuando llega la hora programada y no existe check-in del d├¡a, entonces la notificaci├│n puede enviarse a cualquier hora del d├¡a sin restricciones de silencio.

26. Dado que el usuario no otorg├│ permisos de notificaciones, cuando intenta activar recordatorios push, entonces la app solicita permiso de notificaciones antes de guardar la configuraci├│n.

27. Dado que el usuario rechaza el permiso de notificaciones del navegador o sistema operativo, cuando la app intenta guardar un recordatorio push, entonces:
-el recordatorio no se guarda
-la app muestra un mensaje de error indicando que los permisos son necesarios
---

## No-goals

1. Recuperaci├│n de h├íbitos eliminados.
2. Registro o edici├│n retroactiva de check-ins.
3. Notificaciones por email, SMS o canales alternativos.
4. Reinterpretaci├│n hist├│rica al cambiar frecuencia.
5. H├íbitos con d├¡as espec├¡ficos de la semana.
6. Funcionalidad social, rankings o colaboraci├│n multiusuario.
