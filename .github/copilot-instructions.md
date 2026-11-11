# Repository Instructions

## Project

This repository is a containerized EO/IR perception-stack DevOps challenge. It contains a Python `vision_api`, a Python `inference_worker`, a model engine plus its SHA-256 manifest, PostgreSQL orchestration, and a GitHub Actions workflow. Keep changes focused on operational reliability, security, reproducible releases, and the requirements in `README.md`.

## Architecture

- `vision_api/app.py` serves `/health` and `/api/v1/detect` and checks the worker at `WORKER_URL`.
- `inference_worker/worker.py` serves `/status`, validates the engine against `models/manifest.json`, and writes its execution cache under `TRT_CACHE_DIR`.
- `docker-compose.yml` defines `vision_api`, `inference_worker`, and `postgres`; service health and startup dependencies determine readiness.
- `Dockerfile` builds the Python runtime image. Keep the final image small and run application processes as a non-root user.
- `.github/workflows/ci.yml` is the release gate. Tests and artifact verification must fail closed before image build or promotion; do not publish mutable `latest` tags.

## Working Rules

- Read the relevant section of `README.md` before changing behavior; it contains the acceptance criteria and air-gap release requirements.
- Never commit `.env`, credentials, or other secrets. Use `.env.example` for documented variable names and safe placeholders, and keep `.env` ignored.
- Treat `models/eo_detector_fp16.engine` as a binary release artifact. Do not edit or regenerate it casually; when its bytes intentionally change, update and verify `models/manifest.json` and explain the provenance.
- Preserve checksum verification and ensure worker health checks report invalid or unavailable artifacts as unhealthy.
- Keep image tags immutable and release bundles verifiable offline. Avoid adding runtime network dependencies to the air-gapped deployment path.
- Follow the existing Python standard-library style unless a dependency is justified by the task. Avoid unrelated refactors.

## Validation

- Run unit tests with `python -m unittest discover -s tests` and do not suppress failures.
- For container or orchestration changes, validate with `docker compose config` and, when Docker is available, build and exercise the stack using the README acceptance checks.
- For CI changes, verify that test and artifact-check failures stop the build/promotion path and that generated image references use immutable tags.
- State clearly when Docker, target hardware, or another required environment is unavailable; do not claim an unrun integration check passed.