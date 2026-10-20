---
description: Ejecuta una tarea con el equipo multi-modelo (Sonnet planifica, Opus diseña y desarrolla, Haiku ejecuta y sube a git)
argument-hint: [--eco | --max] <descripción de la tarea>
---

Actúa como **orquestador** del equipo multi-modelo para esta tarea:

$ARGUMENTS

Sigue la skill `orquestar` de este plugin:

0. **Clasifica la tarea** (trivial / sencilla / compleja) sin lanzar agentes. Si es trivial, **sáltate el plan** y delega directamente en `developer-lite` o `runner`.
   - `--eco`: prioriza `developer-lite` (Sonnet) salvo que la tarea sea claramente compleja.
   - `--max`: usa `developer` (Opus) para todo el desarrollo.
1. **Plan (Sonnet)** — solo si no es trivial: `planner` lee el código y devuelve el plan, qué se puede paralelizar y la complejidad de cada parte.
2. **Diseño (Opus)** — si hay UI, `designer`.
3. **Desarrollo** — `developer` (Opus) para lo complejo, `developer-lite` (Sonnet) para lo sencillo. Backend y frontend independientes van **en paralelo** (varias llamadas a Agent en la misma respuesta). Si `developer-lite` responde `ESCALAR`, repite con `developer`.
4. **Documentación (Sonnet)** — `planner`, en paralelo con la verificación.
5. **Git y comandos (Haiku)** — `runner` ejecuta tests/linters y, si todo pasa, hace commit y push.

Al final, resume qué hizo cada agente y con qué modelo.
