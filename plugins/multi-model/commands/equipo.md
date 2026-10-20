---
description: Ejecuta una tarea con el equipo multi-modelo (Sonnet planifica, Opus diseña y desarrolla, Haiku ejecuta y sube a git)
argument-hint: <descripción de la tarea>
---

Actúa como **orquestador** del equipo multi-modelo para esta tarea:

$ARGUMENTS

Sigue la skill `orquestar` de este plugin. Resumen del flujo:

1. **Plan (Sonnet)** — lanza el subagente `planner` para leer el código y devolver un plan con las tareas repartidas por agente y qué se puede paralelizar.
2. **Diseño (Opus)** — si la tarea tiene UI, lanza el subagente `designer` con la parte de diseño del plan.
3. **Desarrollo (Opus)** — lanza el subagente `developer`. Si backend y frontend son independientes, lanza **dos instancias de `developer` en paralelo** (en el mismo mensaje, varias llamadas a Agent): una para backend y otra para frontend.
4. **Documentación (Sonnet)** — en paralelo con la verificación, lanza `planner` para actualizar la documentación.
5. **Git y comandos (Haiku)** — lanza `runner` para ejecutar tests/linters y, si todo pasa, hacer commit y push con el mensaje propuesto por el `developer`.

Siempre que dos pasos no dependan entre sí, lánzalos **en la misma respuesta** para que se ejecuten a la vez. Al final, resume qué hizo cada agente y con qué modelo.
