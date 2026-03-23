---
name: gtd-photo
description: Save a photo sent via Telegram into the Obsidian vault. Creates a meaningful named note with proper project/folder structure. Use whenever a message contains [Photo: /workspace/group/media/FILENAME].
---

# GTD Photo

The image is already downloaded at the path in the message. Save it to the vault with a proper note.

## Step 1 — Determine destination from caption/context

- Mentions "projeto X" → `01_Projetos/slug-do-projeto/`
- Routine/area (health, finance) → `02_Areas/slug-da-area/`
- Generic reference or idea → `03_Recursos/`
- No context → `00_Inbox/`

Extract project slug from caption: lowercase, accents stripped, spaces replaced with hyphens.
Example: "projeto espaço gourmet" → `espaco-gourmet`

## Step 2 — Move image into vault

```bash
VAULT="/workspace/extra/obsidian_vault"
DEST="$VAULT/01_Projetos/slug-do-projeto/assets"
mkdir -p "$DEST"
mv /workspace/group/media/FILENAME "$DEST/FILENAME"
```

## Step 3 — Create a meaningful note

Get current date:
```bash
date +"%Y-%m-%dT%H:%M:%S"
date +"%Y-%m-%d"
```

Write the note — title must be derived from the caption, NOT the filename:
```bash
cat > "$VAULT/01_Projetos/slug-do-projeto/referencia-visual-YYYY-MM-DD.md" << 'OBSIDIAN_EOF'
---
id: "YYYY-MM-DDTHH:MM:SS"
tipo: "PROJECT_SUPPORT"
data_criacao: "YYYY-MM-DDTHH:MM:SS"
projeto: "[[Nome do Projeto]]"
tags: ["referencia", "visual"]
pessoas: []
status: "processado"
---
# Referência Visual — Nome do Projeto

![[assets/FILENAME]]

Breve descrição do que a imagem mostra, inferida do caption.
OBSIDIAN_EOF
```

For `03_Recursos` or `00_Inbox`, omit the `projeto` field and adjust the note title accordingly.

## Step 4 — Respond concisely

`✅ Imagem salva em 01_Projetos/espaco-gourmet/assets/. Nota: referencia-visual-YYYY-MM-DD.md.`
