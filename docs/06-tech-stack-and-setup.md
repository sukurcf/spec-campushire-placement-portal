# Tech stack and setup

Purpose: This document defines the approved CampusHire tools, reference versions, local setup, hardware limits, accounts, and learning order.

## Mandatory stack table

| Category | Tool | Reference version | Purpose | Why |
|---|---|---|---|---|
| Language | Python | 3.12.x | Backend runtime | It is the cohort baseline and supports Django 5.2. |
| Python packages | uv | 0.12.x | Dependency and virtual environment manager | It creates repeatable installs with `uv.lock`. |
| Backend framework | Django | 5.2.x LTS | Web app, ORM, admin, sessions, CSRF | It is LTS and matches the project learning goal. |
| API framework | Django REST Framework | 3.18.x | REST API serialization and views | It is the standard Django API tool. |
| API docs | drf-spectacular | 0.30.x | OpenAPI schema | It defines contracts for Python tests and API clients. |
| Filters | django-filter | 26.x | Job and applicant filters | It keeps filter behavior consistent. |
| Database | PostgreSQL | 18.x | Main relational database | It supports SQL, indexes, and window functions. |
| Test framework | pytest | 9.1.x | Backend tests | It supports fast unit and API tests. |
| Django tests | pytest-django | 4.14.x | Django test integration | It manages test database setup. |
| Test data | factory_boy | 3.3.x | Fictional test records | It reduces duplicate test data setup. |
| Coverage | pytest-cov and coverage | 7.1.x and 7.16.x | Coverage reports | CI enforces backend coverage gates. |
| Types | mypy and django-stubs | 2.4.x and 5.2.x | Backend type checks | They catch model and service mistakes early. |
| Lint and format | Ruff | 0.16.x | Python linting and formatting | It is fast and replaces several Python tools. |
| Hooks | pre-commit | 4.6.x | Local checks before commits | It prevents avoidable PR failures. |
| Security scan | pip-audit | 2.10.x | Python dependency audit | It finds known package vulnerabilities. |
| Secrets scan | Gitleaks | 8.30.x | Secret detection | It protects sessions, keys, and sample data. |
| Container scan | Trivy | 0.75.x | Image and filesystem scan | It checks Docker images and dependencies. |
| Containers | Docker Engine | 29.x | Local services | It makes setup reproducible. |
| Compose | Docker Compose | v5.x | Multi-service local runs | One entry point starts API and DB with an optional local e-mail profile. |
| Optional local e-mail | Mailpit | 1.31.x | E-mail testing | Its built-in console may be inspected; no console implementation or styling is required. |

## Allowed alternatives

Each alternative MUST have an ADR before adoption.

| Default | Allowed alternative | ADR question |
|---|---|---|
| PostgreSQL 18.x | PostgreSQL 17.x | What local or CI limitation prevents PostgreSQL 18.x? |
| Docker Desktop | Colima on macOS or native Docker Engine on Linux | How will Compose networking and resource limits stay equivalent? |

## Not allowed

| Tool or approach | Reason |
|---|---|
| Django 6.x | It is not the selected LTS line for this cohort. |
| SQLite as the main database | It does not demonstrate PostgreSQL indexes and window functions. |
| Replacing session auth with JWT | CampusHire MUST retain Django sessions and explicit CSRF checks. |
| Student-built interfaces or public deployment tasks | Version 1.1 assesses backend APIs and local operations only, including optional work. |
| `python-magic` for PDF checks | Native `libmagic` is hard for freshers on Windows. Check `%PDF-` bytes and size in pure Python. |
| MailHog | It is unmaintained; use Mailpit. |
| Docker image tag `latest` | Builds become non-repeatable. |
| Public media URL for resumes | It bypasses resume authorization rules. |
| Real student or company data | The project must use fictional data only. |
| Bitnami free images or charts as mandatory tools | The free catalog changed in 2025. |

## Local setup checklist

### Windows 11 with WSL2 Ubuntu 24.04

1. Enable WSL2 and install Ubuntu 24.04.
2. Create `%UserProfile%\.wslconfig` with the memory values below.
3. Install Docker Desktop with the WSL2 backend enabled.
4. Install Git inside WSL and configure your GitHub identity.
5. Install Python 3.12 and uv inside WSL.
6. Install VS Code and the WSL, Python, Django, and Ruff extensions if using that editor.
7. Clone the implementation repository `campushire` inside the WSL filesystem, not under `/mnt/c`.
8. Run the single start entry point below; it initializes migrations and fictional fixtures.
9. Run the health checks and deterministic API demonstration.
10. Run pytest and stop through the documented stop entry point.

Commands below are deliverables students MUST implement in their application repository. They are not executable in this Markdown-only specification repository.

### `.wslconfig` cohort decision

| Host RAM | WSL memory | WSL swap |
|---|---|---|
| 8 GB | 4 GB | 4 GB |
| 16 GB | 8 GB | 4 GB |

Change these values only with a written reason in the student README.

### macOS

