---
name: orquestar
description: Reparte el trabajo entre los subagentes multi-modelo del plugin. Úsala cuando una tarea combine varias fases (leer código, planificar, diseñar, desarrollar frontend/backend, documentar, ejecutar comandos o subir a git) para delegar cada fase al modelo adecuado y ejecutar en paralelo lo que sea independiente.
---

# Orquestación multi-modelo

| Fase | Subagente | Modelo | Cuándo |
|---|---|---|---|
| Leer código, planificar, documentar | `planner` | Sonnet | Antes de tocar código y al final para docs |
| Diseño de la app/web (UX, UI, sistema de diseño) | `designer` | Opus | Si hay interfaz nueva o cambios visuales |
| Desarrollo frontend y backend | `developer` | Opus | Toda implementación y tests |
| Git y comandos básicos | `runner` | Haiku | Instalar, ejecutar tests/lint/build, commit, push |

## Flujo

```
planner (Sonnet) ──► designer (Opus) ──┬─► developer backend  (Opus) ─┐
                                       └─► developer frontend (Opus) ─┼─► runner: tests + commit + push (Haiku)
                                           planner: docs (Sonnet) ────┘
```

1. **Planificar.** Lanza `planner` con la petición completa. Pide que marque qué tareas son paralelizables.
2. **Diseñar.** Si hay UI, lanza `designer` con la sección de diseño del plan y el contexto del `planner`. Si no hay UI, sáltate este paso.
3. **Desarrollar en paralelo.** Define primero el contrato entre capas (endpoints, tipos, payloads). Después lanza en **una sola respuesta** varias llamadas a la herramienta Agent:
   - `developer` → backend (con el contrato y la parte backend del plan)
   - `developer` → frontend (con el contrato, el handoff del `designer` y la parte frontend del plan)
   - `planner` → documentación, si ya se conoce el alcance
4. **Verificar y subir.** Lanza `runner` para ejecutar tests, linters y build. Si algo falla, devuelve el error al `developer` (no lo arregla el `runner`). Cuando todo pase, `runner` hace commit y push con el mensaje propuesto.
5. **Resumen final.** Indica qué hizo cada agente, con qué modelo, archivos cambiados y estado de tests.

## Reglas
- Cada subagente arranca sin tu contexto: pásale en el prompt todo lo que necesite (plan, rutas, contrato, handoff).
- Lanza en paralelo solo lo que no dependa entre sí; si dos agentes editarían el mismo archivo, ejecútalos en serie.
- Tareas pequeñas de un solo tipo no necesitan el flujo completo: delega directamente al agente que corresponda (p. ej. "haz push" → solo `runner`).
- Nunca uses `developer` para comandos git ni `runner` para escribir código.
