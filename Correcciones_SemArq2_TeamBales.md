# Correcciones a `SemArq2_TeamBales.pdf`

**Fecha de la revisión:** 12/11/2026 · **Entrega:** 13/11/2026
**Documento revisado:** la versión del PDF que circuló por Telegram (21 páginas, generada el 11/11).

Cuatro revisores independientes revisaron el documento:

1. Cumplimiento de la orientación y formato.
2. Citas de P1 y P3.
3. Citas de P4, P5 y P6.
4. Lógica y pertinencia de los 12 incisos.

Los hallazgos principales se comprobaron después contra el texto de Richards y Ford (2020).

Las páginas son las **impresas al pie**, las mismas del índice. Los textos propuestos son sugerencias listas para pegar: cada autor decide la redacción final.

---

## Resumen

El documento está completo: tiene las 6 preguntas con sus 12 incisos, el índice cuadra y las 5 referencias corresponden con las citas en ambos sentidos. La Declaración de IA cumple la plantilla. Aun así, **hay 7 correcciones que conviene hacer antes de entregar**; casi todas son de minutos.

| # | Corrección | Dónde | Responsable |
|---|---|---|---|
| 1 | Atribución equivocada del *Workflow Event Pattern* y citas que faltan | P3, págs. 12-13 | Enrique |
| 2 | Falta el *bounded context*, que el enunciado exige | P4-a, pág. 13 | Heily |
| 3 | Comillas rotas en la cita de la Primera Ley | Conclusiones, pág. 19 | Heily |
| 4 | El ejemplo se contradice con su propio orden de pasos | P6-b, págs. 18-19 | Enrique |
| 5 | Atribuciones al cap. 13 sin respaldo y premisa de ACID imprecisa | P5, págs. 15-17 | Mauricio |
| 6 | Sigla de connascencia equivocada (CoT por CoE) | P1-a, pág. 1 | Heily |
| 7 | El PDF no coincide con lo que hay en `main` | Repositorio | Heily |

---

## 1. Corregir antes de entregar

### 1.1 P3: atribución equivocada y citas que faltan (Enrique)

**Atribución (P3-b, pág. 12).** El texto dice que el *Workflow Event Pattern* está "descrito en el Capítulo 14 de *Software Architecture: The Hard Parts* (Ford et al., 2021)". El patrón está en *Fundamentals of Software Architecture*, cap. 14, sección "Error Handling", y el enunciado se refiere a ese libro. Según su índice, el cap. 14 de *Hard Parts* trata datos analíticos (esto no pudo verificarse, porque no tenemos ese libro).

> El *Workflow Event Pattern*, descrito por Richards y Ford (2020, cap. 14), ofrece un mecanismo basado en tres principios: delegación, contención y reparación.

**Consecuencia en Referencias.** Ford et al. (2021) solo se cita aquí. Tras el cambio quedará sin citar, así que **hay que quitarla de Referencias**, salvo que se cite en otro sitio donde aporte algo.

**P3-a sin ninguna cita.** Casi todas sus ideas vienen del cap. 14 de Richards y Ford (2020): que nadie se entera del fallo sin un mediador, que los demás procesadores siguen como si nada y que no hay forma de avisar al usuario final. Añadir **(Richards & Ford, 2020, cap. 14)** en:

- **3.1.1**, al cerrar la idea de que no hay orquestador.
- **3.1.2**, tras "sin saber que una fase crítica ha fallado".
- **3.1.4**, tras la mención del bloqueo de la cola.

**Escenario de P3-a.** El texto habla de un procesador "caído", pero el enunciado describe un procesador que falla repetidamente con un tipo concreto de mensaje. Cambiar "ha caído" por "falla sistemáticamente con ese tipo de mensaje".

**Afirmación técnica sin respaldo (P3-a).** "El broker es ligero y carece de capacidades de coordinación transaccional (two-phase commit)" no está en el libro y es discutible, porque algunos brokers admiten transacciones. La causa que da el libro es otra. Texto propuesto:

> En esta topología ningún componente posee el estado de la petición original: nadie puede coordinar la atomicidad entre procesadores ni reiniciar la transacción desde donde quedó (Richards & Ford, 2020, cap. 14). Cada procesador confirma su propio trabajo y el sistema solo ofrece consistencia eventual.

