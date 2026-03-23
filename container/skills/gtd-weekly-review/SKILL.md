---
name: gtd-weekly-review
description: Run the weekly GTD review. Reads the Obsidian inbox, fetches overdue Todoist tasks, and sends a structured review message. Used by the automated Friday 16h cron task.
---

# GTD Weekly Review

## Step 1 — Count Obsidian inbox items

```bash
VAULT="/workspace/extra/obsidian_vault"
ls "$VAULT/00_Inbox/" 2>/dev/null | grep -c "\.md$" || echo 0
```

List the filenames so you can mention them by name.

## Step 2 — Fetch overdue Todoist tasks

```bash
curl -s "https://api.todoist.com/rest/v2/tasks?filter=overdue" \
  -H "Authorization: Bearer $TODOIST_API_TOKEN"
```

Extract `content` and `due.date` from each result.

## Step 3 — Send review message

```
📋 *Revisão Semanal GTD*

🗂 *Inbox Obsidian:* X itens aguardando processamento
[lista dos títulos dos itens, se houver]

⚠️ *Tarefas vencidas:* Y no Todoist
[lista: • tarefa (venceu DD/MM)]

Quer processar a inbox agora? Manda os itens um por um que eu roteio cada um.
```

If inbox is empty and no overdue tasks: `✅ Revisão semanal: inbox limpa, sem tarefas vencidas. Boa semana!`
