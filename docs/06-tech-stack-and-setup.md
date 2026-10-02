# Tech stack and setup

Purpose: This document defines the approved CampusHire tools, reference versions, local setup, hardware limits, accounts, and learning order.

## Mandatory stack table

| Category | Tool | Reference version | Purpose | Why |
|---|---|---|---|---|
| Language | Python | 3.12.x | Backend runtime | It is the cohort baseline and supports Django 5.2. |
| Python packages | uv | 0.12.x | Dependency and virtual environment manager | It creates repeatable installs with `uv.lock`. |
| Backend framework | Django | 5.2.x LTS | Web app, ORM, admin, sessions, CSRF | It is LTS and matches the project learning goal. |
| API framework | Django REST Framework | 3.18.x | REST API serialization and views | It is the standard Django API tool. |
| API docs | drf-spectacular | 0.30.x | OpenAPI schema | It documents endpoints for the React client. |
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
| Compose | Docker Compose | v5.x | Multi-service local runs | One command can start API, DB, SPA, and Mailpit. |
| Local e-mail | Mailpit | 1.31.x | Optional e-mail testing | It replaces unmaintained MailHog. |
| Frontend runtime | Node.js | 24.x LTS | React toolchain | It is the cohort frontend baseline. |
| Frontend framework | React | 19.x | SPA UI | It matches current job requirements. |
| Frontend language | TypeScript | 6.0.x | Typed UI code | It makes API contracts and props safer. |
| Build tool | Vite | 8.x | Frontend dev server and build | It gives a fast React workflow. |
| Routing | React Router | 8.x | SPA route groups and guards | Roles map clearly to route groups. |
| Forms | React Hook Form | 7.x | Profile, posting, and login forms | It manages validation and form state. |
| Styling | Tailwind CSS | 4.x | Responsive UI styles | It supports 360 px, 768 px, and 1280 px layouts. |
| Server state | TanStack Query | 5.x | Should item API caching | It improves loading and retry behavior. |
| Validation | Zod | 4.x | Should item form schemas | It aligns frontend validation with API errors. |
| Charts | Recharts | 3.x | Should item dashboard charts | It visualizes placement numbers. |
| Frontend tests | Vitest | 5.x | Unit and component tests | It integrates with Vite. |
| Component tests | React Testing Library | 16.x | User-focused React tests | It checks forms and route guards. |
| API mocking | MSW | 3.x | Frontend test API mocks | It tests UI states without a live backend. |
| E2E tests | Playwright | 1.63.x | Four required user journeys | It saves the local HTML report. |
| Accessibility | @axe-core/playwright | 4.13.x | Should item axe checks | It checks WCAG 2.2 AA basics. |
| Frontend lint | ESLint | 10.x | TypeScript and React linting | It catches unsafe UI patterns. |
| Frontend format | Prettier | 3.x | Frontend formatting | It keeps PR diffs small. |

## Allowed alternatives

Each alternative MUST have an ADR before adoption.

| Default | Allowed alternative | ADR question |
|---|---|---|
| Django session auth | JWT for a separate API client | Why does CampusHire need cross-site token auth instead of same-site sessions? |
| PostgreSQL 18.x | PostgreSQL 17.x | What local or CI limitation prevents PostgreSQL 18.x? |
| Docker Desktop | Colima on macOS or native Docker Engine on Linux | How will Compose networking and resource limits stay equivalent? |
| React Hook Form | Formik | How will large profile forms stay performant and typed? |
| Tailwind CSS | CSS Modules | How will responsive layouts stay consistent across role dashboards? |
| TanStack Query | Plain typed fetch client | How will loading, retry, and cache invalidation remain clear? |

## Not allowed

| Tool or approach | Reason |
|---|---|
| Django 6.x | It is not the selected LTS line for this cohort. |
| SQLite as the main database | It does not demonstrate PostgreSQL indexes and window functions. |
| JWT as the default auth model | The learning goal is sessions, CSRF, and same-site SPA security. |
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
6. Install Node.js 24 LTS inside WSL.
7. Install VS Code and the WSL, Python, Django, ESLint, Prettier, and Playwright extensions.
8. Clone `campushire` inside the WSL filesystem, not under `/mnt/c`.
9. Start PostgreSQL, Django, and React through the documented Compose or task runner commands.
10. Run only the skeleton checks that exist before the first PR. Complete the four Must Playwright journeys by Friday 30 October.

