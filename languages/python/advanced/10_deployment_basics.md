# 10 - Deployment Basics

## Learning Goal

Learn how to prepare a Python application so it can run predictably outside your laptop. Deployment is not just "copy the code to a server." It means making the code reproducible, configurable, runnable by another process manager, observable when something goes wrong, and safe to release or roll back.

## Why It Matters

Local development hides many assumptions: your installed packages, your shell variables, your current working directory, your open files, your database state, and the fact that you are watching the terminal. A deployed application has to survive without those assumptions.

A good deployment answers these questions:

- Can another machine rebuild the same environment?
- What exact command starts the program?
- How does configuration change between dev, staging, and production?
- Where do logs go?
- How can a platform tell whether the process is healthy?
- How do you test the release quickly?
- How do you return to the previous version if the release is bad?

## Deployment Surfaces

Python code is deployed in several common shapes.

### Scripts

A script deployment runs a command such as:

```bash
python -m reports.daily_summary
```

This is common for jobs, automation, ETL tasks, and scheduled work. You still need pinned dependencies, environment variables, logs, exit codes, and a clear working directory.

### Packages

A package deployment installs your code into an environment:

```bash
python -m pip install .
python -m mytool
```

Packaging is useful when code is shared across projects or when you want console entry points. A package should declare its dependencies and support installation into a fresh environment.

### Web Services

A web-service deployment runs a long-lived process behind a production server:

```bash
gunicorn "myapp:create_app()"
uvicorn app.main:app --host 0.0.0.0 --port 8000
```

Web services need all the usual deployment basics plus port binding, health checks, graceful startup and shutdown, request logging, and often multiple worker processes.

## Deployment Checklist

Before releasing, check each part explicitly.

- Source: the commit, tag, or artifact is known.
- Runtime: the Python version is known.
- Environment: dependencies install into a clean virtual environment or image.
- Dependencies: versions are pinned or locked.
- Startup: one command starts the application.
- Configuration: values such as port, log level, service URLs, and feature flags come from the environment.
- Secrets: passwords, tokens, and private keys are injected by the platform, not committed.
- Logs: application logs go to stdout or stderr so the platform can collect them.
- Health: the app has a cheap endpoint or command that reports basic readiness.
- Smoke test: one or two commands prove the deployed app responds.
- Migration plan: database migrations run once, in a controlled step.
- Rollback: the previous release can be restored quickly.

## Reproducible Environments

Use a virtual environment for local development and CI:

```bash
python -m venv .venv
source .venv/bin/activate
python -m pip install -r requirements.txt
```

On Windows PowerShell:

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install -r requirements.txt
```

The important idea is isolation: the app should not depend on packages that happen to be installed globally. A fresh environment should be enough to install and run the project.

For applications, pin dependency versions:

```text
fastapi==0.115.6
uvicorn[standard]==0.32.1
```

For libraries, dependency ranges may be appropriate, but deployed applications should prefer repeatability. Teams often use tools such as pip-tools, Poetry, uv, or lock files to make this stricter.

## Startup Commands

A deployment platform should not have to guess how to start the app. Write down the command in documentation, a process file, a container `CMD`, or platform configuration.

Good startup commands are explicit:

```bash
python -m my_package.worker
uvicorn app.main:app --host 0.0.0.0 --port "$PORT"
gunicorn "app:create_app()" --workers 3 --bind "0.0.0.0:$PORT"
```

Avoid startup commands that depend on your local shell history, current directory accidents, or an IDE launch profile.

## Configuration and Secrets

Configuration should be separate from code. Environment variables are a common deployment interface because they can change between environments without rebuilding the application.

Examples:

```bash
APP_NAME="Billing API"
LOG_LEVEL=INFO
PORT=8000
DATABASE_URL="postgresql://..."
```

Secrets are configuration, but with stricter handling. Do not commit them, print them, bake them into container images, or store them in example files with real values. Use your platform's secret manager, CI/CD secret store, or runtime environment injection.

## Logs, Health Checks, and Smoke Tests

In production, logging to a local file inside the app directory is usually the wrong default. Containers and cloud platforms expect processes to write logs to stdout and stderr, then the platform collects, stores, searches, and routes those streams.

Use Python's `logging` module instead of scattered `print()` calls:

```python
import logging

