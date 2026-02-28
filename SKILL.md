---
name: spectra-data-analysis
description: "Use this skill whenever the user wants to analyze data using Spectra via REST API. Triggers include: uploading a CSV or Excel file for analysis, asking questions about their data, requesting charts or visualizations, exploring patterns or trends in a dataset, comparing columns, finding anomalies, or generating statistical summaries. Use this skill BEFORE making any Spectra API calls to ensure the best workflow and output quality."
metadata: {"openclaw":{"emoji":"📊"}}
---

# Spectra Data Analysis Skill (REST API)

This skill guides the agent on how to use the Spectra REST API effectively to deliver accurate, insightful, and well-communicated data analysis results.

Spectra is an AI-powered data analysis platform. The agent makes REST API calls to upload files, query data, retrieve context, and manage files.


## Configuration

### Base URL
```
https://api.spectra.nuelo.ai
```

### Authentication
All API calls (except `/health`) require an API key passed as a Bearer token. The key is stored in OpenClaw config at `~/.openclaw/openclaw.json` under `skills.entries.spectra-data-analysis.env.SPECTRA_API_KEY`.

**Always read the API key from config before making API calls:**

```bash
# Read API key from OpenClaw config
SPECTRA_API_KEY=$(cat ~/.openclaw/openclaw.json | python3 -c "import json,sys; d=json.load(sys.stdin); print(d.get('skills',{}).get('entries',{}).get('spectra-data-analysis',{}).get('env',{}).get('SPECTRA_API_KEY',''))")

# Then use in curl commands
curl -s -X GET "https://api.spectra.nuelo.ai/api/v1/files" \
  -H "Authorization: Bearer $SPECTRA_API_KEY"
```


## Available REST API Endpoints

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/api/v1/health` | GET | Check service health |
| `/api/v1/files` | GET | List all uploaded files |
| `/api/v1/files/upload` | POST | Upload CSV/Excel file |
| `/api/v1/files/{file_id}` | GET | Get file details |
| `/api/v1/files/{file_id}/context` | GET | Get AI-generated data brief |
| `/api/v1/files/{file_id}/suggestions` | GET | Get query suggestions |
| `/api/v1/chat/query` | POST | Run natural language analysis |
| `/api/v1/files/{file_id}` | DELETE | Delete a file |


## Core Workflow

### Step 0 — Ensure API Key is Available

Before making any calls, verify the API key is available. If not, ask the user for their `spe_` API key.

### Step 1 — Identify or Upload the File

#### If the file already exists in Spectra

Call `GET /api/v1/files` to list all files and find the one you need.

#### If the file is new — always confirm before uploading

> ⚠️ **Uploading consumes credits.** Each analysis query costs 1 credit. **Always ask the user to confirm** before proceeding with an upload, and present the alternatives below.

When the user shares a new file, respond with:

> *"Before I upload this to Spectra, I want to flag that analysis queries cost credits. Here are your options — which would you prefer?"*
>
> 1. **Upload now and analyze** — I'll upload via curl directly (no LLM tokens used), then run analysis.
> 2. **Upload only** — I'll upload the file so you can analyze later.
> 3. **Upload manually via web** — You upload at https://app.spectra.nuelo.ai, then come back and I'll run the analysis.

**Important:** When uploading via curl, do NOT load the file into the LLM context. Call curl directly with `-F "file=@path"` — this avoids token usage.

### Step 2 — Get Context Before Querying

Always call `GET /api/v1/files/{file_id}/context` **before** running analysis queries. This provides:
- The AI-generated data brief (column names, types, row counts)
- User-provided context (if any)
- Suggested questions tailored to the dataset

### Step 3 — Run Analysis

Use `POST /api/v1/chat/query` with a clear, specific natural language question. Each query costs 1 credit.

**Request body:**
```json
{
  "query": "What is the distribution of sales by region?",
  "file_ids": ["file-uuid-here"],
  "web_search_enabled": false
}
```

**Response includes:**
- `analysis` — Narrative explanation
- `generated_code` — Python code generated
- `execution_result` — Raw computed results (JSON string)
- `chart_specs` — Plotly chart specification (JSON string, or null)
- `credits_used` — Credits deducted

### Step 4 — Render Charts

If `chart_specs` is non-null, render it as an interactive HTML artifact using Plotly.js via CDN.

### Step 5 — Communicate Results

- Present the `analysis` field as the main narrative
- Show the `execution_result` data table if provided
- **ALWAYS render and present the chart** from `chart_specs` if provided (this is mandatory!)
- For chat platforms (Telegram, Discord, etc.): Convert the chart to an image and send it
- Offer follow-up questions from `follow_up_suggestions` if available
- **Do not add personal interpretation or synthesis** — present only what Spectra returns


## Making API Calls

### Helper Pattern

Use this pattern for all Spectra API calls:

```bash
# Read API key from OpenClaw config
SPECTRA_API_KEY=$(cat ~/.openclaw/openclaw.json | python3 -c "import json,sys; d=json.load(sys.stdin); print(d.get('skills',{}).get('entries',{}).get('spectra-data-analysis',{}).get('env',{}).get('SPECTRA_API_KEY',''))")

