# multi-model

Plugin de Claude Code que reparte el trabajo entre varios modelos y los ejecuta a la vez mediante subagentes:

| Agente | Modelo | Rol |
|---|---|---|
| `planner` | **Sonnet** | Leer código, planificar y documentar (solo edita documentación) |
| `designer` | **Opus** | Diseño de la aplicación/web: UX, UI, sistema de diseño, accesibilidad |
| `developer` | **Opus** | Desarrollo complejo de frontend y backend, tests |
| `developer-lite` | **Sonnet** | Modo económico: cambios sencillos y acotados; escala a `developer` si la tarea es compleja |
| `runner` | **Haiku** | Git (commit, push, ramas) y comandos básicos (instalar, tests, lint, build). No escribe código |

El modelo de cada agente se fija con el campo `model:` del frontmatter en `agents/*.md`. La sesión principal (con el modelo que tengas elegido) actúa como **orquestador**: clasifica la tarea, delega en los agentes y, cuando las tareas son independientes, los lanza en paralelo (por ejemplo, backend y frontend a la vez).

## Instalación

Desde Claude Code:

```
/plugin marketplace add ChemaSanchez11/claude
/plugin install multi-model@chemasanchez-plugins
```

O en local, clonando el repo:

```
/plugin marketplace add ./ruta/al/repo/claude
/plugin install multi-model@chemasanchez-plugins
```

## Comandos

| Comando | Qué hace |
|---|---|
| `/multi-model:equipo [--eco\|--max] <tarea>` | Flujo completo: clasifica → plan (Sonnet) → diseño (Opus) → backend + frontend en paralelo + docs (Sonnet) → tests, commit y push (Haiku) |
| `/multi-model:rapido <cambio>` | Modo económico para cambios pequeños: sin plan, `developer-lite` (Sonnet) |
| `/multi-model:planificar <tarea>` | Solo plan (Sonnet) |
| `/multi-model:documentar <qué>` | Documentación (Sonnet) |
| `/multi-model:disenar <qué>` | Diseño de la app/web (Opus) |
| `/multi-model:desarrollar [--eco\|--max] <qué>` | Implementación: Opus para lo complejo, Sonnet para lo sencillo |
| `/multi-model:subir [mensaje]` | Tests, commit y push (Haiku) |
| `/multi-model:ejecutar <comando>` | Comandos básicos (Haiku) |

Flags:
- `--eco`: usa `developer-lite` (Sonnet) siempre que la tarea no sea claramente compleja.
- `--max`: usa `developer` (Opus) para todo el desarrollo, aunque sea sencillo.

También incluye la skill `orquestar`, que Claude usa automáticamente cuando una petición mezcla varias fases, y puedes invocar cualquier agente directamente: *"usa el agente runner para hacer push"*.

## Cómo decide el orquestador

Antes de lanzar ningún agente, el orquestador clasifica la tarea:

| Tipo | Ejemplos | Qué hace |
|---|---|---|
| **Trivial** | Cambiar un texto o color, un typo, hacer push | **Se salta el `planner`** y delega directamente en `developer-lite` o `runner` |
| **Sencilla** | Bug localizado, añadir un campo o un endpoint CRUD, un test | `planner` solo si no sabe qué archivos tocar; desarrolla `developer-lite` (Sonnet) |
| **Compleja** | Funcionalidad nueva en varias capas, migraciones, API pública, autenticación/seguridad | Flujo completo con `planner` y `developer` (Opus) |

Si `developer-lite` descubre que la tarea es más complicada de lo que parecía, se detiene sin dejar cambios a medias y responde `ESCALAR: <motivo>`; entonces el orquestador la repite con `developer` (Opus).

```
                         ┌─ trivial ──► developer-lite (Sonnet) / runner (Haiku)
petición ─► clasificar ──┼─ sencilla ─► [planner] ─► developer-lite (Sonnet) ──(ESCALAR)──► developer (Opus)
                         └─ compleja ─► planner ─► designer ─► developer ×2 en paralelo (Opus) + docs (Sonnet)
                                                                      │
                                                                      ▼
                                                        runner: tests + commit + push (Haiku)
```

