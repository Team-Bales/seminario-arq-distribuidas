# P6-b — Cuándo todavía conviene orquestar

Aunque la coreografía —donde los servicios interactúan directamente sin mediador
central— es ideal para preservar el máximo desacoplamiento, presenta desafíos serios
en la gestión de errores, la trazabilidad y la coordinación cuando los flujos de
negocio se vuelven complejos. Por eso, y a pesar de las críticas históricas al motor
de orquestación de SOA, existen escenarios en los que un arquitecto puede considerar
todavía un enfoque de orquestación (Richards & Ford, 2020, caps. 16 y 17).

## Escenarios que justifican la orquestación

Conviene inclinarse por un enfoque orquestado —ya sea mediante un mediador local
dentro del ecosistema de microservicios, ya sea mediante un *orchestration engine*
heredado de SOA— cuando el flujo de trabajo requiere una secuencia estricta de pasos
dependientes, donde el fallo de un paso intermedio exige una compensación coordinada
del resto. También cuando la lógica de coordinación entre múltiples dominios es
demasiado compleja para delegarse de forma descentralizada mediante eventos de
coreografía sin generar una maraña de dependencias implícitas, difíciles de rastrear
y mantener.

Conviene distinguir aquí dos variantes del patrón *Saga*, formalizadas por Chris
Richardson en *Microservices Patterns* (Richardson, 2018): la **choreography saga**,
donde cada servicio reacciona a eventos y decide su siguiente acción de forma
autónoma; y la **orchestration saga**, donde un componente central —típicamente otro
microservicio más del ecosistema, no un motor externo tipo BPM— coordina las llamadas
y las compensaciones. La segunda es la que interesa aquí, y es la que evita recrear
el cuello de botella clásico de SOA: el orquestador no es un componente omnipresente
que mediatiza todas las comunicaciones, sino un servicio más, con su propio ciclo de
vida y sus propias réplicas para evitar convertirse en *single point of failure*.
Herramientas como AWS Step Functions, Temporal, Camunda o Zeebe implementan este
patrón en la práctica.

## Ejemplo: proceso de checkout y pago en e-commerce

Un ejemplo que se beneficia claramente de la orquestación explícita es el proceso de
**colocación de pedidos** (*Order Placement*) en una tienda en línea. El flujo involucra
varios servicios de grano fino —*OrderService*, *PaymentService*, *InventoryService*,
*ShippingService*, *NotificationService*— y, aunque el camino feliz puede parecer
secuencial y sencillo, los caminos de excepción son donde la orquestación demuestra
su valor.

En el camino feliz, el servicio de pedidos recibe la solicitud, crea la orden en
estado provisional y genera un identificador; a continuación invoca al servicio de
pagos para procesar la transacción, reserva inventario, programa el envío y notifica
al usuario. Cada uno de estos pasos depende del anterior y debe ejecutarse en orden.

Ahora bien, si el pago falla —por ejemplo, porque la tarjeta de crédito ha
expirado— los datos del sistema quedan en un estado inconsistente: la orden existe en
estado provisional, el inventario pudo haberse reservado y quizá se envió una
notificación prematura. Sin un coordinador, cada servicio tendría que conocer la
lógica de compensación de los demás, lo que rompería el desacoplamiento. Con una saga
orquestada, en cambio, un componente central detecta el fallo y ejecuta, de forma
controlada, las transacciones compensatorias: libera el inventario reservado, cancela
el registro de la orden, contrarresta cualquier notificación enviada y notifica al
usuario de forma coherente con el estado final. El orquestador encapsula la lógica de
compensación y la mantiene visible y auditable, sin que los servicios de dominio
tengan que conocerse entre sí.

Este enfoque evita los cuellos de botella del antiguo motor centralizado de SOA
porque el orquestador se limita a coordinar un flujo concreto, no a mediatizar toda
la comunicación del sistema; pero aprovecha, al mismo tiempo, la garantía de
integridad que ofrece la coordinación explícita (Richards & Ford, 2020, cap. 17).
Es precisamente en este punto de equilibrio donde se aprecia el legado útil de SOA:
no el orquestador universal, sino la noción de que ciertos flujos de negocio
reclaman, legítimamente, un componente que los gobierne.
