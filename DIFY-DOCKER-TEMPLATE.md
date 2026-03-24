# Dify + Docker Project Template

## What this is

A battle-tested starting point for building AI products (chatbots, workflows, agents)
on top of Dify + Docker on Windows. Based on real lessons from the dify-etsy-toolkit project.

Use this before starting any new Dify-based project to avoid the hours of debugging we encountered.

---

## Pre-flight checklist (do BEFORE touching any code)

- [ ] Docker Desktop installed and showing green "Engine running"
- [ ] At least 8GB RAM allocated to Docker (Settings → Resources → Memory)
- [ ] Project folder on **C: drive** — never E: or any external/USB drive
- [ ] Windows Defender exclusion added for `C:\Projects` (prevents slow filesystem)
- [ ] DNS configured in Docker Engine JSON (prevents plugin install failures)

### Docker Engine DNS config (Settings → Docker Engine)
```json
{
  "builder": {
    "gc": { "defaultKeepStorage": "20GB", "enabled": true }
  },
  "experimental": false,
  "dns": ["8.8.8.8", "8.8.4.4"]
}
```
Click **Apply & restart** after changing this.

---

## Project setup

### 1. Clone Dify as a git submodule (not a plain clone)
```bash
git init my-project
cd my-project
git submodule add https://github.com/langgenius/dify dify
git submodule update --init --recursive
```
Why: A plain `git clone` of Dify inside your project creates a nested repo that
breaks `git add .` and confuses GitHub. A submodule handles this correctly.

### 2. Configure Docker environment
```bash
cd dify/docker
cp .env.example .env
```
Edit `.env` — the defaults work for local dev. Add your secrets at the bottom:
```
# Your additions go here — never edit above this line
ETSY_TOOLKIT_WORKFLOW_KEY=your_key_here
```

### 3. Start Dify
```bash
cd dify/docker
docker compose up -d
```
Wait for all 11 containers to be healthy. First start takes 3-5 minutes.
Check status: `docker ps --format "{{.Names}} {{.Status}}"`

### 4. Complete Dify setup
Open `http://localhost` → create admin account → done.

---

## Installing model plugins (Anthropic, OpenAI etc.)

### The problem
The plugin daemon downloads Python packages from PyPI inside Docker.
On Windows with default settings, DNS resolution fails and the download times out.

### The fix (already done if you followed the DNS step above)
The `8.8.8.8` DNS in Docker Engine config fixes this. But if a plugin still fails:

1. **Stop the daemon** (this stops the crash loop):
```bash
docker stop docker-plugin_daemon-1
```

2. **Check if packages are already downloaded** (they often are despite the "failed" status):
```bash
dir "dify\docker\volumes\plugin_daemon\cwd\langgenius"
```
If you see a folder with a `.venv` inside it, the packages are there.

3. **Clear failed tasks from the database:**
```bash
docker exec docker-db_postgres-1 psql -U postgres -d dify_plugin -c "DELETE FROM install_tasks;"
```

4. **Restart the daemon clean:**
```bash
cd dify/docker
docker compose up -d plugin_daemon
```
It will find the existing `.venv` and launch without re-downloading.

### Never click "install" multiple times
Each click creates a new `install_task` row. Multiple failed tasks run simultaneously
and compete, making things worse. Click once, wait 5 minutes, then debug.

---

## Adding custom code to Dify without modifying the image

Dify runs from pre-built Docker images. Your custom files don't exist inside them.
Use `docker-compose.override.yml` to mount your files in — Docker auto-merges this file.

### Create `dify/docker/docker-compose.override.yml`:
```yaml
services:
  api:
    environment:
      MY_SECRET_KEY: ${MY_SECRET_KEY:-}
    volumes:
      - ../api/controllers/console/my_feature.py:/app/api/controllers/console/my_feature.py:ro
      - ../api/controllers/console/__init__.py:/app/api/controllers/console/__init__.py:ro
  worker:
    environment:
      MY_SECRET_KEY: ${MY_SECRET_KEY:-}
```