logging.basicConfig(level="INFO")
logger = logging.getLogger(__name__)
logger.info("service started")
```

A health check should be cheap and boring. It should answer "is this process alive enough to receive traffic?" without doing expensive work. A common endpoint is:

```text
GET /health -> {"status": "ok"}
```

A smoke test is a tiny post-deploy test. It does not replace your test suite. It catches obvious release failures:

```bash
curl -fsS http://localhost:8000/health
curl -fsS http://localhost:8000/
```

## Production Servers

Development servers are for local feedback. Do not treat reload-enabled development mode as a production serving strategy.

Python web apps usually speak through one of two interfaces:

- WSGI: the traditional synchronous interface used by frameworks such as Flask and Django.
- ASGI: the async-capable interface used by FastAPI, Starlette, Django async support, and WebSocket-capable services.

Gunicorn is a common production process manager for WSGI apps and can also manage ASGI apps with an ASGI worker class. Uvicorn is an ASGI server often used with FastAPI. In production, these servers are commonly placed behind a reverse proxy or platform load balancer that handles TLS, buffering, routing, and restarts.

The big idea: your framework defines the app; a production server owns the long-running process.

## Containers as a Deployment Boundary

A container image packages your application code, Python runtime, system libraries, Python dependencies, and startup command into one artifact. That makes it easier to move the same release from CI to staging to production.

A useful Python Dockerfile usually:

- Starts from a specific Python base image.
- Sets a working directory.
- Copies dependency files before application code so dependency installation can be cached.
- Installs dependencies without relying on global machine state.
- Copies only the files needed at runtime.
- Runs as a non-root user.
- Defines a clear `CMD`.

Containers do not automatically solve:

- Bad configuration.
- Leaked secrets.
- Missing dependency pins.
- Broken database migrations.
- Unsafe release order.
- Lack of metrics, traces, or logs.
- A service that only binds to `127.0.0.1` inside the container.

Containers make the boundary clearer. You still have to design the application to run well inside that boundary.

## Common Mistakes

- Hard-coding secrets in Python files, Dockerfiles, or committed `.env` files.
- Missing dependency pins, causing different versions to install on different days.
- Binding to `localhost` or `127.0.0.1` inside a container, which can make the service unreachable from outside the container.
- Writing logs only to local files that disappear when the container is replaced.
- Running database migrations from every web worker, causing duplicate or conflicting migration attempts.
- Shipping without a health check, leaving the platform unable to distinguish "started" from "ready."
- Using a development server with auto-reload as the production command.
- Forgetting a smoke test, so broken releases are discovered by users first.

## Exercise

Create a tiny FastAPI service that is ready to containerize.

Requirements:

1. The app has `GET /` and `GET /health`.
2. `APP_NAME`, `LOG_LEVEL`, and `PORT` come from environment variables.
3. The app logs to the console.
4. Dependencies are listed in `requirements.txt`.
5. `.dockerignore` keeps local-only files out of the image.
6. The Dockerfile installs dependencies cache-efficiently, runs as a non-root user, and has a clear startup command.
7. Include build, run, and smoke-test commands.
8. Write a short release checklist.

## Worked Answer

One possible layout:

```text
deploy-demo/
  app/
    main.py
  requirements.txt
  .dockerignore
  Dockerfile
```

`app/main.py`:

```python
import logging
import os

from fastapi import FastAPI


APP_NAME = os.getenv("APP_NAME", "Deployment Demo")
LOG_LEVEL = os.getenv("LOG_LEVEL", "INFO").upper()
PORT = int(os.getenv("PORT", "8000"))

logging.basicConfig(
    level=LOG_LEVEL,
    format="%(asctime)s %(levelname)s %(name)s %(message)s",
)
logger = logging.getLogger(__name__)

app = FastAPI(title=APP_NAME)


@app.on_event("startup")
async def startup() -> None:
    logger.info("starting app=%s port=%s log_level=%s", APP_NAME, PORT, LOG_LEVEL)


@app.get("/")
async def read_root() -> dict[str, str]:
    logger.info("root endpoint called")
    return {"message": f"{APP_NAME} is running"}


@app.get("/health")
async def health() -> dict[str, str]:
    return {"status": "ok"}
```

`requirements.txt`:

```text
fastapi==0.115.6
uvicorn[standard]==0.32.1
```

`.dockerignore`:

```text
.git
.venv
__pycache__
*.pyc
.pytest_cache
.mypy_cache
.ruff_cache
.env
dist
build
*.egg-info
```

`Dockerfile`:

```dockerfile
FROM python:3.13-slim

ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONUNBUFFERED=1

WORKDIR /app

COPY requirements.txt .
RUN python -m pip install --upgrade pip \
    && python -m pip install --no-cache-dir -r requirements.txt

COPY app ./app

RUN useradd --create-home --shell /usr/sbin/nologin appuser
USER appuser

ENV APP_NAME="Deployment Demo"
ENV LOG_LEVEL=INFO
ENV PORT=8000

EXPOSE 8000

CMD ["sh", "-c", "uvicorn app.main:app --host 0.0.0.0 --port ${PORT}"]
```

Build the image:

```bash
docker build -t deploy-demo:latest .
```

Run the container:

```bash
docker run --rm -p 8000:8000 \
  -e APP_NAME="FastAPI Deployment Demo" \
  -e LOG_LEVEL=INFO \
  -e PORT=8000 \
  deploy-demo:latest
```

Smoke test it:

```bash
curl -fsS http://localhost:8000/health
curl -fsS http://localhost:8000/
```

Expected responses:

```json
{"status":"ok"}
```

```json
{"message":"FastAPI Deployment Demo is running"}
```

Release checklist:

- Confirm the image was built from the intended commit.
- Confirm dependency versions are pinned.
- Confirm no real secrets are in the repo or image.
- Confirm required environment variables are set in the deployment platform.
- Confirm the process starts with `uvicorn app.main:app --host 0.0.0.0 --port ${PORT}`.
- Confirm logs appear in the platform log stream.
- Confirm `/health` returns `200 OK`.
- Run the smoke-test commands after deployment.
- Run database migrations once, outside the web-worker startup path, if the app uses a database.
- Keep the previous image tag available for rollback.

## Sources Used

- Python `venv` documentation: https://docs.python.org/3/library/venv.html
- Python Packaging User Guide: https://packaging.python.org/
- Docker Python container guide: https://docs.docker.com/guides/python/containerize/
- Dockerfile reference and build cache docs: https://docs.docker.com/reference/dockerfile/ and https://docs.docker.com/build/cache/
- FastAPI deployment concepts: https://fastapi.tiangolo.com/deployment/concepts/
- Gunicorn deployment documentation: https://gunicorn.org/deploy/
- Twelve-Factor App config and logs: https://12factor.net/config and https://12factor.net/logs
- Python Logging HOWTO: https://docs.python.org/3/howto/logging.html

## Next Step

Return to the advanced README and connect this lesson to packaging, application configuration, logging, and architecture. Deployment is where those ideas stop being separate topics and become one release process.
