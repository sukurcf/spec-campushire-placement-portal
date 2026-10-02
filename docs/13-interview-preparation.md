# Interview preparation

Purpose: This document helps the CampusHire student explain the project, prepare technical answers, improve profiles, and practice the final viva.

## Explain your project in 2 minutes using STAR

| STAR part | Talking points |
|---|---|
| Situation | Sahyadri Institute used spreadsheets, forms, and e-mail for placements. Students missed deadlines, and recruiters saw ineligible applications. |
| Task | I built CampusHire, a Python Django REST backend for students, recruiters, TPO staff, and admins. |
| Action | I used sessions with CSRF, DRF permissions, PostgreSQL/handwritten SQL, exact JSON/CSV reports, Python integration flows, and persistent local Compose services. |
| Result | The backend blocks ineligible applications, returns every reason code, protects resumes, and provides placement reports with SQL/test evidence. |

## Interview questions by topic

### HTTP, REST, sessions, and security

1. What happens when Aarav logs in? Key points: Django creates a server session; API client retains session cookie; CSRF token rotates; logout invalidates the server session.
2. Why retain Django sessions? Key points: server-side revocation; explicit CSRF on unsafe requests and anonymous auth endpoints; cookie-aware Python clients; JWT is not this project's auth model.
3. What is CSRF? Key points: attacker uses victim browser; unsafe methods need token; CampusHire sends `X-CSRFToken`; missing token returns 403.
4. How do error responses avoid leaking data? Key points: stable keys; no SQL exceptions, resume storage keys, credentials, or other users' records.
5. How do session cookie flags work? Key points: HttpOnly, SameSite=Lax, Secure in secure test settings; only loopback local HTTP disables Secure.
6. Explain REST in CampusHire. Key points: resources such as postings and applications; HTTP methods; status codes; JSON error codes.
7. Which status codes did you use? Key points: 201 registration; 400 validation; 403 missing session or forbidden role; 409 duplicate or invalid transition.
8. How do you secure file upload? Key points: PDF content check; 2 MB limit; private media; authorized download view.

### Django, DRF, and ORM

9. How did you model roles? Key points: user role enum; related profiles; server-side role/object permissions on every request.
10. What is a serializer? Key points: validates request data; transforms model data; controls exposed fields; returns error codes.
11. What is a transaction? Key points: groups state change and timeline event; both commit or both roll back; protects FR-APP-02.
12. How do you avoid N+1 queries? Key points: `select_related` for one-to-one or foreign key; `prefetch_related` for many-to-many; inspect query counts.
13. When do you use Django admin? Key points: built-in local master data and staff accounts only; business approvals and student/recruiter operations use REST.
14. How do DRF permissions protect resumes? Key points: check authenticated user; verify owner or owning company; return 403; never expose storage key.
15. How does deadline closure work? Key points: scheduled command; request-time guard; `now >= deadline_at`; closes within 5 minutes.

### SQL and PostgreSQL

16. Why use PostgreSQL instead of SQLite? Key points: production-like SQL; indexes; window functions; query plans.
17. Explain an inner join with CampusHire data. Key points: applications join students; postings join companies; unmatched rows are excluded.
18. Explain GROUP BY for placement reports. Key points: group by branch/company; count accepted full-time applications; count distinct placed students; avoid join multiplication.
19. What is a window function? Key points: ranks companies within each branch; keeps row detail; useful for report ranking.
20. What does an index do? Key points: speeds filtered reads; costs writes and storage; chosen for state, deadline, posting, and branch filters.
21. What is `EXPLAIN ANALYZE`? Key points: shows actual plan and timing; confirms index use; evidence saved for two queries.
22. How do you calculate median CTC? Key points: accepted full-time application amounts; sort values; mean of middle two for even count; JSON null/CSV empty when there are no offers.

### Python services and API contracts

23. Why use Decimal for CTC? Key points: exact salary/percentage arithmetic; two-place serialization; no binary-float rounding artifacts.
24. Where do business rules live? Key points: serializers validate shape; services enforce eligibility/transitions; models/constraints protect persisted invariants.
25. How do you handle time? Key points: aware timestamps; IST representation; UTC persistence; injectable local business clock; real security expiry.
26. How does a Python client handle sessions? Key points: cookie jar; GET CSRF; POST login with header; retain rotated CSRF; no blind write retries.
27. What does OpenAPI verify? Key points: required fields, types, errors, role requirements, decimal strings, pagination, and report representations.
28. How do permission errors differ? Key points: authentication-required versus role-not-allowed versus posting-not-owned; all server checks; no redirects.
29. What is the empty-report contract? Key points: zero registered gives percentage 0.00; missing CTC is null/empty; company CSV may be header-only.
30. How do you test a complete workflow? Key points: Python client changes sessions by role; real PostgreSQL; controlled fixtures/time; assert status, state, and event count.

