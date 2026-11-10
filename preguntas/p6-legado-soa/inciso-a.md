# P6-a — Reutilización en Microservices

La arquitectura de microservicios aborda el dilema de la reutilización de una manera
radicalmente opuesta a la *Service-Oriented Architecture* (SOA) tradicional. Su premisa
de partida es contraintuitiva pero firme: **el diseño altamente desacoplado prioriza la
duplicación sobre el acoplamiento**. Allí donde SOA buscaba maximizar la reutilización
mediante servicios empresariales compartidos, los microservicios aceptan —e incluso
promueven— cierta duplicación de código a cambio de preservar la independencia de
despliegue y evolución de cada servicio (Richards & Ford, 2020, caps. 16 y 17).

## Reutilización de lógica de negocio

En SOA, la reutilización se perseguía creando servicios empresariales de grano fino,
altamente reutilizables, para evitar reescribir lógica de negocio. En la práctica,
esto generaba un fuerte acoplamiento y dependencias rígidas entre aplicaciones
consumidoras y servicios compartidos, lo que dificultaba la evolución independiente
y creaba cuellos de botella en el cambio (Richards & Ford, 2020, cap. 16).

Los microservicios, en cambio, se apoyan en el concepto de *Bounded Context* del
*Domain-Driven Design* (Eric Evans, 2003): cada servicio modela un dominio o flujo
completo, con sus propios datos y reglas de negocio, dentro de una frontera explícita.
Para evitar el acoplamiento y los problemas de despliegue independiente, **se prefiere
duplicar pequeñas porciones de código de negocio** —validaciones específicas, clases de
utilidad, cálculos puntuales— antes que compartir librerías o bases de datos comunes
entre dominios distintos (Richards & Ford, 2020, cap. 17).

Esta postura tensiona conscientemente el principio DRY (*Don't Repeat Yourself*),
válido dentro de un *Bounded Context*, pero deliberadamente relajado entre servicios:
el arquitecto acepta duplicar tres líneas de validación de cliente en dos servicios
antes que introducir una dependencia contractual que frene el despliegue independiente
de cualquiera de ellos. Como advierte Sam Newman en *Building Microservices* (O'Reilly,
2.ª ed., 2021), las librerías compartidas no son la respuesta al problema de la
reutilización en microservicios. La única excepción tolerable, y aun así con precaución,
es la compartición de contratos tipados (como DTOs o esquemas *protobuf*), que no
contienen lógica de negocio pero aseguran compatibilidad de interfaces.

## Reutilización operativa

A diferencia de la lógica de negocio, los **aspectos operativos o transversales**
(*cross-cutting concerns*) sí se benefician enormemente de la estandarización y del
soporte centralizado. *Logging*, monitoreo, circuit breakers, autenticación,
trazabilidad distribuida y retransmisiones son ejemplos donde la duplicación sería
contraproducente: cualquier divergencia entre servicios complicaría la observabilidad
y la operación del sistema.

Los microservicios separan con claridad estos *concerns* operativos del dominio de
negocio y los gestionan mediante infraestructura compartida: herramientas de
*logging* centralizado como Splunk, ELK o Datadog; plataformas de monitoreo como
Prometheus y Grafana; y, de forma más sofisticada, patrones de plataforma como
**sidecars** y **service meshes** (Istio, Linkerd) que inyectan transversalmente
comportamiento uniforme de observabilidad, seguridad y resiliencia sin contaminar el
código de negocio (Richards & Ford, 2020, cap. 17). Así, un servicio puede cambiar su
lógica de dominio sin tocar el *logging*, y la plataforma puede evolucionar su
estrategia de observabilidad sin requerir redeploy de los servicios.
