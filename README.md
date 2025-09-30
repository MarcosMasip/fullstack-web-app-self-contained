# Sports Ticketing Web App

Sports Ticketing is a full‑stack web application for browsing sports matches and purchasing tickets. It’s built with:

- Backend: FastAPI (Python), JWT auth, SQLAlchemy
- Frontend: Vue 2 + Vue Router + BootstrapVue
- Database: Postgres (via Docker) or SQLite (local dev without Docker)

You can run it fully self‑contained with Docker (recommended), or locally without Docker using Python and Node.

Quick links:
- App (frontend): http://localhost:8080
- API (backend): http://localhost:8000

## What you can do

- Sign up, log in, and log out
- View available matches and details
- Add tickets to a cart, set quantities, and place orders
- Admin‑only: manage teams, competitions, and matches

## Table of Contents
1. Project Structure
2. Run with Docker (macOS/Windows/Linux)
3. Run locally without Docker (macOS/Windows/Linux)
4. Environment variables
5. Ports and URLs
6. Shutdown and cleanup procedures
7. Troubleshooting

## 1) Project Structure

Top‑level layout (truncated for clarity):

```
app/
    docker-compose.yml
    services/
        backend/
            Dockerfile
            requirements.txt
            src/
                __init__.py
                database.py
                dependencies.py
                main.py
                models.py
                repository.py
                schemas.py
                utils.py
                test_main.py
                test_main.http
        frontend/
            Dockerfile
            package.json
            index.html
            config/
            src/
                App.vue, main.js, router/, components/, assets/
            bootstrap/
                css/, js/
```

Notes:
- When using Docker, the backend connects to Postgres. When running locally without Docker (production=false), it uses a local SQLite file.
- The frontend calls the backend at http://localhost:8000.

---

## 2) Run with Docker (Recommended)

Prerequisites:
- Docker Desktop (Windows/macOS) or Docker Engine (Linux) installed and running
- Internet connection

Step‑by‑step

1) Clone and enter the repo

```bash
git clone https://github.com/MarcosMasip/fullstack-web-app-self-contained.git
cd fullstack-web-app-self-contained
```

Expected outcome:
- A folder named `fullstack-web-app-self-contained` is created and becomes your working directory.

2) Create the backend .env with secrets (production=true)

The backend requires `app/services/backend/src/.env` with JWT keys and a production flag.

- macOS/Linux:

```bash
mkdir -p app/services/backend/src
printf "jwt_secret_key=%s\njwt_refresh_secret_key=%s\nproduction=true\n" \
    "$(openssl rand -hex 32)" \
    "$(openssl rand -hex 32)" | tee app/services/backend/src/.env
```

- Windows (PowerShell):

```powershell
New-Item -ItemType Directory -Force -Path app/services/backend/src | Out-Null
$jwt = -join ((48..57 + 97..102) | Get-Random -Count 64 | ForEach-Object {[char]$_})
$refresh = -join ((48..57 + 97..102) | Get-Random -Count 64 | ForEach-Object {[char]$_})
@(
    "jwt_secret_key=$jwt"
    "jwt_refresh_secret_key=$refresh"
    "production=true"
) | Set-Content app/services/backend/src/.env
```

Expected outcome:
- A file `app/services/backend/src/.env` exists with three lines: jwt_secret_key, jwt_refresh_secret_key, production=true.

3) Build and start the stack

```bash
cd app
docker compose up --build -d
```

Expected outcome:
- Docker builds images for `backend` and `frontend`, starts `db` (Postgres), and then boots all 3 containers in the background.
- Subsequent runs will be faster due to layer caching.

4) Verify containers

```bash
docker compose ps
```

Expected outcome:
- backend Up with port 8000/tcp published as 0.0.0.0:8000->8000
- frontend Up with port 8080/tcp published as 0.0.0.0:8080->8080
- db Up with 5432/tcp (internal only)

5) Watch logs briefly (optional)

```bash
docker compose logs -f --tail=50 backend frontend db
```

Expected outcome:
- db shows “database system is ready to accept connections”.
- backend shows Uvicorn: “Uvicorn running on http://0.0.0.0:8000”.
- frontend shows Vue dev server with “App running at” and http://localhost:8080.
Press Ctrl+C to stop watching; containers keep running.

6) Quick API checks

```bash
curl -s http://localhost:8000/; echo
curl -s http://localhost:8000/matches/; echo
```

Expected outcome:
- First prints `production`.
- Second prints `[]` (empty matches list on a fresh DB).

7) Open the app

```bash
open http://localhost:8080
```

Expected outcome:
- The Vue app loads. You can sign up, log in, and interact with matches and orders. Admin‑only actions require an admin user.

