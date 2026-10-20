---
description: Revisa, hace commit y push de los cambios (Haiku)
argument-hint: [mensaje de commit opcional]
---

Lanza el subagente `runner` (Haiku) para:

1. Mostrar `git status` y `git diff --stat`.
2. Ejecutar los tests/linters del proyecto si existen.
3. Si todo pasa, hacer `git add` de los cambios relevantes (sin secretos ni artefactos), commit y `git push -u origin <rama actual>`.

Mensaje de commit: $ARGUMENTS
(Si está vacío, el runner propone uno a partir del diff.)
