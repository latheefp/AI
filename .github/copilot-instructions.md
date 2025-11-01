## Purpose
Short, actionable guidelines for AI coding agents working in this repo. Focus on how the project runs locally, where models and data live, and concrete code examples to change or extend.

## Big picture
- This repository is a small Ollama-based model runtime plus training artifacts.
- Runtime: `docker-compose.yaml` runs an `ollama` service exposing the API on port `11434` and an `open-webui` front-end on port `3000` (proxied to Open WebUI). Models are persisted to the `ollama_models` Docker volume and pulled at container startup (the compose entrypoint pulls `llama3`).
- Training / assets: `training/` holds a `Modelfile`, a dataset (`json_extraction_dataset_500.json`), and a Colab link (`colablink`) used for fine-tuning or dataset preparation.

## Quick start (what humans run)
1. Start services: from repo root run `docker-compose up -d` to start Ollama and Open-WebUI. The `ollama` service will pull `llama3` by default (see `entrypoint`).
2. Verify API: Ollama HTTP API is at `http://localhost:11434/api/chat`.

Example: run the included simple client `sample_request.py` to stream responses (it posts JSON and expects newline-delimited JSON in response).

## Useful files & patterns (for code edits)
- `docker-compose.yaml` — service orchestration. Note the `entrypoint` pulls models on container startup: change model names here to change what's pre-pulled.
- `sample_request.py` — a streaming HTTP client example that iterates over `response.iter_lines()` and parses JSON per line. Use this pattern when calling the Ollama `/api/chat` endpoint.
- `package.py` — shows use of the `ollama` Python client library. Either call the local HTTP API (as `sample_request.py` does) or use the client library when available.
- `training/Modelfile` — a simple config/template for local training/prompt templating. It uses PARAMETERS and TEMPLATE blocks — treat it as the ground truth for tuning params used by training scripts.

## Project-specific conventions
- Model names: use the exact Ollama model identifiers (examples in the repo: `llama3`, `llama2`). The code and docker-compose expect those strings.
- Persisted models: models live in the `ollama_models` Docker volume mapped to `/root/.ollama` inside the container. Don't assume models are ephemeral — they persist between container restarts.
- Streaming API handling: code expects newline-delimited JSON streaming. `sample_request.py` shows resilient parsing (skips empty lines, ignores JSON decode errors). Keep that pattern when adding clients.

## Integration points & extension notes
- To add a new model: either update `docker-compose.yaml` entrypoint to `ollama pull <model>` or exec into the running container and run `ollama pull <model>` manually. The model will be saved to `ollama_models`.
- To change HTTP usage vs. Python client: both patterns are present. Use `sample_request.py` for raw HTTP, use `package.py` for higher-level client usage.
- Training artifacts: if you modify `training/json_extraction_dataset_500.json` or `training/Modelfile`, also update any Colab notebook referenced by `training/colablink` for reproducibility.

## Small actionable examples to copy
- Start services:
  - `docker-compose up -d`
- Stream via HTTP (pattern used in `sample_request.py`):
  - POST to `http://localhost:11434/api/chat` with body `{"model":"llama3","messages":[{"role":"user","content":"..."}]}` and iterate `response.iter_lines()`.
- Use Python client (pattern from `package.py`):
  - `import ollama; client = ollama.Client(); client.generate(model='llama2', prompt='...')`

## What to watch for / gotchas
- The repo has no CI/test harness or README; changes to runtime behavior should be sanity-checked locally by starting Docker and running `sample_request.py`.
- The compose entrypoint blocks on `tail -f /dev/null` to keep the container alive after pulling — modify carefully if changing startup ordering.

## Where to look next
- If you need to change model parameters, check `training/Modelfile` and the `entrypoint` in `docker-compose.yaml`.
- For UI/debugging, open `http://localhost:3000` after starting compose (Open WebUI) and use the Ollama API at `http://localhost:11434` for programmatic checks.

If any of this is incomplete or you want specific examples (e.g., adding a new model, creating a local test script, or wiring a new training pipeline), tell me which area and I will expand the instructions.
