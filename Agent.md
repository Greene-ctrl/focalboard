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

### Secrets and Variables
- The environment variable **`HF_TOKEN` will always be provided at execution time**.
- Never hardcode the token. Always read it from the environment.
- All monitoring and log‑streaming commands rely on `$HF_TOKEN`.
- **`your_secure_password`**: This environment variable controls the default admin user password. The application intercepts this variable on startup and will automatically create or update the system user `admin` to authenticate with the password stored in this variable.

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

---

## 4. API Authentication and Usage via curl

Focalboard's API endpoints require explicit Anti-CSRF protection and session authentication. Use the following guidelines to communicate with the REST API using `curl`.

### Mandatory Header (CSRF Protection)
Every REST API request directed to `https://Leon4gr45-kanban.hf.space/api/v2/*` **must** include the following header:
```bash
-H "X-Requested-With: XMLHttpRequest"
```
If this header is omitted, the API will forcefully reject the request with `400 Bad Request: checkCSRFToken FAILED`.

### Authentication & Session Management
Most API operations (such as listing boards or users) are protected and require a valid user session.

#### 1. Authenticate (Login)
To initiate a session, submit your username and password to the `/api/v2/login` endpoint. **Crucially, the payload must include `"type": "normal"`** alongside the credentials.

```bash
curl -X POST "https://Leon4gr45-kanban.hf.space/api/v2/login" \
  -H "X-Requested-With: XMLHttpRequest" \
  -H "Content-Type: application/json" \
  -d '{"type": "normal", "username": "admin", "password": "your_secure_password"}'
```
This will return a JSON response containing your session token, such as `{"token":"...long_token_string..."}`.
*(Note: If no users exist yet, you may need to register first using the `/api/v2/register` endpoint).*

#### 2. Execute Authenticated API Calls
Subsequent requests should include this token in the `Authorization: Bearer <TOKEN>` header.

**Example: Get Current User Data**
```bash
curl -X GET "https://Leon4gr45-kanban.hf.space/api/v2/users/me" \
  -H "X-Requested-With: XMLHttpRequest" \
  -H "Authorization: Bearer <your_token_here>"
```

**Example: List Workspace Boards**
```bash
curl -X GET "https://Leon4gr45-kanban.hf.space/api/v2/teams/0/boards" \
  -H "X-Requested-With: XMLHttpRequest" \
  -H "Authorization: Bearer <your_token_here>"
```

By adhering to these rules, subsequent agents or users can robustly navigate and manage the Kanban via the API.