---

## 3) Run locally without Docker

This path uses Python + Node on your machine. The backend uses SQLite when `production=false`.

Prerequisites:
- Python 3.11+ recommended (3.10+ likely fine)
- Node.js LTS (16+), npm

Backend (FastAPI)

1) Create and activate a virtual environment

- macOS/Linux (bash/zsh):

```bash
cd app/services/backend
python3 -m venv .venv
source .venv/bin/activate
```

- Windows (PowerShell):

```powershell
cd app/services/backend
py -3 -m venv .venv
.venv\Scripts\Activate.ps1
```

Expected outcome:
- Your shell prompt shows the venv active, e.g. `(.venv)`.

2) Install dependencies

```bash
pip install --upgrade pip
pip install -r requirements.txt
```

Expected outcome:
- Packages install successfully (fastapi, uvicorn, sqlalchemy, etc.).

3) Create the backend .env (production=false for SQLite)

- macOS/Linux:

```bash
cat > src/.env << 'EOF'
jwt_secret_key=changeme_dev_only
jwt_refresh_secret_key=changeme_dev_only
production=false
EOF
```

- Windows (PowerShell):

```powershell
@(
    'jwt_secret_key=changeme_dev_only'
    'jwt_refresh_secret_key=changeme_dev_only'
    'production=false'
) | Set-Content src/.env
```

Expected outcome:
- `src/.env` exists and has production=false, so the app uses local SQLite at `app/data.db` (created on first run).

4) Run the API server

```bash
uvicorn src.main:app --reload --host 0.0.0.0 --port 8000 --env-file ./src/.env
```

Expected outcome:
- Uvicorn starts and listens on http://127.0.0.1:8000 (and 0.0.0.0).
- A SQLite file `app/data.db` is created relative to the project root when the app first touches the DB.

Frontend (Vue)

1) Install dependencies

In a separate terminal:

```bash
cd app/services/frontend
npm install
```

Expected outcome:
- Node modules are installed without errors.

2) Start the dev server

```bash
npm run serve
```

Expected outcome:
- Vue CLI dev server starts and shows “App running at” with http://localhost:8080.
- The app loads in the browser and talks to http://localhost:8000.

---

## 4) Environment variables

Backend reads environment from `src/.env` (or actual environment) using Pydantic BaseSettings:

- `jwt_secret_key`: secret for access tokens (required)
- `jwt_refresh_secret_key`: secret for refresh tokens (required)
- `production`: boolean
    - true ⇒ use Postgres via `DATABASE_URL` (Docker Compose sets this for the backend service)
    - false ⇒ use SQLite file at `sqlite:///./app/data.db`

When using Docker Compose, an environment variable `DATABASE_URL` is injected into the backend container:

```
postgresql://postgres:postgres@db:5432/appdb
```

## 5) Ports and URLs

- Frontend: http://localhost:8080 → Vue app
- Backend: http://localhost:8000 → FastAPI
- Database (Docker only): Postgres exposed internally as `db:5432` (not published to host)

---

## 6) Shutdown and cleanup procedures

Docker (stack)

- Stop and remove containers (preserve DB data):

```bash
cd app
docker compose down
```

Expected outcome:
- Containers for backend, frontend, and db are stopped and removed. The `postgres_data` volume persists.

- Stop and remove containers and volumes (reset DB):

```bash
cd app
docker compose down -v
```

Expected outcome:
- Containers and the `postgres_data` volume are removed. Next `up` creates a fresh DB.

Local (without Docker)

- Stop servers:
    - In each terminal running uvicorn or `npm run serve`, press Ctrl+C.
    - Deactivate Python venv: `deactivate` (bash/zsh) or `deactivate` in PowerShell.

- Optional cleanup:

```bash
rm -f app/data.db                 # remove SQLite DB
rm -rf app/services/backend/.venv # remove Python venv
rm -rf app/services/frontend/node_modules
```

Expected outcome:
- Processes stop; optional files/folders are removed.

---

## 7) Troubleshooting

- Ports already in use (8000/8080): stop any processes using those ports or change the ports.
- Docker Desktop not running: start Docker and rerun `docker compose up`.
- “openssl: command not found” on Windows: use the PowerShell snippet provided, or set static dev secrets in `.env`.
- Backend shows CORS errors in browser: ensure you’re serving the frontend on http://localhost:8080 (that origin is allowed), and the backend is reachable at http://localhost:8000.
- Reset Docker DB to a clean state: `docker compose down -v` (from `app/`).
- Node build/install issues on Windows: run PowerShell as Administrator, or use WSL2 with Ubuntu and follow Linux/macOS steps.

---

Happy hacking! If you run into anything unclear, open an issue or PR.