# Entrega en LaTeX

El PDF de la entrega, `SemArq2_TeamBales.pdf`, se genera desde esta carpeta. Cada
issue de inciso entrega **un fragmento `.tex`** con su solución, y `main.tex` los une.

## Cómo compilar

```sh
cd latex
latexmk -pdf -jobname=SemArq2_TeamBales main.tex
```

Sin `latexmk`, ejecuta `pdflatex -jobname=SemArq2_TeamBales main.tex` dos veces (la
segunda genera el índice). Los incisos que todavía no tienen fragmento aparecen como
*Pendiente* y el PDF compila igual.

## Dónde va cada fragmento

| Issue | Fichero |
|---|---|
| P1-a | `preguntas/p1-comunicacion-acoplamiento/latex/inciso-a.tex` |
| P1-b | `preguntas/p1-comunicacion-acoplamiento/latex/inciso-b.tex` |
| P2-a | `preguntas/p2-elasticidad-consistencia/latex/inciso-a.tex` |
| P2-b | `preguntas/p2-elasticidad-consistencia/latex/inciso-b.tex` |
| P3-a | `preguntas/p3-fallos-resiliencia-eda/latex/inciso-a.tex` |
| P3-b | `preguntas/p3-fallos-resiliencia-eda/latex/inciso-b.tex` |
| P4-a | `preguntas/p4-granularidad-microservicios/latex/inciso-a.tex` |
| P4-b | `preguntas/p4-granularidad-microservicios/latex/inciso-b.tex` |
| P5-a | `preguntas/p5-atributos-calidad-tradeoffs/latex/inciso-a.tex` |
| P5-b | `preguntas/p5-atributos-calidad-tradeoffs/latex/inciso-b.tex` |
| P6-a | `preguntas/p6-legado-soa/latex/inciso-a.tex` |
| P6-b | `preguntas/p6-legado-soa/latex/inciso-b.tex` |

## Reglas para los fragmentos

- **Sin preámbulo:** nada de `\documentclass`, `\usepackage` ni `\begin{document}`. Los
  paquetes van en [`preambulo.tex`](preambulo.tex); si falta uno, añádelo allí.
- **Sin título de inciso:** `main.tex` ya pone `\subsection{a) ...}`, etc. Dentro del
  fragmento usa `\subsubsection` y `\paragraph`.
- **Imágenes** con ruta desde la raíz del repo:
  ```latex
  \begin{figure}[H]
    \centering
    \includegraphics[width=\textwidth]{preguntas/p2-elasticidad-consistencia/diagramas/mi-diagrama.png}
    \caption{Diagrama de la solución.}
  \end{figure}
  ```
- **Citas bibliográficas:** cualquier idea, dato o cita de Richards & Ford u otra
  fuente debe llevar su referencia (autor-año o nota al pie, de forma consistente en
  todo el documento). Ver la bibliografía recomendada en el README principal.
- **Tablas** con `booktabs` (`\toprule`, `\midrule`, `\bottomrule`).
- El fragmento debe tener el **mismo contenido** que la respuesta en Markdown del
  inciso (p. ej. `inciso-b.md`), que se mantiene para leerla en GitHub.
- Antes de cerrar el issue, comprueba que `main.tex` compila sin errores con tu
  fragmento.
