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
| Test | `test/<area-short-name>` | `test/playwright-student-apply` |
| Docs | `docs/<topic>` | `docs/api-client-adr` |
| Chore | `chore/<tool-or-config>` | `chore/ruff-precommit` |

## Conventional Commits

| Type | Use | Example |
|---|---|---|
| `feat` | New CampusHire behavior | `feat: add eligibility reasons` |
| `fix` | Bug fix | `fix: reject fake PDF` |
| `test` | Test-only change | `test: deny other company` |
| `docs` | Student repository docs | `docs: add session ADR` |
| `chore` | Tooling or dependency work | `chore(ci): add frontend coverage gate` |
| `refactor` | Design change without behavior change | `refactor: split transitions` |

## Pull request checklist

- The PR title uses a Conventional Commit style.
- The PR links the FR, BR, NFR, or test IDs it changes.
- Backend tests pass for affected Django apps.
- Frontend tests pass for affected React routes.
- API errors use documented identifiers such as `resume-not-pdf`.
- Screenshots or Playwright traces are attached for UI behavior.
- No secrets, real student data, or resume contents appear in logs.
- The author states significant AI help, if used.
- The README or ADRs are updated when behavior changes.

## Branch protection

| Protection | Required setting |
|---|---|
| Required checks | Backend CI and frontend CI must pass. |
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
| Frontend lint | ESLint 10.x | Enforce React hooks rules and TypeScript-safe patterns. |
| Frontend format | Prettier 3.x | Keep React and CSS formatting consistent. |
| Frontend tests | Vitest 5.x and React Testing Library 16.x | Enforce frontend line coverage 60%. |
| E2E | Playwright 1.63.x | Run at least 4 local journeys and save the HTML report. |
| Accessibility | @axe-core/playwright 4.13.x | Should item; fail CI for critical axe issues when enabled. |

## CI pipeline table

| Job | Trigger | Steps in words | Gate or failure condition |
|---|---|---|---|
| Backend lint and type check | Pull request and push to `main` | Install Python with uv, restore lockfile, run Ruff, and run mypy. | Fails on lint errors or type errors. |
| Backend tests | Pull request and push to `main` | Start PostgreSQL 18, run Django migrations, run pytest with coverage. | Fails below 85% line, 75% branch, or 100% eligibility branch coverage. |
| Backend security | Pull request and push to `main` | Run pip-audit and Gitleaks on the repository. | Fails on high-risk packages or committed secrets. |
| Frontend lint and type check | Pull request and push to `main` | Install Node 24 dependencies from lockfile, run ESLint, Prettier check, and TypeScript check. | Fails on lint, format, or type errors. |
| Frontend tests | Pull request and push to `main` | Run Vitest with coverage and mocked API responses. | Fails below 60% frontend line coverage. |
| Build images | Pull request and push to `main` | Build Django and frontend production images with pinned base tags. | Fails if an image cannot build. |
| Container scan | Pull request and push to `main` | Scan built images with Trivy. | Fails on critical vulnerabilities without trainer-approved exception. |
| Playwright smoke | Pull request to `main` | Should run login, student apply, recruiter status, and TPO dashboard journeys. | Must run locally with report; CI is Should. |

## Docker and Compose requirements

- Compose MUST start PostgreSQL, Django, and the React development server for local work.
- The Vite development server proxies `/api` and `/admin` to Django at `http://localhost:8000`, so the browser uses one origin, `http://localhost:5173`.
- Session cookies and CSRF work without CORS in local development.
- Mailpit SHOULD be available through a profile for e-mail features.
- The production-like Should profile uses Gunicorn, built React assets, and Nginx. Nginx serves the SPA and proxies `/api`.
- Every Docker image tag MUST be pinned to a version.
- The README MUST state the standard and lite resource settings.
- Resume media MUST be private and served through an authorized Django view.
- A task runner SHOULD expose setup, lint, test, run, seed, e2e, and clean commands.

## Environment variables

| Name | Example value | Purpose | Secret |
|---|---|---|---|
| `DJANGO_SECRET_KEY` | `change-me-local-only` | Django signing key for local development. | Yes |
| `DJANGO_DEBUG` | `true` | Enables local debug behavior. | No |
| `DJANGO_ALLOWED_HOSTS` | `localhost,127.0.0.1` | Host header allowlist. | No |
| `DATABASE_URL` | `postgres://user:<password>@db:5432/app` | PostgreSQL connection. | Yes |
| `CSRF_TRUSTED_ORIGINS` | `http://localhost:5173` | Same-site SPA CSRF origin. | No |
| `SESSION_COOKIE_SECURE` | `false` | Secure cookie flag for local HTTP. | No |
| `COLLEGE_EMAIL_DOMAIN` | `sitm.example.in` | Student registration domain. | No |
| `MEDIA_ROOT` | `/app/private-media` | Private resume storage root. | No |
| `MAX_RESUME_BYTES` | `2097152` | Resume size limit. | No |
| `MAILPIT_SMTP_URL` | `smtp://mailpit:1025` | Optional local e-mail delivery. | No |
| `FRONTEND_API_BASE_URL` | `/api` | React API base path. | No |
| `PLAYWRIGHT_BASE_URL` | `http://localhost:5173` | E2E test target. | No |

Students MUST commit `.env.example`. They MUST NOT commit real `.env` files.

## Versioning and release

Use SemVer for student releases. Tag the final assessed version as `v1.0.0`. Keep a CHANGELOG with Added, Changed, Fixed, and Security sections. The final tag MUST point to a green `main` build.

## Dependency management

- Commit `uv.lock` for backend dependencies.
- Commit the frontend package lockfile.
- Review dependency updates in a PR.
- Use Dependabot or manual weekly update checks.
- Do not update major versions during week 6 unless needed for a security fix.
- Record any allowed alternative in an ADR before merging.

## Definition of Done

A CampusHire story is Done only when these items are true:

1. The linked FR acceptance criteria pass.
2. Related BR and NFR checks have evidence.
3. Backend or frontend tests cover normal, negative, and boundary behavior.
4. UI changes work at 360 px, 768 px, and 1280 px.
5. Role checks are enforced on the server, not only in React.
6. CI is green on the PR.
7. No secrets, real personal data, or resume contents are committed.
8. README, ADRs, API notes, or screenshots are updated when needed.
9. The Friday demo can show the merged behavior from a clean run.

[Back to README](../README.md)
