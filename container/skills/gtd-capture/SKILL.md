---
name: gtd-capture
description: Process a GTD capture — route actionable tasks to Todoist and reference material to the Obsidian vault. Use for any message that contains something to capture (tasks, ideas, links, notes, references).
---

# GTD Capture

Process the incoming message through the GTD funnel.

## Step 1 — Extract and classify

Read the message and extract:
- **Actionable items** (sentences with action verbs, things to do) → Todoist
- **Reference material** (information, links, ideas, context) → Obsidian

A message can produce both. Reference material NEVER goes to Todoist.

## Step 2 — Create Todoist tasks (for actionable items only)

```bash
curl -s -X POST https://api.todoist.com/rest/v2/tasks \
  -H "Authorization: Bearer $TODOIST_API_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"content": "TASK TEXT", "labels": ["LABEL"]}'
```

Infer the label from context:
- `@aguardando` — delegated, waiting on someone
- `@baixa_energia` — quick, low-effort
- `@alta_energia` — complex, needs deep focus

## Step 3 — Create Obsidian note (for reference material)

Determine the folder using this decision tree:
- No clear context → `00_Inbox`
- Log, meeting, or specific date → `05_Diario`
- Supports an active project → `01_Projetos/slug-do-projeto/`
- Ongoing routine (health, finance) → `02_Areas/slug-da-area/`
- Link, article, generic idea → `03_Recursos`

Get current timestamp:
```bash
date +"%Y-%m-%dT%H:%M:%S"
date +"%Y-%m-%d"
```

Write the note — title and filename must be *human-readable and derived from content*, never from a raw ID:
```bash
VAULT="/workspace/extra/obsidian_vault"
cat > "$VAULT/FOLDER/YYYY-MM-DD_slug-do-titulo.md" << 'OBSIDIAN_EOF'
---
id: "YYYY-MM-DDTHH:MM:SS"
tipo: "INSIGHT"
data_criacao: "YYYY-MM-DDTHH:MM:SS"
projeto: "[[Nome do Projeto]]"
tags: ["tag1"]
pessoas: []
status: "processado"
---
# Título Significativo

Conteúdo sintetizado — não copie a mensagem literalmente, sintetize.
OBSIDIAN_EOF
```

## Step 4 — Respond concisely

Examples:
- `✅ 1 tarefa no Todoist (@alta_energia). 1 nota em 03_Recursos.`
- `✅ 1 tarefa no Todoist (@aguardando). Sem notas (item puramente acionável).`
- `✅ 1 nota em 01_Projetos/espaco-gourmet/.`
