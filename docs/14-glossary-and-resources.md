# Glossary and resources

Purpose: This document defines CampusHire terms and lists official documentation and free learning resources.

## Glossary

| Term | Definition |
|---|---|
| TPO | Training and Placement Office staff who verify students, approve postings, and review reports. |
| Student | A Sahyadri Institute user who maintains a profile and applies to eligible postings. |
| Recruiter | A company HR user who creates postings and updates applicant status after TPO approval. |
| System Admin | A Django admin user who manages master data and TPO accounts. |
| Company profile | Recruiter-maintained company information linked to postings. |
| Job posting | A full-time, internship, or internship-with-PPO opportunity for students. |
| Posting lifecycle | The allowed posting states from `DRAFT` to `PENDING_APPROVAL`, `PUBLISHED`, and `CLOSED`. |
| Draft | A posting state where the recruiter can edit before TPO review. |
| Pending approval | A posting state where the TPO must approve or reject the drive. |
| Published | A posting state where eligible students can apply before the deadline. |
| Closed | A posting state where no new applications are accepted. |
| Eligibility engine | The backend logic that checks profile, posting, deadline, academic, and policy rules. |
| Reason code | A stable machine-readable code such as `CGPA_BELOW_MINIMUM`. |
| CGPA | Cumulative Grade Point Average on the 0.00 to 10.00 scale. |
| Active backlog | A subject the student has not yet cleared. |
| Branch | The engineering branch, such as `CSE`, `ISE`, or `ECE`. |
| Graduation year | The year the student is expected to complete the degree. |
| Deadline | The last timestamp when a published posting accepts applications. |
| Profile verification | TPO approval that a student profile and academics are correct. |
| Resume authorization | The server check that permits only the owner, owning recruiter, TPO, or admin to download a resume. |
| Application | A student's one allowed submission to one posting. |
| Application pipeline | The states from `APPLIED` through interview, offer, acceptance, decline, rejection, or withdrawal. |
| Timeline | The timestamped application event history shown to the student. |
| Shortlist | A recruiter decision that moves an applicant from `APPLIED` to `SHORTLISTED`. |
| Offer | A recruiter decision that moves an applicant to `OFFERED` with CTC or stipend details. |
| Accepted offer | An offer the student accepts, used in placement metrics. |
| Declined offer | An offer the student rejects or that expires in the Should scope. |
| Withdrawn application | An application the student cancels while still in `APPLIED`. |
| Dream-offer policy | A Should rule that allows higher full-time applications after an accepted full-time offer. |
| CTC | Cost to Company, shown in LPA for full-time roles. |
| LPA | Lakhs per annum, used for annual Indian salary figures. |
| Stipend | Monthly internship payment in INR. |
| Full-time | A job posting with annual CTC and placement outcome. |
| Internship | A posting with monthly stipend and no full-time CTC requirement. |
| PPO | Pre-placement offer, usually linked to an internship. |
| CSRF | Cross-Site Request Forgery, prevented by Django token checks on unsafe methods. |
| Session authentication | Server-side login state stored in Django and referenced by a secure cookie. |
| Same-site SPA | A React single-page app served under the same site as the API. |
| Private media | Uploaded files stored outside public static paths. |
| Dashboard | TPO view for student counts, placed percentage, accepted full-time offers, and CTC metrics. |
| Median CTC | The middle accepted full-time CTC value after sorting. |
| Window function | A SQL function that ranks or calculates across related rows without collapsing them. |
| Query plan | PostgreSQL output that explains how a query reads and joins data. |
| Route guard | Frontend logic that blocks a role from opening another role's route. |
| Typed API client | TypeScript fetch wrapper with typed request and response shapes. |
| Playwright journey | A browser test that checks a full CampusHire user workflow. |

## Official documentation links

The links below were checked on 2 October 2026 with HTTP header requests. Use official documentation or official project pages only.

