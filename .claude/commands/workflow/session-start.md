---
allowed-tools: Read(*), Bash(git *), Bash(gh *), LS(*)
description: Configura workspace leyendo CLAUDE.md, verificando estado git y mostrando opciones de desarrollo
---

# Session Start

Configura workspace automáticamente según especificaciones del proyecto.

## Ejecución

Cuando ejecutes este comando, sigue estos pasos:

### 1. Mensaje inicial amable
Mostrar exactamente este texto:
```
*Configurando workspace...*

```

### 2. Leer configuración del proyecto
- Ejecutar: `Read` en `/CLAUDE.md` para obtener configuraciones específicas

### 3. Verificar estado git
- Ejecutar: `git status --porcelain` para ver cambios pendientes
- Ejecutar: `git branch --show-current` para rama actual
- Ejecutar: `git log --oneline -3` para commits recientes
- Mostrar resumen: "📍 Branch: [rama] | Estado: [limpio/[X] cambios pendientes] | Último commit: [mensaje del commit más reciente]"

### 4. Verificar issues del usuario
- Ejecutar: `gh issue list --assignee @me --state open --limit 6` si gh está disponible
- Si comando falla, mostrar: "ℹ️ GitHub CLI no disponible"
- Si exitoso y hay issues:
  - Para issues 1-5: mostrar resumen en una sola linea de cada uno como "• #[número] [título]"
  - Si hay 6 o más: agregar línea "• Ver todos en: github.com/[owner]/[repo]/issues/assigned"
- Si exitoso y no hay issues: mostrar "✓ No tienes issues asignados"

### 5. Restricciones de commit activadas
- NEVER add "Co-authored-by" or Claude signatures
- NEVER include "Generated with Claude Code"
- NEVER modify git config or credentials  
- NEVER use emojis in commits or PRs

### 6. Mostrar estándar de worktrees
Mostrar exactamente este texto:

```
*Para desarrollo (features, bugs, refactoring, etc), debemos usar worktrees:* https://git-scm.com/docs/git-worktree

1. Crear worktree: /worktree:create <purpose> <parent-branch>
2. Cambiar directorio: cd ../worktree-<purpose>
3. Nueva sesión Claude: claude /workflow:session-start

Comandos típicos:
/worktree:create feature-auth develop     # Feature desde develop
/worktree:create fix-payment-bug main     # Hotfix desde main  
/worktree:cleanup worktree-feature-auth   # Limpiar al terminar
```

### 7. Pregunta final
Mostrar exactamente este texto:

```
¿Cuál es tu objetivo para esta sesión?

- Desarrollo → ¡Crea worktree primero! ⬆️
- Análisis/Docs → Continua aquí
```

**IMPORTANTE**:
- Ejecutar pasos 1-7 secuencialmente sin confirmación del usuario
- Si algún paso falla, continuar con warning pero no terminar
- Mantener output conciso y actionable