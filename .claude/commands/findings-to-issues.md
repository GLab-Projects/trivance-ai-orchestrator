# Findings to Issues

Extraigo hallazgos de revisiones de PRs y creo GitHub issues automáticamente con categorización inteligente.

**Argumentos:** `[PR_NUMBER]` (opcional - usa PR del branch actual si no se especifica)

## Ejecución Directa

```bash
#!/bin/bash

# Configuración de constantes de seguridad
MAX_FINDINGS=50
MAX_COMMENTS=1000

# Configuración de colores para output
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
BLUE='\033[0;34m'
NC='\033[0m' # No Color

# Función para logging JSONL seguro
log_event() {
    local event_type="$1"
    local data="$2"
    local timestamp=$(date -u '+%Y-%m-%dT%H:%M:%SZ')
    local today=$(date '+%Y-%m-%d')
    
    # Validar formato de fecha para prevenir directory traversal
    if ! [[ "$today" =~ ^[0-9]{4}-[0-9]{2}-[0-9]{2}$ ]]; then
        echo "Error: Invalid date format" >&2
        return 1
    fi
    
    local logs_dir=".claude/logs/$today"
    mkdir -p "$logs_dir"
    
    # Usar jq para construir JSON seguro
    jq -nc --arg ts "$timestamp" --arg event "$event_type" --argjson data "$data" \
        '{timestamp: $ts, event: $event} + $data' >> "$logs_dir/findings-to-issues.jsonl"
}

# Validaciones iniciales
echo "🔍 Validando entorno..."

# Verificar gh CLI
if ! command -v gh &> /dev/null; then
    echo -e "${RED}❌ GitHub CLI (gh) no encontrado${NC}"
    echo "   Instalar desde: https://cli.github.com"
    log_event "error" '{"message": "gh CLI not found"}'
    exit 1
fi

# Verificar autenticación GitHub
if ! gh auth status &>/dev/null; then
    echo -e "${RED}❌ No autenticado con GitHub${NC}"
    echo "   Ejecutar: gh auth login"
    log_event "error" '{"message": "GitHub auth failed"}'
    exit 1
fi

# Verificar repositorio GitHub
if ! git remote -v | grep -q github.com; then
    echo -e "${RED}❌ No es un repositorio GitHub${NC}"
    log_event "error" '{"message": "Not a GitHub repository"}'
    exit 1
fi

echo -e "${GREEN}✅ Entorno validado${NC}"

# Obtener información del repositorio
repo_info=$(gh repo view --json nameWithOwner --jq '.nameWithOwner')
current_branch=$(git branch --show-current)

# Detectar PR con validación de seguridad
if [ -n "$1" ]; then
    # Validar que PR_NUMBER sea estrictamente numérico
    if ! [[ "$1" =~ ^[0-9]+$ ]]; then
        echo -e "${RED}❌ PR number debe ser un número entero positivo${NC}"
        log_event "error" $(jq -nc --arg input "$1" '{message: "Invalid PR number format", input: $input}')
        exit 1
    fi
    
    pr_number="$1"
    echo "📋 Usando PR especificado: #$pr_number"
else
    echo "🔍 Detectando PR del branch actual: $current_branch"
    pr_number=$(gh pr list --head "$current_branch" --json number --jq '.[0].number' 2>/dev/null)
    
    if [ -z "$pr_number" ]; then
        echo -e "${YELLOW}⚠️  No se encontró PR para el branch '$current_branch'${NC}"
        echo ""
        echo "PRs disponibles:"
        gh pr list --limit 5
        echo ""
        echo "Uso: /findings-to-issues [PR_NUMBER]"
        log_event "error" $(jq -nc --arg branch "$current_branch" '{message: "No PR found for branch", branch: $branch}')
        exit 1
    fi
    echo -e "${GREEN}✅ PR detectado: #$pr_number${NC}"
fi

# Validar formato de repo_info para prevenir API injection
if ! [[ "$repo_info" =~ ^[a-zA-Z0-9_.-]+/[a-zA-Z0-9_.-]+$ ]]; then
    echo -e "${RED}❌ Formato de repositorio inválido${NC}"
    log_event "error" '{"message": "Invalid repository format"}'
    exit 1
fi

# Log inicio de procesamiento con JSON seguro
log_event "start" $(jq -nc --arg pr "$pr_number" --arg branch "$current_branch" --arg repo "$repo_info" \
    '{pr_number: ($pr | tonumber), branch: $branch, repo: $repo}')

# Obtener información del PR con validación JSON
echo ""
echo "📄 Analizando PR #$pr_number..."
pr_info=$(gh pr view "$pr_number" --json title,author,state,url 2>/dev/null)

if [ -z "$pr_info" ]; then
    echo -e "${RED}❌ No se pudo obtener información del PR #$pr_number${NC}"
    log_event "error" $(jq -nc --arg pr "$pr_number" '{message: "PR not found", pr_number: ($pr | tonumber)}')
    exit 1
fi

# Validar que la respuesta sea JSON válido
if ! echo "$pr_info" | jq empty 2>/dev/null; then
    echo -e "${RED}❌ Respuesta JSON inválida del PR${NC}"
    log_event "error" '{"message": "Invalid JSON response from PR API"}'
    exit 1
fi

pr_title=$(echo "$pr_info" | jq -r '.title // "N/A"')
pr_author=$(echo "$pr_info" | jq -r '.author.login // "unknown"')
pr_state=$(echo "$pr_info" | jq -r '.state // "unknown"')
pr_url=$(echo "$pr_info" | jq -r '.url // "N/A"')

echo "  Título: $pr_title"
echo "  Autor: $pr_author"
echo "  Estado: $pr_state"
echo "  URL: $pr_url"

# Extraer comentarios del PR
echo ""
echo "💬 Extrayendo comentarios de revisión..."

# Obtener comentarios del PR con rate limiting y validación
echo "  Obteniendo comentarios..."
sleep 1  # Rate limiting básico
comments=$(gh api "repos/$repo_info/pulls/$pr_number/comments" --jq '.[] | .body' 2>/dev/null)

echo "  Obteniendo reviews..."
sleep 1  # Rate limiting básico
review_comments=$(gh api "repos/$repo_info/pulls/$pr_number/reviews" --jq '.[] | .body' 2>/dev/null)

# Combinar comentarios
all_comments="$comments
$review_comments"

if [ -z "$all_comments" ]; then
    echo -e "${YELLOW}⚠️  No se encontraron comentarios en el PR${NC}"
    log_event "complete" $(jq -nc --arg pr "$pr_number" '{pr_number: ($pr | tonumber), findings_count: 0, issues_created: 0, status: "no_comments"}')
    exit 0
fi

# Función para determinar tipo de issue basado en contenido
determine_issue_type() {
    local content="$1"
    local content_lower=$(echo "$content" | tr '[:upper:]' '[:lower:]')
    
    if echo "$content_lower" | grep -qE "security|vulnerab|inject|xss|csrf|auth|permission|token|secret"; then
        echo "security"
    elif echo "$content_lower" | grep -qE "bug|error|crash|fail|broken|wrong|incorrect|issue"; then
        echo "bug"
    elif echo "$content_lower" | grep -qE "performance|slow|optimi|cache|memory|cpu|latenc"; then
        echo "performance"
    elif echo "$content_lower" | grep -qE "document|docs|readme|comment|explain"; then
        echo "documentation"
    elif echo "$content_lower" | grep -qE "refactor|cleanup|technical debt|reorganize|simplif"; then
        echo "tech-debt"
    elif echo "$content_lower" | grep -qE "feature|add|implement|create|new|enhance|improve"; then
        echo "enhancement"
    else
        echo "enhancement"
    fi
}

# Función para determinar prioridad
determine_priority() {
    local content="$1"
    local content_lower=$(echo "$content" | tr '[:upper:]' '[:lower:]')
    
    if echo "$content_lower" | grep -qE "critical|urgent|blocker|severe|crash|security"; then
        echo "P0-critical"
    elif echo "$content_lower" | grep -qE "high|important|major|significant"; then
        echo "P1-high"
    elif echo "$content_lower" | grep -qE "low|minor|trivial|nice to have"; then
        echo "P3-low"
    else
        echo "P2-medium"
    fi
}

# Parsear hallazgos (buscar patrones de Claude Code review)
echo "🔍 Analizando hallazgos..."

# Arrays para almacenar hallazgos
declare -a findings_titles
declare -a findings_descriptions
declare -a findings_types
declare -a findings_priorities

# Patrones comunes de hallazgos en reviews
# Buscar líneas que empiecen con números, bullets, o headers
while IFS= read -r line; do
    # Buscar patrones como "1.", "- ", "* ", "### ", etc.
    if [[ "$line" =~ ^[0-9]+\. ]] || [[ "$line" =~ ^[-*•] ]] || [[ "$line" =~ ^#{2,3} ]]; then
        # Limpiar el formato
        finding=$(echo "$line" | sed -E 's/^[0-9]+\.\s*//; s/^[-*•]\s*//; s/^#{2,3}\s*//')
        
        # Ignorar líneas vacías o muy cortas
        if [ ${#finding} -gt 10 ]; then
            # Extraer título (primeras 100 caracteres o hasta el primer punto)
            title=$(echo "$finding" | cut -d'.' -f1 | head -c 100)
            
            # Determinar tipo y prioridad
            type=$(determine_issue_type "$finding")
            priority=$(determine_priority "$finding")
            
            findings_titles+=("$title")
            findings_descriptions+=("$finding")
            findings_types+=("$type")
            findings_priorities+=("$priority")
        fi
    fi
done <<< "$all_comments"

findings_count=${#findings_titles[@]}

# Aplicar límite de procesamiento para prevenir resource exhaustion
if [ $findings_count -gt $MAX_FINDINGS ]; then
    echo -e "${YELLOW}⚠️  Limitando procesamiento a $MAX_FINDINGS hallazgos (encontrados: $findings_count)${NC}"
    findings_count=$MAX_FINDINGS
    # Truncar arrays
    findings_titles=("${findings_titles[@]:0:$MAX_FINDINGS}")
    findings_descriptions=("${findings_descriptions[@]:0:$MAX_FINDINGS}")
    findings_types=("${findings_types[@]:0:$MAX_FINDINGS}")
    findings_priorities=("${findings_priorities[@]:0:$MAX_FINDINGS}")
fi

if [ $findings_count -eq 0 ]; then
    echo -e "${YELLOW}⚠️  No se encontraron hallazgos estructurados en los comentarios${NC}"
    log_event "complete" $(jq -nc --arg pr "$pr_number" '{pr_number: ($pr | tonumber), findings_count: 0, issues_created: 0, status: "no_findings"}')
    exit 0
fi

echo -e "${GREEN}✅ Encontrados $findings_count hallazgos${NC}"

# Crear issues
echo ""
echo "🚀 Creando GitHub issues..."
echo ""

issues_created=0
declare -a created_issue_urls
declare -a created_issue_numbers

for i in "${!findings_titles[@]}"; do
    title="${findings_titles[$i]}"
    description="${findings_descriptions[$i]}"
    type="${findings_types[$i]}"
    priority="${findings_priorities[$i]}"
    
    # Formato del título del issue
    issue_title="[PR#$pr_number] $title"
    
    # Emojis para los labels (solo en labels, no en contenido)
    type_emoji=""
    case $type in
        "security") type_emoji="🔒" ;;
        "bug") type_emoji="🐛" ;;
        "performance") type_emoji="⚡" ;;
        "documentation") type_emoji="📚" ;;
        "tech-debt") type_emoji="🔧" ;;
        "enhancement") type_emoji="✨" ;;
    esac
    
    priority_emoji=""
    case $priority in
        "P0-critical") priority_emoji="🔴" ;;
        "P1-high") priority_emoji="🟠" ;;
        "P2-medium") priority_emoji="🟡" ;;
        "P3-low") priority_emoji="🟢" ;;
    esac
    
    # Cuerpo del issue
    issue_body="## What this issue addresses:
$description

**Source:** PR #$pr_number | **Type:** $type_emoji $type | **Priority:** $priority_emoji $priority

## Context:
- Pull Request: $pr_url
- Author: @$pr_author
- Title: $pr_title

## Suggested resolution:
Review the finding in the PR comments and implement the suggested improvement.

## Related PR:
- #$pr_number

---
*Created from PR findings via /findings-to-issues*"
    
    echo "📝 Creando issue: $issue_title"
    echo "   Tipo: $type_emoji $type | Prioridad: $priority_emoji $priority"
    
    # Usar archivo temporal para issue body (previene command injection)
    temp_body=$(mktemp)
    printf '%s' "$issue_body" > "$temp_body"
    
    # Rate limiting para API calls
    sleep 1
    
    # Crear el issue usando archivo temporal
    result=$(gh issue create \
        --title "$issue_title" \
        --body-file "$temp_body" \
        --label "$type,$priority,from-pr-review" \
        2>&1)
    
    # Limpiar archivo temporal
    rm -f "$temp_body"
    
    if [ $? -eq 0 ]; then
        issue_url=$(echo "$result" | grep -oE 'https://github.com/[^[:space:]]+')
        issue_number=$(echo "$issue_url" | grep -oE '[0-9]+$')
        
        created_issue_urls+=("$issue_url")
        created_issue_numbers+=("$issue_number")
        ((issues_created++))
        
        echo -e "   ${GREEN}✅ Issue creado: #$issue_number${NC}"
        echo "   $issue_url"
        
        # Log issue creado
        log_event "issue_created" $(jq -nc \
            --arg pr "$pr_number" \
            --arg issue_num "$issue_number" \
            --arg title "$issue_title" \
            --arg type_val "$type" \
            --arg priority_val "$priority" \
            --arg url "$issue_url" \
            '{
                pr_number: ($pr | tonumber),
                issue_number: ($issue_num | tonumber),
                title: $title,
                type: $type_val,
                priority: $priority_val,
                url: $url
            }')
    else
        echo -e "   ${RED}❌ Error creando issue${NC}"
        echo "   $result"
        
        # Log error
        log_event "issue_error" $(jq -nc \
            --arg pr "$pr_number" \
            --arg title "$issue_title" \
            '{pr_number: ($pr | tonumber), title: $title, error: "Failed to create issue"}')
    fi
    
    echo ""
done

# Resumen final
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo "📊 RESUMEN DE EJECUCIÓN"
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo "  • PR analizado: #$pr_number"
echo "  • Hallazgos encontrados: $findings_count"
echo "  • Issues creados: $issues_created"
echo "  • Log guardado en: .claude/logs/$(date '+%Y-%m-%d')/findings-to-issues.jsonl"

if [ $issues_created -gt 0 ]; then
    echo ""
    echo "🔗 Issues creados:"
    for i in "${!created_issue_urls[@]}"; do
        echo "  • #${created_issue_numbers[$i]}: ${created_issue_urls[$i]}"
    done
fi

# Log resumen final
issue_numbers_json=$(printf '%s\n' "${created_issue_numbers[@]}" | jq -R . | jq -s .)
log_event "complete" $(jq -nc \
    --arg pr "$pr_number" \
    --arg findings_cnt "$findings_count" \
    --arg issues_cnt "$issues_created" \
    --argjson issue_nums "$issue_numbers_json" \
    '{
        pr_number: ($pr | tonumber),
        findings_count: ($findings_cnt | tonumber),
        issues_created: ($issues_cnt | tonumber),
        status: "success",
        issue_numbers: $issue_nums
    }')

echo ""
echo -e "${GREEN}✨ Proceso completado exitosamente${NC}"
```

