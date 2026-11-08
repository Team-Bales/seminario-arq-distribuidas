# P4-b — Nueva estrategia de descomposición

Richards y Ford (2020, cap. 17) proponen tres guías para encontrar la frontera
correcta de un servicio, y las tres fallaron en el diseño original porque nunca se
aplicaron: solo se usó la estructura de la base de datos.

**Propósito.** Cada servicio debe ser funcionalmente cohesivo en torno a un
comportamiento significativo del dominio, no un contenedor de operaciones CRUD
sobre una tabla (Richards & Ford, 2020, cap. 17). Un servicio "Empleado" que solo
expone altas, bajas y modificaciones del registro no tiene un propósito de negocio
propio: es la base de datos con una interfaz de red delante.

**Transacciones.** Las entidades que deben cambiar juntas dentro de la misma
transacción de negocio señalan una buena frontera de servicio (Richards & Ford,
2020, cap. 17). Contratar a un empleado es una única transacción de negocio: el
alta del empleado, la configuración inicial de su nómina y su inscripción en el
plan de beneficios deben confirmarse o revertirse juntas. Por eso deben vivir
dentro del mismo servicio, en vez de repartirse entre tres servicios que luego
necesitan coordinarse por la red.

**Coreografía.** Cuando un conjunto de servicios con buen aislamiento de dominio
termina necesitando comunicación excesiva entre sí para completar una operación,
la señal correcta es reagruparlos en un servicio más grande, no multiplicar las
llamadas entre ellos (Richards & Ford, 2020, cap. 17). Esa es exactamente la
situación de RRHH: el aislamiento por entidad parecía limpio en el papel, pero el
costo real apareció al ejecutar el *workflow* completo de contratación.

Aplicando las tres guías, la nueva frontera deja de ser la tabla de la base de
datos y pasa a ser el *workflow* de contratación: Empleado, la configuración
inicial de Nómina y la inscripción en Beneficios se agrupan en un único
`ServicioContratación`, porque comparten el mismo propósito (dar de alta a un
empleado) y la misma frontera transaccional (deben confirmarse o revertirse como
una unidad). Departamento, en cambio, no participa en esa transacción —solo se
consulta como dato de referencia para validar a qué equipo se asigna el nuevo
empleado—, así que puede seguir siendo un servicio independiente sin introducir
una transacción distribuida: la consulta no necesita ser atómica con el alta.

Todo lo que no forma parte de ese núcleo transaccional se resuelve por
**coreografía** en vez de orquestación síncrona: aprovisionar el equipo de TI,
notificar al gerente del área, actualizar el directorio corporativo. En vez de que
`ServicioContratación` llame uno por uno a esos servicios y espere su respuesta,
publica un único evento `EmpleadoContratado` al confirmar la transacción, y cada
servicio periférico reacciona de forma asíncrona y a su propio ritmo. Esto reduce
la coreografía síncrona obligatoria de "casi todos los servicios" a una sola
llamada atómica dentro de una frontera transaccional bien definida, más un evento
que dispara el resto del proceso sin bloquear la confirmación de la contratación.

| Criterio | Antes (por entidad) | Después (por workflow) |
|---|---|---|
| Propósito | un servicio por tabla (Empleado, Departamento, Nómina, Beneficios) | un servicio por transacción de negocio (`ServicioContratación`) |
| Transacciones | "contratar" cruza síncronamente los 4 servicios | "contratar" es una sola llamada atómica dentro de `ServicioContratación` |
| Dato de referencia | Departamento consultado síncronamente en cada paso | Departamento se consulta fuera de la transacción, como referencia |
| Efectos secundarios | notificar a TI, gerente y directorio: llamadas síncronas adicionales | evento `EmpleadoContratado`; TI, gerente y directorio reaccionan por coreografía |

El costo de esta estrategia es el mismo que ya se discutió para Service-Based
Architecture en P1: `ServicioContratación` concentra más responsabilidad y más
riesgo de despliegue que los cuatro microservicios originales. Pero es un costo
deliberado: se paga acoplamiento interno, local y controlado, a cambio de eliminar
la transacción distribuida que hacía frágil al diseño anterior —el mismo
trade-off de siempre, ahora aplicado a la granularidad en vez de a la
comunicación.
