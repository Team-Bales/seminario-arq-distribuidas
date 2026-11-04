# seminario-arq-distribuidas

Seminario sobre Arquitectura 2: *Arquitecturas de Software Distribuidas*
(Service-Based, Event-Driven, Space-Based, SOA y Microservicios) — Ingeniería de
Software, MATCOM, 3er año (2026-2027).

**Equipo:** 7. Bales
**Organización:** [Team Bales](https://github.com/Team-Bales)

> **Nota sobre la orientación:** la portada del PDF de orientación (`orientacion/ArqDistribuidas-1.pdf`,
> secc. 3) copia literalmente el nombre de la actividad y la convención de fichero del
> seminario anterior (*"...Basados en Capas y Dominio"*, `SemArq1_NombreEquipo`). El
> aviso de la tarea en Telegram lo llama **"Seminario sobre Arquitectura 2"**, así que
> aquí se usa el título real del documento (*Arquitecturas de Software Distribuidas*)
> y el fichero final se llama `SemArq2_TeamBales.pdf`. Si el profesor exige el texto
> literal de la orientación, es un simple cambio de cadena en `latex/main.tex`.

## Estructura del repositorio

```
seminario-arq-distribuidas/
├── README.md                        ← estás aquí
├── orientacion/
│   └── ArqDistribuidas-1.pdf        ← documento de orientación del seminario
├── preguntas/
│   ├── p1-comunicacion-acoplamiento/
│   │   ├── README.md                ← enunciado completo
│   │   ├── inciso-a.md / inciso-b.md
│   │   └── latex/inciso-a.tex / inciso-b.tex
│   ├── p2-elasticidad-consistencia/
│   ├── p3-fallos-resiliencia-eda/
│   ├── p4-granularidad-microservicios/
│   ├── p5-atributos-calidad-tradeoffs/
│   └── p6-legado-soa/
└── latex/
    ├── main.tex                     ← ensambla el PDF final
    ├── preambulo.tex
    └── README.md                    ← cómo compilar y reglas de los fragmentos
```

## Preguntas del seminario

| ID | Tema | Carpeta |
|----|------|---------|
| P1 | Comunicación y acoplamiento (subastas, coreografía vs. orquestación) | [`p1-comunicacion-acoplamiento/`](preguntas/p1-comunicacion-acoplamiento/) |
| P2 | Elasticidad extrema y consistencia (venta de entradas, Space-Based) | [`p2-elasticidad-consistencia/`](preguntas/p2-elasticidad-consistencia/) |
| P3 | Fallos y resiliencia en EDA (Broker, Workflow Event Pattern) | [`p3-fallos-resiliencia-eda/`](preguntas/p3-fallos-resiliencia-eda/) |
| P4 | Granularidad y acoplamiento en Microservicios (RRHH) | [`p4-granularidad-microservicios/`](preguntas/p4-granularidad-microservicios/) |
| P5 | Atributos de calidad y trade-offs (SBA vs. Microservices, CTO) | [`p5-atributos-calidad-tradeoffs/`](preguntas/p5-atributos-calidad-tradeoffs/) |
| P6 | Legado de SOA y reutilización en Microservices | [`p6-legado-soa/`](preguntas/p6-legado-soa/) |

Cada pregunta tiene dos incisos (a, b); cada inciso es un issue propio con etiqueta
`pN` y se entrega como un fragmento LaTeX (ver [`latex/README.md`](latex/README.md)).

## Entrega

- **Fecha límite:** 13/11/2026
- **Fichero:** `SemArq2_TeamBales.pdf`
- **Formato:** LaTeX. Cada issue de inciso entrega su solución como un fragmento
  `.tex`, y el PDF se genera compilando [`latex/main.tex`](latex/main.tex).
- **Dónde subirlo:** en el tópico *Evaluaciones* del grupo de IS en Telegram **y** en
  este repositorio.
- **Portada (primera página) con:**
  - nombre de la actividad y equipo;
  - nombres completos de todos los integrantes;
  - indicación explícita de en qué pregunta(s) trabajó cada integrante;
  - la URL de este repositorio: https://github.com/Team-Bales/seminario-arq-distribuidas
- **Citas:** toda idea, dato o cifra que no sea de elaboración propia necesita cita
  autor-año (o notas al pie, de forma consistente), con la lista de referencias final
  coincidiendo en ambos sentidos con las citas del cuerpo.
- **Declaración de uso de IA:** obligatoria si el equipo usó alguna herramienta de IA,
  como epígrafe propio después de la bibliografía (ver plantilla en la orientación,
  secc. 3).

## Bibliografía recomendada

- Richards, M., & Ford, N. (2020). *Fundamentals of Software Architecture: An
  Engineering Approach*. O'Reilly Media. — Caps. 9, 18 (intro y comparación
  monolito/distribuido); 13 (SBA); 14 (EDA); 15 (Space-Based); 16 (SOA); 17
  (Microservicios); 3, 7, 8 (acoplamiento, connascencia, arquitectura quantum).
- Ford, N., Richards, M., Sadalage, P., & Dehghani, Z. (2021). *Software
  Architecture: The Hard Parts*. O'Reilly Media. — Caps. 4, 6, 7, 9 (datos
  distribuidos, consistencia eventual, Saga, CQRS, Event Sourcing).
- Ford, N., Parsons, R., & Kua, P. (2017). *Building Evolutionary Architectures:
  Automated Software Governance*. O'Reilly Media. — Caps. 2, 3, 5 (fitness
  functions, evolución guiada, gobierno arquitectónico).

## Reparto del trabajo

El trabajo está dividido en issues, uno por inciso (12) más uno de formalización y
entrega del PDF final, con la etiqueta `p1`…`p6` o `entrega`. Las issues ya están
asignadas (ver tabla de Miembros); la columna *Pregunta(s)* de esa tabla y la portada
del PDF se actualizan a medida que se van cerrando las issues de cada pregunta.
Seguimiento en el tablero del proyecto: [Seminario Arq. Distribuidas](https://github.com/orgs/Team-Bales/projects/4).

## Miembros

Esta tabla es la base de la portada del PDF. Cada uno completa su nombre y las
preguntas en que trabajó.

| Nombre | GitHub | Pregunta(s) |
|---|---|---|
| Heily Rodríguez Rodríguez | @heilyrodriguez225 | P1, P4 |
| | @Mackandal04 | P5 |
| | @QuiquiMatCom2004 | P3, P6 |
| | @solerch5510 | P2 |