1. Install Homebrew or use official installers.
2. Install Git, Python 3.12, uv, and Docker Desktop.
3. Set Docker Desktop memory to the selected hardware profile.
4. Install editor extensions for Python, Django, and Ruff if desired.
5. Clone `campushire` and use the same start, health, test, and stop commands below.

### Linux

1. Install Git, Python 3.12, uv, Docker Engine, and Docker Compose.
2. Add your user to the Docker group only if the trainer approves the local setup.
3. Set Compose service memory limits according to the selected profile.
4. Clone `campushire` and use the same local entry points and verification commands below.

## Local operation contract

### Initial downloads and local-only runtime

The first setup needs internet to clone the student repository and download pinned images and `uv.lock` dependencies. GitHub submission and hosted CI also need internet. After those downloads, start, stop, the core API demo, and runtime tests MUST use cached images/packages and recorded local fixtures only. No paid account, public hostname, API key, or cloud resource is required. Local verification MUST not depend on GitHub, a registry, or an e-mail provider.

### Single start and stop entry points

From the implementation repository root, students MUST provide:

```bash
sh scripts/local-start.sh --profile lite
sh scripts/local-stop.sh
```

The alternative `--profile standard` uses the same ports and persistence contract with the larger resource budget. On WSL2 run these commands inside Ubuntu; on macOS/Linux use the POSIX shell.

Start MUST check Docker and occupied host ports, create an untracked local configuration with generated secrets on first use, prepare locked dependencies/images if initially online, and start Compose services. Initial preparation includes host `uv sync --frozen` for the host-side local pytest driver as well as dependencies inside the API image. Cached restarts MUST use existing local images without a build or registry pull. Missing offline caches MUST give a nonzero `local-start: required image or dependency is not cached` diagnostic rather than contact external services.

Start MUST wait for PostgreSQL, run `uv run python manage.py migrate --noinput` inside the API container, and run `uv run python manage.py seed_local_demo`. Seed is idempotent: it creates missing fictional records but never overwrites user edits, deletes applications, or rotates existing local credentials on restart. It then waits for readiness and prints `CampusHire ready at http://127.0.0.1:8000`. A database timeout MUST exit nonzero with `local-start: PostgreSQL unavailable at 127.0.0.1:15432`; an occupied API port reports `local-start: port 8000 is already in use`.

Stop MUST run Compose shutdown without `--volumes` and print `CampusHire stopped; local data preserved`. Neither entry point deletes database or resume volumes.

### Fixed loopback ports and persistence

| Service | Host binding | Container address | Default |
|---|---|---|---|
| Django API, health, built-in admin | `127.0.0.1:8000` | `api:8000` | On |
| PostgreSQL | `127.0.0.1:15432` | `db:5432` | On |
| Mailpit SMTP | `127.0.0.1:11025` | `mailpit:1025` | Optional local e-mail profile |
| Mailpit built-in console | `127.0.0.1:18025` | `mailpit:8025` | Optional operations inspection only |

Container servers may listen on their container interfaces; host-published ports MUST bind to loopback, never `0.0.0.0`. Compose uses named volumes `campushire_pgdata` and `campushire_private_media`. PostgreSQL persists accounts, sessions, postings, applications, and events; the second volume persists private PDFs. Test databases are separate from demo data.

Default `LOCAL_INSTANCE=campushire` selects the named volumes above and `.local/demo-credentials.json`. Acceptance uses `LOCAL_INSTANCE=campushire_acceptance`, separate `campushire_acceptance_pgdata`/`campushire_acceptance_private_media` volumes, and `.local/acceptance/demo-credentials.json`. Only one instance may use the fixed host ports at a time; the test driver must refuse to interfere with an unrelated running stack. Students MUST compare application IDs, event counts, and PDF SHA-256 before and after stop/start.

Reset is a separate destructive command:

```bash
sh scripts/local-reset.sh --confirm-delete-local-data
```

Without that flag, reset MUST fail nonzero with `local-reset: explicit confirmation required` and leave all data intact. With confirmation, it removes only this project's local volumes and generated demo credentials; it must not remove other projects' volumes. A later start recreates migrations and seed data.

### Seed, recorded fixtures, and deterministic demonstration

The seed MUST load the six branches from document 07, Python/SQL/Docker skills, Aarav/Diya/Farhan, Meera/Kavya, approved recruiter Rohan, and the three fictional company postings. Aarav starts `VERIFIED` with complete required fields, a private recorded PDF, and no application to Navira. Navira is `PUBLISHED`, allows CSE/ISE/ECE for 2027, CGPA `7.00`, and zero backlogs.

Use fixed Navira posting UUID `00000000-0000-4000-8000-000000000101` (alias `post-nav-ft-2027`). Keep generated test credentials in untracked `.local/demo-credentials.json`; never commit them or print passwords in logs. The local demo profile sets business clock `LOCAL_DEMO_NOW=2026-11-01T10:00:00+05:30`, so the recorded November deadlines do not become stale. This clock override is allowed only with local debug/demo settings; non-local settings MUST reject it. Security token/session expiry still uses real time. Tests inject a controlled business clock explicitly.