**P3-b sin citas en 3.2.1-3.2.3.** Añadir **(Richards & Ford, 2020, cap. 14)** en la descripción de la delegación, contención y reparación, y en la del *workflow processor* y el *dashboard*.

**Procesamiento fuera de orden (P3-b, 3.2.3).** La idempotencia protege contra duplicados, no contra el desorden. El libro propone una mitigación concreta:

> Cuando el orden importa dentro de un contexto, Richards y Ford (2020, cap. 14) proponen que el consumidor registre la clave del mensaje en error (en su ejemplo, el número de cuenta) y retenga en una cola temporal, en orden FIFO, los mensajes posteriores con esa clave hasta que el corregido se procese. Como el mensaje reinyectado llega como nuevo, conviene además que los consumidores sean idempotentes.

**Detalle de estilo.** P3 escribe "costes"; el resto del documento, "costos".

### 1.2 P4-a: falta el *bounded context* (Heily)

El enunciado dice "basado en los principios de *bounded context* y las guías de granularidad del Capítulo 17", y el término no aparece en toda la P4. Texto para añadir tras el primer párrafo de 4.1:

> El error contradice además el principio rector de los microservicios, el *bounded context*: cada servicio debe modelar un dominio o un *workflow*, no una tabla, y Richards y Ford (2020, cap. 17) advierten expresamente que no hay que modelar los servicios a imagen de entidades individuales de la base de datos. Un servicio por entidad parte un mismo contexto, la gestión del personal, en cuatro fragmentos sin sentido por separado, de modo que cada operación de negocio tiene que reconstruir el contexto por la red.

### 1.3 Conclusiones: comillas rotas (Heily)

En la pág. 19 se imprime `.everything in software archi-tecture is a trade-off"(Richards…`. Son comillas rectas `"`, que babel en español convierte en otra cosa. Además, falta el espacio antes del paréntesis. La misma cita en P1 sale bien.

Corrección en el `.tex`: usar `“…”` o ``` ``…'' ```, como en P1, y dejar un espacio antes de "(Richards".

### 1.4 P6-b: el ejemplo se contradice (Enrique)

El texto fija el orden pedido → pago → inventario → envío → notificación, y dice que cada paso depende del anterior. Luego afirma que, si el pago falla, "el inventario pudo haberse reservado y quizá se envió una notificación prematura". Con ese orden, ninguna de las dos cosas ha ocurrido todavía.

En el ejemplo del libro (cap. 14), el pago, el inventario y la notificación se procesan en paralelo. Sustituir la frase sobre la ejecución en orden por:

> Para reducir la latencia, el pago, la reserva de inventario y el aviso al cliente se lanzan en paralelo, y el envío solo se programa cuando los tres han confirmado (Richards & Ford, 2020, cap. 14).

Con eso, el párrafo de la compensación queda coherente.

**Relacionado (P6-b frente a P1-b).** P6-b afirma que sin coordinador "cada servicio tendría que conocer la lógica de compensación de los demás". En una saga coreografiada, cada servicio compensa lo suyo al recibir un evento de fallo, y P1-b lo describe así. Texto propuesto:

> Sin un coordinador, la compensación queda repartida: cada servicio debe escuchar los eventos de fallo relevantes y deshacer su parte, y nadie ve el estado global de la saga.

### 1.5 P5: atribuciones sin respaldo y premisa de ACID (Mauricio)

**Premisa de ACID (P5-a).** El texto atribuye la garantía ACID a que los servicios "compartan una base de datos". Según el libro, ACID se conserva porque la transacción queda **dentro de un único servicio de dominio**, y si la interfaz o el API gateway orquestan dos servicios hacen falta sagas y BASE. El cap. 13 dice que esta arquitectura "preserves ACID transactions better than any other distributed architecture" por el grano grueso de sus servicios. Texto propuesto para la salvedad:

