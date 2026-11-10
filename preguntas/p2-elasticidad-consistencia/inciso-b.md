# P2-b — Colisiones de datos en cachés replicadas

## El problema: una colisión puede acabar en doble venta

En una caché replicada en la que cualquier instancia puede escribir, una colisión
ocurre cuando un dato se actualiza en una instancia y otra lo modifica mientras ese
cambio se replica; cada réplica acaba sobrescrita con el valor de la otra (Richards &
Ford, 2020, cap. 15). En el ejemplo del libro, el daño es un contador de inventario
erróneo (cap. 15). Con asientos numerados, la consecuencia es más grave: dos unidades
de procesamiento pueden asignar el mismo asiento a dos compradores dentro de la
ventana de replicación, y ambos recibirían una confirmación. Cada asiento debe
venderse como máximo una vez: es un invariante, y un invariante no admite un
porcentaje de error.

## Cuantificar el riesgo

El libro estima la tasa de colisiones con esta fórmula (Richards & Ford, 2020,
cap. 15):

`Tasa de colisiones = N × UR² / S × RL`

donde *N* es el número de instancias (unidades de procesamiento) con la misma caché,
*UR* la tasa de actualización, *S* el tamaño de la caché en filas y *RL* la latencia
de replicación. La fórmula es coherente siempre que *UR* y *RL* usen la misma unidad
de tiempo, y el resultado queda en colisiones por esa unidad; aquí se usan segundos.
Además, Richards y Ford aconsejan hacer el cálculo para la carga de pico (cap. 15).

Estimación de colisiones. La primera fila es el ejemplo de Richards y Ford (2020,
cap. 15), con 14,4 colisiones por hora convertidas a segundos; las demás son valores
ilustrativos de elaboración propia.

| Escenario | N | UR (por s) | S (filas) | RL | Colisiones por s | % de actualizaciones |
|---|---|---|---|---|---|---|
| Ejemplo del libro | 5 | 20 | 50 000 | 100 ms | 0,004 | 0,02 % |
| Pico, demanda uniforme | 20 | 500 | 50 000 | 100 ms | 10 | 2 % |
| Pico, demanda en el 10 % de los asientos | 20 | 500 | 5 000 | 100 ms | 100 | 20 % |
| Pico, latencia de 1 ms | 20 | 500 | 50 000 | 1 ms | 0,1 | 0,02 % |

En los escenarios de pico, *UR* = 500 supone unas tres actualizaciones por asiento
(retener, vender y las retenciones que expiran) sobre las 170 ventas por segundo del
inciso a). Las cifras indican órdenes de magnitud, no predicciones. La fórmula trata
por igual todas las filas de la caché, así que el tercer escenario aproxima la
concentración de la demanda tomando como *S* solo los asientos más solicitados. De la
tabla se desprenden tres conclusiones:

- Los factores de la fórmula (Richards & Ford, 2020, cap. 15) juegan en contra durante
  el pico: *N* crece precisamente por la elasticidad que motiva el estilo, *UR* influye
  al cuadrado y *S* no puede aumentar, porque lo fija el aforo.
- El libro considera viable la replicación en su ejemplo por el bajo porcentaje de
  colisiones (cap. 15), pero ese criterio no sirve para un invariante.
- Incluso con una latencia de 1 ms quedarían 0,1 colisiones por segundo: el mismo
  0,02 % que el libro considera viable, pero unas 30 colisiones si ese pico se
  sostuviera durante cinco minutos, cada una posible doble venta cuando enfrentara a
  dos reservas del mismo asiento.

Por tanto, no basta con reducir la tasa: la solución debe impedir que una colisión se
convierta en una doble venta. Eso lo garantiza un árbitro duradero, sea cual sea el
modelo de caché. Lo que la tabla decide es otra cosa: con caché replicada y árbitro no
habría doble venta, pero cada colisión se descubriría al confirmar, con el comprador
ya en el pago, y en el pico estimado chocaría entre el 2 % y el 20 % de las
actualizaciones. Por eso los asientos no deben ir en caché replicada: no para proteger
el invariante, sino para rechazar al perdedor antes de que pague.

## Mecanismos

El libro aporta el diagnóstico y el criterio para elegir la caché. La copia de solo
lectura, la operación atómica, la retención con expiración, el árbitro en la base de
datos y el dueño por sección son una propuesta propia construida a partir de ese
análisis.

**Separar los datos según su exigencia de consistencia.** El libro aconseja aprovechar
cada modelo de caché donde convenga, en lugar de imponer uno solo, y pone el inventario
como ejemplo de dato que requiere caché distribuida (Richards & Ford, 2020, cap. 15).
El estado de los asientos es muy dinámico, se actualiza con frecuencia y exige priorizar
la consistencia, criterios que apuntan a una caché distribuida (tabla 15-1), aunque el
tamaño no la exige y la tolerancia a fallos favorece a la replicada. Con una única copia
escribible no hay nada que replicar entre unidades y desaparece la colisión tal como la
define el libro (cap. 15). La descripción de los conciertos, casi estática, puede ir en
caché replicada (cap. 15). El modelo *near-cache* se descarta: sus cachés frontales no
se sincronizan entre unidades, lo que según el libro produce un rendimiento desigual, y
por eso no lo recomienda para este estilo (cap. 15).

