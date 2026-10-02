# Interview preparation

Purpose: This document helps the CampusHire student explain the project, prepare technical answers, improve profiles, and practice the final viva.

## Explain your project in 2 minutes using STAR

| STAR part | Talking points |
|---|---|
| Situation | Sahyadri Institute used spreadsheets, forms, and e-mail for placements. Students missed deadlines, and recruiters saw ineligible applications. |
| Task | I built CampusHire, a Django and React portal for students, recruiters, TPO staff, and admins. |
| Action | I used Django sessions with CSRF, DRF APIs, PostgreSQL reports, React 19, TypeScript, route guards, and Playwright journeys. |
| Result | The portal blocks ineligible applications, explains every reason code, protects resumes, and shows placement metrics with SQL evidence. |

## Interview questions by topic

### HTTP, REST, sessions, and security

1. What happens when Aarav logs in? Key points: Django creates a server session; browser stores session cookie; cookie is HttpOnly; logout invalidates the server session.
2. Why not use JWT by default? Key points: same-site SPA; Django sessions are simpler; CSRF protection is built in; JWT is more useful for cross-site APIs.
3. What is CSRF? Key points: attacker uses victim browser; unsafe methods need token; CampusHire sends `X-CSRFToken`; missing token returns 403.
4. What is XSS? Key points: injected script runs in browser; React escapes text; do not render raw posting descriptions; validate rich text if added later.
5. What is CORS? Key points: browser origin policy; local React and API origins must be configured; same-site deployment reduces CORS complexity.
6. Explain REST in CampusHire. Key points: resources such as postings and applications; HTTP methods; status codes; JSON error codes.
7. Which status codes did you use? Key points: 201 registration; 400 validation; 403 missing session or forbidden role; 409 duplicate or invalid transition.
8. How do you secure file upload? Key points: PDF content check; 2 MB limit; private media; authorized download view.

### Django, DRF, and ORM

9. How did you model roles? Key points: user role enum; related profiles; server-side permissions; route guards are not enough.
10. What is a serializer? Key points: validates request data; transforms model data; controls exposed fields; returns error codes.
11. What is a transaction? Key points: groups state change and timeline event; both commit or both roll back; protects FR-APP-02.
12. How do you avoid N+1 queries? Key points: `select_related` for one-to-one or foreign key; `prefetch_related` for many-to-many; inspect query counts.
13. When do you use Django admin? Key points: branches, skills, college settings, TPO accounts; not a replacement for student and recruiter UX.
14. How do DRF permissions protect resumes? Key points: check authenticated user; verify owner or owning company; return 403; never expose storage key.
15. How does deadline closure work? Key points: scheduled command; request-time guard; `now >= deadline_at`; closes within 5 minutes.

### SQL and PostgreSQL

16. Why use PostgreSQL instead of SQLite? Key points: production-like SQL; indexes; window functions; query plans.
17. Explain an inner join with CampusHire data. Key points: applications join students; postings join companies; unmatched rows are excluded.
18. Explain GROUP BY for dashboard metrics. Key points: group by branch or company; count offers; aggregate CTC; use accepted full-time offers only.
19. What is a window function? Key points: ranks companies within each branch; keeps row detail; useful for report ranking.
20. What does an index do? Key points: speeds filtered reads; costs writes and storage; chosen for state, deadline, posting, and branch filters.
21. What is `EXPLAIN ANALYZE`? Key points: shows actual plan and timing; confirms index use; evidence saved for two queries.
22. How do you calculate median CTC? Key points: use accepted full-time offers; sort values; PostgreSQL aggregate or window approach; display `No offers yet` when empty.

### React and TypeScript

23. What is a React component? Key points: reusable UI function; receives props; returns UI; examples are job card and profile form.
24. What are props and state? Key points: props come from parent; state changes locally; job filters are state; applicant row receives props.
25. When do you use hooks? Key points: `useState` for filters; `useEffect` for side effects; form hooks; router hooks for navigation.
26. What is a typed API client? Key points: central fetch wrapper; typed responses; redirects on `403 authentication-required`; handles other 403 and 409 errors.
27. Why use TypeScript? Key points: catches field name mistakes; documents API shapes; improves forms and route guards.
28. How do route guards work? Key points: read current user; compare role; show access message or redirect; server still enforces permissions.
29. How did you handle loading and error states? Key points: show spinner or skeleton; keep safe route; show server error code; retry only safe reads.
30. How did you test React screens? Key points: React Testing Library; MSW for API responses; Playwright for full journeys.

