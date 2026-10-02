# Milestones and deliverables

Purpose: This document gives the CampusHire effort budget, six-week plan, Gantt chart, final checklist, submission steps, and demo script.

## Effort budget by feature area

The total project budget remains about 180 hours; Must scope is 120 hours. It includes 30 hours of compulsory learning: 18 hours of deeper Python, SQL, and API learning and 12 hours of setup, backend fundamentals, testing, and CI learning. Removed interface/deployment work is assessed through replacement backend and local-operation deliverables.

| Feature area | Priority | Hours | Notes |
|---|---|---:|---|
| Setup, Git, CI skeleton, Docker, and other compulsory learning | Must | 12 | Includes first clean clone run and 12 hours of setup, backend, SQL, testing, and CI learning. |
| Django accounts, sessions, CSRF, and roles | Must | 12 | Covers FR-AUTH-01 and FR-AUTH-02. |
| Student profile, academics, skills, verification, and resume upload | Must | 14 | Covers FR-PROFILE-01 and FR-PROFILE-02. |
| Company, posting draft, approval, and deadline closure | Must | 13 | Covers FR-POST-01 and FR-POST-02. |
| Eligibility engine, apply, withdraw, and application pipeline | Must | 16 | Covers FR-ELIG-01, FR-APP-01, and FR-APP-02. |
| Recruiter applicants, job search, JSON/CSV reports, and SQL evidence | Must | 15 | Covers FR-REC-01, FR-JOB-01, and FR-TPO-01. |
| Python, SQL, and API learning | Must | 18 | Decimal/time handling, service boundaries, aggregates/window functions, and API contracts. |
| API permissions, schema/serialization checks, and integration flows | Must | 10 | Covers FR-API-01 and four Must Python/API flows. |
| Local lifecycle, Django admin master data, tests, and Must documentation | Must | 10 | Covers FR-LOCAL-01, FR-ADMIN-01, six local cases, coverage gates, ADRs, and setup evidence. |
| Should features and hardening | Should | 40 | Policy, local notifications, audit, filtered exports, SQL query budgets, and extra tests. |
| Final documentation, interview prep, demo video, and contingency | Should | 20 | Final polishing, viva practice, demo video, and buffer after Must scope. |
| **Total** |  | **180** |  |

## Week-by-week plan

| Week | Goals | Tasks naming FR IDs | Deliverables | Friday demo checkpoint |
|---|---|---|---|---|
| 1: 5-9 Oct | Set up tools and the first vertical slice. | Read docs; create `campushire`; configure Django/PostgreSQL/CI; record session/CSRF ADR; start FR-AUTH-01, FR-API-01, FR-LOCAL-01. | Existing skeleton checks green; start/stop, health, migrations, first seed, PR, Projects board. | Show clean local startup and CI on a PR. |
| 2: 12-16 Oct | Build accounts, profile, and resume foundations. | Finish FR-AUTH-01/02; implement FR-PROFILE-01; begin FR-PROFILE-02; deepen Python validators and SQL constraints. | Registration/login APIs, recruiter approval, profile serialization, Python tests. | Show valid/invalid registration and profile requests. |
| 3: 19-23 Oct | Build postings and eligibility. | Finish FR-PROFILE-02; implement FR-POST-01/02 and FR-ELIG-01; test clocks, reason ordering, and ownership. | Posting approval, deadline guard, ordered JSON eligibility, safe PDF tests. | Show Diya's three exact blocking codes. |
| 4: 26-30 Oct | Complete Must workflows and local acceptance. | Implement FR-APP-01/02, FR-REC-01, FR-JOB-01, FR-TPO-01, FR-ADMIN-01; finish FR-API-01/FR-LOCAL-01 and query plans. | Four API flows; JSON/CSV reports; 55+ meaningful tests; all six local cases. | Show Aarav applies, recruiter shortlists, offline eligibility, and persisted stop/start. |
| 5: 2-6 Nov | Add Should items and harden quality. | Attempt FR-POLICY-01, FR-AUDIT-01, FR-REPORT-01, FR-SQL-01, and selected FR-NOTIF-01; close test gaps. | Dream policy if selected, audit records, filtered CSV, query-budget tests. | Show one Should item and updated Python coverage. |
| 6: 9-13 Nov | Prepare final submission and viva. | Finish documentation, ADRs, API/local JUnit evidence, demo video, clean-clone instructions, and interview practice. | Final tag `v1.0.0`, green `main`, demo video under 5 minutes, submission issue. | Run the 10-minute final demo script. |

