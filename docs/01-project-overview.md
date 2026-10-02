# Project overview

Purpose: This document defines the business goal, scope, constraints, and success criteria for CampusHire.

## Problem statement

Sahyadri Institute of Technology and Management has one Training and Placement Office. The office manages placement drives with spreadsheets, Google Forms, and e-mail. Students miss deadlines because job posts are scattered. Recruiters receive applications from students who do not meet the rules. TPO staff spend many hours checking CGPA, branch, backlogs, and resume links manually.

CampusHire is a same-site web application for this college. It gives students one portal for profiles, jobs, applications, and offer timelines. It gives recruiters one place to publish drives and review only authorized applicants. It gives the TPO clear approval, verification, and reporting workflows.

## Vision

CampusHire MUST make campus placement data accurate before recruiters see it. The system MUST show each student why they are eligible or not eligible. The system MUST keep one application state history for every job application. The TPO SHOULD later use configurable placement policies, notifications, and dashboard exports to reduce manual follow-up.

## Measurable goals

| Goal ID | Goal | Target |
|---|---|---|
| G-01 | Reduce ineligible applications sent to recruiters | 0 recruiter-visible applications that fail the defined eligibility rules |
| G-02 | Make student readiness visible | 100% of application attempts check profile completeness and TPO verification |
| G-03 | Improve deadline control | 0 applications accepted after a posting deadline |
| G-04 | Support placement reporting | TPO dashboard shows counts, placed percentage, CTC metrics, and branch/company tables |
| G-05 | Keep the project buildable by one fresher | MUST scope stays within about 120 hours, including 18 hours frontend learning and 12 hours other compulsory learning |

## Scope by priority

### Must: minimum viable product

- Student self-registration with college e-mail domain `sitm.example.in`.
- Recruiter registration with TPO approval.
- TPO accounts created by System Admin in Django admin.
- Django session authentication with CSRF for a same-site React SPA.
- Student profile, academics, skills, links, TPO verification, and PDF resume upload.
- Company profiles, postings, eligibility rules, selection rounds, and TPO approval.
- Posting lifecycle from `DRAFT` to `PENDING_APPROVAL`, `PUBLISHED`, and `CLOSED`.
- Eligibility engine with exact reason codes and student-visible messages.
- Application pipeline from `APPLIED` to final states.
- Recruiter applicant view for the recruiter's own postings only.
- Job search with filters, pagination, and sorting.
- TPO dashboard with SQL-backed reports and query-plan evidence.
- React 19, TypeScript, Vite, React Router, React Hook Form, and Tailwind CSS.
- Django admin for branches, skills, and college settings.

### Should: recommended after the MVP

- Dream-offer placement policy with the 1.5× CTC rule.
- Profile completeness percentage.
- E-mail verification and password reset through Mailpit.
- Bulk recruiter status updates.
- In-app and e-mail notifications.
- Deadline reminder 24 hours before the posting deadline.
- Offer acceptance deadline of 7 days.
- Audit log for application status changes.
- Dashboard charts and CSV exports.
- TanStack Query, Zod, TypeScript strict mode, Playwright in CI, and axe checks.
- Production-like Compose profile with Gunicorn, built React assets, and Nginx.

### Could: stretch items

- Interview scheduling with `.ics` invites.
- Dark mode.
- Simple resume keyword extraction.
- Free-tier cloud deployment with a cost warning and destroy-after-demo steps.

## Non-goals and out of scope

| Out-of-scope item | Reason |
|---|---|
| Online coding tests | It would add a different assessment product. |
| Video interviews | It needs media infrastructure beyond the training goal. |
| Payments | The college placement process does not need payments. |
| Chat | E-mail and in-app notifications are enough for this project. |
| Native mobile apps | The responsive web UI MUST support 360 px and wider screens. |
| SSO with the college ERP | Session auth and CSRF are the authentication learning goal. |
| Multi-college SaaS | The data model is for one fictional college only. |

## Assumptions

1. The college owns the e-mail domain `sitm.example.in`.
2. A student has one active college roll number.
3. A recruiter user belongs to one company at a time.
4. The TPO approves each recruiter before the recruiter can publish postings.
5. CTC values are in LPA. Internship stipend values are in INR per month.
6. PostgreSQL is the main database for local development and evaluation.
7. The same-site SPA and API run under one site during local development.
8. Mailpit is used locally for optional e-mail features.

## Constraints

| Constraint | Requirement |
|---|---|
| Time | One fresher completes the total project in 6 weeks, about 180 hours. |
| MVP budget | MUST work MUST fit about 120 hours, including 18 hours frontend learning and 12 hours other compulsory learning. |
| Hardware | MUST scope runs on an 8 GB RAM laptop with the lite profile. |
| Cost | MUST tools are free and run locally. |
| Repository | The specification repository contains Markdown only. |
| Privacy | Use fictional people, companies, branches, and e-mail addresses. |
| Auth | Use Django session authentication with CSRF, not JWT. |

## Success criteria

| Criterion | Evidence |
|---|---|
| A student can apply only when eligible | Demo with Aarav Rao and the `PUBLISHED` posting for Navira Systems. |
| The eligibility result explains every failure | UI shows all matching reason codes and messages. |
| Recruiters cannot view other companies' applicants | API or UI test for blocked access returns `403 Forbidden`. |
| The resume upload is safe enough for the MVP | PDF content-type check rejects a renamed text file. |
| TPO can see placement numbers | Dashboard shows registered, verified, placed %, accepted full-time offers, and CTC metrics. |
| SQL learning is visible | Student records `EXPLAIN ANALYZE` evidence for two key queries. |
| Frontend learning is visible | React routes, form validation, loading states, and error states are implemented. |

## Skills learned and job relevance

| Skill | CampusHire use | Job-description keywords |
|---|---|---|
| Django and DRF | Accounts, profiles, postings, applications, reports | Django, REST API, ORM, DRF |
| PostgreSQL | Eligibility queries, dashboards, reports | SQL, indexes, query plans, window functions |
| React and TypeScript | Same-site SPA with role-based navigation | React, TypeScript, Vite, SPA |
| Web security | Sessions, CSRF, resume authorization | CSRF, OWASP, secure file upload |
| Testing | Unit, API, UI, and Playwright journeys | pytest, Vitest, coverage, E2E |
| DevOps basics | Docker Compose and GitHub Actions | CI/CD, Docker, reproducible setup |

## Related documents

- Roles and stories: [02-users-and-roles.md](02-users-and-roles.md)
- Functional rules: [03-functional-requirements.md](03-functional-requirements.md)
- Non-functional targets: [04-non-functional-requirements.md](04-non-functional-requirements.md)
- Data model: [07-data-model.md](07-data-model.md)
- API and UI routes: [08-api-specification.md](08-api-specification.md)

[Back to README](../README.md)
