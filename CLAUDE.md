# Project Rules - dify-etsy-toolkit

## Project Overview

Building an **Etsy Seller Toolkit** feature on top of Dify (github.com/langgenius/dify),
a 90k-star open-source LLM platform. Sellers enter a product idea and receive:
- SEO-optimized Etsy title (3-14 words)
- 13 tags (max 20 chars each)
- Buyer-friendly product description (150-500 words)

**GitHub**: https://github.com/okalangkenneth/dify-etsy-toolkit

---

## Build Progress (Claude Code: update every session)

### COMPLETED
- Project folder and CLAUDE.md created
- Cloned Dify into dify/ as git submodule
- Docker setup - all 11 containers running at http://localhost
- Anthropic plugin installed (v0.3.6) with claude-haiku-4-5-20251001
- Phase 2: Frontend components written (EtsyForm, EtsyOutput, useEtsyToolkit, EtsyToolkit)
- Phase 2: Etsy Seller Toolkit workflow DSL built and imported into Dify
- Phase 2: Workflow PUBLISHED and tested end-to-end successfully
  - Input: product idea (paragraph)
  - LLM node: claude-opus-4 generating JSON title/tags/description
  - Code node: Python validation of Etsy constraints
  - Output: title, tags_display, description, is_valid, validation_summary
- Cloned Dify (shallow) into `dify/` subdirectory
- Configured Docker: copied `dify/docker/.env.example` → `dify/docker/.env`
- Ran `docker compose up -d` from `dify/docker/` — all 11 containers running
  - nginx, web, api, worker, worker_beat, plugin_daemon, db_postgres, redis, weaviate, sandbox, ssrf_proxy
  - Dify UI accessible at http://localhost (redirects to /apps setup page)
- Note: First run requires ~1hr on Windows/Docker Desktop due to large image extraction
- Converted `dify/` from nested git repo to proper git submodule pointing to https://github.com/langgenius/dify (commit ce37059)
- Phase 2: Added Etsy Toolkit frontend components (page.tsx, EtsyForm, EtsyOutput, useEtsyToolkit)
- Phase 3: Added ETSY_TOOLKIT_WORKFLOW_KEY env var to docker-compose.yaml (api service) and dify/docker/.env
  - Workflow API key: app-GDmvpcyfckZlU5m3hx1ZfmKR
  - api container restarted cleanly — migrations ran, gunicorn listening at :5001

### IN PROGRESS

### REMAINING
- Build API route: api/controllers/console/etsy_toolkit.py
- Connect frontend panel to the published workflow via API
- Write validation tests (pytest + vitest)
- Deploy to Railway or Render
- Write README with screenshots and Loom demo link
- Open upstream PR to Dify

---

## Tech Stack

**Dify core (do not change):**
- Backend: Python 3.12 + FastAPI
- Frontend: Next.js 14 + TypeScript
- Database: PostgreSQL + Redis
- Container: Docker Compose

**Our additions:**
- web/app/etsy-toolkit/page.tsx  (new UI panel)
- api/controllers/console/etsy_toolkit.py  (new API route)
- dify-workflows/etsy-seller-toolkit.yml  (workflow DSL)

---

## Etsy Business Rules (NON-NEGOTIABLE)

```
Title:  3-14 words exactly
Tags:   Exactly 13 tags, max 20 characters each (including spaces), no special chars
Desc:   150-500 words, conversational tone, no keyword stuffing
```

Required validators (100% test coverage):
- validateEtsyTitle(title)
- validateEtsyTags(tags)
- formatTagsForDisplay(tags)

---

## Development Workflow

1. Make changes
2. cd web && bun run type-check
3. cd api && python -m mypy .
4. bun run test  /  pytest
5. bun run lint  /  black api/
6. Verify in browser (docker compose up -d)
7. Commit with conventional commits

Docker commands:
```bash
docker compose up -d           # start everything
docker compose logs -f api     # tail API logs
docker compose restart api     # restart after code change
```

---

## Git Conventions

- **Branches**: main = production | feat/name for features
- **Commits**: feat: fix: chore: docs: refactor: test:
- **Never commit**: .env, .env.local, API keys, secrets

**First-time setup (run once):**
```bash
git init
git add .
git commit -m "feat: initial scaffold with CLAUDE.md"
git remote add origin https://github.com/okalangkenneth/dify-etsy-toolkit.git
git branch -M main
git push -u origin main
```

**After every phase:**
```bash
git add . && git commit -m "feat: Phase X - description" && git push
```

---

## Target File Structure

```
dify-etsy-toolkit/
  CLAUDE.md
  MEMORY.md
  docker-compose.yaml
  dify-workflows/
    etsy-seller-toolkit.yml
  api/controllers/console/
    etsy_toolkit.py
  web/app/(commonLayout)/etsy-toolkit/
    page.tsx
    components/
      EtsyForm.tsx
      EtsyOutput.tsx
      useEtsyToolkit.ts
  tests/
    test_etsy_validation.py
    etsy-validation.test.ts
```

---

## Code Quality Rules

- No placeholders (no YOUR_API_KEY, TODO, FIXME)
- Env variables for all secrets (.env.local)
- Remove unused imports
- Log all API calls and errors
- Only modify what was explicitly requested
- Ask if under 90% confident on approach

---

## Corrections Log

| Date | Mistake | Rule |
|------|---------|------|

---

## Session End Prompt

Update the Build Progress section in CLAUDE.md with completed work, then stop.