### Get the real `__init__.py` from the container first:
```bash
docker cp docker-api-1:/app/api/controllers/console/__init__.py dify/api/controllers/console/__init__.py
```
Then add your import to it, and mount it back via the override.

### Recreate the api container to pick up the mounts:
```bash
cd dify/docker
docker compose up -d --no-deps --force-recreate api
```

### Verify your files are inside:
```bash
docker exec docker-api-1 ls /app/api/controllers/console/my_feature.py
docker exec docker-api-1 grep -n "my_feature" /app/api/controllers/console/__init__.py
```

---

## Writing a Flask API route for Dify

### Template route file
```python
import logging
import os
import requests
from flask import request
from flask_restx import Resource
from pydantic import BaseModel, Field
from controllers.console import console_ns
from controllers.console.wraps import account_initialization_required, setup_required
from libs.login import login_required

logger = logging.getLogger(__name__)

DIFY_WORKFLOW_KEY = os.environ.get("MY_WORKFLOW_KEY", "")

class MyPayload(BaseModel):
    user_input: str = Field(..., min_length=1, max_length=500)

@console_ns.route("/my-feature/generate")
class MyFeatureApi(Resource):
    @setup_required
    @login_required
    @account_initialization_required
    def post(self):
        try:
            payload = MyPayload.model_validate(request.json or {})
        except Exception as e:
            return {"message": f"Invalid input: {e}"}, 400

        if not DIFY_WORKFLOW_KEY:
            return {"message": "Feature not configured."}, 503

        try:
            resp = requests.post(
                "http://api:5001/v1/workflows/run",
                headers={"Authorization": f"Bearer {DIFY_WORKFLOW_KEY}"},
                json={"inputs": {"user_input": payload.user_input},
                      "response_mode": "blocking", "user": "my-feature"},
                timeout=60,
            )
            resp.raise_for_status()
        except requests.Timeout:
            return {"message": "Workflow timed out."}, 504
        except requests.RequestException as e:
            return {"message": "Workflow unreachable."}, 502

        outputs = resp.json().get("data", {}).get("outputs", {})
        return outputs, 200
```

### CSRF auth — required for all console API calls from the browser
The Dify console API requires a CSRF token. Get it from the cookie:
```javascript
const csrf = document.cookie.split('; ')
  .find(r => r.startsWith('csrf_token='))?.split('=')[1] ?? ''

fetch('/console/api/my-feature/generate', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json', 'X-CSRF-Token': csrf },
  credentials: 'include',
  body: JSON.stringify({ user_input: 'hello' })
})
```

---

## Building a Dify workflow DSL

The fastest way to build a workflow is in the Dify UI first, then export it.
To import from a YAML file: Studio → Import DSL file.

