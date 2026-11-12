# P1 — Notas de bibliografía (Richards & Ford 2020)

> Material extraído de *Fundamentals of Software Architecture* (Richards & Ford, 2020)
> para responder los incisos a y b de la P1. Las citas textuales están en inglés con
> la referencia exacta (capítulo/sección) para poder citar autor-año en el documento final.

## Enunciado (orientación, secc. 2, pregunta 1)

Sistema de subastas en línea; al finalizar una subasta hay que realizar múltiples
acciones: **notificar al ganador**, **procesar el pago** y **actualizar el inventario**.

| Diseño | Cómo resuelve el flujo |
|---|---|
| Microservicios + **coreografía** | eventos asíncronos, sin coordinador central |
| Microservicios + **orquestación** | servicio mediador que coordina el flujo |
| **SBA** | las acciones se ejecutan *dentro* de un mismo servicio de dominio de grano grueso |

- **Inciso a:** ¿cómo afecta cada enfoque al **acoplamiento dinámico** entre componentes?
- **Inciso b:** implicaciones para **tolerancia a fallos** y **consistencia de datos
  (ACID vs. BASE)** si el servicio de pago falla *justo después* de notificar al ganador.

---

## 1. Connascencia (cap. 3, "Modularity" → sección *Connascence*)

**Definición** (Page-Jones 1996, retomada por Richards & Ford):

> "Two components are connascent if a change in one would require the other to be
> modified in order to maintain the overall correctness of the system."

Tipos **dinámicos** (los centrales para el inciso a, son acoplamiento en tiempo de
ejecución):

| Tipo | Descripción | Conexión con el escenario |
|---|---|---|
| **CoE** — execution | el orden de ejecución importa | en orquestación, el mediador impone orden (pago antes de inventario); en SBA es una llamada a método interna |
| **CoT** — timing | el momento de la ejecución importa (race conditions) | coreografía: los 3 consumidores reaccionan en paralelo al mismo evento; nadie controla el orden |
| **CoV** — values | varios valores deben cambiar juntos | **la clave del inciso b**: "the more common and problematic case involves transactions, especially in distributed systems… all the values must change together or not at all" |
| **CoI** — identity | componentes comparten/actualizan una estructura común (ej. cola distribuida) | el event broker como estructura compartida |

Propiedades de la connascencia:
- **Strength** (fuerza): los arquitectos deben preferir connascencia estática a dinámica,
  porque es detectable por análisis de código y las herramientas facilitan refactorizarla.
- **Locality** (localidad): "as the distance between software elements increases, use
  weaker forms of connascence" (regla de localidad, Jim Weirich). → En SBA la
  connascencia fuerte está *dentro* del servicio (local); en microservicios está
  *entre* servicios (remota).
- **Degree** (grado): cuántos módulos afecta.

Guías de Page-Jones: 1) minimizar la connascencia total rompiendo el sistema en
elementos encapsulados; 2) minimizar la connascencia que cruza fronteras de
encapsulación; 3) maximizar la connascencia *dentro* de las fronteras.

---

## 2. Architecture quantum y connascencia síncrona vs. asíncrona (cap. 7)

Definición: "An independently deployable artifact with high functional cohesion and
**synchronous connascence**".

- Las llamadas **síncronas** entre servicios distribuidos crean connascencia dinámica
  durante la llamada: si quien llama es mucho más escalable que quien responde,
  aparecen timeouts y problemas de fiabilidad — sus características operativas deben
  coincidir durante la llamada.
- Las llamadas **asíncronas** (fire-and-forget) permiten que dos servicios difieran en
  sus características operativas.

**⚠ Ejemplo EXACTO del escenario de P1** (sección *Coupling and Connascence*, cap. 7):

> "consider a microservices architecture with a Payment service and an Auction
> service. When an auction ends, the Auction service sends payment information to the
> Payment service. However, let's say that the payment service can only handle a
> payment every 500 ms — what happens when a large number of auctions end at once?
> A poorly designed architecture would allow the first call to go through and allow
> the others to time out. Alternatively, an architect might design an asynchronous
> communication link between Payment and Auction, allowing the message queue to
> temporarily buffer differences. In this case, **asynchronous connascence creates a
> more flexible architecture**."

