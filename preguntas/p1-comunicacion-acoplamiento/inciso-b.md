# P1-b — Tolerancia a fallos y consistencia

El inciso extiende el escenario de la subasta: el servicio de pago falla justo
después de notificarse al ganador. Evaluamos qué garantiza cada diseño sobre la
consistencia de los datos —ACID (atomicidad, consistencia, aislamiento,
durabilidad) frente a BASE (disponibilidad básica, estado flexible, consistencia
eventual; Richards & Ford, 2020, cap. 13)— y sobre la tolerancia a fallos, es
decir, qué tan rápido el sistema detecta el fallo y recupera un estado correcto.
La connascencia de valores (CoV) es otra vez la clave: cuando una transacción de
negocio cruza límites de servicio, se genera la peor forma de connascencia
dinámica (Richards & Ford, 2020, cap. 17), porque varios estados remotos deben
cambiar juntos o no cambiar.

En la coreografía, el servicio de pago falla sin que nadie más lo note: nadie en
el sistema se entera de ese fallo, el procesador de inventario sigue
decrementando el stock y el resto de los procesadores reacciona como si todo
estuviera bien (Richards & Ford, 2020, cap. 14). En el escenario, el ganador
ya fue notificado —un efecto irreversible— y el pago queda bloqueado mientras el
resto del flujo avanza como si hubiera tenido éxito: es BASE en su forma más
débil, disponible pero sin garantía de completar la transacción de negocio ni de
deshacer lo ya hecho. La recuperación exige mecanismos añadidos por fuera de la
topología: colas con reintento y cola de mensajes fallidos (*dead-letter queue*),
un patrón *outbox* para no perder el evento, o una saga coreografiada donde el
propio servicio de inventario escuche un evento de compensación (Richardson,
2018).

En la orquestación, el mediador conoce el estado de cada paso: si el pago falla,
detiene el flujo, registra el estado de la petición en su propio almacén y, una
vez que el pago se aplique, retoma el flujo justo donde quedó (Richards & Ford,
2020, cap. 14), o dispara las compensaciones del patrón Saga:
en esta secuencia solo la notificación precede al pago, y como no puede
retirarse, se compensa con un segundo aviso; el inventario ni siquiera llega a
tocarse (Richards & Ford, 2020, cap. 17). Sigue siendo
BASE —no hay rollback atómico real entre servicios—, pero es una consistencia
eventual *gobernada*: el mediador garantiza que el proceso termine en un estado
conocido, a cambio del acoplamiento que ya se pagó en el inciso a.

En Service-Based Architecture (SBA), las tres acciones viven en un único servicio
con una base de datos compartida, así que pago e inventario pueden envolverse en
una transacción ACID real: si el pago falla, la base de datos revierte íntegra y
automáticamente (Richards & Ford, 2020, cap. 13). El matiz es la notificación: si
ocurre, como indica el enunciado, antes de intentar el pago, ya escapó de la
transacción y no se puede deshacer con un rollback de base de datos. Pero como
todo el flujo corre en un mismo proceso síncrono, el servicio conoce el fallo de
inmediato y puede emitir una compensación explícita (un segundo aviso de que el
pago no se procesó) sin depender de ningún mecanismo distribuido. El costo de
esta fortaleza, visto en el inciso a, es que el servicio grueso concentra el
riesgo de despliegue y no escala el pago por separado.

| Criterio | Coreografía | Orquestación | SBA |
|---|---|---|---|
| Modelo de consistencia | BASE sin gobierno | BASE gobernado (Saga) | ACID dentro del servicio |
| Detección del fallo | ninguna: nadie lo nota | el mediador la detecta y persiste el estado | inmediata (mismo proceso) |
| Riesgo de inconsistencia | alto: el inventario avanza igual | bajo: compensaciones explícitas | nulo en la BD; solo en la notificación ya emitida |
| Recuperación | manual/externa (DLQ, *outbox*) | automática: reintento o rollback lógico | automática en la BD; compensación manual de la notificación |

El fallo del pago expone el mismo trade-off del inciso a, visto ahora desde la
consistencia: la coreografía paga su desacoplamiento con BASE sin red de
seguridad —el inventario puede decrementarse sobre un pago que nunca ocurrió—; la
orquestación compra consistencia eventual gobernada (Saga) con el acoplamiento
hacia el mediador; y el SBA logra la consistencia más fuerte (ACID) porque el
flujo nunca cruza la red, aunque la notificación —al ser un efecto externo previo
al fallo— sigue necesitando una compensación explícita en los tres diseños.
Ninguna arquitectura evita por completo la necesidad de compensar: la diferencia
es si existe un lugar —el mediador o el propio servicio— que sepa que debe
hacerlo.
