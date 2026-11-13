# P5-b — Despliegue independiente y trade-offs

## Capacidad de despliegue independiente

Ambos estilos puntúan cuatro estrellas en facilidad de despliegue. En
**Microservices Architecture**, cada microservicio puede versionarse, probarse y
desplegarse de forma independiente cuando sus contratos son compatibles. Esto
permite limitar el alcance de un cambio a los servicios afectados, aunque los
cambios de contratos, esquemas o flujos distribuidos todavía requieren
coordinación (Richards & Ford, 2020, fig. 17-13).

La diferencia está en el alcance y el riesgo de cada cambio, y en la
evolutividad: **Service-Based Architecture (SBA)** obtiene tres estrellas frente
a cinco de Microservices. Como los servicios de SBA son más grandes, un cambio
despliega más código, puede exigir probar el servicio completo —incluido el flujo
de pago— y conlleva más riesgo (Richards & Ford, 2020, cap. 13, figs. 13-9 y
17-13).

## Trade-offs si se prioriza la consistencia

Si el CTO prioriza la consistencia ACID absoluta, debe aceptar que la agilidad no
se obtiene principalmente por dividir más los servicios, sino por diseñar límites
de dominio que mantengan cada transacción financiera dentro de un único servicio.
SBA facilita este objetivo mediante servicios de grano grueso; si una interfaz o
un API gateway debe orquestar varios servicios, el flujo vuelve a depender de
sagas y BASE (Richards & Ford, 2020, cap. 13). En consecuencia, puede implicar:

- releases coordinados y ventanas de despliegue más amplias;
- mayor acoplamiento entre equipos y servicios que comparten datos;
- menor libertad para elegir tecnologías o escalar cada parte por separado; y
- que un cambio pequeño tenga un ciclo de validación más grande.

Microservices puede construir comportamiento transaccional entre servicios
mediante sagas, pero eso va contra la razón de elegir el estilo y solo ofrece
consistencia eventual; los autores aconsejan usar las sagas con moderación
(Richards & Ford, 2020, caps. 9 y 17).

El compromiso razonable es elegir SBA para los flujos financieros que requieren
ACID y limitar la coordinación mediante contratos estables, compatibilidad hacia
atrás y migraciones graduales. Los componentes que no participan en esas
transacciones pueden evolucionar con mayor independencia. Así, el CTO prioriza la
corrección financiera sin asumir que todo el sistema debe desplegarse siempre como
un único bloque.