El caso de subastas en línea es el *kata* **"Going, Going, Gone"** (introducido en el
cap. 5, analizado en caps. 7 y 18) — el enunciado del seminario se inspira en él.

---

## 3. SBA: orquestación interna y transacciones ACID (cap. 13)

Topología: UI desplegada aparte + **4–12 servicios de dominio de grano grueso** +
**base de datos compartida** (monolítica).

Orquestación a nivel de clase (sección *Service Design and Granularity*):

> La API facade del `OrderService` "internally orchestrates the single business
> request: place the order, generate an order ID, apply the payment, and update the
> product inventory". En microservicios esto implicaría "the orchestration of many
> separately deployed remote single-purpose services". **Orquestación interna a nivel
> de clase vs. orquestación externa a nivel de servicio** — una de las diferencias
> clave de granularidad entre SBA y microservicios.

Transacciones (misma sección):

> "Because domain services are coarse-grained, regular **ACID** database transactions
> involving database commits and rollbacks are used to ensure database integrity
> within a single domain service. Highly distributed architectures like microservices,
> on the other hand, usually have fine-grained services and use a distributed
> transaction technique known as **BASE** transactions (basic availability, soft
> state, eventual consistency)… that rely on eventual consistency and hence do not
> support the same level of database integrity as ACID transactions."

Ejemplo del libro (checkout con tarjeta vencida) — trasladable al inciso b:
- **SBA**: es una transacción atómica en el mismo servicio → rollback de todo y aviso
  al cliente.
- **Microservicios**: la orden ya fue insertada, el pago falló → estado inconsistente;
  ¿y el inventario? ¿se decrementa? ¿se lo reservamos al cliente con tarjeta vencida?
  "These are just a few of the questions that would need to be addressed when
  orchestrating a business process with multiple finer-grained services."

Trade-off de SBA (contrapeso, útil para no idealizar): un cambio en la funcionalidad
de órdenes obliga a probar y redesplegar **todo** el servicio grueso (incluido el
procesamiento de pagos) → más riesgo por despliegue que en microservicios.

---

## 4. EDA: topologías broker vs. mediator (cap. 14)

Las topologías del cap. 14 son la base conceptual de coreografía (≈broker) y
orquestación (≈mediator) en microservicios (ver §5).

### Broker topología (→ coreografía)
Sin mediador central; flujo distribuido "in a chain-like broadcasting fashion" por un
broker ligero; modelo publish-subscribe; cada procesador anuncia lo que hizo
(fire-and-forget).

Tabla 14-1:

| Ventajas | Desventajas |
|---|---|
| Procesadores de eventos altamente desacoplados | Control del workflow |
| Alta escalabilidad | Manejo de errores |
| Alta capacidad de respuesta | Recuperabilidad |
| Alto rendimiento | Capacidades de reinicio |
| Alta tolerancia a fallos | Inconsistencia de datos |

**⚠ Párrafo EXACTO para el inciso b** (fallo de Payment en broker):

> "if a failure occurs (such as the Payment event processor crashing and not
> completing its assigned task), **no one in the system is aware of that crash**. The
> business process gets stuck… Furthermore, all other processes are moving along
> without regard for the error. For example, **the Inventory event processor still
> decrements the inventory**, and all other event processors react as though
> everything is fine."

También: la topología broker **no soporta reiniciar** la transacción de negocio —
nadie posee el estado del request original. En el ejemplo minorista de la Fig. 14-4,
Payment publica `payment-applied` / `payment-denied` (casi idéntico al escenario de
la subasta).

### Mediator topología (→ orquestación)
El mediador "manages and controls the workflow", mantiene el estado del evento en su
propio datastore persistente, y controla manejo de errores, recuperación y reinicio:

> "suppose in the prior example the payment was not applied due to the credit card
> being expired… the mediator… stops the workflow and records the state of the request
> in its own persistent datastore. Once payment is eventually applied, the workflow
> can be restarted from where it left off."

