# P5-a — Consistencia de datos

## Comparación

La **Service-Based Architecture (SBA)** organiza el sistema en servicios de grano
relativamente amplio. Ese grano permite acotar una operación financiera y su
transacción dentro de un único servicio de dominio; por ello SBA preserva las
transacciones ACID mejor que otros estilos distribuidos (Richards & Ford, 2020,
cap. 13).

En **Microservices Architecture**, cada microservicio suele ser dueño de sus datos
y de su esquema. Una operación financiera que atraviesa varios microservicios
cruza también varios límites transaccionales. Microservices puede construir
comportamiento transaccional entre servicios mediante sagas, pero eso va contra la
razón de elegir el estilo y solo ofrece consistencia eventual; los autores
aconsejan usar las sagas con moderación (Richards & Ford, 2020, caps. 9 y 17).

Respecto al **despliegue independiente**, ambos estilos puntúan cuatro estrellas
en facilidad de despliegue. La diferencia está en el alcance y el riesgo de cada
cambio, y en la evolutividad: SBA obtiene tres estrellas frente a cinco de
Microservices. Un cambio en SBA despliega más código, puede exigir probar el
servicio completo —incluido el flujo de pago— y conlleva más riesgo (Richards &
Ford, 2020, cap. 13, figs. 13-9 y 17-13).

Para un **equipo con experiencia mixta**, SBA suele ser más accesible. Las
puntuaciones del libro lo confirman: SBA obtiene tres estrellas en simplicidad y
cuatro en coste global, frente a una y una de Microservices, respectivamente
(Richards & Ford, 2020, figs. 13-9 y 17-13).

## Decisión

Para el requisito prioritario de **consistencia absoluta**, SBA es la opción más
adecuada, porque permite diseñar servicios de dominio suficientemente gruesos
como para que cada transacción financiera permanezca local a uno de ellos
(Richards & Ford, 2020, caps. 13 y 17).

Esto no significa que SBA garantice ACID automáticamente: la garantía solo se
mantiene mientras cada transacción financiera quede acotada a un único servicio de
dominio; si la interfaz o el API gateway tienen que orquestar dos o más servicios,
la transacción vuelve a depender de sagas y BASE (Richards & Ford, 2020, cap. 13).
Por eso los límites de los servicios deben trazarse alrededor de las transacciones
financieras.
