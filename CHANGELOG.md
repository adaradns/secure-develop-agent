# Changelog

All notable changes to this project are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.1.0] - 2026-10-13

### Changed
- Split the monolithic skill into **three topic-based skills** so only the relevant guidance loads per context:
  - **secure-web** — OWASP Top 10:2025 (backend/frontend: Java/Quarkus, Spring Boot, Node/NestJS, Next.js)
  - **secure-llm** — OWASP Top 10 for LLM Applications 2025 (prompt injection, I/O validation, etc.)
  - **secure-infra** — infrastructure security (Docker, Kubernetes, CI/CD)

### Added
- Infrastructure coverage: containers, Kubernetes, and CI/CD pipelines.

### Includes
- Security audit agent (`security-auditor`).
- Checklists based on OWASP Top 10:2025 and OWASP Top 10 for LLM Applications 2025.

## [1.0.0] - 2026-10-13

### Added
- Initial release of the `secure-develop` plugin.
- Security auditing and guidance based on OWASP Top 10 and OWASP Top 10 for LLM Applications.
- Security audit agent.

[1.1.0]: https://github.com/adaradns/secure-develop-agent/releases/tag/1.1.0
[1.0.0]: https://github.com/adaradns/secure-develop-agent/releases/tag/1.0.0