> Esto no significa que SBA garantice ACID automáticamente: la garantía solo se mantiene mientras cada transacción financiera quede acotada a un único servicio de dominio; si la interfaz o el API gateway tienen que orquestar dos o más servicios, la transacción vuelve a depender de sagas y BASE (Richards & Ford, 2020, cap. 13). Por eso los límites de los servicios deben trazarse alrededor de las transacciones financieras.

**"Microservices puede alcanzar consistencia fuerte… (cap. 13)".** El cap. 13 dice lo contrario: las transacciones BASE no alcanzan la integridad de ACID. El cap. 17 aconseja usar la saga con moderación. Texto propuesto:

> Microservices puede construir comportamiento transaccional entre servicios mediante sagas, pero eso va contra la razón de elegir el estilo y solo ofrece consistencia eventual; los autores aconsejan usar la saga con moderación (Richards & Ford, 2020, caps. 13 y 17).

**Two-phase commit y sagas citados como cap. 13.** El libro no menciona el *two-phase commit*. Las sagas están en los caps. 9 y 17. Opciones:

- Dejar el *two-phase commit* sin cita, como conocimiento general.
- Citar las sagas como (Richards & Ford, 2020, caps. 9 y 17).

**"SBA ofrece menos independencia de despliegue que Microservices" (P5-b).** El libro da **4 estrellas de *deployability* a ambos estilos** (figs. 13-9 y 17-13; comprobado). La diferencia está en el alcance y el riesgo de cada cambio y en la evolutividad (3 frente a 5). Texto propuesto:

> Ambos estilos puntúan 4 estrellas en facilidad de despliegue; la diferencia está en el alcance de cada cambio y en la evolutividad (3 estrellas en SBA frente a 5 en Microservices), porque un cambio en SBA despliega más código, obliga a probar el servicio completo —pagos incluidos— y conlleva más riesgo (Richards & Ford, 2020, cap. 13, y figs. 13-9 y 17-13).

**Equipo con experiencia mixta.** Hoy se resuelve en una frase. El libro lo respalda con las puntuaciones de simplicidad (SBA 3 estrellas, Microservices 1) y de coste global (SBA 4, Microservices 1). Puede añadirse:

> Las puntuaciones del libro lo confirman: SBA obtiene 3 estrellas en simplicidad y 4 en coste global, frente a 1 y 1 de Microservices (Richards & Ford, 2020, figs. 13-9 y 17-13).

**Consejo del libro que P5 no usa y P4 sí.** El cap. 17 recomienda: "Don't do transactions in microservices—fix granularity instead!". Para un sistema financiero ACID, eso empuja hacia servicios más gruesos, lo que refuerza la elección de SBA.

### 1.6 P1-a: sigla equivocada (Heily)

En la pág. 1 dice "nadie garantiza el orden (CoT)". En el libro (cap. 3), que el orden importe es **connascencia de ejecución (CoE)**; CoT es el momento, como en las condiciones de carrera. El propio texto da esas definiciones dos párrafos antes. Texto propuesto:

> …nadie garantiza el orden de ejecución (CoE) ni evita las carreras entre consumidores (CoT), el estado global queda eventualmente consistente (CoV latente) y…

### 1.7 Repositorio: el PDF no coincide con `main` (Heily)

El PDF contiene cosas que no están en `main`: la sección de Referencias con las 5 obras, la Declaración de IA, la portada completa y las citas corregidas de P6. La orientación exige que el mismo documento esté en el repositorio, junto con su fuente LaTeX. Hay que hacer push de esos cambios y subir también el PDF final.

La tabla **Miembros** del `README.md` sigue con solo el nombre de Heily y la columna de preguntas vacía; conviene sincronizarla con la portada.

---

## 2. Si hay tiempo