# GET request
curl -s -X GET "https://api.spectra.nuelo.ai/api/v1/files" \
  -H "Authorization: Bearer $SPECTRA_API_KEY"

# POST request with JSON body
curl -s -X POST "https://api.spectra.nuelo.ai/api/v1/chat/query" \
  -H "Authorization: Bearer $SPECTRA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"query": "Your question", "file_ids": ["file-id"], "web_search_enabled": false}'

# Upload file
curl -s -X POST "https://api.spectra.nuelo.ai/api/v1/files/upload" \
  -H "Authorization: Bearer $SPECTRA_API_KEY" \
  -F "file=@/path/to/yourfile.csv"
```

### Response Format

**Success:**
```json
{
  "success": true,
  "data": { ... },
  "credits_used": 1.0
}
```

**Error:**
```json
{
  "success": false,
  "error": {
    "code": "FILE_NOT_FOUND",
    "message": "File not found."
  }
}
```


## Common Operations

> ⚠️ **Before running any of these commands**, make sure to read the API key first using the helper pattern above. Add this line before each curl command:
> ```bash
> SPECTRA_API_KEY=$(cat ~/.openclaw/openclaw.json | python3 -c "import json,sys; d=json.load(sys.stdin); print(d.get('skills',{}).get('entries',{}).get('spectra-data-analysis',{}).get('env',{}).get('SPECTRA_API_KEY',''))")
> ```

### 1. Check Health
```bash
curl -s https://api.spectra.nuelo.ai/api/v1/health
```

### 2. List Files
```bash
curl -s -X GET "https://api.spectra.nuelo.ai/api/v1/files" \
  -H "Authorization: Bearer $SPECTRA_API_KEY"
```

### 3. Upload File
```bash
curl -s -X POST "https://api.spectra.nuelo.ai/api/v1/files/upload" \
  -H "Authorization: Bearer $SPECTRA_API_KEY" \
  -F "file=@sales_data.csv"
```

Response returns `file_id` — save this for subsequent calls.

### 4. Get File Context
```bash
curl -s -X GET "https://api.spectra.nuelo.ai/api/v1/files/{file_id}/context" \
  -H "Authorization: Bearer $SPECTRA_API_KEY"
```

Returns: `data_brief`, `user_context`, `query_suggestions`.

### 5. Run Analysis Query
```bash
curl -s -X POST "https://api.spectra.nuelo.ai/api/v1/chat/query" \
  -H "Authorization: Bearer $SPECTRA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "query": "What is total revenue by region?",
    "file_ids": ["3fa85f64-5717-4562-b3fc-2c963f66afa6"],
    "web_search_enabled": false
  }'
```

### 6. Delete File
```bash
curl -s -X DELETE "https://api.spectra.nuelo.ai/api/v1/files/{file_id}" \
  -H "Authorization: Bearer $SPECTRA_API_KEY"