### Minimal working workflow structure:
```yaml
app:
  description: 'What this workflow does'
  icon: 🤖
  icon_background: '#FFEAD5'
  mode: workflow
  name: My Workflow
dependencies: []
kind: app
version: 0.3.1
workflow:
  conversation_variables: []
  environment_variables: []
  features:
    file_upload: {enabled: false}
    speech_to_text: {enabled: false}
    text_to_speech: {enabled: false}
  graph:
    edges:
    - {id: e1, source: start_node, sourceHandle: source,
       target: llm_node, targetHandle: target, type: custom,
       data: {isInIteration: false, isInLoop: false, sourceType: start, targetType: llm}}
    - {id: e2, source: llm_node, sourceHandle: source,
       target: end_node, targetHandle: target, type: custom,
       data: {isInIteration: false, isInLoop: false, sourceType: llm, targetType: end}}
    nodes:
    - data:
        type: start
        title: Start
        variables:
        - {label: Input, variable: user_input, type: paragraph, required: true, max_length: 500}
      id: start_node
      position: {x: 30, y: 200}
      type: custom
      width: 244
      height: 116
    - data:
        type: llm
        title: LLM
        model:
          provider: anthropic
          name: claude-haiku-4-5-20251001
          mode: chat
          completion_params: {temperature: 0.7, max_tokens: 1500}
        prompt_template:
        - role: system
          text: "You are a helpful assistant. Respond concisely."
        - role: user
          text: '{{#start_node.user_input#}}'
        vision: {enabled: false}
        memory: {enabled: false}
        context: {enabled: false}
        structured_output: {enabled: false}
        retry_config: {enabled: true, max_retries: 2, retry_interval: 1000}
      id: llm_node
      position: {x: 334, y: 200}
      type: custom
      width: 244
      height: 122
    - data:
        type: end
        title: End
        outputs:
        - {variable: result, value_selector: [llm_node, text], value_type: string}
      id: end_node
      position: {x: 638, y: 200}
      type: custom
      width: 244
      height: 122
    viewport: {x: 0, y: 0, zoom: 0.75}
```

### Code node (for validation or data transformation):
```yaml
    - data:
        type: code
        title: Validate Output
        code_language: python3
        code: |
          import json, re
          def main(llm_output: str) -> dict:
              clean = re.sub(r'```(?:json)?|```', '', llm_output).strip()
              try:
                  data = json.loads(clean)
              except Exception as e:
                  return {"result": "", "is_valid": "false", "error": str(e)}
              return {"result": data.get("result", ""), "is_valid": "true", "error": ""}
        variables:
        - {variable: llm_output, value_selector: [llm_node, text], label: llm_output}
        outputs:
          result: {type: string, children: null}
          is_valid: {type: string, children: null}
          error: {type: string, children: null}
      id: code_node
      position: {x: 638, y: 200}
      type: custom
      width: 244
      height: 122
```

### Key DSL rules learned the hard way:
- `outputs` in code node must be a **dict** with `{type, children: null}` — not a list
- `outputs` in end node must be a **list** with `{variable, value_selector, value_type}`
- `variables` in code node is a **list** with `{variable, value_selector, label}`
- Output types: use `string` not `array[string]` — boolean outputs should return `"true"`/`"false"` as strings
- Model name must match exactly what's in your plugin (check Plugins page for exact string)
- Always include `retry_config` in LLM nodes to handle transient API errors

---

## Docker troubleshooting reference

### Postgres crash recovery (happens after unclean Docker shutdown)
**Symptom:** Dify loads forever, `docker ps` shows `db_postgres` as unhealthy,
`plugin_daemon` is restarting in a loop.

**Cause:** Postgres is replaying WAL logs from the unclean shutdown. Takes 5-10 minutes.

**Fix:** Just wait. Watch the logs:
```bash
docker logs docker-db_postgres-1 --tail 5
```
When you see `database system is ready to accept connections` — it's done.
Then restart the plugin daemon:
```bash
docker compose restart plugin_daemon
```

### Plugin daemon crash loop
**Symptom:** `docker ps` shows `plugin_daemon` as `Restarting (1) X seconds ago`

**Most common cause:** Postgres isn't ready yet (see above). Wait for Postgres first.

**Second cause:** Failed install tasks blocking startup. Fix:
```bash
docker stop docker-plugin_daemon-1
docker exec docker-db_postgres-1 psql -U postgres -d dify_plugin -c "DELETE FROM install_tasks;"
docker compose up -d plugin_daemon
```

### API container not picking up your code changes
After editing mounted files, force recreate (not just restart):
```bash
docker compose up -d --no-deps --force-recreate api
```
Then check the logs for import errors:
```bash
docker logs docker-api-1 --tail 20
```

### Testing an API route from inside the container
```bash
docker exec docker-api-1 curl -s http://localhost:5001/console/api/ping
```
If this hangs, the gunicorn server hasn't finished starting yet. Wait 30 seconds.