**P1 (Heily)**
- **Orquestación síncrona (P1-a, pág. 1).** El texto da por hecho que la orquestación es síncrona ("tres saltos síncronos", "un solo quantum"), pero el mediador de la topología *mediator* del cap. 14 envía comandos asíncronos. Basta una aclaración: "Suponemos un mediador que invoca cada paso de forma síncrona (petición-respuesta); si enviara comandos asíncronos, conservaría el control del flujo sin fundir los servicios en un solo quantum". Además, el quantum y la connascencia síncrona son del cap. 7, no de los caps. 9 y 14.
- **"Dos reglas gobiernan su uso" (P1-a).** Mezcla las tres propiedades de la connascencia (fuerza, localidad y grado) con las dos reglas de Weirich (cap. 3). Tampoco es que las formas fuertes sean tolerables "solo si son locales": el libro dice que son menos dañinas cuanto más próximos están los elementos.
- **"La única connascencia es el contrato del evento" (P1-a).** Choca con la frase siguiente, que nombra CoT y CoV latentes. Cambiar por "la única connascencia *explícita*".
- **Compensaciones (P1-b).** La secuencia fijada es notificar → pagar → actualizar, pero la compensación incluye "liberar el inventario reservado", que aún no se ha tocado, y "retirar la notificación", que antes se calificó de irreversible. Propuesta: "En esta secuencia solo la notificación precede al pago, y como no puede retirarse, se compensa con un segundo aviso; el inventario ni siquiera llega a tocarse".
- **Patrones sin cita (P1-b).** El *outbox* y la saga coreografiada pueden citarse a (Richardson, 2018), que ya está en Referencias.

**P4 (Heily)**
- **Connascencia de valor (P4-a).** El libro la menciona como efecto *añadido* de las transacciones entre servicios. El problema central, según el libro, es que violan el desacoplamiento. Puede citarse el consejo del cap. 17 ("Don't do transactions in microservices—fix granularity instead!"), que respalda toda la tesis de P4.
- **Servicios demasiado pequeños (P4-a).** "Cuando el grano es demasiado pequeño…" se cita al cap. 8, pero la fuente directa para servicios es el cap. 17, que habla del error de hacer servicios demasiado pequeños.
- **Matices del libro (P4-b).** El libro dice "puede considerar reagruparlos" y que las entidades que comparten transacción "a menudo" señalan la frontera. El texto lo presenta como regla fija.
- **Terminología (P4-b).** "Coreografía en vez de orquestación síncrona" confunde los términos: el diseño original ya era una coreografía síncrona. Mejor: "coreografía asíncrona basada en eventos, en vez de llamadas síncronas encadenadas".
- **Premisa transaccional (P4-b).** No se justifica que nómina y beneficios deban ser atómicos con el alta. Basta con declararlo como supuesto.

**P6 (Enrique)**
- **Cita a Evans (P6-a).** "Cada servicio modela un dominio o flujo completo" es una paráfrasis casi literal de Richards y Ford (2020, cap. 17), no de Evans (2003). Atribuir esa frase a Richards y Ford y dejar a Evans para el concepto de *bounded context*.
- **"La única excepción tolerable" (P6-a).** Es demasiado absoluto, y compartir contratos tipados también crea connascencia estática (cap. 7). Mejor: "una excepción habitual, que aun así acopla".
- **Herramientas y "sin redeploy" (P6-a).** El cap. 17 trata el *sidecar* y el *service mesh*, pero no nombra Splunk, ELK, Datadog, Prometheus, Istio ni Linkerd: la cita no debería cubrirlos. "Sin requerir redeploy" es inexacto, porque actualizar el *sidecar* suele implicar reiniciar los pods. Mejor: "sin modificar el código de los servicios".
- **Motores BPM (P6-b).** El texto dice que el orquestador "no es un motor externo tipo BPM", pero luego nombra Camunda y Zeebe, que son motores BPMN, y AWS Step Functions, que es un orquestador externo.
- **"Garantía de integridad" (P6-b).** El libro habla de "mejor consistencia" y de usar la saga con moderación, no de una garantía.
- **Escenarios sin cita (P6-b, 6.2.1).** Vienen de los caps. 14 y 17. Añadir (Richards & Ford, 2020, caps. 14 y 17), y (Richards & Ford, 2020, cap. 14) tras "single point of failure".

**P5 (Mauricio)**
- **Cita a contratos (P5-b).** "Contratos estables, compatibilidad hacia atrás…" se cita al cap. 13, que no trata eso. La mitigación que sí propone el cap. 13 es particionar lógicamente la base de datos con librerías compartidas por dominio.

---

## 3. Formato y consistencia (menor)

