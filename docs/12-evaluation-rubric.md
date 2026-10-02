# Evaluation rubric

Purpose: This document defines mandatory gates, grading weights, project-specific criteria, bonus rules, deductions, grade bands, and viva questions.

## Mandatory gates

If any mandatory gate fails, the result is Rework required for any score.

| Gate | Evidence |
|---|---|
| CI is green on `main`. | Latest GitHub Actions run passes backend and frontend jobs. |
| Coverage thresholds are met. | Backend line ≥ 85%, branch ≥ 75%, eligibility branch 100%, frontend line ≥ 60%. |
| The system starts from a clean clone. | README setup steps work on the standard or lite profile. |
| There are no secrets in Git history. | Gitleaks report and trainer review pass. |
| The student can explain their code in the viva. | Student changes or debugs one small CampusHire behavior live. |
| Four Playwright journeys run locally. | Saved HTML report covers login, student apply, recruiter pipeline, and TPO dashboard. |
| Resume upload security works. | Demo rejects a renamed text file and a file over 2 MB. |

## Weighted rubric

| Category | Points |
|---|---:|
| Functional completeness (Must FRs work in the demo) | 30 |
| Code quality and design | 15 |
| Testing and coverage | 15 |
| DevOps: CI/CD, Docker, reproducibility | 10 |
| Documentation (README, ADRs, API/CLI/pipeline docs) | 10 |
| Git and engineering practices (PRs, commits, issues) | 5 |
| Final demo and viva | 15 |
| **Total** | **100** |

## Category criteria

### Functional completeness: 30 points

| Level | CampusHire criteria |
|---|---|
| Excellent | All Must FRs work end to end. Aarav can apply only when eligible. Diya sees all blocking reason codes. Rohan cannot access another company's applicants. Meera sees correct dashboard metrics. |
| Good | Most Must FRs work. One minor Must flow needs a small fix, but no security or data-loss rule is broken. |
| Needs work | A core flow such as login, eligibility, application creation, recruiter authorization, or dashboard reporting fails. |

### Code quality and design: 15 points

| Level | CampusHire criteria |
|---|---|
| Excellent | Django apps have clear service boundaries. Eligibility and application state changes are testable. React uses typed props, route guards, and clear API client types. |
| Good | Code is understandable and mostly typed. Some views or components are larger than ideal, but behavior is correct. |
| Needs work | Business rules are duplicated across views, models, and React. State transitions or permissions are hard to audit. |

### Testing and coverage: 15 points

| Level | CampusHire criteria |
|---|---|
| Excellent | Tests cover positive, negative, boundary, security, and E2E cases. Eligibility reason codes have 100% branch coverage. |
| Good | Coverage gates pass. A few Should items or UI edge states lack tests. |
| Needs work | Coverage gates fail, Playwright report is missing, or tests avoid the real eligibility and permission rules. |

### DevOps: CI/CD, Docker, reproducibility: 10 points

| Level | CampusHire criteria |
|---|---|
| Excellent | A clean clone starts with documented steps. Backend and frontend CI jobs are separate. Docker resource limits match the lite profile. |
| Good | CI is green and Docker works, but one setup step needs trainer clarification. |
| Needs work | The trainer cannot start PostgreSQL, Django, and React from the README steps. |

### Documentation: 10 points

| Level | CampusHire criteria |
|---|---|
| Excellent | README, ADRs, API notes, query-plan evidence, and screenshots explain sessions, CSRF, SQL reports, and file-upload security. |
| Good | Required documents exist and are mostly current. One ADR or screenshot is thin. |
| Needs work | Documentation does not match the implemented routes, environment variables, or demo behavior. |

### Git and engineering practices: 5 points

| Level | CampusHire criteria |
|---|---|
| Excellent | PRs are small, linked to issues, named with Conventional Commits, and reviewed with clear checklists. |
| Good | Most work goes through PRs. Some commits are too broad, but history is understandable. |
| Needs work | Direct pushes to `main`, unclear commits, or missing issue links hide project progress. |

### Final demo and viva: 15 points

| Level | CampusHire criteria |
|---|---|
| Excellent | The student completes the 10-minute script, explains trade-offs, and changes one small rule live. |
| Good | The demo covers core flows, but one explanation needs prompting. |
| Needs work | The student cannot explain sessions versus JWT, CSRF, eligibility, ORM queries, or React state used in the project. |

## Bonus rules

Bonus is up to 10 points. Bonus applies only to Could requirements. Bonus applies only when the base score is 60 or more and all mandatory gates pass. The final score is capped at 100.

| Bonus item | Maximum |
|---|---:|
| Interview scheduling with `.ics` invites and timezone-safe display | 3 |
| Dark mode with `light`, `dark`, and `system` choices | 2 |
| Resume keyword extraction with student-visible suggestions | 2 |
| Cost-aware free-tier deployment notes with destroy-after-demo steps | 3 |

## Deductions

| Issue | Deduction |
|---|---:|
| Uses real student, recruiter, or company personal data | Rework required |
| Commits a secret, token, or password | Rework required |
| Publicly exposes resume files without authorization | Rework required |
| Skips local Playwright report | Up to 8 |
| Missing `EXPLAIN ANALYZE` evidence for required queries | Up to 5 |
| README setup takes more than 10 steps or has stale commands | Up to 5 |
| Unexplained AI-generated code in a PR | Up to 10 or Rework required |
| Broken responsive layout at 360 px for Must pages | Up to 5 |

## Grade bands

| Score | Grade |
|---:|---|
| 85–100 | Distinction |
| 70–84 | Merit |
| 60–69 | Pass |
| Below 60 | Rework required |

## Viva questions

1. Why did CampusHire use Django sessions and CSRF instead of JWT for the React SPA?
2. How does the API reject `aarav.rao@gmail.com` during student registration?
3. How does the eligibility engine return all failures for Diya instead of stopping at the first failure?
4. What database constraint prevents Aarav from applying twice to `NAV-FT-2027`?
5. How do `select_related` and `prefetch_related` prevent N+1 queries in the recruiter applicant view?
6. Which indexes support job search by state, deadline, type, and CTC?
7. How did you prove the branch-wise placement report uses a window function efficiently?
8. How does the resume upload check content and protect private media?
9. What React state, effects, and route guard logic run when a recruiter opens `/recruiter/postings`?
10. How would you debug a `403` caused by a missing `X-CSRFToken` header?

[Back to README](../README.md)
