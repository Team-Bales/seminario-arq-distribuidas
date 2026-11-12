# P4-a — Error fundamental de diseño

El error fue caer en la **entity trap**: Richards y Ford (2020, cap. 8) advierten
que este anti-patrón surge cuando un arquitecto confunde las relaciones de la base
de datos con los *workflows* reales de la aplicación, una correspondencia que casi
nunca se cumple en la práctica. El equipo tomó cada entidad de la base de datos
(Empleado, Departamento, Nómina, Beneficios) y la convirtió directamente en un
servicio —lo que los autores describen como un mapeo componente-relacional de un
framework hacia la base de datos, no una arquitectura (Richards & Ford, 2020,
cap. 8)—, en vez de preguntarse qué *workflows* de negocio necesitaba soportar el
sistema.

El error contradice además el principio rector de los microservicios, el
*bounded context*: cada servicio debe modelar un dominio o un *workflow*, no
una tabla, y Richards y Ford (2020, cap. 17) advierten expresamente que no hay
que modelar los servicios a imagen de entidades individuales de la base de
datos. Un servicio por entidad parte un mismo contexto, la gestión del
personal, en cuatro fragmentos sin sentido por separado, de modo que cada
operación de negocio tiene que reconstruir el contexto por la red.

Contratar a un nuevo empleado es precisamente uno de esos *workflows*: un único
proceso de negocio en el que varias entidades deben cooperar dentro de la misma
transacción. Richards y Ford (2020, cap. 17) señalan justo ese criterio —las
entidades que cooperan en una transacción apuntan a una buena frontera de
servicio— como una de las guías de granularidad que un arquitecto de
microservicios debe seguir. Al diseñar primero por entidad y descubrir recién
después que "contratar" necesitaba a casi todos los servicios, el equipo invirtió
el orden correcto: dejó que la estructura de la base de datos dictara la
granularidad, en vez de que el *workflow* de negocio dictara dónde debían estar
los límites.

La consecuencia es doble. En términos de **acoplamiento**, la transacción de
contratación cruza ahora límites de servicio en cada paso: construir
transacciones a través de fronteras de microservicios viola el principio de
desacoplamiento que sostiene al estilo, y además genera connascencia de
valores entre servicios, la peor forma de connascencia dinámica (Richards &
Ford, 2020, cap. 17). El consejo de los propios autores es elocuente: "Don't
do transactions in microservices—fix granularity instead!" (Richards & Ford,
2020, cap. 17). En términos de
**rendimiento y fiabilidad**, cada llamada síncrona adicional entre servicios suma
latencia de red y un nuevo punto de fallo (Richards & Ford, 2020, cap. 9), así que
encadenar a casi todos los servicios para una sola operación de negocio multiplica
ambos riesgos en vez de distribuirlos.

El resultado es, en la práctica, un **monolito distribuido**: los servicios
quedaron demasiado finos para actuar con autonomía, porque cuando el grano es
demasiado pequeño el diseño termina exigiendo más comunicación entre componentes
de la que se ahorra en independencia (Richards & Ford, 2020, cap. 17), pero siguen
tan acoplados entre sí como las tablas del monolito original. El equipo pagó el
costo operativo completo de la distribución —red, serialización, descubrimiento de
servicios, fallos parciales— sin obtener ninguno de sus beneficios: ni despliegue
independiente real de la operación de contratación, ni aislamiento de fallos, ni
escalabilidad diferenciada.