## Ejemplos

```
/multi-model:equipo Añade una página de registro de usuarios con API REST, validación y tests
```

1. El orquestador la clasifica como **compleja**.
2. `planner` (Sonnet) lee el repo y devuelve el plan.
3. `designer` (Opus) define el formulario, estados de error y tokens.
4. Dos `developer` (Opus) en paralelo: endpoint `/api/register` y la página del formulario. Mientras, `planner` (Sonnet) actualiza la documentación.
5. `runner` (Haiku) ejecuta tests y lint, hace commit y push.

```
/multi-model:rapido Cambia el texto del botón "Enviar" por "Crear cuenta" en el formulario de registro
```

→ Sin plan; `developer-lite` (Sonnet) hace el cambio directamente.

## ¿Ahorra tokens?

Reduce el **coste**, pero no necesariamente el **número de tokens**. Conviene entender por qué para sacarle partido.

### Dónde ahorra
- **Precio por token.** Sonnet es bastante más barato que Opus, y Haiku todavía más. En planes Pro/Max, Opus además consume el límite de uso más rápido.
- **Leer código es lo que más tokens gasta** (se cargan muchos archivos). Hacerlo con Sonnet en lugar de Opus es donde está el mayor ahorro.
- Planificar y documentar (Sonnet), comandos y git (Haiku) y los cambios sencillos (`developer-lite`, Sonnet) salen más baratos que si los hiciera Opus.

### Dónde no ahorra (o gasta más)
- **Cada subagente empieza de cero**: no ve tu conversación, así que hay que pasarle contexto y a veces vuelve a leer archivos que otro agente ya leyó. En total puede haber *más* tokens que una sola sesión haciéndolo todo.
- **El orquestador es tu sesión principal**: si la tienes en Opus, la coordinación (repartir, leer resultados, resumir) se cobra a precio de Opus.
- **El diseño y el desarrollo complejo siguen en Opus**, y suelen ser la parte más larga.
- **El paralelismo ahorra tiempo, no tokens**: dos `developer` a la vez gastan lo mismo (o algo más) que uno detrás de otro.

**En resumen:** en tareas grandes con mucha lectura de código suele salir más barato que hacerlo todo con Opus; en tareas pequeñas el flujo completo puede costar igual o más por el arranque de los subagentes. Por eso el orquestador se salta el plan en las tareas triviales y existe `/multi-model:rapido`.

### Consejos para gastar menos
1. **Pon la sesión principal en Sonnet** (`/model sonnet`). El plugin sigue usando Opus para diseñar y desarrollar lo complejo, pero la orquestación sale más barata.
2. **No uses `/multi-model:equipo` para cosas pequeñas.** Para un push, `/multi-model:subir`; para un cambio pequeño, `/multi-model:rapido`. El flujo completo compensa en tareas grandes.
3. **Usa `--eco`** cuando el coste importe más que la calidad máxima.
4. **Da rutas concretas.** Si le dices al orquestador qué archivos tocar, o el `planner` devuelve rutas `archivo:línea`, los desarrolladores van directos sin volver a explorar el repo con Opus.
5. **Divide tareas enormes** en varias peticiones: el contexto de cada agente se mantiene pequeño.

## Personalización

- Cambia el modelo de un agente editando su frontmatter: `model: sonnet | opus | haiku | inherit` o un ID completo de modelo.
- Las herramientas permitidas de cada agente están en el campo `tools:`.
- Los criterios para clasificar tareas (trivial/sencilla/compleja) y para que `developer-lite` escale están en `skills/orquestar/SKILL.md` y `agents/developer-lite.md`.

## Estructura

```
plugins/multi-model/
├── .claude-plugin/plugin.json
├── agents/
│   ├── planner.md          # Sonnet
│   ├── designer.md         # Opus
│   ├── developer.md        # Opus
│   ├── developer-lite.md   # Sonnet (modo económico)
│   └── runner.md           # Haiku
├── commands/               # equipo, rapido, planificar, documentar, disenar, desarrollar, subir, ejecutar
└── skills/orquestar/SKILL.md
```