## Gantt chart

```mermaid
gantt
    title CampusHire six-week plan
    dateFormat  YYYY-MM-DD
    section Setup and learning
    Tools, Git, Docker, CI           :a1, 2026-10-05, 5d
    Python, SQL, API learning        :a2, 2026-10-05, 20d
    section Must backend
    Auth and roles                   :b1, 2026-10-08, 7d
    Profile and resume               :b2, 2026-10-13, 8d
    Postings and eligibility         :b3, 2026-10-21, 8d
    Applications and recruiter views :b4, 2026-10-26, 5d
    REST reports and SQL evidence    :b5, 2026-10-27, 4d
    section API and local contracts
    Role and ownership tests         :c1, 2026-10-12, 12d
    Schema and API integration       :c2, 2026-10-26, 5d
    Offline and persisted lifecycle  :c3, 2026-10-28, 3d
    section Quality and delivery
    Python tests and local checks    :d1, 2026-10-20, 11d
    Should items and hardening       :d2, 2026-11-02, 5d
    Final docs and viva              :d3, 2026-11-09, 5d
```

## Final deliverables checklist

- Public GitHub repository named `campushire`.
- Trainer `@sukurcf` added as collaborator.
- README setup steps from a clean clone in 10 steps or fewer.
- Architecture diagram and at least 5 ADRs.
- Green CI on `main`.
- Backend coverage report with required gates.
- At least 55 meaningful Python cases and JUnit evidence for four Must API flows.
- TC-LOCAL-001 through TC-LOCAL-006 evidence: health, offline result, persisted IDs/PDF hashes, errors, loopback bindings, safe reset.
- Evidence for branch-wise and job-search `EXPLAIN ANALYZE`.
- `.env.example` without secrets.
- CHANGELOG and tag `v1.0.0`.
- Demo video of maximum 5 minutes.
- Final demo script practiced once before evaluation.

## Submission process

1. Merge all required PRs to `main`.
2. Confirm CI is green on the final commit.
3. Create tag `v1.0.0`.
4. Upload or link Python API/local acceptance reports and sanitized response/hash evidence in the student repository.
5. Add the demo video link to the student README.
6. Open a GitHub Issue in this specification repository with title `[Submission] CampusHire - <student name>`.
7. Include the repository link, final tag, CI run link, coverage summary, and demo video link.

## 10-minute demo script

| Minute | What to show |
|---:|---|
| 0 | State the problem: TPO spreadsheets allow ineligible applications and missed deadlines. |
| 1 | Run the local start entry point; show exact health responses and login with a cookie-aware API client. |
| 2 | Show the registration test for `aarav.rao@sitm.example.in` and CSRF rejection on an unsafe request. |
| 3 | Show Aarav's profile, verified status, and PDF resume upload rule. |
| 4 | Show Rohan's approved recruiter account and Navira Systems posting. |
| 5 | Show Meera approves a posting and explain `DRAFT` to `PUBLISHED`. |
| 6 | Show Diya blocked by `CGPA_BELOW_MINIMUM`, `BRANCH_NOT_ALLOWED`, and `ACTIVE_BACKLOGS_EXCEEDED`. |
| 7 | Show Aarav applies once, then the duplicate application is rejected. |
| 8 | Show Rohan moves Aarav to `SHORTLISTED` and the timeline updates. |
| 9 | Request JSON/CSV TPO reports and show SQL query-plan evidence; show offline eligibility and persisted stop/start results. |
| 10 | Show CI, Python coverage, local/API JUnit reports, logout denial, and answer one viva question. |

[Back to README](../README.md)