**Copia de solo lectura para el mapa.** Las lecturas masivas del mapa de asientos no
deberían cargar el servidor central. Pueden atenderse desde una copia en cada unidad,
alimentada por un único origen. Como las unidades no escriben en ella, no hay
colisiones. El retraso no afecta a la corrección, pero tampoco es gratuito: a la
frecuencia del pico la replicación puede no dar abasto (Richards & Ford, 2020, cap. 15),
por lo que conviene difundir instantáneas periódicas por sección. Además, un mapa
atrasado concentra intentos en asientos ya tomados, con más rechazos y más carga sobre
el servidor central.

**Reserva atómica por asiento.** La caché distribuida no evita por sí sola la condición
de carrera: dos unidades pueden leer "libre" y escribir después. Comprobar y escribir
deben ser una sola operación condicional (*compare-and-set*) sobre la clave del asiento:
pasar a retenido únicamente si sigue libre. Solo un intento gana, y el resto recibe un
rechazo inmediato para que el usuario elija otro asiento. Se prefiere a un bloqueo
distribuido clásico porque el perdedor no espera y en un asiento muy disputado no se
forma una cola; la retención posterior sí actúa como un bloqueo con plazo, y por eso la
decisión final corresponde a la base de datos. Cada compra lleva un identificador de
reserva, guardado en la retención y en la venta, de modo que un reintento que encuentra
su propio identificador es un éxito y no un conflicto. Una compra de varios asientos
los retiene en un orden fijo, lo que evita que dos compras solapadas se hagan fallar
mutuamente, y libera los ya retenidos si alguno falla. Esa retención múltiple no es
atómica: otros usuarios pueden ver por un instante parte del grupo tomada, y si el
proceso cae, la expiración libera lo retenido.

**Retención con expiración.** Cada asiento pasa por los estados libre, retenido (con
comprador y hora de expiración), confirmando y vendido. La expiración la evalúa el
servidor de caché, de modo que hay un único reloj. La retención dura unos minutos, el
tiempo de pagar, y, al expirar, el asiento vuelve a estar libre. Así, la reserva no
depende de la pasarela de pago. La venta sí: si la pasarela se satura, las retenciones
expiran durante el pago, por lo que conviene limitar la entrada al pago y fijar la
expiración con margen. Una expiración corta penaliza a quien paga y una larga inmoviliza
asientos, y para evitar abusos hay que limitar las retenciones por usuario.

**La base de datos como árbitro de la venta firme.** La venta se registra de forma
síncrona, con una restricción de unicidad sobre el par concierto-asiento, antes de
confirmar al usuario. Es una excepción deliberada al estilo, cuyos *data pumps* son
siempre asíncronos, y el libro advierte que en este tipo de sistema acceder de forma
síncrona y continua a la base de datos probablemente no funcionaría (Richards & Ford,
2020, cap. 15). Resulta asumible porque solo llegan aquí las retenciones ganadoras, del
orden de una inserción por asiento y no una por intento: 50 000 asientos en cinco
minutos suponen, de media, unas 170 inserciones por segundo. El aforo limita el total de
ventas firmes; su tasa depende además de cómo se reparta el pago en el tiempo. El precio
es que la base de datos vuelve al quantum de la confirmación: si no está disponible, las
reservas continúan, pero no se emiten ventas firmes.

**Confirmar sin dejar estados inconsistentes.** Una operación atómica pasa el asiento de
retenido por este comprador a confirmando, lo que detiene su expiración; si la retención
ya expiró, se rechaza y se anula la autorización del cobro. Después se insertan en una
sola transacción todos los asientos de la compra, registrados como pendientes de captura,
y los asientos pasan a vendidos en la caché, donde ya no expiran. Si la inserción se
rechaza porque otro comprador tiene el asiento, se anula la autorización y la caché se
corrige a vendido. Si el proceso cae durante la confirmación, antes de liberar el asiento
se consulta la base de datos. Un proceso aparte captura (hace efectivo) el cobro con una
clave idempotente; si la captura falla definitivamente, se compensa anulando la venta y
liberando el asiento.

