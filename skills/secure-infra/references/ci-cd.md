# CI/CD pipelines — secure checklist

Covers GitHub Actions, GitLab CI, and similar. Format: **What to check** · **Why** · **Fix**.

## 1. Secrets management
- **Check:** secrets come from the platform secret store (GitHub/GitLab secrets, OIDC to cloud),
  never hardcoded in the workflow YAML or echoed to logs.
- **Fix:** mask secrets; prefer short-lived OIDC federation over long-lived static cloud keys.

## 2. Least-privilege job permissions (GitHub Actions)
- **Why:** the default `GITHUB_TOKEN` can be write-scoped to the whole repo.
- **Fix:** set `permissions: contents: read` at the top and widen per-job only when needed.

## 3. Pin third-party actions by SHA
- **Why:** `uses: some/action@v3` follows a mutable tag — a compromised tag runs in your pipeline.
- **Fix:** pin to a full commit SHA (`uses: some/action@<40-char-sha>`); review before bumping.

## 4. Guard untrusted triggers
- **Why:** `pull_request_target` and workflows that check out PR code run with repo secrets in scope —
  a classic exfiltration path from forks.
- **Check:** don't expose secrets to workflows triggered by untrusted PRs; don't run untrusted code
  with elevated tokens; avoid `script injection` via `${{ github.event.* }}` interpolated into `run:`.

## 5. Supply-chain & scanning gates
- **Check:** dependency audit (`npm audit`, OWASP Dependency-Check), SAST, secret scanning
  (gitleaks/trufflehog), and image scanning (Trivy/Grype) run in CI and fail the build on
  high/critical findings.

## 6. Artifact integrity
- **Check:** build artifacts/images are signed (cosign) and/or a provenance/SBOM is produced; deploy
  only from trusted, pinned sources.

## 7. Environment protection
- **Check:** production deploys require approvals/protected environments; secrets scoped per
  environment, not shared globally.

> TODO: adapt examples to the exact CI platform in use once standardized.