En mediator los eventos de procesamiento son **comandos** ("things that need to
happen"), no eventos ("things that have already happened").

Tabla 14-2:

| Ventajas | Desventajas |
|---|---|
| Control del workflow | Más acoplamiento de los event processors |
| Manejo de errores | Menor escalabilidad |
| Recuperabilidad | Menor rendimiento |
| Reinicio | Menor tolerancia a fallos |
| Mejor consistencia de datos | Modelar workflows complejos |

Conclusión del capítulo:

> "The choice between the broker and mediator topology essentially comes down to a
> trade-off between workflow control and error handling capability versus high
> performance and scalability."

### Comunicación síncrona vs. asíncrona (cap. 14, *Asynchronous Capabilities*)
- Ejemplo 3.100 ms (REST síncrono) vs. 25 ms (mensajería asíncrona) para el usuario.
- Distinción **responsiveness** (avisar que se aceptó) vs. **performance** (acelerar el
  proceso completo).
- Caveat: con síncrono hay garantía de que se hizo; con asíncrono solo un *ack* y una
  promesa futura → el principal problema de la comunicación asíncrona es el **manejo
  de errores**.

---

## 5. Microservicios: coreografía y orquestación (cap. 17)

Mapeo explícito del libro:

> "For asynchronous communication, architects often use events and messages, thus
> internally utilizing an event-driven architecture, covered in Chapter 14; **the
> broker and mediator patterns manifest in microservices as choreography and
> orchestration**."

**Coreografía**: "no central coordinator exists in this architecture, respecting the
bounded context philosophy". Preserva el desacoplamiento filosófico del estilo, pero
"common problems like error handling and coordination become more complex".

**Orquestación**: el arquitecto crea un **mediador localizado** (ej.
`ReportCustomerInformation`). "While this creates coupling between these services, it
allows the architect to focus coordination into a single service". Si un servicio
coreografiado termina actuando como mediador además de su lógica de dominio, es el
*front controller pattern* → complejidad añadida en el servicio.

La API layer **no debe** usarse como mediador si se quiere respetar la filosofía
(= acoplamiento operacional que el estilo intenta evitar).

Transacciones (sección *Transactions and Sagas*):

> "Building transactions across service boundaries violates the core decoupling
> principle of the microservices architecture (and also **creates the worst kind of
> dynamic connascence, connascence of value**). The best advice… is: don't! Fix the
> granularity components instead."

Patrón **Saga** con **transacciones de compensación**: el mediador registra
éxito/fallo de cada paso y, si uno falla, envía undo a los que ya tuvieron éxito; cada
request queda en *pending state* hasta confirmación global; "this design becomes
complex if asynchronous requests must be juggled" y genera mucho tráfico de
coordinación; consejo: usar saga con moderación.

---

## 6. Trade-offs síncrono/asíncrono y falacias (caps. 18 y 9)

- Cap. 18 (sección de decisiones de diseño): "Synchronous communication is more
  convenient in most cases, but it can lead to scalability, reliability, and other
  undesirable characteristics. Asynchronous communication can provide unique benefits
  in terms of performance and scale but can present a host of headaches: data
  synchronization, deadlocks, race conditions, debugging…". Consejo: **"Use
  synchronous by default, asynchronous when necessary."**
- Cap. 18, caso GGG (subastas): Payment procesa un pago cada 500 ms; si muchas
  subastas terminan a la vez, la comunicación síncrona produce timeouts → las colas de
  mensajes "add reliability to a critical part of the architecture that exhibits
  fragility". El diseño resultante tiene **5 quanta** (Payment, Auctioneer, Bidder,
  Bidder Streams, Bid Tracker).
- Cap. 9 (fallacies of distributed computing, Peter Deutsch et al. 1994): #1 *the
  network is reliable* (justifica timeouts y circuit breakers), #2 *latency is zero*
  (encadenar 10 llamadas a 100 ms añade 1 s), #3 *bandwidth is infinite*. Útiles para
  justificar por qué en microservicios cada salto síncrono encadena acoplamiento
  dinámico y latencia.