### Checking if an env var reached the container
```bash
docker exec docker-api-1 env | findstr MY_KEY
```

---

## Frontend: skip the Next.js dev server on Windows

The Dify web container runs a pre-built Next.js production bundle.
Running `bun run dev` to develop custom pages has serious issues on Windows:
- Turbopack freezes on Windows filesystems (even C: drive with antivirus)
- First compile of the full Dify codebase takes 30+ minutes
- Desktop Commander and other tools lose connection during the long compile

### The solution: serve a standalone HTML file via nginx

For demos, prototypes, and internal tools — a vanilla HTML file is faster,
more reliable, and frankly looks just as good.

1. Create your HTML file with the UI and JS fetch calls
2. Copy it into the nginx container:
```bash
docker cp my-feature.html docker-nginx-1:/usr/share/nginx/html/my-feature.html
```
3. Add a location block to the nginx config inside the container:
```bash
docker exec docker-nginx-1 sh -c "sed -i 's|location /explore {|location /my-feature.html {\n      root /usr/share/nginx/html;\n    }\n\n    location /explore {|' /etc/nginx/conf.d/default.conf"
docker exec docker-nginx-1 nginx -t && docker exec docker-nginx-1 nginx -s reload
```
4. Open `http://localhost/my-feature.html` — instant, no compilation

If you genuinely need the Next.js dev server:
- Disable Turbopack: remove the `turbopack:` block from `next.config.ts`
- Add `C:\Projects` to Windows Defender exclusions
- Navigate directly to your route URL — don't go to `localhost:3000` which triggers the full app compile

---

## Git conventions for Dify projects

```bash
# Correct: submodule
git submodule add https://github.com/langgenius/dify dify

# If you already did a plain clone by mistake:
git rm --cached dify
git submodule add https://github.com/langgenius/dify dify

# Never commit the Docker .env (has secrets):
echo "dify/docker/.env" >> .gitignore
echo "dify/docker/.env.local" >> .gitignore

# Commit after every phase:
git add . && git commit -m "feat: Phase X - description" && git push
```

---

## CLAUDE.md template for new Dify projects

Copy this to the root of every new Dify project:

```markdown
# Project Rules

## What we are building
[One paragraph describing the feature being added to Dify]

GitHub: https://github.com/okalangkenneth/[repo-name]

## Build Progress (Claude Code: update every session)

### COMPLETED

### IN PROGRESS

### REMAINING
- [ ] Set up Docker + Dify locally
- [ ] Install model provider plugin (Anthropic)
- [ ] Build workflow DSL
- [ ] Write API route
- [ ] Build frontend
- [ ] Test end-to-end
- [ ] Write README
- [ ] Record Loom demo

## Tech Stack
- Backend: Python 3.12 + Flask (Dify API service)
- Frontend: Vanilla HTML or Next.js
- Container: Docker Compose (dify/docker/)
- Workflow: Dify DSL in dify-workflows/

## Business Rules (NON-NEGOTIABLE)
[List your domain-specific validation rules here]

## Session End Prompt
Update the Build Progress section in CLAUDE.md with completed work, then stop.
```

---

## Quick reference: useful Docker commands

```bash
# Check all containers
docker ps --format "{{.Names}} {{.Status}}"

# Tail logs for a service
docker logs docker-api-1 --tail 30 -f

# Run a command inside a container
docker exec docker-api-1 env | findstr MY_KEY

# Copy file into container
docker cp myfile.py docker-api-1:/app/api/controllers/console/myfile.py

# Copy file out of container
docker cp docker-api-1:/app/api/controllers/console/__init__.py ./local-copy.py

# Restart one service without affecting others
docker compose up -d --no-deps --force-recreate api

# Full restart of everything
docker compose down && docker compose up -d

# Check if a route exists
docker exec docker-api-1 curl -s http://localhost:5001/console/api/ping
```
