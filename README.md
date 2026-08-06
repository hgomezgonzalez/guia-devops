# Guía de entrevistas técnicas

Guía visual de estudio para entrevistas de perfiles técnicos: **24 cursos** organizados en 6 niveles, con
chuleta de bolsillo, glosario, banco de preguntas y simulador cronometrado.

**Sitio:** https://hgomezgonzalez.github.io/guia-devops/
**Inicio:** https://hgomezgonzalez.github.io/

---

## Qué contiene

| Bloque | Qué es |
|---|---|
| **Dashboard** | Los cursos por niveles, con buscador que encuentra por tema y no solo por nombre |
| **Cursos** | Contenedores, Kubernetes, cloud (Azure y GCP), backend (Java, .NET, Python, Node), frontend, APIs, seguridad, arquitectura, DevOps y liderazgo |
| **Chuleta** | Pitch de 20 segundos y las frases listas por tema |
| **Glosario** | Cada término del CV explicado en una frase |
| **Entrenamiento** | Banco de preguntas con respuesta modelo |
| **Simulador** | Práctica cronometrada con rúbrica de evaluación |
| **Assessments** | System Design, pruebas de código, psicométricos y assessment center |

Todos los cursos siguen la misma estructura: analogía cotidiana → conceptos → código → el dato que
demuestra profundidad → **una frase en primera persona con métrica** → preguntas típicas.

## Cómo está hecha

Un solo `index.html` con HTML, CSS y JS en línea. Sin build, sin dependencias, sin `node_modules`: carga
de una, funciona offline y se despliega con `git push`. El caso de uso es abrirla en el móvil cinco
minutos antes de una entrevista.

## Mantenimiento

**→ [`RUNBOOK.md`](RUNBOOK.md)** — arquitectura del sistema, cómo añadir un curso (los 10 puntos de
registro), cómo se escribe el contenido, verificación y despliegue. Incluye los errores ya cometidos, que
es la parte que ahorra tiempo.
