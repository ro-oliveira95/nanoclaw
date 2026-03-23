# NanoClaw Customization Guide

This document tracks architectural decisions and customizations made to this fork of [qwibitai/nanoclaw](https://github.com/qwibitai/nanoclaw).

## Branching Strategy

**Stay on `main` for everything**, with short-lived feature branches for risky core changes.

- The upstream sync automation and skill merges are designed around `main` — a long-lived custom branch creates permanent divergence overhead
- `main` IS the customization; the fork's identity diverges from upstream by design
- **Safe to commit directly to main:** CLAUDE.md edits, new skills, docs, config values
- **Use a short-lived branch:** any change to `src/` core files (container-runner.ts, config.ts, channel files)

**Known conflict point:** upstream may update `groups/global/CLAUDE.md`. Our GTD routing table lives in the bottom section — resolve manually on upstream sync.

## Active Channels

| Channel | Registered as |
|---------|---------------|
| WhatsApp | `whatsapp_main` (main) |
| Telegram | `telegram_main` (main) |

## Customizations

### 1. CONTAINER_ENV_VARS passthrough (`src/config.ts`, `src/container-runner.ts`)

Adds a `CONTAINER_ENV_VARS` setting in `.env` — a comma-separated list of var names whose values are read from `.env` and injected as `-e` flags into agent containers. Keeps secrets out of `process.env` while making application tokens available inside the sandbox.

Current value: `CONTAINER_ENV_VARS=TODOIST_API_TOKEN`

### 2. Telegram photo download (`src/channels/telegram.ts`)

When a photo is received, the best-quality JPEG is downloaded from Telegram's servers via `getFile` and saved to `groups/<folder>/media/`. The message content becomes `[Photo: /workspace/group/media/<file>] caption` so the container agent can move the file into the Obsidian vault and create a meaningful note.

### 3. GTD/PARA system (`container/skills/`, `groups/global/CLAUDE.md`)

Implements a full GTD capture system backed by Todoist and an Obsidian vault. Uses a skill-based architecture so instructions only load into context when invoked.

See [GTD_AGENT_SPECS.md](GTD_AGENT_SPECS.md) for the full specification.

**Skills:**
| Skill | Trigger |
|-------|---------|
| `gtd-capture` | Any message with something to capture |
| `gtd-photo` | Message contains `[Photo: /workspace/group/media/...]` |
| `gtd-briefing` | "Me atualize sobre X", "status de X" |
| `gtd-weekly-review` | Automated cron — Fridays 16h |

**Infrastructure:**
- Obsidian vault mounted at `/workspace/extra/obsidian_vault` (host path configured in mount allowlist)
- Mount allowlist: `~/.config/nanoclaw/mount-allowlist.json`
- Todoist token: `TODOIST_API_TOKEN` in `.env`, passed to containers via `CONTAINER_ENV_VARS`
- Weekly review task in `scheduled_tasks` table (cron `0 16 * * 5`, group `telegram_main`)

## Assistant Identity

- **Name:** Jarvis (set via `ASSISTANT_NAME=Jarvis` in `.env`)
- **Personality:** `groups/global/CLAUDE.md` — calm, sharp, GTD-aware Chief of Staff
- **Per-channel config:** `groups/telegram_main/CLAUDE.md` — vault path (gitignored)

## Debugging

Enable full container stdout/stderr in logs:
```
LOG_LEVEL=debug  # in .env
```

Logs land in `groups/<folder>/logs/container-<timestamp>.log`.
Remove when done to keep logs lean.
