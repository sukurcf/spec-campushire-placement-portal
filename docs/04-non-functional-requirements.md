# Non-functional requirements

Purpose: This document defines measurable quality targets for CampusHire and how students verify them.

## Non-functional requirement table

| ID | Category | Target | Verification method |
|---|---|---|---|
| NFR-PERF-01 | API performance | 95% of authenticated list endpoints respond within 700 ms with 2,000 students, 80 postings, and 5,000 applications on the standard profile. | Local performance run with warm database and recorded summary. |
| NFR-PERF-02 | Report performance | TPO summary responds within 2 seconds for 5,000 applications. | Measure HTTP duration using Python against the loopback API. |
| NFR-PERF-03 | Resume upload | A valid 2 MB PDF upload completes within 5 seconds on loopback. | Python integration test with recorded PDF fixture. |
| NFR-SCALE-01 | Data volume | MVP supports 2,000 students, 100 recruiters, 80 postings, and 5,000 applications. | Seed data run and smoke test. |
| NFR-REL-01 | Deadline closure reliability | A published posting closes no later than 5 minutes after `deadline_at` when the scheduled command runs. | Scheduled command log and test clock scenario. |
| NFR-REL-02 | State history durability | Every application status change creates exactly one timeline event in the database transaction. | Integration test that checks state and timeline count. |
| NFR-REL-03 | Local persistence | PostgreSQL records, sessions, and private resume files survive the documented stop/start; only a confirmed reset deletes them. | TC-LOCAL-003 and TC-LOCAL-006 compare IDs, event counts, and PDF hashes. |
| NFR-SEC-01 | Session security | Session cookies use `HttpOnly`, `SameSite=Lax`, and `Secure` outside local HTTP. | Automated response-cookie assertions with local and secure test settings. |
| NFR-SEC-02 | CSRF protection | 100% of unsafe API requests reject missing or invalid CSRF tokens. | API tests for `POST`, `PUT`, `PATCH`, and `DELETE`. |
| NFR-SEC-03 | File upload security | Resume processing rejects non-PDF content and files over 2 MB. Stored files are not web-public. | API tests and storage path review. |
| NFR-SEC-04 | Authorization | Recruiters cannot access postings, applicants, resumes, or status updates for other companies. | API tests for cross-company access. |
| NFR-PRIV-01 | Privacy | The app stores only placement data needed for this project. It never logs passwords, CSRF tokens, or resume contents. | Code review checklist and log sample review. |
| NFR-PRIV-02 | DPDP awareness | README and privacy note explain purpose, retention, correction, and deletion contact for student data. | Documentation review. |
| NFR-MAINT-01 | Backend maintainability | Ruff and mypy pass on application packages. Migrations and generated files are excluded from mypy. | CI output. |
| NFR-DATA-01 | Report correctness | JSON and CSV agree on counts, decimal precision, null/empty CTC values, distinct students, accepted full-time filtering, and company ranks. | Python report equivalence and boundary tests. |
| NFR-OBS-01 | Logging | Backend logs include timestamp, level, request ID, user ID when authenticated, path, status, and duration. | Sample JSON log inspection. |
| NFR-OBS-02 | Query evidence | Student records `EXPLAIN ANALYZE` evidence for job search and branch-wise placement report. | Notes in student repository documentation. |
| NFR-LOCAL-01 | Offline local runtime | After dependency/image downloads, seeded core API operations and runtime tests use only local fixtures without external network access. | TC-LOCAL-002 with external egress disabled, loopback and Compose network retained. |
| NFR-LOCAL-02 | Actionable local errors | Invalid input returns documented 400/409 keys; unavailable DB/storage returns 503 keys; startup reports occupied ports and fails nonzero without deleting data. | TC-API-008, TC-LOCAL-004, and storage failure integration test. |
| NFR-PORT-01 | Local portability | One documented start/stop contract runs on Windows 11 with WSL2 Ubuntu, macOS, and Linux using Docker Compose. | TC-LOCAL-001 and trainer OS setup evidence. |
| NFR-LIC-01 | Licence compliance | Must tools use free licences suitable for public student portfolios. | Dependency list review. |
| NFR-TEST-01 | Backend coverage | Backend line coverage ≥ 85%; branch coverage ≥ 75%. Eligibility module has 100% branch coverage. | pytest coverage report. |
| NFR-TEST-02 | API test breadth | At least 55 meaningful Python test cases cover positive, negative, boundary, schema, and permission behavior; catalog rows cannot be replaced by duplicate assertions. | pytest collection and pass summary plus document 09 mapping. |
| NFR-TEST-03 | API integration flows | At least 4 critical multi-request Python/API flows run locally with saved JUnit and response evidence. | TC-FLOW-001 through TC-FLOW-004. |
| NFR-HW-01 | Lite hardware | Must scope runs on an 8 GB RAM laptop with the lite profile below. | Trainer pre-check before week 1. |