## Uso

```bash
# Detectar PR del branch actual automáticamente
/findings-to-issues

# Especificar número de PR
/findings-to-issues 123

# Ver PRs disponibles si no hay PR en el branch actual
gh pr list
```

## Características

### Categorización Automática
- **security** 🔒 - Vulnerabilidades, permisos, autenticación
- **bug** 🐛 - Errores, crashes, comportamiento incorrecto
- **performance** ⚡ - Optimización, latencia, uso de recursos
- **documentation** 📚 - Documentación faltante o desactualizada
- **tech-debt** 🔧 - Refactoring, limpieza de código
- **enhancement** ✨ - Nuevas funcionalidades, mejoras

### Priorización Inteligente
- **P0-critical** 🔴 - Bloqueante, urgente
- **P1-high** 🟠 - Importante, alta prioridad
- **P2-medium** 🟡 - Prioridad normal
- **P3-low** 🟢 - Menor prioridad

### Logging Auditable
Todos los eventos se registran en formato JSONL:
- `.claude/logs/YYYY-MM-DD/findings-to-issues.jsonl`
- Timestamps ISO 8601
- Eventos: start, issue_created, issue_error, complete
- Información completa para auditoría

### Validaciones Robustas
- Verificación de gh CLI instalado
- Autenticación GitHub activa
- Repositorio GitHub válido
- Manejo de errores con mensajes claros

## Mejoras sobre versión anterior

✅ **Ejecución directa en bash** - Sin dependencias de agents
✅ **Categorización inteligente** - Detecta tipo automáticamente
✅ **Priorización automática** - Basada en keywords
✅ **Formato profesional** - Template estructurado para issues
✅ **Logging completo** - JSONL con toda la información
✅ **Validaciones robustas** - Manejo de todos los casos de error
✅ **Output informativo** - Resumen claro y URLs de issues