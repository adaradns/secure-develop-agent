---
name: secure-infra
description: Secure infrastructure and build/deploy configuration — Dockerfiles, container images, Kubernetes manifests, and CI/CD pipelines. Use when writing or reviewing a Dockerfile, docker-compose, Kubernetes/Helm manifest, or a CI/CD workflow. Triggers on "Dockerfile", "docker", "container"/"contenedor", "image"/"imagen", "kubernetes"/"k8s", "helm", "CI/CD", "pipeline", "workflow", "GitHub Actions", "secrets in CI".
allowed-tools: Read, Grep, Glob
---

# Secure Infrastructure (containers, Kubernetes, CI/CD)

Read it before writing or reviewing a Dockerfile, a container/compose file, a Kubernetes or
Helm manifest, or a CI/CD pipeline. This covers how code is built, packaged, and shipped —
distinct from the application code itself (**secure-web**).

## When it activates

On mentions of "Dockerfile", "docker", "container", "image", "kubernetes"/"k8s", "helm",
"CI/CD", "pipeline", "workflow", "GitHub Actions", or whenever you open a file named
`Dockerfile`, `docker-compose*.yml`, `*.yaml` under `k8s/`/`helm/`, or a workflow under
`.github/workflows/`. Triggers work in English and Spanish.

## Instructions

1. Identify the artifact type and load ONLY the matching reference:
   - Dockerfile / docker-compose / container image → [references/dockerfile.md](references/dockerfile.md)
   - Kubernetes / Helm manifests → [references/kubernetes.md](references/kubernetes.md)
   - CI/CD pipelines (GitHub Actions, GitLab CI, etc.) → [references/ci-cd.md](references/ci-cd.md)
2. Apply the checklist against the real file; read context before reporting.
3. Report findings with severity and a concrete fix anchored to `file:line`.

## Hard rules (never negotiate)

- Never run containers as root when the workload does not require it — require a non-root `USER`.
- Never bake secrets, tokens, or `.env` files into an image layer or commit them to a repo.
- Never pull base images by floating tag (`latest`) for production — pin by digest or version.
- Never grant a CI job broader permissions/secrets than the job needs.
- Never disable TLS/certificate verification to "make it work".

## Additional resources

- Dockerfiles & container images: [references/dockerfile.md](references/dockerfile.md)
- Kubernetes & Helm: [references/kubernetes.md](references/kubernetes.md)
- CI/CD pipelines: [references/ci-cd.md](references/ci-cd.md)