| Tool | Official link |
|---|---|
| Python | <https://docs.python.org/3/> |
| uv | <https://docs.astral.sh/uv/> |
| Django | <https://docs.djangoproject.com/en/5.2/> |
| Django REST Framework | <https://www.django-rest-framework.org/> |
| drf-spectacular | <https://drf-spectacular.readthedocs.io/> |
| django-filter | <https://django-filter.readthedocs.io/> |
| PostgreSQL | <https://www.postgresql.org/docs/> |
| pytest | <https://docs.pytest.org/> |
| pytest-django | <https://pytest-django.readthedocs.io/> |
| factory_boy | <https://factoryboy.readthedocs.io/> |
| pytest-cov | <https://pytest-cov.readthedocs.io/> |
| coverage.py | <https://coverage.readthedocs.io/> |
| mypy | <https://mypy.readthedocs.io/en/stable/> |
| django-stubs | <https://github.com/typeddjango/django-stubs> |
| Ruff | <https://docs.astral.sh/ruff/> |
| pre-commit | <https://pre-commit.com/> |
| pip-audit | <https://pypi.org/project/pip-audit/> |
| Gitleaks | <https://github.com/gitleaks/gitleaks> |
| Trivy | <https://trivy.dev/> |
| Docker Engine | <https://docs.docker.com/engine/> |
| Docker Compose | <https://docs.docker.com/compose/> |
| Mailpit | <https://mailpit.axllent.org/> |
| Node.js | <https://nodejs.org/en/learn> |
| React | <https://react.dev/> |
| TypeScript | <https://www.typescriptlang.org/docs/> |
| Vite | <https://vite.dev/guide/> |
| React Router | <https://reactrouter.com/> |
| React Hook Form | <https://react-hook-form.com/> |
| Tailwind CSS | <https://tailwindcss.com/docs> |
| TanStack Query | <https://tanstack.com/query/latest> |
| Zod | <https://zod.dev/> |
| Recharts | <https://recharts.org> |
| Vitest | <https://vitest.dev/> |
| React Testing Library | <https://testing-library.com/docs/react-testing-library/intro/> |
| MSW | <https://mswjs.io/docs/> |
| Playwright | <https://playwright.dev/> |
| @axe-core/playwright | <https://github.com/dequelabs/axe-core-npm/tree/develop/packages/playwright> |
| ESLint | <https://eslint.org/docs/latest/> |
| Prettier | <https://prettier.io/docs/> |

## Free learning resources

| Topic | Resource | Why to read it |
|---|---|---|
| Django basics | <https://docs.djangoproject.com/en/5.2/intro/> | Build models, views, admin, sessions, and forms. |
| DRF tutorial | <https://www.django-rest-framework.org/tutorial/quickstart/> | Understand serializers, viewsets, and permissions. |
| PostgreSQL tutorial | <https://www.postgresql.org/docs/current/tutorial.html> | Revise SQL queries, joins, and aggregates. |
| PostgreSQL indexes | <https://www.postgresql.org/docs/current/indexes.html> | Learn why job search and dashboard filters need indexes. |
| React learn | <https://react.dev/learn> | Learn components, props, state, hooks, and effects. |
| TypeScript handbook | <https://www.typescriptlang.org/docs/handbook/intro.html> | Learn types, interfaces, generics, and narrowing. |
| Vite guide | <https://vite.dev/guide/> | Understand local frontend development and builds. |
| MDN HTTP | <https://developer.mozilla.org/en-US/docs/Web/HTTP> | Revise methods, headers, cookies, and status codes. |
| OWASP file upload | <https://cheatsheetseries.owasp.org/cheatsheets/File_Upload_Cheat_Sheet.html> | Understand PDF upload risks and controls. |
| OWASP CSRF | <https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html> | Understand why unsafe methods need tokens. |
| Playwright docs | <https://playwright.dev/docs/intro> | Build the four required browser journeys. |
| GitHub Actions docs | <https://docs.github.com/actions> | Understand CI jobs, triggers, and logs. |
| GitHub Flow | <https://docs.github.com/get-started/using-github/github-flow> | Learn branch and PR workflow. |
| Conventional Commits | <https://www.conventionalcommits.org/en/v1.0.0/> | Write searchable commit messages. |
| Docker getting started | <https://docs.docker.com/get-started/> | Run PostgreSQL, Django, and React consistently. |

## Suggested reading order

1. Read the README and documents 01 to 04 to understand scope, roles, FRs, BRs, and NFRs.
2. Read documents 05, 07, and 08 before creating models or APIs.
3. Read this document's Django, DRF, and PostgreSQL links during week 1.
4. Read the React, TypeScript, Vite, React Router, and React Hook Form links before building forms.
5. Read the CSRF, XSS, and file upload resources before implementing login or resume upload.
6. Read the testing resources before week 3 ends.
7. Read documents 10, 11, 12, and 13 before final hardening and interview preparation.

[Back to README](../README.md)
