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
| API client | A cookie-aware tool or Python test that sends HTTP requests and inspects responses. |
| Private media | Uploaded files stored outside public static paths. |
| Placement report | TPO REST endpoint returning student counts, placed percentage, accepted full-time offers, and CTC metrics as JSON or CSV. |
| Median CTC | The middle accepted full-time CTC after sorting, or the mean of the middle two values for an even count. |
| Window function | A SQL function that ranks or calculates across related rows without collapsing them. |
| Query plan | PostgreSQL output that explains how a query reads and joins data. |
| Object permission | Server-side check that a role owns or is authorized to access a particular resource. |
| API integration flow | Python test of a multi-request business workflow against real database-backed endpoints. |
| Local fixture | Recorded fictional seed/PDF input that requires no external network during runtime tests. |
| Named volume | Persistent local Docker storage retained through normal stop/start. |
| Confirmed reset | A separate command that deletes only project-local data after explicit confirmation. |

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

## Free learning resources

| Topic | Resource | Why to read it |
|---|---|---|
| Django basics | <https://docs.djangoproject.com/en/5.2/intro/> | Build models, views, built-in admin, and sessions. |
| DRF tutorial | <https://www.django-rest-framework.org/tutorial/quickstart/> | Understand serializers, viewsets, and permissions. |
| PostgreSQL tutorial | <https://www.postgresql.org/docs/current/tutorial.html> | Revise SQL queries, joins, and aggregates. |
| PostgreSQL indexes | <https://www.postgresql.org/docs/current/indexes.html> | Learn why job search and report filters need indexes. |
| Python standard library | <https://docs.python.org/3/> | Revise Decimal, datetime, CSV, JSON, exceptions, and context managers. |
| MDN HTTP | <https://developer.mozilla.org/en-US/docs/Web/HTTP> | Revise methods, headers, cookies, and status codes. |
| OWASP file upload | <https://cheatsheetseries.owasp.org/cheatsheets/File_Upload_Cheat_Sheet.html> | Understand PDF upload risks and controls. |
| OWASP CSRF | <https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html> | Understand why unsafe methods need tokens. |
| Python test fixtures | <https://docs.pytest.org/> | Build API flows, controlled-clock tests, and local lifecycle verification. |
| GitHub Actions docs | <https://docs.github.com/actions> | Understand CI jobs, triggers, and logs. |
| GitHub Flow | <https://docs.github.com/get-started/using-github/github-flow> | Learn branch and PR workflow. |
| Conventional Commits | <https://www.conventionalcommits.org/en/v1.0.0/> | Write searchable commit messages. |
| Docker getting started | <https://docs.docker.com/get-started/> | Run PostgreSQL/Django locally, retain named volumes, and inspect readiness. |

## Suggested reading order

1. Read the README and documents 01 to 04 to understand scope, roles, FRs, BRs, and NFRs.
2. Read documents 05, 07, and 08 before creating models or APIs.
3. Read this document's Django, DRF, and PostgreSQL links during week 1.
4. Read the Python, API permission, and SQL resources before implementing serializers, services, and reports.
5. Read the CSRF and file upload resources before implementing login or resume upload.
6. Read the testing resources before week 3 ends.
7. Read documents 10, 11, 12, and 13 before final hardening and interview preparation.
8. Follow document 06's local operation contract and document 09's local cases before claiming reproducibility.

[Back to README](../README.md)
