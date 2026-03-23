# Etsy Seller Toolkit

> An AI-powered Etsy listing generator built as a feature extension on top of [Dify](https://github.com/langgenius/dify) — a 90k-star open-source LLM platform.

![Etsy Seller Toolkit Demo](./images/demo-screenshot.png)

## What it does

Paste in a product idea. Get back a complete, SEO-optimized Etsy listing in seconds:

- **Title** — 3–14 words, highest-value keywords first, naturally phrased
- **13 Tags** — each ≤20 characters, a mix of broad and specific buyer search terms
- **Description** — 150–500 words, warm conversational tone, buyer-first opening, soft call to action

All outputs are automatically validated against Etsy's listing constraints before being returned.

## Demo

**Input:**
```
A printable weekly planner for busy moms who want to stay organized
```

**Output:**
```json
{
  "title": "Weekly Planner Printable for Busy Moms Organization PDF Download",
  "tags": [
    "weekly planner", "mom planner", "printable planner",
    "busy mom organizer", "weekly schedule", "mom organization",
    "planner pdf", "digital planner", "family organizer",
    "week at a glance", "mom life planner", "planning printable",
    "instant download"
  ],
  "is_valid": true,
  "validation_summary": "All checks passed"
}
```

## Architecture

```
Browser (Next.js UI)
    ↓
Flask API Route  (/console/api/etsy-toolkit/generate)
    ↓
Dify Workflow Engine
    ↓  Start → LLM Node → Code Node → End
Claude (claude-haiku-4-5-20251001)
    ↓
Python validation (title word count, tag length, special chars)
    ↓
JSON response
```

## Tech Stack

| Layer | Technology |
|---|---|
| Platform | [Dify](https://github.com/langgenius/dify) v1.13.2 |
| LLM | Anthropic Claude (via Dify Anthropic plugin) |
| Backend | Python 3.12 + Flask (Dify's API service) |
| Frontend | Next.js 14 + TypeScript |
| Infrastructure | Docker Compose (11 services) |
| Workflow | Dify DSL (YAML) with LLM + Code nodes |

## What I built on top of Dify

This project extends Dify with three original additions:

**1. Etsy Seller Toolkit Workflow** (`dify-workflows/etsy-seller-toolkit.yml`)
A 4-node Dify workflow: Start → LLM (Claude with a custom Etsy SEO prompt) → Code (Python validation) → End. Importable as a DSL file into any Dify instance.

**2. Python API Route** (`dify/api/controllers/console/etsy_toolkit.py`)
A new Flask-RESTx endpoint that accepts a product idea, calls the published Dify workflow via its internal API, validates the output, and returns structured JSON. Mounted into the running Docker container via `docker-compose.override.yml`.

**3. Next.js Frontend Panel** (`dify/web/app/(commonLayout)/etsy-toolkit/`)
A new page inside Dify's console UI with a form component, output display, per-field copy buttons, real-time Etsy constraint validation (title word count, tag character limits), and error state handling.

## Why I built this

I run an Etsy digital product business and know the SEO constraints cold — titles must be 3–14 words, tags max 20 characters each, descriptions need to be buyer-first, not keyword-stuffed. Most AI tools ignore these rules. This one enforces them automatically.

The project demonstrates:
- Contributing a feature to a large open-source codebase (90k stars)
- Full-stack development across Python and TypeScript
- LLM workflow design with validation logic
- Docker-based development with service overrides
- Domain knowledge applied as product constraints

## Setup

### Prerequisites
- Docker Desktop
- An Anthropic API key

### Run locally

```bash
git clone https://github.com/okalangkenneth/dify-etsy-toolkit
cd dify-etsy-toolkit

# Start Dify
cd dify/docker
cp .env.example .env
# Add your ETSY_TOOLKIT_WORKFLOW_KEY to .env
docker compose up -d
```

Then open `http://localhost` and complete the Dify setup wizard.

**Import the workflow:**
1. Go to Studio → Import DSL file
2. Select `dify-workflows/etsy-seller-toolkit.yml`
3. Configure your Anthropic API key in Settings → Model Provider
4. Publish the workflow and copy its API key to `.env`

## Project Structure

```
dify-etsy-toolkit/
├── CLAUDE.md                          # AI session rules and build progress
├── README.md                          # This file
├── dify-workflows/
│   └── etsy-seller-toolkit.yml        # Dify workflow DSL (importable)
└── dify/
    ├── docker/
    │   └── docker-compose.override.yml  # Mounts our files + env vars
    ├── api/controllers/console/
    │   ├── etsy_toolkit.py              # New Flask API route
    │   └── __init__.py                  # Route registration
    └── web/app/(commonLayout)/etsy-toolkit/
        ├── page.tsx
        └── components/
            ├── EtsyToolkit.tsx
            ├── EtsyForm.tsx
            ├── EtsyOutput.tsx
            └── useEtsyToolkit.ts
```

## Contributing

This is a portfolio project. For contributions to Dify itself, see
[langgenius/dify](https://github.com/langgenius/dify/blob/main/CONTRIBUTING.md).

## License

Built on top of [Dify](https://github.com/langgenius/dify) which is licensed under the
Apache 2.0 License. Our additions follow the same license.