### DevOps and testing

31. What does CI check? Key points: Python lint/types, API/report tests, local lifecycle, image/security checks, coverage and 55-case floor.
32. Why use Docker Compose? Key points: repeatable PostgreSQL; one local stack; standard and lite profiles; easier trainer setup.
33. What is a lockfile? Key points: pins transitive dependencies; reproducible installs; reviewed in PRs.
34. How do you prove offline persistence? Key points: initial downloads cached; external egress off; seed demo exact response; stop/start preserves IDs/events/PDF hash; host test driver survives container shutdown.
35. How do you handle secrets? Key points: `.env.example` only; no real `.env`; Gitleaks; rotate if exposed.

## Project deep-dive questions

1. How did you prevent duplicate applications? Key points: unique student plus posting; API checks; 409 response; repeated request test.
2. How did you return all eligibility reasons? Key points: evaluate each rule; fixed reason order; no early return; aligned JSON reason/message arrays.
3. How did you protect recruiter applicant data? Key points: company ownership check; queryset filter; 403 on direct ID access; API tests.
4. How did you keep timeline events correct? Key points: state transition table; transaction; one event per change; invalid transition records none.
5. How did you design the report query? Key points: accepted full-time offers only; distinct students; branch grouping; dense rank; query-plan evidence.
6. How did you test CSRF? Key points: unsafe methods; missing token test; expected 403; no database change.
7. How does the lite hardware profile affect tests? Key points: one Python worker; optional Mailpit off; small fixtures; 768 MB DB/API budgets; recorded resource measurements.
8. How would you add dream-offer policy later? Key points: TPO setting; highest accepted full-time CTC; 1.5 multiplier; reason `DREAM_POLICY_BLOCKED`.

## Fundamentals check

| Area | CampusHire topics to revise |
|---|---|
| Python core | Functions, classes, exceptions, context managers, type hints, lists, dictionaries, and datetime handling. |
| SQL | Joins, GROUP BY, window functions, transactions, indexes, constraints, and `EXPLAIN ANALYZE`. |
| Git | Branches, rebase basics, merge conflicts, Conventional Commits, PR reviews, and tags. |
| Linux | `cd`, `ls`, `grep`, `find`, permissions, processes, environment variables, and log inspection. |
| HTTP and networking | TCP, loopback bindings, methods, headers, cookies, status codes, CSRF, and same-site cookie behavior. |
| Local operations | Compose readiness, migrations, idempotent seed, named volumes, offline caches, private-media hashes, explicit reset, and actionable diagnostics. |

## Resume bullet templates

- Built CampusHire, a Python Django REST backend, with `<N>` role-protected endpoints and `<N>` meaningful Python tests.
- Implemented an eligibility engine that returned `<N>` reason codes and blocked duplicate or late applications with 409 responses.
- Optimized PostgreSQL placement reports for `<N>` applications using indexes, GROUP BY, and a window function.
- Secured PDF resume upload with content validation, a 2 MB limit, private media, and authorized downloads.
- Added CI with Ruff, mypy, pytest coverage, Docker image checks, API integration flows, and offline/persistence acceptance.

## GitHub and LinkedIn tips

- Pin the `campushire` repository and add the final demo video link.
- Keep the README short, accurate, and runnable from a clean clone.
- Include sanitized API examples, JSON/CSV report samples, and local lifecycle/JUnit evidence.
- Write LinkedIn project summary with Python, Django, DRF, PostgreSQL, SQL, Docker, and CI keywords.
- Do not post real student data, secrets, or resume files.

## Mock-interview checklist

- Explain CampusHire in 2 minutes without reading notes.
- Draw the main architecture on paper.
- Explain sessions versus JWT and CSRF.
- Walk through the eligibility reason codes.
- Explain one SQL query plan and one index.
- Explain one serializer/permission check and one JSON/CSV report boundary.
- Debug one failing test live.
- Show CI, Python coverage, four API flows, and persisted offline local acceptance evidence.

[Back to README](../README.md)
