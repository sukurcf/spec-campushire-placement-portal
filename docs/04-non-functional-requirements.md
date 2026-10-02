# Non-functional requirements

Purpose: This document defines measurable quality targets for CampusHire and how students verify them.

## Non-functional requirement table

| ID | Category | Target | Verification method |
|---|---|---|---|
| NFR-PERF-01 | API performance | 95% of authenticated list endpoints respond within 700 ms with 2,000 students, 80 postings, and 5,000 applications on the standard profile. | Local performance run with warm database and recorded summary. |
| NFR-PERF-02 | Dashboard performance | Dashboard summary panels respond within 2 seconds for 5,000 applications. | Measure backend timing and browser network timing. |
| NFR-PERF-03 | Resume upload | A valid 2 MB PDF upload completes within 5 seconds on local Wi-Fi. | Manual demo and integration test with sample PDF. |
| NFR-SCALE-01 | Data volume | MVP supports 2,000 students, 100 recruiters, 80 postings, and 5,000 applications. | Seed data run and smoke test. |
| NFR-REL-01 | Deadline closure reliability | A published posting closes no later than 5 minutes after `deadline_at` when the scheduled command runs. | Scheduled command log and test clock scenario. |
| NFR-REL-02 | State history durability | Every application status change creates exactly one timeline event in the database transaction. | Integration test that checks state and timeline count. |
| NFR-SEC-01 | Session security | Session cookies use `HttpOnly`, `SameSite=Lax`, and `Secure` outside local HTTP. | Browser devtools screenshot or automated header assertion. |
| NFR-SEC-02 | CSRF protection | 100% of unsafe API requests reject missing or invalid CSRF tokens. | API tests for `POST`, `PUT`, `PATCH`, and `DELETE`. |
| NFR-SEC-03 | File upload security | Resume processing rejects non-PDF content and files over 2 MB. Stored files are not web-public. | API tests and storage path review. |
| NFR-SEC-04 | Authorization | Recruiters cannot access postings, applicants, resumes, or status updates for other companies. | API tests for cross-company access. |
| NFR-PRIV-01 | Privacy | The app stores only placement data needed for this project. It never logs passwords, CSRF tokens, or resume contents. | Code review checklist and log sample review. |
| NFR-PRIV-02 | DPDP awareness | README and privacy note explain purpose, retention, correction, and deletion contact for student data. | Documentation review. |
| NFR-MAINT-01 | Backend maintainability | Ruff and mypy pass on application packages. Migrations and generated files are excluded from mypy. | CI output. |
| NFR-MAINT-02 | Frontend maintainability | ESLint and Prettier pass. TypeScript uses strict mode when FR-FEQ-01 is selected. | CI output. |
| NFR-OBS-01 | Logging | Backend logs include timestamp, level, request ID, user ID when authenticated, path, status, and duration. | Sample JSON log inspection. |
| NFR-OBS-02 | Query evidence | Student records `EXPLAIN ANALYZE` evidence for job search and branch-wise placement report. | Notes in student repository documentation. |
| NFR-USE-01 | Responsive UI | All Must pages work at 360 px, 768 px, and 1280 px widths without horizontal scrolling. | Manual checklist and Playwright viewport test. |
| NFR-ACC-01 | Accessibility basics | Must pages have keyboard navigation, visible focus, and labels for form fields. | Manual checklist for each Must page at 360 px, 768 px, and 1280 px. Automated axe checks are Should. |
| NFR-PORT-01 | Local portability | System runs on Windows 11 with WSL2 Ubuntu, macOS, and Linux using Docker Compose. | Trainer or student setup evidence. |
| NFR-LIC-01 | Licence compliance | Must tools use free licences suitable for public student portfolios. | Dependency list review. |
| NFR-TEST-01 | Backend coverage | Backend line coverage ≥ 85%; branch coverage ≥ 75%. Eligibility module has 100% branch coverage. | pytest coverage report. |
| NFR-TEST-02 | Frontend coverage | Frontend line coverage ≥ 60% for React components and API client. | Vitest coverage report. |
| NFR-TEST-03 | E2E coverage | At least 4 critical Playwright journeys run locally with saved HTML report. | Playwright report committed or linked in student evidence. |
| NFR-HW-01 | Lite hardware | Must scope runs on an 8 GB RAM laptop with the lite profile below. | Trainer pre-check before week 1. |

