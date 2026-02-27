# Spectra OpenClaw Skill

An [OpenClaw](https://openclaw.ai) agent skill that connects your AI assistant to [Spectra](https://nuelo.ai) — Nuelo's AI-powered data analytics platform.

## What It Does

Enables OpenClaw to interact with the Spectra REST API to:

- **Upload files** — CSV, Excel, or JSON datasets (via direct curl, no token waste)
- **Analyze data** — Ask natural language questions about your data
- **Generate charts** — Get Plotly visualizations rendered as interactive HTML
- **Manage files** — List, retrieve context, and delete uploaded datasets

## Requirements

- An active [Spectra](https://app.spectra.nuelo.ai) account
- A Spectra API key (`spe_...`)
- [OpenClaw](https://openclaw.ai) installed and configured

## Installation

1. Copy the `spectra/` folder into your OpenClaw workspace skills directory:
   ```
   ~/.openclaw/workspace/skills/spectra/
   ```

2. Add your API key to `~/.openclaw/openclaw.json`:
   ```json
   {
     "skills": {
       "entries": {
         "spectra-data-analysis": {
           "enabled": true,
           "env": {
             "SPECTRA_API_KEY": "your_key_here"
           }
         }
       }
     }
   }
   ```

3. Start a new OpenClaw session — the skill will be auto-detected.

## Usage

Just talk to your OpenClaw assistant naturally:

> "Upload this sales CSV and tell me which region had the highest revenue last quarter."

> "Show me a chart of monthly signups."

> "What are the top 5 products by total orders?"

The skill handles the API calls, credit usage warnings, and chart rendering automatically.

## API Reference

Base URL: `https://api.spectra.nuelo.ai`

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/api/v1/files` | GET | List uploaded files |
| `/api/v1/files/upload` | POST | Upload a dataset |
| `/api/v1/files/{id}/context` | GET | Get AI-generated data brief |
| `/api/v1/chat/query` | POST | Run a natural language query |
| `/api/v1/files/{id}` | DELETE | Delete a file |

## Notes

- Each analysis query costs **1 credit**
- Max file size: **50MB**
- Supported formats: `.csv`, `.xlsx`, `.xls`

---

Built by [Nuelo](https://nuelo.ai) · [GitHub](https://github.com/nuelo-ai/spectra-openclaw-skills)
