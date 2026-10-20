# multi-model

Plugin de Claude Code que reparte el trabajo entre varios modelos y los ejecuta a la vez mediante subagentes:

| Agente | Modelo | Rol |
|---|---|---|
| `planner` | **Sonnet** | Leer código, planificar y documentar |
| `designer` | **Opus** | Diseño de la aplicación/web (UX, UI, sistema de diseño, accesibilidad) |
| `developer` | **Opus** | Desarrollo frontend y backend, tests |
| `runner` | **Haiku** | Git (commit, push, ramas) y comandos básicos (instalar, tests, lint, build) |

El modelo de cada agente se fija con el campo `model:` del frontmatter en `agents/*.md`. La sesión principal (con el modelo que tengas elegido) actúa como orquestador y delega en ellos; cuando las tareas son independientes los lanza en paralelo (por ejemplo, backend y frontend a la vez con dos instancias de `developer`).

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
| `/multi-model:equipo <tarea>` | Flujo completo: plan (Sonnet) → diseño (Opus) → backend + frontend en paralelo (Opus) + docs (Sonnet) → tests, commit y push (Haiku) |
| `/multi-model:planificar <tarea>` | Solo plan (Sonnet) |
| `/multi-model:documentar <qué>` | Documentación (Sonnet) |
| `/multi-model:disenar <qué>` | Diseño de la app/web (Opus) |
| `/multi-model:desarrollar <qué>` | Implementación frontend/backend (Opus) |
| `/multi-model:subir [mensaje]` | Tests, commit y push (Haiku) |
| `/multi-model:ejecutar <comando>` | Comandos básicos (Haiku) |

También incluye la skill `orquestar`, que Claude usa automáticamente cuando una petición mezcla varias fases, y puedes invocar cualquier agente directamente: *"usa el agente runner para hacer push"*.

## Ejemplo

```
/multi-model:equipo Añade una página de registro de usuarios con API REST, validación y tests
```

1. `planner` (Sonnet) lee el repo y devuelve el plan.
2. `designer` (Opus) define el formulario, estados de error y tokens.
3. Dos `developer` (Opus) en paralelo: endpoint `/api/register` y la página del formulario. Mientras, `planner` (Sonnet) actualiza la documentación.
4. `runner` (Haiku) ejecuta tests y lint, hace commit y push.

## Personalización

Cambia el modelo de un agente editando su frontmatter (`model: sonnet | opus | haiku | inherit`, o un ID completo de modelo). Las herramientas permitidas de cada agente están en el campo `tools:`.
