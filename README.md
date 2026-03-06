# Automated Real Estate Monitoring — Montreal Multiplexes

> [Lire en français](README.fr.md)

An n8n workflow that automatically analyzes real estate listing alerts received by email (Centris, DuProprio, Realtor) and generates a daily investment report using AI.

## How it works

1. **Trigger** — Fires every day at 5 PM on a dedicated Gmail label
2. **Extraction** — Parses alert emails from real estate platforms
3. **Historical context** — Fetches the last 30 analyzed properties from Google Sheets
4. **AI analysis** — Sends data to Claude (Anthropic), which computes financial metrics and assigns a score out of 10
5. **Storage** — Saves each analyzed property to Google Sheets
6. **Report** — Sends an HTML email summary only if at least one property scores ≥ 7/10

## Target investment criteria

| Parameter | Value |
|---|---|
| Budget | $700,000 – $1,000,000 |
| Property types | Triplex, Quadruplex, Quintuplex |
| Area | Island of Montreal |
| Down payment | 10% (owner-occupant) |
| Stress-test rate | 5.75%, 25-year amortization |

## Metrics computed by the AI

- **GRM** (Gross Rent Multiplier) — Price / gross annual rents. Threshold: < 14 excellent, 14–17 acceptable
- **Cap rate** — NOI / price (assuming 35% expenses)
- **DSCR** — Debt service coverage ratio. Threshold: > 1.0
- **Net cost to live** — Mortgage payment + taxes − rental income from other units

## Tech stack

- **[n8n](https://n8n.io)** — Workflow orchestration (self-hosted via Docker)
- **Gmail** — Alert source + report delivery
- **Claude API** (Anthropic) — Property analysis and scoring
- **Google Sheets** — History tracker for analyzed properties

## Prerequisites

- Docker & Docker Compose
- n8n account (self-hosted or cloud)
- Anthropic API key
- Gmail account with OAuth2 configured in n8n
- Google Sheets account with OAuth2 configured in n8n

## Setup

### 1. Clone the repo

```bash
git clone https://github.com/Danforthhh/n8n-immobilier.git
cd n8n-immobilier
```

### 2. Configure environment variables

```bash
cp .env.txt .env
```

Fill in `.env`:

```env
N8N_ENCRYPTION_KEY=a_long_random_key
```

### 3. Start n8n

```bash
docker-compose up -d
```

n8n will be available at [http://localhost:5678](http://localhost:5678).

### 4. Import the workflow

1. In n8n → **Workflows** → **Import from file**
2. Select `workflows/veille-immobiliere.json.json`
3. Reconnect your credentials (Gmail, Google Sheets, Anthropic)

### 5. Configure personal values

Refer to `config.example.json` for the parameters to adapt:

| Parameter | Where to update in n8n |
|---|---|
| Recipient emails | **Send a message** node → `sendTo` field |
| Google Sheets ID | **Get row(s)** and **Append or update** nodes → select your file |
| Gmail label | **Gmail Trigger** node → `labelIds` field |

### 6. Set up Gmail filters

Create a Gmail label (e.g. `Real Estate Alerts`) and configure Gmail filters to automatically route emails from Centris, DuProprio, and Realtor to that label.

## Project structure

```
n8n-immobilier/
├── docker-compose.yml                    # Docker config for n8n
├── workflows/
│   └── veille-immobiliere.json.json      # n8n workflow to import
├── config.example.json                   # Template for values to personalize
├── .env.txt                              # Environment variables template
└── .gitignore
```

## Customization

The prompt sent to Claude is fully editable in the **Code3** node of the workflow. You can adjust:

- Priority neighborhoods
- GRM / DSCR thresholds
- Budget range
- Target property types
- Email report format

## License

MIT