**Fallos de la caché.** El árbitro cubre los fallos de integridad de la caché: el libro
advierte que un servidor espejo puede quedar inconsistente si el primario cae
inesperadamente (Richards & Ford, 2020, cap. 15), y en ese caso la restricción impide la
doble venta. Esta garantía exige que la restricción sea global sobre un primario único y
que el commit sea duradero, con replicación síncrona o por quórum: si una conmutación por
error perdiera una venta ya confirmada, el asiento podría venderse de nuevo. El árbitro
protege la integridad, no la disponibilidad: si cae la caché distribuida, las unidades no
pueden operar (cap. 15) y la venta se detiene durante la conmutación o la recarga. Tras
un fallo total, la caché se reconstruye desde la base de datos, con los asientos vendidos,
mediante un componente análogo al *data reader* que el libro describe para la caída de
todas las unidades (cap. 15). Las retenciones en curso se pierden, lo que es una molestia
y no una doble venta. El historial y la analítica pueden actualizarse mediante *data
pumps* asíncronos, como en el ejemplo de subastas del libro (cap. 15), con colas
persistentes, envío síncrono al *broker*, confirmación del consumidor y confirmación
transaccional en destino para no perder mensajes (cap. 14).

**Alternativa: un único dueño por sección.** El libro señala que dedicar unidades a cada
subasta asegura la consistencia de sus datos (Richards & Ford, 2020, cap. 15). De forma
análoga, cada sección del recinto podría tener una unidad dueña, para lo cual el
*messaging grid* tendría que enrutar por sección, algo que va más allá del balanceo que
describe el libro (cap. 15). Las reservas de la sección se serializarían en memoria, sin
servidor central. Repartiría las escrituras igual que un servidor de caché particionado
por sección, pero llevaría el estado a las unidades, con costos propios: una sección muy
demandada quedaría limitada a una unidad, habría que gestionar el relevo del dueño, con
pérdida de retenciones si cae, y se complicarían las compras entre secciones. Solo
compensaría si las pruebas mostraran que esa capa de caché, ya particionada, sigue siendo
el límite. El árbitro seguiría siendo necesario.

## Flujo resultante

1. **Consulta.** El mapa se lee de la copia de la unidad, que puede estar algo
   desactualizada.
2. **Reserva.** Una operación atómica en la caché distribuida retiene el asiento con
   expiración; quien pierde recibe un rechazo inmediato.
3. **Pago.** Se autoriza el cobro mientras dura la retención.
4. **Confirmación.** Una operación atómica pasa el asiento a confirmando, se inserta la
   venta con la restricción de unicidad, el asiento pasa a vendido en la caché y se
   confirma al usuario.
5. **Cobro y propagación.** El cobro se captura de forma idempotente, y el historial y la
   analítica se actualizan de forma asíncrona.

## Trade-offs

Mecanismos propuestos y su costo. Elaboración propia a partir de Richards y Ford (2020,
cap. 15).

| Mecanismo | Qué protege | Qué se paga |
|---|---|---|
| Caché distribuida para asientos | la reserva frente a las colisiones de replicación | menos rendimiento y tolerancia a fallos; dependencia de un servidor central |
| Copia de solo lectura | el servidor central frente a las lecturas masivas | mapa desactualizado; más rechazos en los asientos disputados |
| Reserva atómica por asiento | la reserva frente a la carrera entre leer y escribir | más rechazos visibles para el usuario; requiere que el producto de caché la ofrezca |
| Retención con expiración | la reserva frente a la lentitud del pago | asientos inmovilizados y posible abuso, que obliga a limitar las retenciones por usuario |
| Árbitro en la base de datos | el invariante, incluso si falla la caché | escritura síncrona en ese paso; más latencia al confirmar; sin base de datos no hay ventas firmes |
| Dueño por sección | la consistencia sin servidor central | relevo del dueño y pérdida de retenciones si cae; compras entre secciones; sección muy demandada limitada a una unidad |

El *trade-off* central es consistencia a cambio de rendimiento y tolerancia a fallos. El
libro plantea en esos términos la elección cuando la caché es pequeña y el motor de
réplica puede seguir el ritmo de actualización; con tasas muy altas, advierte además que
la réplica puede no dar abasto (Richards & Ford, 2020, cap. 15). Aquí se acepta el costo
solo en la ruta crítica. El servidor de caché pasa a ser el límite de escritura: añadir
unidades añade clientes, no capacidad. Se acepta porque una operación en memoria sobre
una clave es mucho más barata que una transacción duradera y porque las operaciones por
clave permiten repartir las secciones entre varios servidores; esa capacidad debe
confirmarse con pruebas. Se conserva lo esencial del inciso a): una capa en memoria
absorbe el pico de intentos y la base de datos solo recibe las ventas firmes y el flujo
diferido.

En este diseño, la latencia de replicación solo afecta a la frescura del mapa y a la
ventana de pérdida del espejo, y debe medirse en producción; además, las pruebas a carga
máxima son difíciles (Richards & Ford, 2020, cap. 15). Por ello conviene vigilar durante
la venta los rechazos de la restricción de unicidad que no correspondan a reintentos:
señalan que la caché falló como primera barrera.
