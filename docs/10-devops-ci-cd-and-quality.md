# DevOps, CI/CD, and quality

Purpose: This document defines the CampusHire Git workflow, quality checks, CI gates, Docker rules, configuration, release process, and Definition of Done.

## Git workflow

CampusHire uses GitHub Flow. The `main` branch is always buildable. Every feature, fix, and documentation change goes through a pull request.

| Rule | Requirement |
|---|---|
| Branch source | Create branches from the latest `main`. |
| Branch size | Keep PRs small enough for a 30-minute trainer review. |
| Direct pushes | Do not push directly to `main`. |
| Issues | Link each PR to a GitHub Issue or Projects task. |
| Review | Complete the PR checklist before merge. |
| Merge | Squash merge is recommended for student PRs. |

## Branch naming

| Branch type | Pattern | CampusHire example |
|---|---|---|
| Feature | `feature/<fr-id>-short-name` | `feature/fr-elig-01-reasons` |
| Fix | `fix/<bug-short-name>` | `fix/resume-content-type-check` |
| Test | `test/<area-short-name>` | `test/api-student-apply` |
| Docs | `docs/<topic>` | `docs/api-client-adr` |
| Chore | `chore/<tool-or-config>` | `chore/ruff-precommit` |

## Conventional Commits

| Type | Use | Example |
|---|---|---|
| `feat` | New CampusHire behavior | `feat: add eligibility reasons` |
| `fix` | Bug fix | `fix: reject fake PDF` |
| `test` | Test-only change | `test: deny other company` |
| `docs` | Student repository docs | `docs: add session ADR` |
| `chore` | Tooling or dependency work | `chore(ci): add local lifecycle gate` |
| `refactor` | Design change without behavior change | `refactor: split transitions` |

## Pull request checklist

- The PR title uses a Conventional Commit style.
- The PR links the FR, BR, NFR, or test IDs it changes.
- Backend tests pass for affected Django apps.
- Python API/permission and report tests pass for affected endpoints.
- API errors use documented identifiers such as `resume-not-pdf`.
- Sanitized request/response and JUnit evidence are attached for changed API behavior.
- No secrets, real student data, or resume contents appear in logs.
- The author states significant AI help, if used.
- The README or ADRs are updated when behavior changes.

## Branch protection

| Protection | Required setting |
|---|---|
| Required checks | Python quality, API tests, image/security checks, and local lifecycle CI must pass. |
| Pull requests | Require a PR before merge. |
| Review | Trainer review is required for at least 2 substantive PRs per week. |
| Stale branches | Require branch update when `main` changes important files. |
| Secrets | Block pushes that contain detected secrets. |
| Force pushes | Disable force pushes to `main`. |

## Source-quality tools and rules

| Area | Tool | Rules |
|---|---|---|
| Python lint | Ruff 0.16.x | Enforce import order, unused source removal, and common bug rules. |
| Python format | Ruff 0.16.x | Format backend source before each PR. |
| Python types | mypy 2.4.x with django-stubs 5.2.x | Check application packages; exclude migrations and generated OpenAPI. |
| Backend tests | pytest 9.1.x and pytest-django 4.14.x | Enforce line coverage 85%, branch coverage 75%, and eligibility branch coverage 100%. |
| Dependencies | pip-audit 2.10.x | Fail CI for high-risk known vulnerabilities unless a trainer-approved note exists. |
| Secrets | Gitleaks 8.30.x | Fail on tokens, private keys, database passwords, or resume samples. |
| Containers | Trivy 0.75.x | Scan local images and fail on critical unfixed vulnerabilities. |
| API integration | pytest and pytest-django | Run four Must multi-request flows and validate JSON/CSV and role/ownership contracts. |
| Local operations | Host-side Python/pytest driver | Run all six local acceptance cases from document 09 without runtime external egress. |

## CI pipeline table

| Job | Trigger | Steps in words | Gate or failure condition |
|---|---|---|---|
| Backend lint and type check | Pull request and push to `main` | Install Python with uv, restore lockfile, run Ruff, and run mypy. | Fails on lint errors or type errors. |
| Backend tests | Pull request and push to `main` | Start PostgreSQL 18, migrate, load fictional fixtures, and run pytest with coverage, report contracts, and four Must API flows. | Fails below 85% line, 75% branch, 100% eligibility branch coverage, or 55 meaningful cases; all Must API flows must pass. |
| Backend security | Pull request and push to `main` | Run pip-audit and Gitleaks on the repository. | Fails on high-risk packages or committed secrets. |
| Build image | Pull request and push to `main` | Build the local Django runtime image with locked Python dependencies and pinned base tags. | Fails if the image cannot build. |
| Container scan | Pull request and push to `main` | Scan built images with Trivy. | Fails on critical vulnerabilities without trainer-approved exception. |
| Local lifecycle | Pull request and push to `main` | Cache dependencies/images, then use the host Python driver to run TC-LOCAL-001 through TC-LOCAL-006 with external egress disabled and isolated test volumes. | Fails for startup, health/demo mismatch, data loss, unclear dependency errors, public bindings, or unsafe reset. Save JUnit and hash evidence. |

