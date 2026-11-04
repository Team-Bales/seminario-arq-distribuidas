# P1 — Análisis Comparativo de Comunicación y Acoplamiento

> Un sistema de subastas en línea requiere que, al finalizar una subasta, se realicen
> múltiples acciones: notificar al ganador, procesar el pago y actualizar el inventario.
> En un diseño con **Microservicios**, un arquitecto podría optar por una coreografía
> de eventos (comunicación asíncrona) o una orquestación mediante un servicio mediador.
> En un diseño con **Service-Based Architecture**, estas acciones probablemente se
> ejecutarían dentro de un mismo servicio de dominio de grano grueso.
>
> Basado en los conceptos de *connascence* (connascencia) y los trade-offs de la
> comunicación síncrona vs. asíncrona, analice:

## a) Acoplamiento dinámico

¿Cómo afecta cada enfoque (orquestación vs. coreografía en microservicios, y servicios
de grano grueso en SBA) al acoplamiento dinámico entre los componentes del sistema?

- Respuesta: [`inciso-a.md`](inciso-a.md) · fragmento LaTeX: [`latex/inciso-a.tex`](latex/inciso-a.tex)

## b) Tolerancia a fallos y consistencia

¿Qué implicaciones tiene cada diseño para la tolerancia a fallos y la consistencia de
los datos (ACID vs. BASE) si el servicio de pago falla justo después de que se notifica
al ganador?

- Respuesta: [`inciso-b.md`](inciso-b.md) · fragmento LaTeX: [`latex/inciso-b.tex`](latex/inciso-b.tex)
