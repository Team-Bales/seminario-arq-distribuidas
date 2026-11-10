# P3-a — Impacto del fallo en una topología de Broker

En una arquitectura dirigida por eventos con topología de Broker, el flujo de mensajes se distribuye entre procesadores independientes a través de un intermediario ligero, sin que exista un componente central que orqueste la transacción global. Cuando un procesador falla de forma repetida sobre un mensaje concreto debido a un error en los datos, se desencadenan varios efectos en cadena que conviene analizar por separado.

## Invisibilidad del fallo

El primero y más problemático es la invisibilidad del fallo. Al no haber un mediador con visión global, ningún otro componente del sistema se entera de que el procesador ha caído ni de que el flujo de negocio asociado a ese mensaje se ha detenido. La transacción queda atrapada en un estado incompleto sin que se dispare ninguna alerta automática. Este bloqueo silencioso es precisamente la consecuencia directa de la ausencia de orquestador.

## Problemas de consistencia

A esto se suma un problema de consistencia. El resto de procesadores continúa ejecutándose con normalidad ---registrando pagos, modificando inventarios, emitiendo notificaciones paralelas--- sin saber que una fase crítica ha fallado. Se producen así asimetrías y estados inconsistentes en todo el ecosistema distribuido, lo que agrava el modelo de consistencia eventual inherente a la arquitectura. Cabe recordar que en esta topología el broker es ligero y carece de capacidades de coordinación transaccional (two-phase commit), por lo que tampoco es posible garantizar atomicidad entre varios procesadores.

## Impacto en el usuario final

Desde el punto de vista del usuario final, la experiencia se degrada de forma notable. Al operar de manera totalmente asíncrona y sin canal de retorno directo, el usuario no recibe ninguna retroalimentación sobre el fallo de su solicitud: la transacción queda en una especie de limbo indefinido, lo que afecta a la confianza y a la percepción general del servicio.

## Efecto poison pill y degradación del sistema

Por último, el mensaje que origina el fallo se comporta como un clásico poison pill: un mensaje que nunca podrá procesarse correctamente y que, si el procesador intenta reintentos síncronos o bloqueantes, acaba deteniendo la lectura de la cola (head-of-line blocking). Esto degrada drásticamente la responsiveness del sistema para los mensajes posteriores y, además, consume recursos ---CPU, memoria, conexiones--- en cada reintento inútil, con riesgo de entrar en un bucle infinito si no se define una política de máximo de reintentos.
