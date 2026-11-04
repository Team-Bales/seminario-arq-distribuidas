# P2 — Diseño para Elasticidad Extrema y Consistencia de Datos

> Considere el sistema de venta de entradas para conciertos descrito en el libro
> (Capítulo 15). Durante el lanzamiento de entradas para un artista popular, el
> sistema experimenta un aumento repentino de usuarios de cientos a decenas de miles
> en segundos. Se le pide como arquitecto diseñar la solución.

## a) Space-Based vs. Microservices

Justifique por qué una **Space-Based Architecture** sería una elección más adecuada
que una **Microservices Architecture** para este escenario específico, enfocándose en
las características de elasticidad y escalabilidad.

- Respuesta: [`inciso-a.md`](inciso-a.md) · fragmento LaTeX: [`latex/inciso-a.tex`](latex/inciso-a.tex)

## b) Colisiones de datos en cachés replicadas

La Space-Based Architecture introduce el problema de las colisiones de datos en
cachés replicadas. Describa cómo un arquitecto podría mitigar el riesgo de colisiones
de datos en el contexto de la venta de asientos numerados, donde la consistencia es
crucial. ¿Qué mecanismos o trade-offs consideraría?

- Respuesta: [`inciso-b.md`](inciso-b.md) · fragmento LaTeX: [`latex/inciso-b.tex`](latex/inciso-b.tex)
