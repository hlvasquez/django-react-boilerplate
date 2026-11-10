# Django + React/Vite Boilerplate

A minimal, functional full-stack skeleton. It provides the core runtime and infrastructure; build application features on top of it.

## Stack

- **Backend:** Python 3.13, Django 6, Django REST Framework, Daphne (ASGI)
- **Frontend:** Node.js 22, React 19, Vite
- **Database:** PostgreSQL + PostGIS
- **Background jobs/cache:** Celery + Dragonfly (Redis-compatible)
- **Reverse proxy:** NGINX
- **Development containers:** rootless Podman + `podman-compose`

## Repository layout

```text
backend/       Django project, settings, API and Celery configuration
frontend/      React + Vite application
nginx/         NGINX reverse-proxy configuration
docker-compose.yml
.env.example
```

Keep this structure small. Add Django apps, frontend features, and extra infrastructure only when the application needs them.

## Requirements

Target environment: Ubuntu 26.04 LTS.

Install the basic tools:

```bash
sudo apt update
sudo apt install -y podman podman-compose python3 python3-venv python3-pip nodejs npm
```

Check that they are available:

```bash
podman --version
podman-compose --version
python3 --version
node --version
npm --version
```

Django 6 requires Python 3.12 or newer. Use a supported Node.js release for the Vite frontend.

## 1. Configure environment

From the repository root:

```bash
cp .env.example .env
```

Set the database credentials and Django secret key in `.env`. Generate a secret key with:

```bash
python3 -c "import secrets; print(secrets.token_urlsafe(64))"
```

Never commit `.env`.

## 2. Start database and cache

```bash
podman-compose up -d
podman-compose ps
```

This starts the PostgreSQL/PostGIS and Dragonfly services defined in `docker-compose.yml`.

## 3. Run the backend

```bash
cd backend
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements/development.txt
python manage.py migrate
```

Start the ASGI server:

```bash
daphne -b 127.0.0.1 -p 8000 config.asgi:application
```

The Django development server is also available for quick iteration:

```bash
python manage.py runserver 127.0.0.1:8000
```

Use a second terminal to run the background worker:

```bash
cd backend
source .venv/bin/activate
celery -A config worker -l info
```

## 4. Run the frontend

In another terminal:

```bash
cd frontend
npm install
npm run dev
```

Open the local URL printed by Vite (normally `http://localhost:5173`). The Vite configuration should proxy `/api` requests to the backend at `http://127.0.0.1:8000`.

## 5. Run behind NGINX (optional)

To run the complete containerized stack, including the backend, worker, frontend build, and NGINX:

```bash
podman-compose --profile app up -d --build
```

Open `http://localhost:8080` if that is the port configured in `.env`/Compose.

## Useful commands

```bash
# Backend checks
cd backend
python manage.py check
python manage.py test

# Frontend build
cd ../frontend
npm run build
```

## Scope

This boilerplate intentionally includes only the shared foundation:

- Django API and ASGI entry point
- React/Vite frontend
- PostgreSQL/PostGIS persistence
- Celery worker and Dragonfly broker/cache
- NGINX reverse proxy configuration

Authentication policies, domain-specific models, AI integrations, business workflows, seed data, production deployment procedures, and extensive API documentation belong in the application that uses this skeleton—not in this README.
