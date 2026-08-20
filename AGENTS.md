# Repository Guidelines

## Project Structure & Module Organization

This repository primarily stores the local Scriberr deployment configuration.

- `k8s/scriberr-deployment.yaml` defines the Ingress, Service, and Deployment.
- `k8s/*-secret.yaml` and `k8s/*-external-ingress.yaml` are local-only configuration and are ignored by Git.
- `data/` contains the SQLite database, uploads, transcripts, and temporary files. Treat it as generated runtime state.
- `whisperx-env/` contains downloaded models, Python virtual environments, and an embedded WhisperX checkout. It is runtime/vendor material, not the primary source tree.

Keep new maintained manifests under `k8s/`. Do not add generated audio, databases, model weights, virtual environments, or logs.

## Build, Test, and Development Commands

There is no root application build. Validate and deploy the Kubernetes configuration directly:

```sh
kubectl apply --dry-run=client -f k8s/scriberr-deployment.yaml
kubectl apply -f k8s/scriberr-deployment.yaml
kubectl rollout status deployment/scriberr
kubectl logs deployment/scriberr --follow
```

For intentional WhisperX development inside its nested checkout:

```sh
cd whisperx-env/WhisperX
uv sync --extra dev
uv run pytest
```

## Coding Style & Naming Conventions

Use two-space indentation for YAML and four spaces for Python. Prefer lowercase, hyphenated Kubernetes resource names (for example, `scriberr-secret`) and `snake_case` for Python modules and functions. Keep related Kubernetes resources in one multi-document YAML file separated by `---`. Preserve existing `uv` lockfiles when Python dependencies change.

## Testing Guidelines

Run Kubernetes client-side validation for every manifest change. After deployment, confirm rollout status and inspect logs. WhisperX tests use `pytest`; name additions `test_<behavior>.py` and keep focused assertions near the changed module. No repository-wide coverage threshold is currently defined.

## Commit & Pull Request Guidelines

The root workspace has no Git history to establish a local convention. Use concise, imperative Conventional Commit-style subjects such as `fix: increase transcription timeout` or `chore: update Scriberr image`. Pull requests should explain the operational impact, list validation commands run, link relevant issues, and call out changes to ports, volumes, resources, or environment variables. Include screenshots only for user-visible behavior.

## Security & Configuration

Never commit live tokens, JWT secrets, databases, uploads, or transcripts. Keep secrets in ignored `k8s/*-secret.yaml` files or a cluster secret manager, and commit only redacted examples. Rotate any credential that has appeared in plaintext or version control.
