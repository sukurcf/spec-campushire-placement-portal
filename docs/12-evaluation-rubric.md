# Evaluation rubric

Purpose: This document defines mandatory gates, grading weights, project-specific criteria, bonus rules, deductions, grade bands, and viva questions.

## Mandatory gates

If any mandatory gate fails, the result is Rework required for any score.

| Gate | Evidence |
|---|---|
| CI is green on `main`. | Latest GitHub Actions run passes Python/API, security/image, and local lifecycle jobs. |
| Coverage thresholds and test floor are met. | Python line ≥ 85%, branch ≥ 75%, eligibility branch 100%; at least 55 meaningful cases. |
| Local operation contract passes. | TC-LOCAL-001 through TC-LOCAL-006 prove clean clone, exact health/demo responses, offline runtime, persistence, actionable errors, loopback, and confirmed reset. |
| There are no secrets in Git history. | Gitleaks report and trainer review pass. |
| The student can explain their code in the viva. | Student changes or debugs one small CampusHire behavior live. |
| Four Python/API flows run locally. | TC-FLOW-001 through TC-FLOW-004 have JUnit and sanitized response evidence. |
| Resume upload security works. | Demo rejects a renamed text file and a file over 2 MB. |

## Weighted rubric

Version 1.1 removes all frontend and public-deployment deliverables and their grading criteria. The same 100-point category weights now assess the replacement Python, SQL, API-contract, permission, reporting, testing, and local-operation evidence. No optional interface work earns points.

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
| Excellent | All Must FRs work through the API. Aarav applies only when eligible; Diya receives every blocking code; Rohan cannot access another company's applicants; Meera receives exact JSON/CSV report values. |
| Good | Most Must FRs work. One minor Must flow needs a small fix, but no security or data-loss rule is broken. |
| Needs work | A core flow such as login, eligibility, application creation, recruiter authorization, or REST reporting fails. |

### Code quality and design: 15 points

| Level | CampusHire criteria |
|---|---|
| Excellent | Django apps have clear typed service boundaries. Eligibility and atomic state/event changes are testable; Decimal, datetime, serializers, permissions, and handwritten SQL preserve exact contracts. |
| Good | Code is understandable and mostly typed. Some API views or services are larger than ideal, but behavior is correct. |
| Needs work | Business rules are duplicated across views and models. State transitions, ownership, or aggregate semantics are hard to audit. |

### Testing and coverage: 15 points

| Level | CampusHire criteria |
|---|---|
| Excellent | Tests cover positive, negative, boundary, security, JSON/CSV, multi-request, offline, and persistence cases. Eligibility reason codes have 100% branch coverage. |
| Good | All mandatory coverage/local gates pass. A few selected Should edge cases lack tests. |
| Needs work | Coverage/test-floor gates fail, API/local evidence is missing, or tests avoid the real eligibility and permission rules. |

### DevOps: CI/CD, Docker, reproducibility: 10 points

| Level | CampusHire criteria |
|---|---|
| Excellent | One documented start initializes migrations/fixtures; stop/start retains DB and PDFs; offline runtime and error cases pass. CI separates Python/API and local lifecycle verification; resource limits match lite. |
| Good | CI is green and Docker works, but one setup step needs trainer clarification. |
| Needs work | The trainer cannot start PostgreSQL/Django from the documented entry point or local acceptance is not reproducible. |

### Documentation: 10 points

| Level | CampusHire criteria |
|---|---|
| Excellent | README, ADRs, OpenAPI/CSV notes, query plans, JUnit, and sanitized response/hash evidence explain sessions, CSRF, SQL reports, upload security, and local lifecycle. |
| Good | Required documents exist and are mostly current. One ADR or evidence record is thin. |
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
| Needs work | The student cannot explain sessions versus JWT, CSRF, eligibility, ORM queries, report semantics, or persisted local operation. |

## Bonus rules

Bonus is up to 10 points. Bonus applies only to Could requirements. Bonus applies only when the base score is 60 or more and all mandatory gates pass. The final score is capped at 100.

| Bonus item | Maximum |
|---|---:|
| FR-SCHED-01: API-generated `.ics` invites with timezone/boundary tests | 5 |
| FR-RESUME-02: Python keyword extraction, 20-term cap, and repeatable API tests | 5 |

The removed bonus allocation is reassigned to these existing backend Could items. No cloud deployment, custom console, or student-built interface is a bonus.

## Deductions

| Issue | Deduction |
|---|---:|
| Uses real student, recruiter, or company personal data | Rework required |
| Commits a secret, token, or password | Rework required |
| Publicly exposes resume files without authorization | Rework required |
| Skips required API/local evidence | Rework required under mandatory gates |
| Missing `EXPLAIN ANALYZE` evidence for required queries | Up to 5 |
| README setup takes more than 10 steps or has stale commands | Up to 5 |
| Unexplained AI-generated code in a PR | Up to 10 or Rework required |
| Stale OpenAPI/CSV contracts or unclear local dependency diagnostics | Up to 5; mandatory behavior failures still require rework |

## Grade bands

| Score | Grade |
|---:|---|
| 85–100 | Distinction |
| 70–84 | Merit |
| 60–69 | Pass |
| Below 60 | Rework required |

## Viva questions

1. How do Python/API clients use Django sessions and CSRF, including anonymous login protection?
2. How does the API reject `aarav.rao@gmail.com` during student registration?
3. How does the eligibility engine return all failures for Diya instead of stopping at the first failure?
4. What database constraint prevents Aarav from applying twice to `NAV-FT-2027`?
5. How do `select_related` and `prefetch_related` prevent N+1 queries in the recruiter applicant view?
6. Which indexes support job search by state, deadline, type, and CTC?
7. How did you prove the branch-wise placement report uses a window function efficiently?
8. How does the resume upload check content and protect private media?
9. Which permission and ownership checks run for `/api/recruiter/postings/`, and how do the JSON error keys distinguish wrong role from missing session?
10. How would you debug a `403` caused by a missing `X-CSRFToken` header?

[Back to README](../README.md)
