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
contratación cruza ahora límites de servicio en cada paso, generando la peor forma
de connascencia dinámica entre servicios —connascencia de valores—, exactamente lo
que Richards y Ford (2020, cap. 17) identifican como el problema central de
construir transacciones a través de fronteras de microservicios. En términos de
**rendimiento y fiabilidad**, cada llamada síncrona adicional entre servicios suma
latencia de red y un nuevo punto de fallo (Richards & Ford, 2020, cap. 9), así que
encadenar a casi todos los servicios para una sola operación de negocio multiplica
ambos riesgos en vez de distribuirlos.

El resultado es, en la práctica, un **monolito distribuido**: los servicios
quedaron demasiado finos para actuar con autonomía, porque cuando el grano es
demasiado pequeño el diseño termina exigiendo más comunicación entre componentes
de la que se ahorra en independencia (Richards & Ford, 2020, cap. 8), pero siguen
tan acoplados entre sí como las tablas del monolito original. El equipo pagó el
costo operativo completo de la distribución —red, serialización, descubrimiento de
servicios, fallos parciales— sin obtener ninguno de sus beneficios: ni despliegue
independiente real de la operación de contratación, ni aislamiento de fallos, ni
escalabilidad diferenciada.
