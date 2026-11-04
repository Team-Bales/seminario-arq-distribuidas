# P3 — Análisis de Fallos y Resiliencia en Arquitecturas Dirigidas por Eventos

> En una **Event-Driven Architecture** con topología de **Broker**, un procesador de
> eventos (*Event Processor*) falla repetidamente al procesar un tipo específico de
> mensaje debido a un error en los datos.

## a) Impacto del fallo

Describa el impacto de este fallo en el flujo general del sistema y en la experiencia
del usuario final, considerando que no hay un mediador central.

- Respuesta: [`inciso-a.md`](inciso-a.md) · fragmento LaTeX: [`latex/inciso-a.tex`](latex/inciso-a.tex)

## b) Workflow Event Pattern

Proponga una solución arquitectónica utilizando el *Workflow Event Pattern* (Patrón
de Eventos de Flujo de Trabajo) descrito en el Capítulo 14. Explique detalladamente
cómo este patrón mitiga el problema y qué componentes adicionales (como el *workflow
processor* y el *dashboard*) serían necesarios, así como los trade-offs que introduce
(ej. procesamiento fuera de orden).

- Respuesta: [`inciso-b.md`](inciso-b.md) · fragmento LaTeX: [`latex/inciso-b.tex`](latex/inciso-b.tex)
