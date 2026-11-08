# P1-a — Acoplamiento dinámico

La **connascencia**, concepto que Richards y Ford (2020, cap. 3) aplican al análisis
arquitectónico, mide el acoplamiento dinámico: dos componentes son connascentes si un
cambio en uno obliga a modificar el otro. Interesan sus formas en tiempo de ejecución: **CoE** (el orden de
ejecución importa), **CoT** (el momento importa; condiciones de carrera), **CoV**
(varios valores deben cambiar juntos — "the worst kind of dynamic connascence" en
transacciones distribuidas, Richards & Ford, 2020, cap. 17) y **CoI** (varios
componentes comparten una estructura, como una cola).

Dos reglas gobiernan su uso (Richards & Ford, 2020, cap. 3): la **localidad** (las
formas fuertes son tolerables solo si son locales a un módulo) y la **fuerza**
(preferir connascencia estática, detectable por herramientas, sobre la dinámica).
Además, cada llamada **síncrona** entre servicios crea connascencia dinámica
mientras dura la espera —acoplando disponibilidad y latencia de ambos lados—,
mientras que la **asíncrona** (*fire-and-forget*) los desacopla operativamente
(Richards & Ford, 2020, cap. 7).

**Coreografía.** Sin coordinador central: los servicios reaccionan de forma
asíncrona al evento `SubastaFinalizada` (topología *broker* de las arquitecturas
dirigidas por eventos, EDA; Richards & Ford, 2020, cap. 17). **Minimiza** el
acoplamiento dinámico: la única connascencia es el contrato del evento (forma
débil), no hay espera síncrona, y agregar un consumidor no afecta a los existentes.
El costo es la pérdida de control: nadie garantiza el orden (CoT), el estado global
queda eventualmente consistente (CoV latente) y "no one in the system really knows
when the business transaction... is actually complete" (Richards & Ford, 2020,
cap. 14).

**Orquestación.** Un mediador posee la secuencia *notificar → pagar → actualizar*
(topología *mediator*, cap. 17). **Concentra** el acoplamiento: la connascencia de
ejecución queda explícita en un solo lugar, pero el mediador y cada paso quedan
connascentes entre sí, y cada espera de respuesta crea connascencia síncrona que une
a los cuatro servicios en un solo quantum —con la latencia sumándose salto a salto y
el mediador como posible cuello de botella (Richards & Ford, 2020, caps. 9 y 14). A
cambio, el mediador puede detectar fallos, persistir estado y reintentar o
compensar: es acoplamiento a cambio de control.

**Service-Based Architecture (SBA).** Las tres acciones son llamadas a método
dentro de un servicio de dominio de grano grueso — "orquestación a nivel de clase"
(Richards & Ford, 2020, cap. 13). El acoplamiento dinámico se **localiza**: dentro
del servicio es fuerte (CoE, CoV sobre una transacción ACID), pero por la regla de
localidad es aceptable porque no cruza fronteras; entre servicios, el flujo no toca
la red, así que el acoplamiento dinámico es prácticamente nulo. El costo se paga en
otro lado: redesplegar todo el servicio ante cualquier cambio y perder la
escalabilidad fina del pago (Richards & Ford, 2020, cap. 13).

| Criterio | Coreografía | Orquestación | SBA |
|---|---|---|---|
| Connascencia dominante | débil: contrato del evento (≈ nombre) | fuerte: CoE en el mediador + síncrona por paso | fuerte (CoE, CoV) pero **local** |
| Alcance del acoplamiento dinámico | solo el esquema del mensaje | mediador ↔ todos los pasos | dentro de la frontera del servicio |
| Dependencia de la red en el flujo | nula en el disparo (asíncrona) | alta: tres saltos síncronos | nula (llamadas a método) |
| Visibilidad y control del flujo | difusa: nadie ve la secuencia completa | máxima: el mediador posee el flujo y su estado | máxima dentro del servicio; invisible desde fuera |

Los tres diseños son puntos del mismo continuo del acoplamiento dinámico: la
coreografía lo **minimiza** a costa de visibilidad del flujo; la orquestación lo
**concentra** en un mediador a cambio de control explícito; el SBA lo encierra como
connascencia fuerte pero **local**, a costa de agilidad de despliegue y
escalabilidad fina. Como dicta la Primera Ley, "everything in software architecture
is a trade-off" (Richards & Ford, 2020, cap. 1): conviene orquestación o grano
grueso si pesa el control del flujo y la auditoría del cobro; coreografía si pesan
la extensibilidad y el desacoplamiento de equipos.
