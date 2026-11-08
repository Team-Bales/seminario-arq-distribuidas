# P2-a — Space-Based vs. Microservices

## Marco: qué exige el escenario

La **escalabilidad** es la capacidad de atender a muchos usuarios concurrentes sin
una degradación seria del rendimiento; la **elasticidad**, la de absorber ráfagas de
peticiones (Richards & Ford, 2020, cap. 5). El libro las distingue y pone la venta
de entradas como ejemplo de sistema que necesita elasticidad. En este escenario se
pasa de cientos a decenas de miles de usuarios en segundos y, según la descripción
del caso (cap. 15), las entradas se agotan en minutos.

Dos rasgos condicionan el diseño. La carga llega de golpe, sin un aumento gradual
que dé tiempo a reaccionar. Además, la demanda se concentra en un conjunto pequeño de
datos muy disputados: el mapa de asientos de un mismo concierto, cuya disponibilidad
debe actualizarse continuamente (cap. 15). Conviene separar los tipos de carga. Las
ventas efectivas están acotadas por el aforo: 50 000 asientos vendidos en cinco
minutos son unas 170 ventas por segundo (cifra ilustrativa). Lo difícil no es ese
caudal, sino las lecturas masivas del mapa y, sobre todo, los intentos simultáneos
sobre los mismos asientos.

## Por qué las puntuaciones no deciden

En las tablas del libro, ambos estilos tienen cinco estrellas tanto en elasticidad
como en escalabilidad (Richards & Ford, 2020, figs. 15-15 y 17-13), por lo que esas
puntuaciones no justifican la elección. Ante una necesidad de elasticidad, el libro
plantea buscar primero el factor limitante (cap. 23), que en aplicaciones de alta
concurrencia suele ser la base de datos (cap. 15). Aquí se adopta como hipótesis de
trabajo el acceso duradero al estado de los asientos, que habría que confirmar con
pruebas de carga; la pasarela de pago es otro candidato y se trata al final.

## Escalabilidad: qué recurso se agota primero

**Microservicios.** Cada servicio escala por separado, lo que funciona muy bien
cuando la carga se reparte entre dominios; con este estilo se han construido algunos
de los sistemas más escalables (Richards & Ford, 2020, cap. 17). Aquí, sin embargo,
la carga no se reparte entre dominios. Por el aislamiento de datos, cada servicio
posee su base de datos, y el arquitecto debe designar un dominio como fuente de
verdad o distribuir el dato con replicación o caché (cap. 17). En el diseño base, la
disponibilidad de asientos queda a cargo de un único servicio por el que pasa la
compra. En aplicaciones web tradicionales, escalar la capa de aplicación suele
trasladar el cuello de botella a la base de datos, la capa más difícil de escalar
(cap. 15), y cabe esperar lo mismo aquí: añadir instancias de ese servicio tiende a
llevar el problema a su almacén. El libro considera poco viable, a esta escala y
frecuencia de actualización, acceder de forma síncrona y continua a una base de datos
central, porque a una base de datos típica le costaría atender tantas transacciones
concurrentes (cap. 15). Que la base de datos sea de un servicio y no compartida no
cambia el problema: la granularidad reparte dominios, no la contención sobre los
mismos registros. Las llamadas de red y la verificación de seguridad en cada
*endpoint* añaden, además, latencia (cap. 17; dos estrellas de rendimiento en la
fig. 17-13).

**Space-Based.** La clave de su escalabilidad es, según Richards y Ford (2020,
cap. 15), "removing the central database as a synchronous constraint". Cada unidad
de procesamiento resuelve la petición contra datos en memoria y envía la
actualización de forma asíncrona por un *data pump* hasta un *data writer*, que
escribe en la base de datos (cap. 15). Lo decisivo es que la compra no espera una
confirmación duradera. De ahí se derivan dos consecuencias:

- La capacidad de procesamiento y de lectura crece con las unidades, sin depender de
  la base de datos durante el pico (cap. 15).
- La cola del *data pump* amortigua las escrituras: la base de datos las recibe al
  ritmo que puede sostener, y el procesamiento continúa aunque el *data writer* no
  esté disponible (caps. 7 y 15).

