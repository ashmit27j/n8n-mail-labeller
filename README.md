# Mail Labeller

![n8n workflow](./assets/architecture.png)

Self-hosted n8n automation that watches an inbox over IMAP, classifies each incoming email with an LLM into one of a fixed set of categories, and automatically sorts it into the matching folder — no manual filing required.

---

## Architecture

- **n8n** (Docker, self-hosted, local) — runs all workflow logic
- **IMAP** (mail server, e.g. Gmail via app password) — polls for new mail and moves classified messages into folders
- **SMTP** (app password, optional) — sends notification/error emails if configured
- **LLM API** (OpenAI or Anthropic) — reads subject/sender/snippet and assigns a label

One workflow:
1. **Ingest & Label** — polls the inbox every 5 minutes, classifies each new email, and moves it into the corresponding folder

---

## Prerequisites

- Docker installed and running
- A mail account with IMAP and SMTP access enabled
- 2-Step Verification turned on for that account, so an app password can be generated
- An OpenAI or Anthropic API key

---

## Setup Steps

### 1. Clone and start n8n

```bash
git clone <this-repo>
cd mail-labeller
docker compose up -d
```

### 2. Generate an app password

1. Confirm 2-Step Verification is **On**: [myaccount.google.com/security](https://myaccount.google.com/security)
2. Generate an app password: [myaccount.google.com/apppasswords](https://myaccount.google.com/apppasswords)
3. Use `imap.gmail.com` port `993` for IMAP and `smtp.gmail.com` port `587` for SMTP, with your email address as the user and the 16-character app password (not your regular account password)

### 3. Get an LLM API key

1. OpenAI: [platform.openai.com/api-keys](https://platform.openai.com/api-keys)
2. Or Anthropic: [console.anthropic.com](https://console.anthropic.com)

### 4. Fill in environment values

Copy `.env.example` to `.env` and fill in every value from steps 2–3, plus your desired `LABELS` list (comma-separated). Never commit `.env`.

### 5. First-time n8n login

1. Open [http://localhost:5678](http://localhost:5678)
2. Create an owner account
3. Add credentials inside n8n: **IMAP**, **SMTP** (optional), and your **LLM API** key

### 6. Create the target folders

Make sure each folder/label in your `LABELS` list already exists on the mail server (Gmail: create the labels in Gmail settings first), since most IMAP servers don't auto-create folders on move.

### 7. Import/run the workflow

The workflow JSON export lives in `workflows/`. Import via n8n's UI (Workflows → Import from File) or its REST API, then activate it.

---

## Environment Variables

| Variable | Notes |
|---|---|
| `IMAP_HOST` / `IMAP_PORT` | e.g. `imap.gmail.com` / `993` |
| `IMAP_USER` | Your email address |
| `IMAP_APP_PASSWORD` | 16-character app password |
| `SMTP_HOST` / `SMTP_PORT` | e.g. `smtp.gmail.com` / `587` (optional) |
| `LLM_PROVIDER` | `openai` or `anthropic` |
| `LLM_API_KEY` | From OpenAI or Anthropic |
| `LABELS` | Comma-separated folder/label names, e.g. `Work,Personal,Finance,Newsletters,Promotions,Urgent,Other` |

---

## Repo Structure

```
mail-labeller/
├── docker-compose.yml
├── .env                  # not committed
├── .env.example
├── workflows/
│   └── ingest-and-label.json
├── assets/
│   └── architecture.png
└── README.md
```

---

## Status / Future Work

- [x] IMAP polling for new mail
- [x] LLM-based classification into a fixed label set
- [x] Auto-move into matching folder
- [ ] Auto-create missing folders instead of requiring them upfront
- [ ] Confidence threshold + fallback review folder for low-confidence classifications
- [ ] Restart-resilience (`restart: unless-stopped` in compose, or a proper VPS deployment) so polling survives a machine reboot
