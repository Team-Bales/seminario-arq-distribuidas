# P1-a — Acoplamiento dinámico

La connascencia, concepto que Richards y Ford (2020, cap. 3) aplican al análisis
arquitectónico, mide el acoplamiento dinámico: dos componentes son connascentes si
un cambio en uno obliga a modificar el otro. Interesan sus formas en tiempo de
ejecución: CoE (el orden de ejecución importa), CoT (el momento importa;
condiciones de carrera), CoV (varios valores deben cambiar juntos — la peor
forma de connascencia dinámica en transacciones distribuidas, Richards & Ford,
2020, cap. 17) y CoI (varios componentes comparten una estructura, como una cola).

Tres propiedades gobiernan su uso (Richards & Ford, 2020, cap. 3): la fuerza
(preferir connascencia estática, detectable por herramientas, sobre la dinámica),
la localidad (las formas fuertes son menos dañinas cuanto más próximos están los
elementos que conectan, según la regla de Weirich) y el grado. Además, cada llamada síncrona entre servicios crea connascencia dinámica mientras
dura la espera —acoplando disponibilidad y latencia de ambos lados—, mientras que
la asíncrona (*fire-and-forget*) los desacopla operativamente (Richards & Ford,
2020, cap. 7).

En la coreografía no existe coordinador central: los servicios reaccionan de
forma asíncrona al evento `SubastaFinalizada` (topología *broker* de las
arquitecturas dirigidas por eventos, EDA; Richards & Ford, 2020, cap. 17). Este
enfoque minimiza el acoplamiento dinámico: la única connascencia explícita es
el contrato del evento (forma débil), no hay espera síncrona, y agregar un
consumidor no afecta a los existentes. El costo es la pérdida de control: nadie
garantiza el orden de ejecución (CoE) ni evita las carreras entre consumidores
(CoT), el estado global queda eventualmente consistente (CoV latente) y nadie
en el sistema sabe realmente cuándo se completa la transacción de negocio
(Richards & Ford, 2020, cap. 14).

En la orquestación, un mediador posee la secuencia *notificar → pagar →
actualizar* y la ejecuta paso a paso (topología *mediator*, cap. 17). Suponemos
aquí un mediador que invoca cada paso de forma síncrona (petición-respuesta); si
en cambio enviara comandos asíncronos, conservaría el control del flujo sin
fundir los servicios en un solo quantum. Bajo ese supuesto, el enfoque concentra
el acoplamiento: la connascencia de ejecución queda explícita en un solo lugar,
pero el mediador y cada paso quedan connascentes entre sí, y cada espera de
respuesta crea connascencia síncrona que une a los cuatro servicios en un solo
quantum (Richards & Ford, 2020, cap. 7), con la latencia sumándose salto a salto
y el mediador como posible cuello de botella (Richards & Ford, 2020, cap. 9). A
cambio, el mediador puede detectar fallos, persistir estado y reintentar o
compensar: es acoplamiento a cambio de control.

En Service-Based Architecture (SBA), las tres acciones son llamadas a método
dentro de un servicio de dominio de grano grueso, lo que Richards y Ford (2020,
cap. 13) llaman orquestación a nivel de clase. El acoplamiento dinámico se
localiza: dentro del servicio es fuerte (CoE, CoV sobre una transacción ACID),
pero por la regla de localidad es aceptable porque no cruza fronteras; entre
servicios, el flujo no toca la red, así que el acoplamiento dinámico es
prácticamente nulo. El costo se paga en otro lado: redesplegar todo el servicio
ante cualquier cambio y perder la escalabilidad fina del pago (Richards & Ford,
2020, cap. 13).

| Criterio | Coreografía | Orquestación | SBA |
|---|---|---|---|
| Connascencia dominante | débil: contrato del evento (≈ nombre) | fuerte: CoE en el mediador + síncrona por paso | fuerte (CoE, CoV) pero local |
| Alcance del acoplamiento dinámico | solo el esquema del mensaje | mediador ↔ todos los pasos | dentro de la frontera del servicio |
| Dependencia de la red en el flujo | nula en el disparo (asíncrona) | alta: tres saltos síncronos | nula (llamadas a método) |
| Visibilidad y control del flujo | difusa: nadie ve la secuencia completa | máxima: el mediador posee el flujo y su estado | máxima dentro del servicio; invisible desde fuera |

Los tres diseños son puntos del mismo continuo del acoplamiento dinámico: la
coreografía lo minimiza a costa de visibilidad del flujo; la orquestación lo
concentra en un mediador a cambio de control explícito; el SBA lo encierra como
connascencia fuerte pero local, a costa de agilidad de despliegue y escalabilidad
fina. Como dicta la Primera Ley, todo en la arquitectura de software es un
trade-off (Richards & Ford, 2020, cap. 1): conviene orquestación o grano grueso
si pesa el control del flujo y la auditoría del cobro; coreografía si pesan la
extensibilidad y el desacoplamiento de equipos.
