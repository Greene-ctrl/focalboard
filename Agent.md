## 1. Deployment Configuration

### Target Space
- **Profile:** `Leon4gr45`
- **Space:** `kanban`
- **Full Identifier:** `Leon4gr45/kanban`
- **Frontend Port:** `7860` (mandatory for all Hugging Face Spaces)

### Deployment Method
Choose the correct SDK based on the app type based on the codebase language:

- **Gradio SDK** — for Gradio applications
- **Streamlit SDK** — for Streamlit applications
- **Docker SDK** — for all other applications (recommended default for flexibility)

### HF Token
- The environment variable **`HF_TOKEN` will always be provided at execution time**.
- Never hardcode the token. Always read it from the environment.
- All monitoring and log‑streaming commands rely on `$HF_TOKEN`.

### Required Files
- `Dockerfile` (or `app.py` for Gradio/Streamlit SDKs)
- `README.md` with Hugging Face YAML frontmatter:
  ```yaml
  ---
  title: <APP NAME>
  sdk: docker | gradio | streamlit
  app_port: 7860
  ---
  ```
- `.hfignore` to exclude unnecessary files
- This `Agent.md` file (must be committed before deployment)

---

## 2. API Exposure and Documentation

### Mandatory Endpoints
Every deployment **must** expose:

- **`/health`**
  - Returns HTTP 200 when the app is ready.
  - Required for Hugging Face to transition the Space from *starting* → *running*.

- **`/api-docs`**
  - Documents **all** available API endpoints.
  - Must be reachable at:
    `https://Leon4gr45-kanban.hf.space/api-docs`

### Functional Endpoints

### /health
- Method: GET
- Purpose: Check if the application is healthy and running.
- Request:
  None
- Response:
  Returns HTTP Status 200 OK without a payload body when healthy.

### /api-docs
- Method: GET
- Purpose: Retrieve the Swagger API documentation.
- Request:
  None
- Response:
  Returns an HTML file containing the API documentation interface.

---

## 3. Deployment Workflow

Precondition: Use the huggingface hub cli hf to check that the space is empty of files nd delete any which are still in there and not belonging to the project to be uploaded

### Standard Deployment Command
After any code change, run:

```bash
hf upload Leon4gr45/kanban --repo-type=space
---

Scan build and run logs # Get build logs (SSE) curl -N
-H "Authorization: Bearer $HF_TOKEN"
"https://huggingface.co/api/spaces/Leon4gr45/kanban/logs/build"

Get run logs (SSE) once the build logs succeed
curl -N
-H "Authorization: Bearer $HF_TOKEN"
"https://huggingface.co/api/spaces/Leon4gr45/kanban/logs/run"

after 300 seconds to see if the deployment has been successful, and if not, fix the errors of deployment, and redeploy and monitor in a cycle until the space is running and reacts to the api endpoints you created.