The Vite development server runs at `http://localhost:5173`. It proxies `/api` and `/admin` to Django at `http://localhost:8000`, so local development does not need CORS. In the production-like profile, Nginx serves the SPA and proxies `/api`.

### `.wslconfig` cohort decision

| Laptop RAM | `memory` | `swap` | Reason |
|---|---|---|---|
| 8 GB | `4GB` | `4GB` | Leaves RAM for Windows, VS Code, browser, and Docker. |
| 16 GB | `8GB` | `4GB` | Gives Docker and Playwright enough headroom. |

Change these values only with a written reason in the student README.

### macOS

1. Install Homebrew or use official installers.
2. Install Git, Python 3.12, uv, Node.js 24 LTS, and Docker Desktop.
3. Set Docker Desktop memory to the selected hardware profile.
4. Install VS Code extensions for Python, Django, ESLint, Prettier, and Playwright.
5. Clone `campushire`, install dependencies, and run backend and frontend checks.

### Linux

1. Install Git, Python 3.12, uv, Node.js 24 LTS, Docker Engine, and Docker Compose.
2. Add your user to the Docker group only if the trainer approves the local setup.
3. Set Compose service memory limits according to the selected profile.
4. Clone `campushire`, install dependencies, and run the verification commands.

## Hardware profiles

| Profile | Laptop | Docker memory | Docker swap | Services | Limits |
|---|---|---|---|---|---|
| Standard | 16 GB RAM | 8 GB | 4 GB | Django, React, PostgreSQL, Mailpit, Playwright browser | PostgreSQL up to 1 GB; Django up to 1 GB; React up to 1 GB. |
| Lite | 8 GB RAM | 4 GB | 4 GB | Django, PostgreSQL, React, Mailpit only when needed | PostgreSQL 768 MB; Django 768 MB; React 512 MB; Mailpit 256 MB. |

In the lite profile, run one Playwright worker. Switch off the production-like Nginx and Gunicorn profile. Seed only the MVP data volume unless a performance run needs more.

## Trainer pre-check

- Verify the lite profile on an 8 GB laptop before week 1.
- Confirm PostgreSQL starts within the 768 MB limit.
- Confirm Django and React can run together.
- Confirm one Playwright journey completes with one worker.
- Confirm a 2 MB PDF upload completes within 5 seconds.
- Confirm Docker cleanup instructions free disk space after the demo.

## Free accounts needed

| Account | Purpose |
|---|---|
| GitHub | Student repository, issues, pull requests, Actions, and Projects board. |
| Docker Hub | Optional base image pulls if rate limits affect class Wi-Fi. |
| Browser vendor account | None required for local Playwright. |
| Cloud provider | Not required. Use only for the optional deployment Could item. |

## Suggested learning order

| Order | Topic | Hours | Outcome |
|---|---:|---:|---|
| 1 | GitHub Flow, Conventional Commits, and PR review | 2 | First PR follows the project rules. |
| 2 | Django models, admin, sessions, and CSRF | 4 | Student can explain session versus JWT. |
| 3 | DRF serializers, permissions, and filters | 2 | Student can build role-protected APIs. |
| 4 | PostgreSQL joins, indexes, window functions, and `EXPLAIN ANALYZE` | 2 | Student can justify dashboard indexes. |
| 5 | React components, props, state, hooks, and effects | 10 | Student can build role screens and loading states. |
| 6 | TypeScript basics, typed API clients, and form types | 8 | Student can type route data and API responses. |
| 7 | Testing with pytest, Vitest, and Playwright | 1 | Student can meet coverage and E2E requirements. |
| 8 | Docker Compose, CI, and security scans | 1 | Student can reproduce the app from a clean clone. |

The compulsory learning total is 30 hours. It has 18 hours of frontend learning and 12 hours of other setup, backend, SQL, testing, and CI learning. All 30 hours count inside Must scope during weeks 1 to 4.

[Back to README](../README.md)
