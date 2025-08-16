# Guía Práctica de Comandos Claude Code

## 🚀 Comandos Básicos

### 💾 `/commit` - Commits inteligentes
```bash
/commit "descripción del cambio"
```
**Qué hace**: Analiza tus cambios y crea commits semánticos con validaciones automáticas.
**Cuándo usarlo**: Después de completar cualquier cambio en código.
**Ejemplo**: `commit "fix user login validation"` → genera commit con mensaje optimizado.

### ✅ `/test` - Ejecutar tests con auto-fix
```bash
/test
```
**Qué hace**: Ejecuta todos los tests, detecta fallos y sugiere fixes automáticos.
**Cuándo usarlo**: Antes de commits importantes o después de cambios significativos.

### `/format` - Formateo consistente
```bash
/format
```
**Qué hace**: Formatea todo el código siguiendo las convenciones del proyecto.
**Cuándo usarlo**: Antes de commits o cuando el código se ve inconsistente.

---

## 🔧 Comandos de Desarrollo

### ⚙️ `/implement` - Motor de implementación
```bash
/implement "nueva feature de dashboard con notificaciones"
```
**Qué hace**: Planifica e implementa features completas paso a paso.
**Cuándo usarlo**: Para implementar nuevas funcionalidades desde cero.
**Flujo**: Analiza → Planifica → Implementa → Valida → Documenta

### `/scaffold` - Generar estructuras
```bash
/scaffold "API para gestión de usuarios"
```
**Qué hace**: Genera estructura completa de archivos siguiendo patrones del proyecto.
**Cuándo usarlo**: Al empezar nuevos módulos, APIs o componentes.

### 🔄 `/refactor` - Refactoring inteligente
```bash
/refactor "mejorar performance de consultas de base de datos"
```
**Qué hace**: Reestructura código manteniendo funcionalidad, mejorando calidad.
**Cuándo usarlo**: Cuando el código funciona pero necesita mejoras estructurales.

### `/fix-imports` - Reparar imports rotos
```bash
/fix-imports
```
**Qué hace**: Encuentra y repara sistemáticamente imports rotos por file moves o renames.
**Cuándo usarlo**: Después de refactoring, reestructuración de carpetas o cuando hay errores de imports.

### `/make-it-pretty` - Mejora de legibilidad
```bash
/make-it-pretty
```
**Qué hace**: Mejora legibilidad del código preservando funcionalidad exacta (naming, organización).
**Cuándo usarlo**: Cuando el código funciona pero es difícil de leer o mantener.

### `/remove-comments` - Eliminar comentarios obvios
```bash
/remove-comments
```
**Qué hace**: Limpia comentarios redundantes preservando los que añaden valor real.
**Cuándo usarlo**: Para limpiar código con exceso de comentarios obvios o redundantes.

---

## 🔍 Comandos de Análisis

### 🧪 `/understand` - Comprensión profunda
```bash
/understand
```
**Qué hace**: Analiza todo el codebase y explica arquitectura, patrones y funcionamiento.
**Cuándo usarlo**: Al empezar en un proyecto nuevo o cuando necesitas entender código complejo.

### 🔎 `/review` - Revisión de código
```bash
/review
```
**Qué hace**: Revisa cambios recientes y sugiere mejoras de calidad, seguridad y performance.
**Cuándo usarlo**: Después de implementar features importantes o antes de hacer merge.

### 🔒 `/security-scan` - Auditoría de seguridad
```bash
/security-scan
```
**Qué hace**: Escanea vulnerabilidades, credenciales expuestas y problemas de seguridad.
**Cuándo usarlo**: Antes de deployments o periódicamente en código crítico.

### 🧠 `/deep` - Razonamiento profundo
```bash
/deep "problema complejo o decisión arquitectónica"
```
**Qué hace**: Activa capacidades máximas de análisis para problemas complejos y decisiones estratégicas.
**Cuándo usarlo**: Planificación estratégica, decisiones arquitectónicas críticas, auditorías profundas.
**Flujo**: Análisis multi-perspectiva → Investigación de causa raíz → Pensamiento sistémico → Soluciones alternativas

### `/e-team` - Análisis de equipo experto
```bash
/e-team "challenge técnico o arquitectónico"
```
**Qué hace**: Análisis estratégico con equipos de expertos virtuales especializados.
**Cuándo usarlo**: Evaluación de challenges complejos, validación de enfoques técnicos.
**Flujo**: Claude Code Strategist → Security Architect → Technical Architect → Strategic Director