```

**Always confirm with user before deleting.**


## Chart Rendering & Delivery

### Key Instructions

1. **Always present the chart** — If Spectra returns a non-null `chart_specs`, you MUST render and present it to the requester. Never skip charts.

2. **Make it sleek and presentable** — When rendering charts:
   - Add a clear, descriptive title in the chart layout
   - Include a brief description/analysis result from Spectra (from the `analysis` field) as a caption or subtitle
   - Use clean styling: white/transparent background, readable fonts, appropriate colors
   - Ensure axes have clear labels

3. **For chat platforms (Telegram, Discord, etc.) — convert to image** — If the user is on a messaging platform (Telegram, Discord, WhatsApp, Signal, etc.):
   - Navigate to the rendered chart in a browser
   - Take a screenshot
   - Send the image via the messaging tool with a brief caption explaining what the chart shows

### Rendering Charts

If `chart_specs` is returned, parse the JSON string and render using Plotly.js in an HTML artifact:

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8"/>
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <script src="https://cdn.plot.ly/plotly-2.35.2.min.js"></script>
  <style>
    body { font-family: Inter, sans-serif; background: transparent; margin: 0; padding: 16px; }
  </style>
</head>
<body>
  <div id="chart"></div>
  <script>
    // Paste figure.data and figure.layout from chart_specs
    const data = [/* figure.data array */];
    const layout = { /* figure.layout object */, paper_bgcolor: "rgba(0,0,0,0)", plot_bgcolor: "rgba(0,0,0,0)" };
    Plotly.newPlot("chart", data, layout, { responsive: true, displayModeBar: false });
  </script>
</body>
</html>
```


### Converting Charts to Images (for Telegram, Discord, etc.)

To send a chart as an image:

1. Start a local HTTP server in the workspace directory:
   ```bash
   cd ~/.openclaw/workspace && python3 -m http.server 8888 &
   ```

2. Navigate to the chart in the OpenClaw browser (profile="openclaw"):
   ```
   http://localhost:8888/your-chart-file.html
   ```

3. Take a screenshot using the browser tool

4. Send the image via the message tool:
   ```json
   {
     "action": "send",
     "channel": "telegram",  // or discord, whatsapp, etc.
     "target": "telegram:7581487482",
     "media": "/path/to/screenshot.png",
     "caption": "Your chart description"
   }
   ```

5. Stop the browser and HTTP server when done

**Note:** When taking the screenshot, ensure the chart is fully loaded and visible. Adjust the viewport if needed.


## Transparency — Always Label the Source

When presenting results, be clear about what comes from Spectra vs. any additions you make:

- **Spectra-sourced content:** The `analysis` narrative, data table, chart, and `generated_code` — present these as-is
- **Your additions:** Only add minimal formatting (tables, emojis for readability). Do NOT add insights, synthesis, or conclusions beyond what Spectra provides

**File Upload Best Practice:** When uploading files to Spectra, always use curl directly — never load the file into the LLM context. This avoids unnecessary token usage:
```bash
# Read API key from config
SPECTRA_API_KEY=$(cat ~/.openclaw/openclaw.json | python3 -c "import json,sys; d=json.load(sys.stdin); print(d.get('skills',{}).get('entries',{}).get('spectra-data-analysis',{}).get('env',{}).get('SPECTRA_API_KEY',''))")

# Upload file
curl -X POST "https://api.spectra.nuelo.ai/api/v1/files/upload" \
  -H "Authorization: Bearer $SPECTRA_API_KEY" \
  -F "file=@/path/to/file.csv"
```

If you must add something, clearly label it, for example: > *"The analysis above is from Spectra. The following is my summary:"*

**Never present your own interpretation as if it were Spectra's output.**


## Error Handling

| Error Code | Meaning | What to Do |
|------------|---------|------------|
| `UNAUTHORIZED` (401) | Invalid API key | Ask for valid key |
| `FILE_NOT_FOUND` (404) | File doesn't exist | List files to find correct ID |
| `INSUFFICIENT_CREDITS` (402) | No credits left | Ask user to top up |
| `INVALID_FILE_TYPE` (400) | Wrong format | Use .csv, .xlsx, or .xls |
| `FILE_TOO_LARGE` (400) | File > 50MB | Ask user to split file |


## Example Interaction Flow

**User:** Here's my sales data. Can you analyze it?

**Agent should:**
1. Ask for API key if not available
2. Confirm: *"This will use credits. Upload now?"*
3. Upload file via `POST /api/v1/files/upload`
4. Get context via `GET /api/v1/files/{id}/context`
5. Run targeted analysis queries
6. Present findings with charts if applicable
7. Suggest follow-up questions