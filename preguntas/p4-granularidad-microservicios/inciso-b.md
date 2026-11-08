# P4-b — Nueva estrategia de descomposición

Richards y Ford (2020, cap. 17) ofrecen tres guías para fijar los límites de un
servicio: **propósito** (cada servicio debe ser funcionalmente cohesivo en torno a
un comportamiento significativo), **transacciones** (las entidades que deben
cambiar juntas en una misma transacción de negocio señalan una buena frontera de
servicio) y **coreografía** (si el aislamiento de dominio exige demasiada
comunicación entre servicios, hay que reagruparlos: "the architect may consider
bundling these services back into a larger service to avoid the communication
overhead", Richards & Ford, 2020, cap. 17).

Aplicadas a RRHH, la nueva frontera no es la tabla de la base de datos sino el
*workflow* de contratación: Empleado, la configuración inicial de Nómina y la
inscripción en Beneficios participan todos en la misma transacción atómica de
"contratar" y deben vivir dentro de un mismo servicio —`ServicioContratación`— para
no construir una transacción distribuida. Departamento no cambia durante la
contratación, solo se consulta como dato de referencia, así que puede seguir siendo
un servicio aparte sin introducir acoplamiento transaccional. Todo lo que no es
parte de ese núcleo atómico —aprovisionar el equipo de TI, notificar al gerente,
actualizar el directorio corporativo— se mueve a **coreografía**:
`ServicioContratación` publica un evento `EmpleadoContratado` al confirmar la
transacción, y cada servicio periférico reacciona por su cuenta, de forma asíncrona.

| Criterio | Antes (por entidad) | Después (por workflow) |
|---|---|---|
| Propósito | un servicio por tabla (Empleado, Departamento, Nómina, Beneficios) | un servicio por transacción de negocio (`ServicioContratación`) |
| Transacciones | "contratar" cruza síncronamente los 4 servicios | "contratar" es una sola llamada atómica dentro de `ServicioContratación` |
| Dato de referencia | Departamento consultado síncronamente en cada paso | Departamento se consulta fuera de la transacción, como referencia |
| Efectos secundarios | notificar a TI, gerente y directorio: llamadas síncronas adicionales | evento `EmpleadoContratado`; TI, gerente y directorio reaccionan por coreografía |

El resultado reduce la coreografía síncrona de "casi todos los servicios" a una sola
llamada atómica más un evento publicado: el criterio de diseño deja de ser "un
servicio por tabla" y pasa a ser "un servicio por transacción de negocio", y la
coreografía absorbe todo lo que no necesita consistencia inmediata.