Hosted CI and dependency/image/security-database downloads need internet. They MUST not be local startup prerequisites. Local runtime acceptance runs after downloads with only loopback/Compose networking; online audit scanners run separately. No public deployment or hosted business interface is a CI job.

## Docker and Compose requirements

- The single start entry point in document 06 MUST prepare local configuration, start PostgreSQL/Django, migrate, seed idempotently, and verify health.
- API clients use `http://127.0.0.1:8000`, retain cookies, and send CSRF on unsafe requests. No cross-origin dependency is required.
- Mailpit SHOULD be available through a profile for e-mail features.
- Every Docker image tag MUST be pinned to a version.
- The README MUST state the standard and lite resource settings.
- Resume media MUST be private and served through an authorized Django view.
- Stop MUST preserve `campushire_pgdata` and `campushire_private_media`; reset requires explicit confirmation and deletes only project-local data.
- Host-published ports MUST match document 06 and bind to `127.0.0.1`; optional built-in Mailpit/admin consoles are operations tools only.
- Additional task commands MAY wrap lint, unit/API tests, seed, and reports; they must not replace the canonical start/stop contract.

## Environment variables

| Name | Example value | Purpose | Secret |
|---|---|---|---|
| `DJANGO_SECRET_KEY` | `change-me-local-only` | Django signing key for local development. | Yes |
| `DJANGO_DEBUG` | `true` | Enables local debug behavior. | No |
| `DJANGO_ALLOWED_HOSTS` | `localhost,127.0.0.1` | Host header allowlist. | No |
| `DATABASE_URL` | `postgres://user:<password>@db:5432/app` | PostgreSQL connection. | Yes |
| `CSRF_TRUSTED_ORIGINS` | `http://127.0.0.1:8000` | Allowed local API origin, not a wildcard. | No |
| `SESSION_COOKIE_SECURE` | `false` | Secure cookie flag for local HTTP. | No |
| `COLLEGE_EMAIL_DOMAIN` | `sitm.example.in` | Student registration domain. | No |
| `MEDIA_ROOT` | `/app/private-media` | Private resume storage root. | No |
| `MAX_RESUME_BYTES` | `2097152` | Resume size limit. | No |
| `MAILPIT_SMTP_URL` | `smtp://mailpit:1025` | Optional local e-mail delivery. | No |
| `LOCAL_DEMO_NOW` | `2026-11-01T10:00:00+05:30` | Fixed business clock permitted only in local debug/demo mode; not session/token expiry. | No |
| `LOCAL_INSTANCE` | `campushire` | Project-scoped local volumes/config; acceptance uses a disposable isolated instance. | No |

Students MUST commit `.env.example`. They MUST NOT commit real `.env` files.

## Versioning and release

Use SemVer for student releases. Tag the final assessed version as `v1.0.0`. Keep a CHANGELOG with Added, Changed, Fixed, and Security sections. The final tag MUST point to a green `main` build.

## Dependency management

- Commit `uv.lock` for backend dependencies.
- Review dependency updates in a PR.
- Use Dependabot or manual weekly update checks.
- Do not update major versions during week 6 unless needed for a security fix.
- Record any allowed alternative in an ADR before merging.

## Definition of Done

A CampusHire story is Done only when these items are true:

1. The linked FR acceptance criteria pass.
2. Related BR and NFR checks have evidence.
3. Python tests cover normal, negative, and boundary behavior.
4. OpenAPI schemas, error keys, and JSON/CSV outputs match implemented contracts.
5. Role and ownership checks are enforced on every server endpoint.
6. CI is green on the PR.
7. No secrets, real personal data, or resume contents are committed.
8. README, ADRs, API notes, and sanitized response/test evidence are updated when needed.
9. The Friday demo can show the merged behavior from a clean local run, offline after downloads, with persisted stop/start evidence.

[Back to README](../README.md)
