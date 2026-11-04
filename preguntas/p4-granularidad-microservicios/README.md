# P4 — El Dilema de la Granularidad y el Acoplamiento en Microservicios

> Un equipo de desarrollo está migrando un sistema monolítico de gestión de recursos
> humanos a Microservicios. Inicialmente, crean servicios muy pequeños para cada
> entidad (Empleado, Departamento, Nómina, Beneficios). Pronto descubren que completar
> una operación simple, como contratar a un nuevo empleado, requiere una compleja
> coreografía de llamadas síncronas entre casi todos los servicios, lo que resulta en
> un alto acoplamiento y una fragilidad similar a la del monolito original.

## a) Error fundamental de diseño

Basado en los principios de *bounded context* y las guías de granularidad del
Capítulo 17, ¿cuál fue el error fundamental en el diseño de los servicios?

- Respuesta: [`inciso-a.md`](inciso-a.md) · fragmento LaTeX: [`latex/inciso-a.tex`](latex/inciso-a.tex)

## b) Nueva estrategia de descomposición

Proponga una nueva estrategia de descomposición. ¿Cómo aplicaría los conceptos de
propósito, transacciones y coreografía para redefinir los límites de los servicios y
reducir el acoplamiento? Proporcione un ejemplo de cómo agruparía los servicios para
la operación de "contratación".

- Respuesta: [`inciso-b.md`](inciso-b.md) · fragmento LaTeX: [`latex/inciso-b.tex`](latex/inciso-b.tex)
