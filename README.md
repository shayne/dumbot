# Dumbot

Dumbot is a persistent AI assistant with long-term memory and Telegram integration.
It stores conversation history in SQLite so context carries across sessions.

## Quick Start (Docker)

1. Create a Telegram bot with `@BotFather` and copy `TELEGRAM_BOT_TOKEN`.
2. Choose an LLM provider:
   - Anthropic (default): set `ANTHROPIC_API_KEY`
   - OpenAI: set `DUMBOT_LLM_PROVIDER=openai` and `OPENAI_API_KEY`
3. Run the container.

OpenAI example:

```bash
docker run -d \
  --name dumbot \
  -e TZ=Etc/UTC \
  -e TELEGRAM_BOT_TOKEN=YOUR_TELEGRAM_BOT_TOKEN \
  -e DUMBOT_LLM_PROVIDER=openai \
  -e OPENAI_API_KEY=YOUR_OPENAI_API_KEY \
  -e DUMBOT_OPENAI_EMPTY_COMPLETED_MESSAGE_RETRY_LIMIT=5 \
  -e DUMBOT_DB_FILE=/data/dumbot.db \
  -e DUMBOT_STORAGE_DIR=/data \
  -v /path/on/host/dumbot:/data \
  --restart unless-stopped \
  ghcr.io/shayne/dumbot:latest
```

Anthropic example:

```bash
docker run -d \
  --name dumbot \
  -e TZ=Etc/UTC \
  -e TELEGRAM_BOT_TOKEN=YOUR_TELEGRAM_BOT_TOKEN \
  -e ANTHROPIC_API_KEY=YOUR_ANTHROPIC_API_KEY \
  -e DUMBOT_DB_FILE=/data/dumbot.db \
  -e DUMBOT_STORAGE_DIR=/data \
  -v /path/on/host/dumbot:/data \
  --restart unless-stopped \
  ghcr.io/shayne/dumbot:latest
```

## Docker Compose

```yaml
services:
  dumbot:
    image: ghcr.io/shayne/dumbot:latest
    container_name: dumbot
    environment:
      - TELEGRAM_BOT_TOKEN=YOUR_TELEGRAM_BOT_TOKEN
      # Provider selection:
      - DUMBOT_LLM_PROVIDER=openai
      - OPENAI_API_KEY=YOUR_OPENAI_API_KEY
      # or use Anthropic instead:
      # - ANTHROPIC_API_KEY=YOUR_ANTHROPIC_API_KEY

      # Optional model override:
      # - DUMBOT_LLM_MODEL=gpt-5.3-codex

      # Optional: retries before showing retry button for empty OpenAI responses
      - DUMBOT_OPENAI_EMPTY_COMPLETED_MESSAGE_RETRY_LIMIT=5

      - DUMBOT_DB_FILE=/data/dumbot.db
      - DUMBOT_STORAGE_DIR=/data
      # Optional:
      # - TAVILY_API_KEY=YOUR_TAVILY_KEY
      # - DUMBOT_DEBUG=1
      # - TZ=Etc/UTC
    volumes:
      - ./data:/data
    restart: unless-stopped
```

## Environment Variables

### Core

- `TELEGRAM_BOT_TOKEN` (required for Telegram bot operation)
- `DUMBOT_LLM_PROVIDER` (`anthropic` default, or `openai`)
- `DUMBOT_LLM_MODEL` (optional model override)
- `DUMBOT_DB_FILE` (default `data/dumbot.db`)
- `DUMBOT_STORAGE_DIR` (default `./storage`)
- `DUMBOT_PUBLIC_URL` (optional override for public base URL)
- `DUMBOT_DEBUG` (`1/true/yes/on` enables verbose diagnostics)

### Provider keys

- `ANTHROPIC_API_KEY` (required when provider is `anthropic`)
- `OPENAI_API_KEY` (required when provider is `openai`)

### OpenAI-specific runtime tuning

- `DUMBOT_OPENAI_EMPTY_COMPLETED_MESSAGE_RETRY_LIMIT`
  - Retries top-level transient empty OpenAI Responses replies before returning a retry error to the user.
  - Default behavior in Dumbot is `5` retries.

### Optional integrations

- `TAVILY_API_KEY` (web search tools)
- `SLACK_CLIENT_ID`, `SLACK_CLIENT_SECRET`, `SLACK_SIGNING_SECRET`
- `TWILIO_AUTH_TOKEN`, `TWILIO_WEBHOOK_URL`, `TWILIO_ACCOUNT_SID`

## Notes

- The container runs as an unprivileged user. Ensure the mounted data directory is writable.
- If using OpenAI, keep `DUMBOT_OPENAI_EMPTY_COMPLETED_MESSAGE_RETRY_LIMIT` at a positive value (the default `5` is recommended).
- If retries are exhausted for a transient empty model reply, Dumbot returns a friendly failure message and presents a Telegram retry button.

## Troubleshooting

- No responses in Telegram:
  - verify `TELEGRAM_BOT_TOKEN`
  - verify provider key for the selected `DUMBOT_LLM_PROVIDER`
- Bot starts but search tools missing:
  - set `TAVILY_API_KEY`
- Need deeper diagnostics:
  - set `DUMBOT_DEBUG=1` and inspect container logs