---

## 7. Brecha bibliográfica

*Software Architecture: The Hard Parts* (Ford, Richards, Sadalage & Dehghani, 2021) —
recomendado en el README (caps. 4, 6, 7, 9: datos distribuidos, consistencia eventual,
Saga, CQRS, Event Sourcing) — **no está** en `bibliografia/`. Ahí es donde aparece
formalmente el par **static coupling / dynamic coupling** que nombra el enunciado.
Con el cap. 3 (connascencia dinámica) + cap. 7 (connascencia síncrona/asíncrona) de
Richards & Ford 2020 se puede responder igual, citando solo la referencia principal
que pide la orientación. Si conseguimos el PDF, añadir §"dynamic coupling" de su
cap. 2-3 a estas notas.

---

## 8. Borrador de esqueleto para cada inciso

### Inciso a — acoplamiento dinámico
1. Marco: connascencia dinámica = acoplamiento en tiempo de ejecución; el *quantum*
   mide cuántos servicios quedan atados por connascencia síncrona (§1, §2).
2. **Coreografía**: minimiza el acoplamiento dinámico — connascencia asíncrona, cada
   servicio solo conoce el contrato del evento (esquema del mensaje), no a los demás;
   quanta separados. Costo: CoT/CoE residuales (nadie controla orden ni timing).
3. **Orquestación**: aumenta el acoplamiento dinámico — el mediador está
   connascentemente ligado (CoE) a todos los pasos y su disponibilidad/latencia se
   propaga a los demás (connascencia síncrona durante cada llamada); el flujo queda
   dentro de UN quantum más grande.
4. **SBA**: las tres acciones son llamadas a método dentro del mismo servicio →
   connascencia dinámica fuerte PERO local (regla de locality: es aceptable); el
   quantum es uno solo; no hay acoplamiento *entre servicios* porque el flujo no cruza
   la red.
5. Mini-tabla comparativa de los tres enfoques × (CoE/CoT/CoV, alcance del quantum,
   latencia de red, extensibilidad).

### Inciso b — tolerancia a fallos y consistencia (fallo de pago tras notificar al ganador)
1. Marco: ACID (SBA) vs. BASE (microservicios); CoV como "worst kind of dynamic
   connascence" (§1, §3, §5).
2. **SBA**: notificación y pago viven en el mismo servicio → si es una transacción
   ACID, un fallo de pago hace rollback de todo; el caso "notificar y luego fallar"
   solo ocurre si se notifica *fuera* de la transacción → analizar ese matiz.
3. **Coreografía**: exactamente el fallo del cap. 14 — nadie sabe que Payment murió;
   Inventory sigue; el ganador ya fue notificado pero el pago quizá nunca se procese →
   inconsistencia de datos; recuperación: nadie reinicia el flujo. Mitigaciones:
   compensación (saga sin mediador), colas con reintento/DLQ, *outbox*.
4. **Orquestación**: el mediador detecta el fallo, persiste el estado y reintenta o
   dispara compensaciones (rollback lógico) → mejor consistencia (eventual) y
   recuperabilidad, a costa de acoplamiento (a) y menor tolerancia al fallo del propio
   mediador.
5. Cierre: trade-off explícito — consistencia fuerte (SBA) vs. disponibilidad y
   desacoplamiento con consistencia eventual (µs); recomendación justificada para la
   subasta (ej. orquestación + saga, o coreografía + compensaciones para el inventario).

## Entradas de referencia (autor-año, para la lista final)

- Richards, M., & Ford, N. (2020). *Fundamentals of Software Architecture: An
  Engineering Approach*. O'Reilly Media. — caps. 3, 7, 9, 13, 14, 17, 18.
- (Opcional, si se consigue) Ford, N., Richards, M., Sadalage, P., & Dehghani, Z.
  (2021). *Software Architecture: The Hard Parts*. O'Reilly Media.
- (Opcional, para el origen de la connascencia) Page-Jones, M. (1996). *What Every
  Programmer Should Know About Object-Oriented Design*. Dorset House.
