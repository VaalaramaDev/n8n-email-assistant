# AI Email Assistant

An n8n workflow that monitors a Gmail inbox, classifies incoming emails with GPT-4o-mini, generates a professional draft reply, and sends the result to Telegram for human approval. The approval step uses inline buttons so you can review each message before anything is sent back to the original email thread.

## Features

- Automatic Gmail monitoring with polling every minute
- Email classification: inquiry, complaint, spam, newsletter, support, other
- Priority detection: high / medium / low
- Draft reply generation in the language of the original email
- Telegram summary card with category, priority, summary, and draft reply
- Inline approval buttons: Send / Edit / Reject
- Filtering for `noreply` and `no-reply` senders

## Architecture

```text
Gmail Inbox
  -> n8n Trigger
  -> Filter
  -> OpenAI GPT-4o-mini
  -> Parse
  -> Telegram Bot
  -> [Approve / Edit / Reject]
  -> Gmail Send
  -> Telegram Confirm
```

## Tech Stack

- n8n 1.x (self-hosted with Docker)
- OpenAI API (`gpt-4o-mini`)
- Gmail API (OAuth 2.0)
- Telegram Bot API

## Quick Start

### Local

1. Clone the repository:

   ```bash
   git clone <your-repository-url>
   cd n8n-email-assistant
   ```

2. Copy the environment template and update the values:

   ```bash
   cp .env.example .env
   ```

3. Start the stack:

   ```bash
   docker compose up -d
   ```

4. Open `http://localhost:5678`.
5. Import `workflows/email_assistant.json`.
6. Configure the Gmail OAuth, OpenAI, and Telegram credentials in n8n.
7. Activate the workflow.

## Deployment on VPS

1. Connect to your server:

   ```bash
   ssh user@178.104.202.207
   ```

2. Clone the project:

   ```bash
   cd ~/projects
   git clone <your-repository-url> n8n-email-assistant
   cd n8n-email-assistant
   ```

3. Create the environment file:

   ```bash
   cp .env.example .env
   nano .env
   ```

4. Start the service:

   ```bash
   docker compose up -d --build
   ```

5. Open `http://178.104.202.207:5678`.
6. Import the workflow and configure all credentials in the n8n UI.

## Gmail OAuth Setup

See [docs/setup-gmail-oauth.md](docs/setup-gmail-oauth.md).

## Environment Variables

| Variable | Description | Example |
| --- | --- | --- |
| `N8N_USER` | Basic auth username for the n8n web UI | `admin` |
| `N8N_PASSWORD` | Basic auth password for the n8n web UI | `changeme` |
| `N8N_HOST` | Public VPS IP or domain used by n8n and webhooks | `178.104.202.207` |
| `N8N_ENCRYPTION_KEY` | Random string used by n8n to encrypt credentials | `replace_with_32char_random_string` |
| `OPENAI_API_KEY` | API key used by the HTTP Request node for OpenAI | `sk-...` |
| `TELEGRAM_BOT_TOKEN` | Bot token created with BotFather | `123456:ABCDEF...` |
| `TELEGRAM_CHAT_ID` | Telegram user ID or chat ID that receives approval messages | `123456789` |

## Useful Commands

```bash
docker compose up -d                # start
docker compose down                 # stop
docker compose logs -f n8n          # logs
docker compose pull && docker compose up -d  # update n8n
```

## Workflow Import

1. Open the n8n UI.
2. Go to **Settings** -> **Workflows** -> **Import**.
3. Select `workflows/email_assistant.json`.
4. Open the imported workflow.
5. Configure credentials and adjust placeholder node settings as needed.
6. Save and activate the workflow.

![n8n](https://img.shields.io/badge/n8n-Workflow%20Automation-EA4B71?style=for-the-badge)
![OpenAI](https://img.shields.io/badge/OpenAI-GPT--4o--mini-412991?style=for-the-badge)
![Telegram](https://img.shields.io/badge/Telegram-Bot%20API-26A5E4?style=for-the-badge)
![Docker](https://img.shields.io/badge/Docker-Compose-2496ED?style=for-the-badge)
