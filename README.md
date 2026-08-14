[README.md](https://github.com/user-attachments/files/31061463/README.md)
# applyfollow Semantic Job Parser

Job application tracker with AI-powered extraction. Paste a job description, let the AI extract structured fields, then save it into your tracker.

## Features

- **Semantic Job Parser** — Paste a job description → AI extracts company, position, platform, status, follow-up date, and more
- **Dual AI Provider** — Groq (`openai/gpt-oss-120b`) first, OpenRouter (`nvidia/nemotron-3-ultra-550b-a55b:free`) as fallback
- **Fallback Table Parser** — Works without API key using regex-based extraction
- **Entry Tab** — Full job list with preview/editor modes, sorting, search
- **Workspace Tab** — Quick Add form + Semantic Parser + recent entries
- **Follow-up System** — Auto-marks "No Response" when follow-up date passes
- **Travel Maps** — Paste screenshot from clipboard, compressed to WebP
- **Import/Export** — JSON backup and restore
- **Gugu Pet Chatbot** — Penguin assistant for page guidance

## AI Extraction Logic

### Provider Priority

```
1. Groq (openai/gpt-oss-120b) — fast, free tier
2. OpenRouter (nvidia/nemotron-3-ultra-550b-a55b:free) — fallback
3. Local table parser — no API key needed
```

### Extracted Fields

| Field | Description |
|-------|-------------|
| `appliedDate` | Application date (YYYY-MM-DD) |
| `company` | Company name |
| `position` | Job title |
| `platform` | JobStreet, LinkedIn, Indeed, etc. |
| `status` | Applied, No Response, Replied, Interview, etc. |
| `followDate` | Follow-up date (auto-set to +3 working days) |
| `whatsapp` | Questions to ask before interview |
| `followupPm` | Follow-up message template |
| `interviewAsk` | Interview questions to ask |
| `response` | HR response / notes |
| `travelNote` | Commute / travel notes |

### How It Works

1. User pastes job description into textarea
2. API key detection:
   - Has key → Send to Groq first, OpenRouter if Groq fails
   - No key → Use local table parser (regex-based)
3. AI returns JSON with extracted fields
4. Fields auto-fill the Quick Add Job form
5. User reviews and clicks "Add Job"

## Setup

### GitHub Secrets

Add to your repo Settings → Secrets → Actions:

- `GROQ_API_KEY` — Get from [console.groq.com](https://console.groq.com)
- `OPENROUTER_API_KEY` — Get from [openrouter.ai](https://openrouter.ai)

### build.yml

```yaml
name: Build HTML with API Key
on:
  push:
    branches: [main]
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repo
        uses: actions/checkout@v4

      - name: Inject API keys into index.html
        run: |
          sed -i "s|__GROQ_API_KEY__|${{ secrets.GROQ_API_KEY }}|g" index.html
          sed -i "s|__OPENROUTER_API_KEY__|${{ secrets.OPENROUTER_API_KEY }}|g" index.html

      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: .

  deploy:
    needs: build
    runs-on: ubuntu-latest
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

## API Key Handling

- Keys stored in `localStorage` (never sent to server)
- User can paste their own key in the parser toolbar
- Build.yml injects keys at deploy time via `sed`
- Placeholders: `__GROQ_API_KEY__`, `__OPENROUTER_API_KEY__`

## Local Development

Just open `index.html` in a browser. No build step needed.

- Without API key: fallback table parser works
- With API key: paste in the Semantic Parser toolbar

## Tech

- Single HTML file, no dependencies
- localStorage for persistence
- Groq + OpenRouter for AI
- Compressed WebP for travel images
- Gugu pet sprite (optional)

## License

Personal project by ZiJun Looi.