**Terminología**
- El estilo aparece como "Microservices" (unas 15 veces: títulos 2.1 y 6.1, y P5) y como "Microservicios" (unas 14: P2 y portada). Elegir una forma.
- La sigla "SBA" se define cuatro veces; basta una.

**Tipografía**
- Las cursivas de los términos en inglés no son uniformes: van en cursiva en P3 y P6 y en redonda en P1, P4 y P5.
- El guion largo es espaciado en P1 y P4 y pegado en la Introducción, P3 y P6.
- "*Broker*" aparece con y sin mayúscula, y "inciso a" con y sin paréntesis.

**Maquetación**
- Los títulos 4 y 6 se parten con guion ("Mi-croservicios", "Mo-dernas"). Se evita con `\hyphenation{}` o un salto manual.

**Fuente de las tablas**
- Las tablas 3, 4 y 5 indican "Elaboración propia a partir de…", pero las 1, 2 y 6 no. Unificar.

**Citas abreviadas en P2 (Ernesto)**
- Algunos párrafos citan solo "(cap. 15)" sin la forma completa: el segundo párrafo de 2.1.1, "En términos de quantum…" y "El libro recomienda…".
- Con cinco obras en la bibliografía es ambiguo: poner "(Richards & Ford, 2020, cap. 15)" en la primera cita de cada párrafo.

**Forma narrativa de las citas en P6**
- "Como advierte Sam Newman en *Building Microservices* (Newman, 2021)" debería ser "Newman (2021) advierte…".
- Lo mismo con Richardson.

**Referencias**
- Faltan los subtítulos: *Building Microservices: Designing Fine-Grained Systems* (Newman) y *Microservices Patterns: With Examples in Java* (Richardson).
- Si se mantuviera Ford et al. (2021), su subtítulo es *Modern Trade-Off Analyses for Distributed Architectures*.

**Declaración de IA**
- Gemini y Codex aparecen sin versión ni modelo. La orientación la pide "cuando sea posible".
- La declaración afirma que "los números de capítulo… se cotejaron con los textos originales"; el error de P3 lo contradice, así que corregir 1.1 es importante también por esto.

**Portada**
- La orientación pide "nombres completos". "Mauricio Bridon" lleva un solo apellido y "Ernesto A. Soler Choong" usa una inicial. Si aplica, completarlos también en la Declaración.

**Nombre del fichero**
- `SemArq2_TeamBales.pdf`, cuando la portada llama al equipo "Bales". La orientación usa `SemArq1_NombreEquipo`, copiado del seminario anterior.
- No es grave, pero conviene decidirlo a propósito: `SemArq2_TeamBales.pdf` o `SemArq2_Bales.pdf`.

---

## 4. Lo que está bien

- **Portada:** equipo, los 4 integrantes con su GitHub, la pregunta de cada uno y el enlace al repositorio.
- **Contenido:** los 12 incisos están completos, sin "Pendiente" ni relleno, y el índice coincide con las páginas.
- **Citas literales:** las 6 en inglés de P1 coinciden carácter a carácter con el libro, igual que las de P2.
- **Referencias:** 5 entradas en APA, en orden alfabético y en correspondencia exacta con las citas, salvo lo dicho sobre Ford et al. (2021).
- **Declaración de IA:** va después de Referencias, con el título exacto, y nombra a los 4 integrantes, sus herramientas, el propósito y la forma de uso.
- **Maquetación:** 6 tablas numeradas, sin cortes ni desbordes, y numeración de páginas continua.
- **Coherencia entre preguntas:** P1, P4 y P5 coinciden en SBA y ACID (salvo el matiz de 1.5), y P2 encaja con el resto.

## Notas sobre la verificación

- Se comprobó contra *Fundamentals of Software Architecture* (Richards & Ford, 2020): las citas literales, los capítulos atribuidos, las puntuaciones de las figuras 13-9, 15-15 y 17-13, y los pasajes que respaldan los textos propuestos.
- **No se pudieron verificar** las afirmaciones atribuidas a Ford et al. (2021), Evans (2003), Newman (2021) y Richardson (2018), porque no tenemos esos libros. Las observaciones sobre ellas se basan en conocimiento general y conviene que alguien con acceso las confirme.
