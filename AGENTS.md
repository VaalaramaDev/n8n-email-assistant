# AGENTS.md — AI Email Assistant

## Project overview

n8n workflow that monitors a Gmail inbox, classifies incoming emails and
generates a draft reply via OpenAI GPT-4o-mini, then sends the draft to
Telegram for human approval before sending.

Stack: n8n 1.x (self-hosted, Docker) · OpenAI API · Gmail API (OAuth 2.0) ·
Telegram Bot API

---

## Architecture

Gmail Inbox
└─ Gmail Trigger (polling · 60s · unread)
└─ IF Filter (skip noreply / no-reply addresses)
└─ HTTP Request → OpenAI GPT-4o-mini
└─ Code node (parse JSON response)
└─ Telegram: send card + inline buttons
└─ Telegram Trigger (webhook callback)
└─ Switch
├─ send → Gmail: send reply → Telegram: confirm
├─ edit → Telegram: prompt for new text → Gmail: send
└─ reject → Telegram: confirm dismissal

---

## Nodes reference

| #   | Node             | Type         | Key settings                                                                                                                              |
| --- | ---------------- | ------------ | ----------------------------------------------------------------------------------------------------------------------------------------- |
| 1   | Gmail Trigger    | Gmail        | Label: INBOX · Mark as read: true · Poll every 60s                                                                                        |
| 2   | IF Filter        | IF           | Condition: `{{ $json.from }}` does not contain `noreply` or `no-reply`                                                                    |
| 3   | OpenAI Request   | HTTP Request | POST `https://api.openai.com/v1/chat/completions` · model: `gpt-4o-mini` · Auth: Header `Authorization: Bearer {{ $env.OPENAI_API_KEY }}` |
| 4   | Parse response   | Code (JS)    | `JSON.parse($input.item.json.choices[0].message.content)`                                                                                 |
| 5   | Telegram card    | Telegram     | Send message with `parse_mode: HTML` · inline keyboard: Send / Edit / Reject                                                              |
| 6   | Telegram Trigger | Webhook      | Path: `/telegram-callback` · responds to `callback_query`                                                                                 |
| 7   | Switch           | Switch       | Routes on `{{ $json.body.callback_query.data }}`: `send` · `edit` · `reject`                                                              |
| 8   | Gmail Send       | Gmail        | Operation: Reply · threadId from original trigger data                                                                                    |
| 9   | Telegram confirm | Telegram     | Sends confirmation message to `TELEGRAM_CHAT_ID`                                                                                          |

---

## OpenAI prompt

System prompt used in node #3:
You are an email assistant. Given an incoming email, do two things:

Classify it into one of: inquiry, complaint, spam, newsletter, support, other
Write a professional draft reply in the same language as the email.

Respond ONLY with valid JSON and nothing else:
{
"category": "inquiry",
"priority": "high|medium|low",
"summary": "2-3 sentence summary of the email",
"draft_reply": "Full draft reply text"
}

User message passed to the API:
From: {{ $json.from }}
Subject: {{ $json.subject }}
Body:
{{ $json.text }}

---

## Telegram card format

📧 <b>New email</b>
<b>From:</b> {{ from }}
<b>Subject:</b> {{ subject }}
🏷 Category: {{ category }} | Priority: {{ priority }}
📝 <b>Summary:</b>
{{ summary }}
✍️ <b>Draft reply:</b>
{{ draft_reply }}

Inline keyboard:

```json
{
  "inline_keyboard": [
    [
      { "text": "✅ Send", "callback_data": "send" },
      { "text": "✏️ Edit", "callback_data": "edit" },
      { "text": "❌ Reject", "callback_data": "reject" }
    ]
  ]
}
```

---

## Environment variables

| Variable             | Description                                     |
| -------------------- | ----------------------------------------------- |
| `N8N_USER`           | n8n basic auth username                         |
| `N8N_PASSWORD`       | n8n basic auth password                         |
| `N8N_HOST`           | VPS IP or domain (e.g. `178.104.202.207`)       |
| `N8N_ENCRYPTION_KEY` | Random 32-char string for credential encryption |
| `OPENAI_API_KEY`     | OpenAI API key (`sk-...`)                       |
| `TELEGRAM_BOT_TOKEN` | Bot token from @BotFather                       |
| `TELEGRAM_CHAT_ID`   | Your Telegram user/chat ID                      |

