---
name: gtd-briefing
description: Generate a context briefing for a project or topic. Searches the Obsidian vault and Todoist for relevant notes and open tasks, then synthesizes a summary. Use when the user asks to be updated on something ("Me atualize sobre X", "Status de X", "O que temos sobre X").
---

# GTD Briefing

Generate a context briefing for the requested topic.

## Step 1 — Search vault for relevant notes

```bash
VAULT="/workspace/extra/obsidian_vault"
grep -rl "TOPIC" "$VAULT" --include="*.md" 2>/dev/null
```

Read the most relevant files found (prioritize `01_Projetos` and `05_Diario`).

## Step 2 — Query open Todoist tasks

```bash
curl -s https://api.todoist.com/rest/v2/tasks \
  -H "Authorization: Bearer $TODOIST_API_TOKEN"
```

Filter results for tasks related to the topic (check `content` field).

## Step 3 — Synthesize and respond

Reply in this structure:
```
*[Topic] — Briefing*

*Tarefas abertas:*
• [task 1] (label)
• [task 2] (label)
(or "Nenhuma tarefa aberta.")

*Notas relevantes:*
• [note title] — [one-line summary]
(or "Nenhuma nota encontrada.")

*Próximos passos sugeridos:*
• [suggestion based on what's missing or pending]
```

Keep it concise — this is a briefing, not a report.