## Security requirements by risk

| Risk | CampusHire control |
|---|---|
| Broken access control | Role guards in the SPA and server-side permission checks on every API. |
| CSRF | Django CSRF cookie and `X-CSRFToken` header on unsafe requests. |
| XSS | React escaping, no raw HTML rendering for posting descriptions in Must scope. |
| Insecure upload | PDF content check, 2 MB limit, private media storage, authorized download view. |
| Sensitive logging | Logs omit passwords, session IDs, CSRF tokens, resume content, and raw cookies. |
| SQL injection | Django ORM and parameterized handwritten SQL. No string concatenation in SQL. |
| Weak password choices | Django password validators and minimum 12 characters. |

## Hardware profiles

### Standard profile

| Resource | Value |
|---|---|
| Laptop RAM | 16 GB recommended |
| Docker memory | 8 GB |
| Docker swap | 4 GB |
| Services | Django, React dev server, PostgreSQL, Mailpit, optional Playwright browser |
| PostgreSQL memory target | Up to 1 GB container memory |
| Browser tests | Playwright can run locally and in CI |

### Lite profile for 8 GB laptops

| Resource | Exact limit |
|---|---|
| Docker memory | 4 GB |
| Docker swap | 4 GB |
| PostgreSQL container | Limit to 768 MB |
| Django container | Limit to 768 MB |
| React dev server | Run on host Node.js or one 512 MB container |
| Mailpit | Run only when e-mail Should items are active; limit 256 MB |
| Browser tests | Run one Playwright worker at a time |
| Switched off | Production-like Nginx/Gunicorn profile, charts seed load above MVP, optional task queue |

### Windows WSL2 guidance

| Laptop RAM | `.wslconfig` memory | `.wslconfig` swap |
|---|---|---|
| 8 GB | 4 GB | 4 GB |
| 16 GB | 8 GB | 4 GB |

The trainer MUST validate the lite profile on an 8 GB laptop before week 1. Students SHOULD keep only one browser, editor, and Docker Desktop open during Playwright runs on 8 GB laptops.

## Coverage and test targets

| Scope | Line coverage | Branch coverage | Notes |
|---|---|---|---|
| Backend application packages | ≥ 85% | ≥ 75% | Exclude migrations and generated OpenAPI files. |
| Eligibility module | 100% | 100% | Must cover every reason code. |
| Placement-policy module | 100% if built | 100% if built | Applies to dream-offer Should item. |
| Frontend | ≥ 60% | Not enforced | Measure React components and API client. |
| E2E | 4 journeys Must | 8 journeys Should | Save local HTML report. |

## Critical verification scenarios

| Scenario | Requirement links | Evidence |
|---|---|---|
| Student blocked by incomplete profile | FR-PROFILE-01, FR-ELIG-01 | API test and UI screenshot |
| Student applies successfully | FR-APP-01 | Playwright journey |
| Recruiter cross-company access denied | FR-REC-01, NFR-SEC-04 | API security test |
| Deadline closes posting | FR-POST-02, NFR-REL-01 | Unit or integration test with controlled time |
| Dashboard metrics correct | FR-TPO-01, NFR-PERF-02 | Report test and timing note |
| PDF content rejected | FR-PROFILE-02, NFR-SEC-03 | Upload test |

## Reliability and recovery rules

1. Application state changes MUST be atomic with timeline creation.
2. Deadline closure MAY run through a management command and MUST also be enforced during apply.
3. Notification delivery, when built, is at least once with a stable de-duplication key.
4. If resume storage fails, the old current resume MUST remain active.
5. If dashboard report generation fails, other dashboard panels SHOULD still render.

[Back to README](../README.md)
