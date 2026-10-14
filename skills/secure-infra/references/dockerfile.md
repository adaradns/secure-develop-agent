# Dockerfile & container image — secure checklist

Format: **What to check** · **Why** · **Fix**. Verify against the real Dockerfile/compose file.

## 1. Non-root user
- **Why:** a process running as root in the container that escapes has root on the host namespace.
- **Check:** is there a `USER` instruction switching to a non-root user before `CMD`/`ENTRYPOINT`?
- **Fix:**
  ```dockerfile
  RUN addgroup -S app && adduser -S app -G app
  USER app
  ```

## 2. Pinned base image
- **Why:** `FROM node:latest` is non-reproducible and can pull in a changed/compromised image.
- **Check:** base image pinned to a specific version, ideally by digest (`@sha256:...`).
- **Fix:** `FROM node:20.11-alpine@sha256:...`

## 3. No secrets in layers
- **Why:** `ENV`, `ARG`, or `COPY .env` bake secrets into image history — recoverable with `docker history`.
- **Check:** no secrets in `ENV`/`ARG`; no `COPY . .` that drags in `.env`, keys, or `.git`.
- **Fix:** use build secrets (`RUN --mount=type=secret,...`) or inject at runtime; add a `.dockerignore`
  for `.env`, `.git`, `node_modules`, credentials.

## 4. Minimal image & multi-stage builds
- **Why:** build tools, compilers, and dev deps in the final image enlarge the attack surface.
- **Check:** multi-stage build; final stage based on `-slim`/`-alpine`/`distroless`; no `curl`/`bash`
  left in unless needed.
- **Fix:** copy only the built artifact into a minimal runtime stage.

## 5. Least-privilege filesystem & capabilities
- **Check:** read-only root FS where possible (`--read-only` / compose `read_only: true`),
  dropped capabilities (`cap_drop: [ALL]`), `no-new-privileges`.

## 6. Healthcheck & pinned deps
- **Check:** `HEALTHCHECK` present; dependency install uses a lockfile (`npm ci`, `mvn -B`, not `npm install`).

## 7. docker-compose specifics
- No `privileged: true` unless strictly required; don't mount the Docker socket
  (`/var/run/docker.sock`) into app containers; don't hardcode secrets in `environment:` — use
  `env_file`/secrets and keep them out of git.

## 8. Image scanning
- **Check:** CI scans the built image (Trivy, Grype, `docker scout`) and fails on high/critical CVEs.
