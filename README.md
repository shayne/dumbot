# Dumbot

Dumbot is a virtual AI assistant that never forgets. Every conversation is stored in a dynamic database so it can remember what you told it yesterday, last week, or last month and keep the context consistent over time.

---

## What Dumbot does

- **Long‑term memory**: Dumbot stores everything it is told in a database, so it can keep context across conversations.
- **Telegram integration**: Chat with Dumbot inside Telegram.
- **Optional web search**: Connect Tavily to give Dumbot live web search.

---

## What you need

- A Linux server (or any machine that can run Docker).
- A Telegram bot token from @BotFather.
- An Anthropic API key (required).
- Optional: a Tavily API key for web search.

---

## Step 1 — Create a Telegram bot

1. Open Telegram and search for **@BotFather**.
2. Send `/newbot` and follow the prompts to create your bot.
3. Copy the **bot token** that BotFather gives you.

Helpful Telegram docs:

- BotFather guide: `https://core.telegram.org/bots/features#botfather`
- Bot tutorial: `https://core.telegram.org/bots/tutorial`

---

## Step 2 — Get your API keys

### Anthropic (required)

You must set `ANTHROPIC_API_KEY` or Dumbot will not start.

Create an Anthropic account and generate an API key in the Anthropic Console (Account Settings):

- Anthropic Console: `https://console.anthropic.com`
- API docs (getting started): `https://docs.anthropic.com/en/api/getting-started`

### Tavily (optional, enables web search)

If you want Dumbot to search the web, create a Tavily account and copy an API key from your dashboard:

- Tavily website: `https://tavily.com`
- API key help: `https://help.tavily.com/articles/9170796666-how-can-i-create-an-api-key`

---

## Step 3 — Run with Docker (linuxserver.io style)

Replace values in ALL_CAPS with your own:

```bash
docker run -d \
  --name dumbot \
  -e TZ=Etc/UTC \
  -e ANTHROPIC_API_KEY=YOUR_ANTHROPIC_KEY \
  -e TELEGRAM_BOT_TOKEN=YOUR_TELEGRAM_BOT_TOKEN \
  -e TAVILY_API_KEY=YOUR_TAVILY_KEY \
  -e DUMBOT_DB_FILE=/data/dumbot.db \
  -e DUMBOT_STORAGE_DIR=/data \
  -v /path/on/host/dumbot:/data \
  --restart unless-stopped \
  ghcr.io/shayne/dumbot:latest
```

Notes:

- `DUMBOT_DB_FILE` controls the main app database path. If you set it under `/data` (as above), the SQLite DB is persisted on the host.
- `DUMBOT_STORAGE_DIR` holds per-channel databases, media, and attachments. This is also persisted on the host.
- The container runs as an unprivileged user. Ensure the mounted host directory is writable by the container user or the service will fail to open the SQLite DB.

---

## Step 4 — Docker Compose example

Create a file named `compose.yml`:

```yaml
services:
  dumbot:
    image: ghcr.io/shayne/dumbot:latest
    container_name: dumbot
    environment:
      - ANTHROPIC_API_KEY=YOUR_ANTHROPIC_KEY
      - TELEGRAM_BOT_TOKEN=YOUR_TELEGRAM_BOT_TOKEN
      - DUMBOT_DB_FILE=/data/dumbot.db
      - DUMBOT_STORAGE_DIR=/data
      # Optional:
      - TAVILY_API_KEY=YOUR_TAVILY_KEY
      - TZ=Etc/UTC
    volumes:
      # Ensure the host directory is writable by the container user.
      - ./data:/data
    restart: unless-stopped
```

Start it:

```bash
docker compose up -d
```

---

## Troubleshooting

- **Dumbot won’t start**: Make sure `ANTHROPIC_API_KEY` is set.
- **Telegram messages not working**: Confirm your `TELEGRAM_BOT_TOKEN` is correct and the bot is started in Telegram.
- **Web search missing**: Set `TAVILY_API_KEY` to enable Tavily.

---

## Security tips

- Treat all API keys like passwords.
- Do not share them publicly.
- Use environment variables (as shown above) instead of hard‑coding keys.