En términos de quantum, la base de datos queda fuera del quantum arquitectónico
(cap. 15), mientras que en Microservicios forma parte del quantum de cada servicio
(cap. 7).

El límite no desaparece: se traslada a la caché, y su forma depende del modelo
elegido (Richards & Ford, 2020, cap. 15):

- **Caché replicada.** El acceso es local y rápido, pero cada unidad recibe todas
  las actualizaciones. Más instancias significan más colisiones posibles, y con una
  tasa de actualización muy alta la replicación puede no dar abasto.
- **Caché distribuida.** Los datos residen en un servidor central. Ofrece mayor
  consistencia a cambio de menos rendimiento y tolerancia a fallos, y el servidor se
  convierte en una dependencia síncrona donde se concentra la contención.

El libro recomienda la caché distribuida para datos muy consistentes como el
inventario, y la replicada para datos casi estáticos (cap. 15). Los asientos caen en
el primer grupo y la descripción de los conciertos en el segundo; el inciso b)
desarrolla esa decisión.

## Elasticidad: qué capa debe crecer durante la ráfaga

**Microservicios.** Las herramientas de *service discovery* vigilan el volumen de
peticiones y arrancan nuevas instancias (Richards & Ford, 2020, cap. 17). El cómputo
sin estado arranca con facilidad, pero la capa duradera no se amplía en segundos:
añadirle nodos o particiones durante la ráfaga exige redistribuir datos.

**Space-Based.** El *deployment manager* vigila tiempos de respuesta y carga, y
arranca o apaga unidades de procesamiento (Richards & Ford, 2020, cap. 15). Con
caché replicada, una unidad nueva obtiene los datos de otra unidad activa, sin leer
de la base de datos, siempre que siga activa al menos una con esa caché (cap. 15);
con caché distribuida, es un cliente más del servidor de caché. En ambos casos,
crecer no presiona la base de datos, aunque sí la caché. Cuando las entradas se
agotan, las unidades sobrantes se apagan.

El libro confía en que el *deployment manager* detecte el aumento y arranque muchas
unidades, pero señala que lo óptimo es arrancarlas poco antes de abrir la venta
(Richards & Ford, 2020, cap. 15). A nuestro juicio, esa anticipación es decisiva,
porque arrancar unidades y preparar su caché lleva un tiempo que una subida en
segundos no concede; es viable porque el momento del pico se conoce de antemano,
aunque no su magnitud. Una arquitectura de Microservicios también puede arrancar
instancias por adelantado, de modo que la diferencia no está en la velocidad de
arranque del cómputo, sino en qué capa debe prepararse para el pico. En Microservicios
es el almacén duradero, que debe soportar el pico de intentos. En Space-Based es una
capa en memoria, y el almacén duradero solo absorbe el caudal diferido.

## Comparación

Ruta de compra durante el pico en cada estilo. Elaboración propia a partir de
Richards y Ford (2020, caps. 15 y 17).

| Criterio | Microservicios (diseño base) | Space-Based |
|---|---|---|
| Recurso limitante | almacén duradero del servicio de asientos | servidor de caché (distribuida) o replicación entre unidades (replicada) |
| Acceso al dato en la compra | transacción síncrona con confirmación duradera | operación en memoria, remota (distribuida) o local (replicada), sin esperar a la base de datos |
| Efecto de una instancia nueva | más clientes sobre el mismo almacén | más cómputo; otro cliente del servidor de caché, o una réplica que carga el estado de otra unidad |
| Escrituras duraderas en el pico | síncronas, dentro de la transacción | encoladas y aplicadas de forma asíncrona |
| Qué se prepara para el pico | el almacén duradero | la capa de caché |
| Consistencia con la base de datos | inmediata dentro del servicio | eventual |

## Las objeciones: particionar, añadir caché o encolar

**Particionar o cambiar de almacén.** Microservicios no está atado a una base
relacional única: cada servicio puede elegir el almacenamiento que más le convenga
(Richards & Ford, 2020, cap. 17). Una escritura condicional por asiento sobre un
almacén particionado es una alternativa seria, que además conserva un árbitro
duradero. Tiene, no obstante, tres costos, según nuestro análisis. En un lanzamiento
la demanda se concentra en un solo concierto, de modo que particionar por concierto
no reparte nada. Particionar más fino complica reservar varios asientos de forma
atómica. Y cada intento sigue esperando una confirmación duradera, por lo que esa
capa debe dimensionarse de antemano para el pico.