### `/explain-like-senior` - Explicación nivel senior
```bash
/explain-like-senior
```
**Qué hace**: Explica código como desarrollador senior, enfocándose en el por qué detrás de las decisiones.
**Cuándo usarlo**: Para mentorización, entendimiento profundo de arquitectura y patrones de código.

---

## 📄 Comandos de Documentación

### 📚 `/docs` - Gestión de documentación
```bash
/docs
```
**Qué hace**: Analiza y actualiza TODA la documentación del proyecto automáticamente (README, CHANGELOG, docs/*).
**Cuándo usarlo**: Después de features, cambios importantes, para mantener documentación actualizada.
**Flujo**: Analiza conversación → Lee documentación existente → Identifica cambios → Actualiza sistemáticamente

---

## 🗂️ Comandos de Workflow

### 🔄 `/workflow:switch` - Cambio seguro de rama validando PR
```bash
/workflow:switch <target_branch>
```
**Qué hace**: Valida PR mergeado, cambia a rama objetivo y limpia workspace temporal.
**Cuándo usarlo**: Al finalizar PRs mergeados para cambiar a main/develop con cleanup completo.
**Flujo**: Bloquea si PR no mergeado → Switch seguro → Limpieza

### 📝 `/workflow:changelog` - Actualización inteligente de changelog
```bash
/workflow:changelog --pr <number>       # Single PR
/workflow:changelog --prs <n1,n2,n3>   # Multiple PRs
```
**Qué hace**: Actualiza CHANGELOG.md con PRs mergeados, detecta duplicados automáticamente.
**Cuándo usarlo**: Después de merge para documentar cambios en proyecto.
**Flujo**: Valida PRs mergeados → Detecta duplicados → Actualización atómica con backup

### `/workflow:session-start` - Iniciar sesión documentada
```bash
/workflow:session-start
```
**Qué hace**: Inicia sesión de código documentada con objetivos claros y tracking.
**Cuándo usarlo**: Al comenzar trabajo significativo para mantener continuidad.


---

## 📋 Gestión de TODOs

### `/todos:create` - Crear TODOs contextuales
```bash
/todos:create
```
**Qué hace**: Convierte hallazgos de análisis en TODOs específicos en el código.
**Cuándo usarlo**: Después de security-scan, review o cuando encuentras issues que no puedes arreglar inmediatamente.

### `/todos:find` - Buscar TODOs existentes
```bash
/todos:find
```
**Qué hace**: Escanea y categoriza todos los TODOs/FIXMEs del proyecto.
**Cuándo usarlo**: Para auditoría de deuda técnica o planificación de sprints.

### `/todos:fix` - Resolver TODOs
```bash
/todos:fix
```
**Qué hace**: Encuentra y resuelve TODOs existentes de forma sistemática.
**Cuándo usarlo**: Cuando quieres limpiar deuda técnica acumulada.

### `/todos:to-issues` - TODOs a issues GitHub
```bash
/todos:to-issues
```
**Qué hace**: Escanea TODOs en código y crea issues profesionales en GitHub automáticamente.
**Cuándo usarlo**: Para convertir deuda técnica en trabajo trackeable y organizado.

---

## 🧹 Comandos de Mantenimiento

### 🧽 `/mantenimiento:cleanproject` - Limpieza integral
```bash
/mantenimiento:cleanproject
```
**Qué hace**: Limpia dead code, optimiza imports, remueve archivos innecesarios.
**Cuándo usarlo**: Antes de releases o periódicamente para mantener el proyecto limpio.

---

## 🔗 Comandos GitHub

### `/github:to-contributing` - Estrategia completa de contribución
```bash
/github:to-contributing
```
**Qué hace**: Análisis contextual completo para contribuciones - detecta tu trabajo, escanea issues remotos, crea PRs profesionales.
**Cuándo usarlo**: Para contribuciones a cualquier repo con máxima probabilidad de aceptación.
**Flujo**: Análisis contexto → Pre-flight checks → Escaneo remoto → Linking issues → PR optimizado

### 🔄 `/pr` - Crear pull requests
```bash
/pr [target-branch]
```
**Qué hace**: Crea PRs con validación automática de branch, push seguro y metadata completa.
**Cuándo usarlo**: Para crear PRs que faciliten review y mantengan estándares de calidad.
**Flujo**: Valida target branch → Pre-fetch remoto → Crea branch temporal → Push seguro → PR con metadata


### 🎯 `/github:findings-to-issues` - Hallazgos a issues GitHub
```bash
/github:findings-to-issues
```
**Qué hace**: Convierte hallazgos de PR reviews en issues GitHub trackeable con categorización inteligente.
**Cuándo usarlo**: Después de reviews importantes para gestionar deuda técnica y seguimiento.

### `/github:issues-to-solved` - Resolver issues de PR automáticamente
```bash
/github:issues-to-solved <pr_number>
```
**Qué hace**: Extrae issues asociados a PR, analiza prioridades, genera plan de implementación y ejecuta fixes seguros automáticamente.
**Cuándo usarlo**: Después de crear issues con github:findings-to-issues, para planificar y resolver sistemáticamente.
**Flujo**: Extrae issues → Prioriza (CRITICAL/HIGH/MEDIUM/LOW) → Plan estructurado → Ejecución opcional

---

## 🌳 Comandos Worktree

### `/worktree:create` - Crear worktree aislado
```bash
/worktree:create <purpose> <parent-branch>
```
**Qué hace**: Crea worktree en directorio sibling con rama nueva y upstream remoto.
**Cuándo usarlo**: SIEMPRE para desarrollo (features, bugs, refactoring).
**Flujo**: Valida argumentos → Verifica parent branch → Crea worktree → Configura upstream → Guía para cambio de directorio

### `/worktree:cleanup` - Limpiar worktrees
```bash
/worktree:cleanup <worktree1> [worktree2] [...]
```
**Qué hace**: Elimina worktrees específicos con validación de ownership y estado limpio.
**Cuándo usarlo**: Después de mergear PRs o cuando worktrees ya no se necesiten.
**Flujo**: Valida ownership → Verifica estado limpio → Confirmación → Triple cleanup (worktree/local/remote)

**Estándar del equipo:**
- Todo desarrollo se hace en worktrees aislados
- Mantiene workspace principal siempre limpio  
- Permite sesiones Claude Code paralelas
- Rollback instantáneo sin conflictos

---

## 🎯 Flujos Típicos

### Desarrollo de Feature Nueva
```bash
1. /worktree:create feature-name develop  # Crear worktree aislado
2. cd ../worktree-feature-name            # Cambiar al worktree
3. claude /workflow:session-start         # Nueva sesión en worktree
4. /understand                            # Entender contexto
5. /implement "nueva feature"             # Implementar
6. /test                                 # Validar funcionamiento  
7. /review                               # Revisar calidad
8. /security-scan                        # Verificar seguridad
9-12. Seguir workflow AI-First           # Ver: ai-first-workflow.md
```

> 📚 **Para workflow completo de PR + findings + issues:** Ver `ai-first-workflow.md`

### Bug Fix Urgente
```bash
1. /worktree:create fix-bug-name main     # Worktree desde main
2. cd ../worktree-fix-bug-name            # Cambiar al worktree
3. claude /workflow:session-start         # Nueva sesión en worktree
4. /understand                            # Entender el problema
5. Arreglar el código
6. /test                                 # Validar fix
7. /commit "fix: descripción"            # Commit inmediato
```

### Limpieza de Código
```bash
1. /todos:find                  # Ver deuda técnica
2. /todos:fix                   # Resolver pendientes
3. /mantenimiento:cleanproject  # Limpiar proyecto
4. /desarrollo:make-it-pretty   # Mejorar legibilidad
5. /format                     # Formatear todo
6. /commit "chore: cleanup"    # Documentar limpieza
```

### Análisis Estratégico Completo
```bash
1. /analisis:deep "problema arquitectónico"  # Razonamiento profundo
2. /analisis:e-team "challenge complejo"     # Análisis multi-experto
3. /understand                      # Mapear codebase
4. /review                         # Revisar estado actual
5. Implementar solución
6. /documentacion:docs             # Documentar decisiones
```

---

## 💡 Tips de Uso

- **Combina comandos**: Usa flujos secuenciales para máximo valor
- **Iterativo**: Los comandos recuerdan contexto entre ejecuciones
- **Seguridad primero**: Siempre usa /analisis:security-scan antes de production
- **Test frecuente**: Ejecuta /test después de cambios significativos
- **Documenta cambios**: Usa /documentacion:docs para mantener documentación actualizada
- **Análisis profundo**: Usa /analisis:deep para decisiones arquitectónicas críticas
- **Gestión de deuda**: Convierte TODOs en issues con /todos:to-issues