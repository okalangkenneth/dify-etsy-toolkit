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
- Anthropic plugin installed (v0.3.6)
- Phase 2: Frontend components written (EtsyForm, EtsyOutput, useEtsyToolkit, EtsyToolkit)
- Phase 2: Etsy Seller Toolkit workflow DSL built, imported and PUBLISHED in Dify
- Phase 2: Workflow tested end-to-end successfully (title, 13 tags, description, validation)
- Phase 3: Python API route (etsy_toolkit.py) written and mounted via docker-compose.override.yml
- Phase 3: CSRF token auth wired and verified working
- Phase 3: Full stack tested - browser to Flask to Dify workflow to Claude to response
- Phase 3: Standalone HTML demo page (etsy-toolkit-demo.html) served at localhost/etsy-toolkit.html
- Phase 3: README.md written with architecture, demo output, setup instructions

### IN PROGRESS

### REMAINING
- Record Loom demo (2 min walkthrough)
- Write LinkedIn post
- Pin repo on GitHub profile
- Deploy to Railway or Render (optional - for public URL)
- Phase 4: dify/web/.env.local created with correct NEXT_PUBLIC_* vars pointing to http://localhost
- Phase 4: bun install completed in dify/web/ (1024 packages)
- Phase 4: Added 'use client' directive to all 4 custom components (EtsyForm, EtsyOutput, EtsyToolkit, useEtsyToolkit) — required by Next.js App Router
- Phase 4: Next.js 16.2.0 (Turbopack) dev server running at http://localhost:3000
- Phase 4: /etsy-toolkit route compiled and returned HTTP 200 ✅
  - Note: First Turbopack compile on Windows E:\ drive takes ~27 min (slow filesystem warning expected); subsequent compiles are fast

### IN PROGRESS

### REMAINING
- Take screenshot of http://localhost:3000/etsy-toolkit for README images folder
- Record Loom demo (2 min walkthrough)
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