Gmail OAuth credentials (`GMAIL_CLIENT_ID`, `GMAIL_CLIENT_SECRET`) are
configured directly in the n8n credentials UI — never stored in `.env`.

---

## Agent instructions

This project is a **configuration-driven workflow**, not a traditional
codebase. The agent's job is to generate the scaffolding files; the actual
workflow logic lives inside the n8n UI and is exported as JSON.

### What the agent must create

1. `docker-compose.yml` — n8n service with basic auth, volume, port 5678
2. `.env.example` — all variables with inline comments
3. `.gitignore` — exclude `.env`, `n8n_data/`, `*.log`, `.DS_Store`
4. `workflows/email_assistant.json` — valid n8n workflow JSON stub with all
   9 nodes listed above (nodes array + connections map + `"active": false`)
5. `docs/setup-gmail-oauth.md` — step-by-step OAuth guide (English)
6. `README.md` — full portfolio README per conventions below

### README conventions (apply to every project)

Sections in order:
`Features` · `Architecture` (ASCII flow) · `Tech Stack` · `Quick Start` ·
`Deployment on VPS` · `Gmail OAuth Setup` (link to docs/) ·
`Environment Variables` (table) · `Useful Commands` · `Workflow Import`

Rules:

- English only
- No `.env` committed — reference `.env.example` everywhere
- `data/` and `n8n_data/` mounted as Docker volumes, not copied in
- Sensitive values never appear in logs or README examples

### What the agent must NOT do

- Do not implement real HTTP calls to Gmail, OpenAI, or Telegram — n8n
  handles all integrations via its node system
- Do not create a Python or Node.js application — this project has no
  custom runtime code beyond the Code node snippet above
- Do not hardcode API keys, tokens, or passwords anywhere
- Do not commit or reference `.env` — only `.env.example`

### workflow/email_assistant.json stub requirements

The JSON must be valid and importable by n8n. Minimum required fields:

```json
{
  "name": "AI Email Assistant",
  "active": false,
  "nodes": [
    /* 9 node objects with id, name, type, position, parameters */
  ],
  "connections": {
    /* wiring between nodes */
  },
  "settings": { "executionOrder": "v1" },
  "id": "ai-email-assistant-v1"
}
```

Each node object must include: `id`, `name`, `type` (e.g.
`n8n-nodes-base.gmail`), `typeVersion`, `position` [x, y], `parameters`.

Use descriptive placeholder values in parameters — they will be replaced
by real credentials in the n8n UI.

---

## Testing checklist (manual, after deploy)

- [ ] n8n accessible at `http://178.104.202.207:5678` with basic auth
- [ ] Workflow imported and visible in n8n UI
- [ ] Gmail OAuth credential connected and authorized
- [ ] OpenAI credential added (Header Auth with `OPENAI_API_KEY`)
- [ ] Telegram credential added (Bot Token)
- [ ] Send test email to monitored inbox → Telegram card appears within 60s
- [ ] Tap ✅ Send → reply arrives in original thread
- [ ] Tap ❌ Reject → Telegram confirms dismissal
- [ ] noreply email sent → no Telegram notification triggered

---

## Useful commands

```bash
# Start
docker compose up -d

# Stop
docker compose down

# Logs
docker compose logs -f n8n

# Update n8n to latest
docker compose pull && docker compose up -d --build

# Export workflow from n8n UI
# Settings → Workflows → ··· → Download → save to workflows/email_assistant.json
# Then commit: git add workflows/email_assistant.json && git commit -m "update workflow export"
```

---

## Related projects

| Repo                | Stack                                 |
| ------------------- | ------------------------------------- |
| `tg-todo-bot`       | Python · python-telegram-bot · SQLite |
| `tg-currency-bot`   | Python · REST API · APScheduler       |
| `discord-mod-bot`   | discord.py · slash commands · SQLite  |
| `discord-music-bot` | discord.py · yt-dlp · FFmpeg          |
