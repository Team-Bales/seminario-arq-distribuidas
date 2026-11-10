# P3-b — Workflow Event Pattern

El *Workflow Event Pattern*, descrito en el Capítulo 14 de *Software Architecture: The
Hard Parts* (Ford, Richards, Sadalage y Dehghani, O'Reilly, 2021), ofrece un mecanismo
basado en tres principios: **delegación, contención y reparación**.

## Funcionamiento: delegación, contención y reparación

La idea central es sencilla pero potente. Cuando el procesador de eventos encuentra un
error al procesar un mensaje por datos incorrectos, en lugar de bloquear el flujo o
reintentar indefinidamente, **delega inmediatamente el error** mediante mensajería
asíncrona hacia un componente especializado y continúa al instante con el siguiente
mensaje de la cola. Con esto se preserva la *responsiveness* global del sistema, que
es precisamente la preocupación principal en arquitecturas reactivas.

## Componentes adicionales del patrón

Para llevar el patrón a la práctica hacen falta tres componentes adicionales. El
primero es el *workflow processor*, que recibe los mensajes con error delegados y
aplica lógica programática —reglas deterministas, heurísticas o algoritmos de análisis
y *machine learning*— para intentar reparar automáticamente los datos defectuosos. Si
lo consigue, reinyecta el mensaje en la cola original para que siga su curso normal.
El segundo es el *dashboard*, una interfaz gráfica de gestión conceptualmente similar
a un cliente de correo electrónico, que entra en juego cuando el *workflow processor*
no logra determinar el error por sí solo. Los mensajes no reparables se enrutan hacia
este tablero, donde un operador humano o analista puede inspeccionar el error,
aplicar correcciones manuales y reenviar el mensaje a la cola original. El tercer
componente, transversal a los dos anteriores, son los **canales de eventos de error**,
dedicados al transporte asíncrono de los mensajes fallidos desde los consumidores hacia
el *workflow processor* y el *dashboard*.

## Trade-offs introducidos

El patrón, sin embargo, no es gratuito. El *trade-off* principal es el
**procesamiento fuera de orden** (*out-of-order processing*). Al desviar el mensaje
erróneo hacia el flujo de reparación mientras el consumidor sigue procesando los
mensajes siguientes, se altera la secuencia cronológica original de los eventos:
mensajes posteriores pueden completarse antes de que el mensaje corregido se
reincorpore y resuelva su estado previo. Esto obliga a diseñar los consumidores bajo
principios de *idempotencia* y tolerancia al procesamiento desordenado, un requisito
de diseño nada trivial.

Un segundo *trade-off*, menos evidente pero igualmente relevante, es la **latencia
individual**. Aunque la *responsiveness* global del sistema se mantiene, los mensajes
que entran en el flujo de reparación experimentan una latencia considerablemente mayor
—de milisegundos pasan a minutos, e incluso horas si requieren intervención humana—.
Esta latencia debe ser asumible desde la perspectiva del negocio.

Por último, la incorporación de todos estos componentes —*workflow processor*, colas de
errores, *dashboards*, mecanismos de reinyección— incrementa la **complejidad
arquitectónica y operativa** del sistema, con costes asociados de mantenimiento y de
personal para gestionar el *dashboard*. A pesar de estos costes, el patrón sigue
siendo una de las soluciones más pragmáticas para el manejo de errores en
arquitecturas asíncronas descentralizadas sin mediador.
