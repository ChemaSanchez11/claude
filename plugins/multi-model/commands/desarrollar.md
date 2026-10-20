---
description: Implementa frontend y/o backend (Opus para lo complejo, Sonnet para lo sencillo; en paralelo si son independientes)
argument-hint: [--eco | --max] <qué implementar>
---

Implementa lo siguiente:

$ARGUMENTS

- Elige el agente según la complejidad (ver skill `orquestar`): `developer-lite` (Sonnet) para cambios sencillos y acotados, `developer` (Opus) para lo complejo. `--eco` fuerza `developer-lite` salvo que sea claramente complejo; `--max` fuerza `developer`.
- Si `developer-lite` responde `ESCALAR: <motivo>`, repite la tarea con `developer` pasándole el motivo.
- Si hay una parte de backend y otra de frontend que no dependen entre sí, lanza **dos agentes en paralelo** en la misma respuesta (uno por capa) y acuerda antes el contrato de la API.
- No hagas commit: al terminar, muéstrame los cambios y el mensaje de commit propuesto.