**Añadir caché.** El libro admite la caché y la replicación en Microservicios
(Richards & Ford, 2020, cap. 17), e incluso propone cachear como solución pragmática
si el límite es la base de datos (cap. 23). La escritura asíncrona tampoco distingue
a los estilos, porque ambos usan mensajería como *data pumps* (cap. 14). Lo propio de
Space-Based es resolver toda la transacción contra datos en memoria e incorporar, como
parte del estilo, la sincronización de la caché y la gestión elástica de las unidades.

Este análisis no afirma que Microservicios no pueda alcanzar esta carga. Afirma que
debe elegir entre preparar el almacén duradero para el pico o construir en la ruta de
compra mecanismos que Space-Based ya define, y que en este escenario la segunda
opción acerca mucho ambos diseños: una unidad de procesamiento puede contener
servicios pequeños de un solo propósito (Richards & Ford, 2020, cap. 15).

**Encolar la entrada.** Una cola de admisión suaviza la ráfaga en cualquiera de los
dos estilos, pero regula la llegada sin aumentar la capacidad: traslada la espera al
usuario, un límite que el libro señala para las colas en otro caso de elasticidad
(Richards & Ford, 2020, cap. 20). Complementa la elección, pero no la sustituye.

## Qué se paga y hasta dónde aplicar la decisión

Space-Based también tiene costos importantes. Frente a Microservicios, tiene menos
testabilidad (una estrella contra cuatro), capacidad evolutiva (tres contra cinco),
tolerancia a fallos (tres contra cuatro) y facilidad de despliegue (tres contra
cuatro); en simplicidad, ambos tienen la puntuación mínima (Richards & Ford, 2020,
figs. 15-15 y 17-13). Las pruebas a carga máxima son tan difíciles que suelen acabar
haciéndose en producción (cap. 15), un riesgo serio cuando cada lanzamiento es
irrepetible. El libro añade que es un estilo relativamente caro, por las licencias
de caché y el alto consumo de recursos, y que hay que cuidar que ningún fallo pierda
datos (cap. 15), un riesgo directo cuando la compra confirmada se escribe de forma
asíncrona. Si cayeran todas las unidades, la caché tendría que recargarse desde la
base de datos (cap. 15). Space-Based tampoco resuelve otros cuellos de botella: una
pasarela de pago externa no se vuelve elástica porque lo sea el sistema, y habrá que
separar la reserva del cobro.

Conviene, pues, limitar el estilo a la ruta crítica de consulta de disponibilidad y
reserva de asientos. Las características operativas se definen por quantum, lo que
puede conducir a arquitecturas híbridas (Richards & Ford, 2020, cap. 7), y el propio
Space-Based admite repartir la aplicación en varias unidades por área funcional
(cap. 15). El perfil de usuario, la administración del catálogo o las notificaciones
previsiblemente no sufren este pico del mismo modo y pueden seguir como servicios
convencionales. El resultado es un híbrido, y la elección de este inciso se refiere a
esa ruta, donde se concentra el requisito de elasticidad.

El libro presenta Space-Based como un buen encaje para este caso por sus requisitos
de elasticidad (Richards & Ford, 2020, cap. 15), pero la razón de fondo no es una
puntuación más alta. Los dos estilos pueden alcanzar esta carga, y la diferencia está
en qué hay que preparar para el pico: Microservicios necesita un almacén duradero
dimensionado de antemano para los intentos, mientras que Space-Based traslada ese
pico a una capa en memoria y deja a la base de datos solo el flujo diferido. Si la
carga se repartiera entre dominios y la prioridad fuera la evolución independiente,
Microservicios sería la mejor opción. El precio de la elección es la consistencia
eventual con la base de datos y, según el modelo de caché, el riesgo de colisiones o
la dependencia de un servidor central (cap. 15), dos problemas críticos cuando los
asientos son numerados; se tratan en el inciso b).