Students MUST commit fictional seed records and local PDF fixtures in their implementation repository: valid `aarav-resume.pdf` of `1,468,006` bytes with `%PDF-` signature, plain-text `notes.pdf`, and an oversized `2,100,000` byte PDF. Record each fixture's SHA-256 at creation. Optional e-mail targets Mailpit, not live SMTP. External integrations, if discussed, are separate opt-in modes and never a runtime/test dependency.

Health checks:

```bash
curl --fail --silent http://127.0.0.1:8000/health/live/
curl --fail --silent http://127.0.0.1:8000/health/ready/
```

Expected status/body pairs are `200 {"status":"ok"}` and `200 {"status":"ready","database":"ok","media":"ok"}`. If PostgreSQL is stopped, readiness returns `503 {"status":"not_ready","database":"unavailable","media":"ok"}`.

For the deterministic core operation, use Python tests or a cookie-aware API client: GET `/api/csrf/`, POST login as seeded Aarav with `X-CSRFToken`, retain the new CSRF cookie after login, then GET `/api/jobs/00000000-0000-4000-8000-000000000101/eligibility/`. Expected status is `200` and exact JSON is:

```json
{"label":"Eligible","reason_ids":[],"messages":[]}
```

With external egress disabled but loopback/Compose networking retained, the same result MUST hold. Run the student-implemented local acceptance suite with cached dependencies:

```bash
uv run --offline pytest -q backend/tests/local/ --junitxml=docs/test-reports/local.xml
```

Document 09 links this contract to TC-LOCAL-001 through TC-LOCAL-006 and FR-LOCAL-01.

The host lifecycle driver runs unit/API/database runtime selections without recursively invoking `backend/tests/local/`. It must remain alive while it stops/restarts the API containers.

## Hardware profiles

| Profile | Laptop | Docker memory | Docker swap | Services | Limits |
|---|---|---|---|---|---|
| Standard | 16 GB RAM, four cores, 20 GB free disk | 8 GB | 4 GB | Django, PostgreSQL, optional local Mailpit | PostgreSQL up to 1 GB; Django up to 1 GB; Mailpit 256 MB. |
| Lite | 8 GB RAM, four cores, 20 GB free disk | 4 GB | 4 GB | Django, PostgreSQL; Mailpit only when needed | PostgreSQL 768 MB; Django 768 MB; Mailpit 256 MB. |

These are proposed resource limits, not measured results. In the lite profile, run one Python test worker and keep optional e-mail services off. Use the small local demo fixture; load the performance volume only into a separate test database for a measured run.

## Trainer pre-check

- Verify the lite profile on an 8 GB, four-core laptop with 20 GB free disk before week 1.
- Confirm PostgreSQL starts within the 768 MB limit.
- Confirm the single start entry point migrates and seeds on a clean implementation clone.
- Confirm both health responses and the exact Aarav eligibility JSON.
- Block external egress after initial downloads and rerun the demo and runtime tests.
- Confirm TC-LOCAL-003 preserves the application, event count, and private PDF hash across stop/start.
- Confirm invalid CGPA, unavailable PostgreSQL/storage, occupied ports, and unconfirmed reset return the documented errors.
- Confirm a 2 MB PDF upload completes within 5 seconds.
- Confirm normal stop retains data and only explicit reset deletes this project's local volumes.

## Free accounts needed

| Account | Purpose |
|---|---|
| GitHub | Student repository, issues, pull requests, Actions, and Projects board. |
| Container registry | No account required for local work; initial public base-image downloads need internet. |
| API client | No account or hosted synchronization required. |

## Suggested learning order

| Order | Topic | Hours | Outcome |
|---|---:|---:|---|
| 1 | GitHub Flow, Conventional Commits, and PR review | 2 | First PR follows the project rules. |
| 2 | Django models, admin, sessions, and CSRF | 4 | Student can explain session versus JWT. |
| 3 | DRF serializers, permissions, and filters | 2 | Student can build role-protected APIs. |
| 4 | PostgreSQL joins, indexes, window functions, and `EXPLAIN ANALYZE` | 6 | Student can justify report indexes and distinct-count semantics. |
| 5 | Python services, Decimal, datetime, validation, and exceptions | 6 | Student can implement exact eligibility and CTC calculations. |
| 6 | API contracts, permission matrices, and JSON/CSV serialization | 6 | Student can document and test role-protected operations. |
| 7 | pytest, database fixtures, CSRF tests, and integration flows | 2 | Student can meet coverage and API-flow gates. |
| 8 | Docker Compose, offline fixtures, persistence, and CI | 2 | Student can reproduce clean startup and safe stop/start. |

The compulsory learning total remains 30 hours. Removed frontend learning time is reassigned to Python, SQL, API contracts, testing, and local operations. All 30 hours count inside Must scope during weeks 1 to 4; no interface or public deployment exercise is a bonus.

[Back to README](../README.md)
