---
name: orquestar
description: Reparte el trabajo entre los subagentes multi-modelo del plugin. Úsala cuando una tarea combine varias fases (leer código, planificar, diseñar, desarrollar frontend/backend, documentar, ejecutar comandos o subir a git) para delegar cada fase al modelo adecuado, gastar lo mínimo y ejecutar en paralelo lo que sea independiente.
---

# Orquestación multi-modelo

| Fase | Subagente | Modelo | Cuándo |
|---|---|---|---|
| Leer código, planificar, documentar | `planner` | Sonnet | Tareas no triviales, antes de tocar código, y al final para docs |
| Diseño de la app/web (UX, UI, sistema de diseño) | `designer` | Opus | Si hay interfaz nueva o cambios visuales importantes |
| Desarrollo complejo frontend/backend | `developer` | Opus | Lógica nueva, varias capas, arquitectura, seguridad |
| Desarrollo sencillo (modo económico) | `developer-lite` | Sonnet | Cambios acotados y de bajo riesgo |
| Git y comandos básicos | `runner` | Haiku | Instalar, ejecutar tests/lint/build, commit, push |

## Paso 0: clasificar la tarea (hazlo tú, sin lanzar agentes)

Lee la petición y, como mucho, echa un vistazo rápido a los archivos que nombre. Clasifícala:

- **Trivial** — una sola acción mecánica o un cambio de 1–2 archivos que ya sabes dónde está (cambiar un texto, un color, arreglar un typo, añadir un campo obvio, hacer push).
  → **Sáltate el `planner`.** Delega directamente: `developer-lite` para código, `runner` para comandos/git.
- **Sencilla** — cambio acotado en pocos archivos, sin decisiones de arquitectura.
  → `planner` solo si no sabes qué archivos tocar; desarrollo con `developer-lite`.
- **Compleja** — funcionalidad nueva, varias capas, modelos de datos/migraciones, API pública, seguridad, o no está claro el alcance.
  → Flujo completo con `planner` y `developer` (Opus).

Ante la duda entre sencilla y compleja, empieza con `developer-lite`: si responde `ESCALAR: <motivo>`, repite la tarea con `developer` pasándole el motivo y lo que ya se sabe.

Si el usuario pide explícitamente modo económico (`--eco` o "barato"), usa `developer-lite` siempre que no sea claramente compleja. Si pide calidad máxima (`--max`), usa `developer` aunque sea sencilla.

## Flujo completo (tareas complejas)

```
planner (Sonnet) ──► designer (Opus) ──┬─► developer backend  (Opus) ─┐
                                       └─► developer frontend (Opus) ─┼─► runner: tests + commit + push (Haiku)
                                           planner: docs (Sonnet) ────┘
```

1. **Planificar.** Lanza `planner` con la petición completa. Pide que marque qué tareas son paralelizables y que clasifique cada una como sencilla o compleja.
2. **Diseñar.** Si hay UI, lanza `designer` con la sección de diseño del plan. Si no hay UI, sáltate este paso.
3. **Desarrollar en paralelo.** Define primero el contrato entre capas (endpoints, tipos, payloads). Después lanza en **una sola respuesta** varias llamadas a la herramienta Agent:
   - `developer` (o `developer-lite` si esa parte es sencilla) → backend
   - `developer` (o `developer-lite`) → frontend, con el handoff del `designer`
   - `planner` → documentación, si ya se conoce el alcance
4. **Verificar y subir.** Lanza `runner` para ejecutar tests, linters y build. Si algo falla, devuelve el error al desarrollador (no lo arregla el `runner`). Cuando todo pase, `runner` hace commit y push con el mensaje propuesto.
5. **Resumen final.** Indica qué hizo cada agente, con qué modelo, archivos cambiados y estado de tests.

## Reglas para ahorrar tokens
- Cada subagente arranca sin tu contexto: pásale en el prompt **todo lo que necesite** (plan, rutas exactas `archivo:línea`, contrato, handoff) para que no vuelva a explorar el repo.
- No le pases archivos enteros en el prompt si basta con la ruta; tampoco repitas todo el plan a cada agente, solo su parte.
- Pide respuestas concisas a los subagentes; tú resumes al usuario.
- Lanza en paralelo solo lo que no dependa entre sí; si dos agentes editarían el mismo archivo, ejecútalos en serie.
- Nunca uses `developer`/`developer-lite` para comandos git ni `runner` para escribir código.