## Security requirements by risk

| Risk | CampusHire control |
|---|---|
| Broken access control | Server-side role and object-ownership checks on every API. |
| CSRF | Django CSRF cookie and `X-CSRFToken` header on unsafe requests. |
| Untrusted descriptions | Store and return plain text as JSON; never interpret posting descriptions as executable content. |
| Insecure upload | PDF content check, 2 MB limit, private media storage, authorized download view. |
| Sensitive logging | Logs omit passwords, session IDs, CSRF tokens, resume content, and raw cookies. |
| SQL injection | Django ORM and parameterized handwritten SQL. No string concatenation in SQL. |
| Weak password choices | Django password validators and minimum 12 characters. |

## Hardware profiles

### Standard profile

| Resource | Value |
|---|---|
| Laptop RAM | 16 GB recommended |
| CPU and free disk | At least four cores and 20 GB available for images, database, and private fixtures |
| Docker memory | 8 GB |
| Docker swap | 4 GB |
| Services | Django, PostgreSQL, optional local Mailpit |
| PostgreSQL memory target | Up to 1 GB container memory |
| Python tests | Unit, database/API integration, report, and local lifecycle checks |

### Lite profile for 8 GB laptops

| Resource | Exact limit |
|---|---|
| Docker memory | 4 GB |
| Docker swap | 4 GB |
| PostgreSQL container | Limit to 768 MB |
| Django container | Limit to 768 MB |
| Mailpit | Run only when e-mail Should items are active; limit 256 MB |
| Python tests | One worker; small fixtures for normal development |
| CPU and free disk | Four cores and 20 GB available |
| Switched off | Optional notification services and large performance seed until needed |

### Windows WSL2 guidance

| Host RAM | WSL memory | WSL swap |
|---|---|---|
| 8 GB | 4 GB | 4 GB |
| 16 GB | 8 GB | 4 GB |

These resource budgets are proposed limits, not measured results. The trainer MUST validate the lite profile on an 8 GB, four-core laptop before week 1 using the pre-check in document 06.

## Coverage and test targets

| Scope | Line coverage | Branch coverage | Notes |
|---|---|---|---|
| Backend application packages | ≥ 85% | ≥ 75% | Exclude migrations and generated OpenAPI files. |
| Eligibility module | 100% | 100% | Must cover every reason code. |
| Placement-policy module | 100% if built | 100% if built | Applies to dream-offer Should item. |
| API integration | 4 flows Must | 8 flows with Should scope | Save JUnit and exact response evidence; included in Python coverage. |

## Critical verification scenarios

| Scenario | Requirement links | Evidence |
|---|---|---|
| Student blocked by incomplete profile | FR-PROFILE-01, FR-ELIG-01 | API status and ordered JSON reason assertion |
| Student applies successfully | FR-APP-01 | Python/API integration flow |
| Recruiter cross-company access denied | FR-REC-01, NFR-SEC-04 | API security test |
| Deadline closes posting | FR-POST-02, NFR-REL-01 | Unit or integration test with controlled time |
| Placement metrics correct | FR-TPO-01, NFR-PERF-02, NFR-DATA-01 | JSON/CSV report test and HTTP timing note |
| PDF content rejected | FR-PROFILE-02, NFR-SEC-03 | Upload test |
| Local lifecycle and offline operation | FR-LOCAL-01, NFR-REL-03, NFR-LOCAL-01, NFR-PORT-01 | TC-LOCAL-001 through TC-LOCAL-003 |

## Reliability and recovery rules

1. Application state changes MUST be atomic with timeline creation.
2. Deadline closure MAY run through a management command and MUST also be enforced during apply.
3. Notification delivery, when built, is at least once with a stable de-duplication key.
4. If resume storage fails, the old current resume MUST remain active.
5. If one report endpoint fails, it returns `503 report-timeout`; other report endpoints remain independently callable.
6. Stop MUST preserve the named database and private-media volumes; reset MUST require explicit confirmation.

[Back to README](../README.md)