### DevOps and testing

31. What does CI check? Key points: backend lint and tests; frontend lint and tests; builds; security scans; coverage gates.
32. Why use Docker Compose? Key points: repeatable PostgreSQL; one local stack; standard and lite profiles; easier trainer setup.
33. What is a lockfile? Key points: pins transitive dependencies; reproducible installs; reviewed in PRs.
34. What are Playwright journeys? Key points: browser E2E tests; login, apply, recruiter status, dashboard; saved HTML report.
35. How do you handle secrets? Key points: `.env.example` only; no real `.env`; Gitleaks; rotate if exposed.

## Project deep-dive questions

1. How did you prevent duplicate applications? Key points: unique student plus posting; API checks; 409 response; test for double click.
2. How did you show all eligibility reasons? Key points: evaluate each rule; fixed reason order; no early return; typed UI list.
3. How did you protect recruiter applicant data? Key points: company ownership check; queryset filter; 403 on direct ID access; API tests.
4. How did you keep timeline events correct? Key points: state transition table; transaction; one event per change; invalid transition records none.
5. How did you design the dashboard query? Key points: accepted full-time offers only; branch grouping; window rank; query-plan evidence.
6. How did you test CSRF? Key points: unsafe methods; missing token test; expected 403; no database change.
7. How does the lite hardware profile affect tests? Key points: one Playwright worker; Mailpit only when needed; smaller seed data; Docker memory limits.
8. How would you add dream-offer policy later? Key points: TPO setting; highest accepted full-time CTC; 1.5 multiplier; reason `DREAM_POLICY_BLOCKED`.

## Fundamentals check

| Area | CampusHire topics to revise |
|---|---|
| Python core | Functions, classes, exceptions, context managers, type hints, lists, dictionaries, and datetime handling. |
| SQL | Joins, GROUP BY, window functions, transactions, indexes, constraints, and `EXPLAIN ANALYZE`. |
| Git | Branches, rebase basics, merge conflicts, Conventional Commits, PR reviews, and tags. |
| Linux | `cd`, `ls`, `grep`, `find`, permissions, processes, environment variables, and log inspection. |
| HTTP and networking | DNS, TCP, HTTP methods, headers, cookies, status codes, CORS, CSRF, and same-site behavior. |
| Local tools to AWS | Django container maps to ECS or Elastic Beanstalk; PostgreSQL maps to RDS; private media maps to S3; Mailpit maps to SES sandbox; GitHub Actions maps to CodeBuild-style CI; logs map to CloudWatch. |

## Resume bullet templates

- Built CampusHire, a Django and React placement portal, with `<N>` role-protected REST endpoints and `<N>` automated tests.
- Implemented an eligibility engine that returned `<N>` reason codes and blocked duplicate or late applications with 409 responses.
- Optimized PostgreSQL placement reports for `<N>` applications using indexes, GROUP BY, and a window function.
- Secured PDF resume upload with content validation, a 2 MB limit, private media, and authorized downloads.
- Added CI with Ruff, mypy, pytest coverage, ESLint, Vitest, Docker build, and Playwright journey evidence.

## GitHub and LinkedIn tips

- Pin the `campushire` repository and add the final demo video link.
- Keep the README short, accurate, and runnable from a clean clone.
- Include screenshots of the student apply flow and TPO dashboard.
- Write LinkedIn project summary with Django, DRF, PostgreSQL, React, TypeScript, Docker, and CI keywords.
- Do not post real student data, secrets, or resume files.

## Mock-interview checklist

- Explain CampusHire in 2 minutes without reading notes.
- Draw the main architecture on paper.
- Explain sessions versus JWT and CSRF.
- Walk through the eligibility reason codes.
- Explain one SQL query plan and one index.
- Explain one React component and its state.
- Debug one failing test live.
- Show CI, coverage, and Playwright evidence.

[Back to README](../README.md)
