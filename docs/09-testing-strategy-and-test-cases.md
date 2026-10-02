# Testing strategy and test cases

Purpose: This document defines the CampusHire test approach, required tools, environments, coverage gates, test catalog, traceability, performance conditions, defect fields, and release criteria.

## Test strategy

CampusHire uses Python tests throughout. Unit tests cover rules and Decimal calculations. Database/API tests cover permissions, CSRF, constraints, and state. Multi-request API flows replace business-interface automation. The implementation MUST retain at least 55 meaningful test cases; the catalog below exceeds that floor and all Must rows require evidence.

| Level | Approximate count | Main scope | Required evidence |
|---|---:|---|---|
| Unit | 22 | Eligibility reasons, state transitions, report calculations, validators | pytest report |
| Integration and API | 40+ | DRF endpoints, DB constraints, CSRF, role/ownership, OpenAPI, uploads, JSON/CSV | pytest-django report |
| API flows | 8 | Four Must multi-request flows and four Should flows | JUnit and exact response assertions |
| Local lifecycle | 6 | Clean clone, offline fixtures, persistence, errors, loopback, reset | Host-side Python driver and lifecycle evidence |
| Performance and security | 8+ | Report timing, job search timing, upload timing, authorization | Timing summary and security test output |

## Tools and versions

| Area | Tool | Reference line |
|---|---|---|
| Backend tests | pytest | 9.1.x |
| Django integration | pytest-django | 4.14.x |
| Coverage | pytest-cov and coverage | 7.1.x and 7.16.x |
| Test data | factory_boy | 3.3.x |
| Backend quality | Ruff and mypy | 0.16.x and 2.4.x |

## Test environments

| Environment | Purpose | Data volume |
|---|---|---|
| Local lite | Must feature development on 8 GB RAM | Three-student demo seed; isolated small factories for test variants |
| Local standard | API integration and report performance verification | 2,000 students, 80 postings, 5,000 applications in a separate performance DB |
| CI | Automated PR gates | Small deterministic fixtures, four API flows, and local lifecycle acceptance |
| Manual demo | Friday and final demo | Fictional accounts for Aarav, Rohan, Meera, and Kavya |

## Test data strategy

Factories MUST create fictional users and companies only. Seeds MUST include branch identifiers `CSE`, `ISE`, `ECE`, `EEE`, `ME`, and `CV`. Recorded resume fixtures MUST include a valid `1,468,006` byte PDF, plain text named `notes.pdf`, and a `2,100,000` byte PDF. Tests verify recorded SHA-256 and use isolated databases plus controlled clocks. No runtime test fetches external files, sends real e-mail, or contacts a live service. Optional e-mail tests use local Mailpit.

## Local acceptance execution

Implement TC-LOCAL-001 through TC-LOCAL-006 as a host-side Python/pytest driver under `backend/tests/local/`; it invokes document 06's shell entry points and sends HTTP requests through loopback. It MUST not run inside a container that the stop test terminates. Use an isolated Compose project/test volume set so acceptance reset never deletes the student's normal demo data. Save JUnit results plus health JSON, application IDs, event counts, and PDF hashes.

Download/cache dependencies and images before the offline phase. Disable external egress while retaining loopback and Compose networking. The offline phase may neither skip tests nor silently substitute SQLite. Audit scanners and GitHub publication are separate online activities, not runtime prerequisites.

## Coverage thresholds

| Scope | Line coverage | Branch coverage | Measured packages | Exclusions |
|---|---:|---:|---|---|
| Backend application | 85% | 75% | accounts, profiles, companies, postings, applications, reports, notifications | migrations, generated OpenAPI, settings, `__main__` blocks |
| Eligibility module | 100% | 100% | eligibility service and reason helpers | none |
| Placement policy Should | 100% | 100% | dream-offer policy module | none |
| API flows | 4 Must flows | 8 with Should scope | real multi-request business operations | none |

CI MUST fail when line or branch coverage is below a gate.

## Test case catalog

| ID | Title | Type | Priority | Linked requirement IDs | Preconditions | Steps | Test data | Expected result |
|---|---|---|---|---|---|---|---|---|
| TC-API-001 | Register student with college e-mail | API | Must | FR-AUTH-01; BR-01 | New API-client cookie jar has `csrftoken` | POST student register | `aarav.rao@sitm.example.in`, generated `<password>` | Status `201`; role `STUDENT`; profile status `DRAFT` |
| TC-API-002 | Reject non-college student e-mail | API | Must | FR-AUTH-01; BR-01 | New registration session has `csrftoken` cookie | POST student register | `aarav.rao@gmail.com` | Status `400`; error key `college-email-required`; no User row |
| TC-SEC-001 | Reject profile PATCH without CSRF | SEC | Must | FR-AUTH-01; BR-03; CSRF target NFR-SEC-02 | Student session exists with CGPA `8.20` | PATCH `/api/me/profile/` without `X-CSRFToken` | New CGPA `9.00` | Status `403 csrf-failed`; stored CGPA stays `8.20` |
| TC-SEC-006 | Reject login without CSRF | SEC | Must | FR-AUTH-01; BR-03; login target NFR-SEC-02 | Anonymous login request omits the CSRF header | POST login | Aarav credentials | Status `403 csrf-failed`; no session is created |
| TC-SEC-007 | Reject login with invalid CSRF | SEC | Must | FR-AUTH-01; BR-03; invalid-login target NFR-SEC-02 | Anonymous API-client cookie jar has stale CSRF | POST login | Aarav credentials with bad header | Status `403 csrf-failed`; session table has no new row |
| TC-SEC-008 | Reject student registration without CSRF | SEC | Must | FR-AUTH-01; BR-03; student-register target NFR-SEC-02 | Student registration request omits CSRF | POST student register | Aarav e-mail `aarav.rao@sitm.example.in` | Status `403 csrf-failed`; no Student user row |
| TC-SEC-009 | Reject student registration with invalid CSRF | SEC | Must | FR-AUTH-01; BR-03; invalid-student target NFR-SEC-02 | Anonymous test client supplies a bad token | POST student register | Diya e-mail `diya.kulkarni@sitm.example.in` | Status `403 csrf-failed`; no Diya user row |
| TC-SEC-010 | Reject recruiter registration without CSRF | SEC | Must | FR-AUTH-01; BR-03; recruiter-register target NFR-SEC-02 | Recruiter registration request omits CSRF | POST recruiter register | Rohan and Navira Systems | Status `403 csrf-failed`; no recruiter or company row |
| TC-SEC-011 | Reject recruiter registration with invalid CSRF | SEC | Must | FR-AUTH-01; BR-03; invalid-recruiter target NFR-SEC-02 | Recruiter registration cookie/header tokens mismatch | POST recruiter register | Kavita and Teralite Motors | Status `403 csrf-failed`; no recruiter or company row |
| TC-API-003 | Logout invalidates session | API | Must | FR-AUTH-01; BR-02 | Aarav is logged in | POST logout, then GET session | Valid CSRF token | Logout status `204`; session response has `authenticated=false` |
| TC-SEC-002 | Session cookie flags are set | SEC | Must | BR-02; NFR-SEC-01 | Login succeeds outside local HTTP | Inspect response cookies | Recruiter approved account | Cookie has `HttpOnly`, `SameSite=Lax`, and `Secure` |
| TC-API-004 | Recruiter starts pending | API | Must | FR-AUTH-02; BR-04 | New recruiter session has `csrftoken` cookie | POST recruiter register; login; GET recruiter postings | Rohan, Navira Systems | Register `201`; login `200` with `approval_status=PENDING`; postings returns `403 recruiter-pending-approval` |
| TC-API-005 | TPO approves recruiter | API | Must | FR-AUTH-02; BR-04 | Rohan is `PENDING`; Meera is TPO | POST approve; login as Rohan | `rec-rohan` for Navira Systems | Approval response `200 APPROVED`; company endpoint status `200` |
| TC-API-006 | Rejected recruiter sees reason | API | Must | FR-AUTH-02; BR-04 | Recruiter is pending | POST reject then login | Reason `Company email could not be verified` | Login returns `403 recruiter-rejected` with exact reason |
| TC-API-007 | Student saves valid academics | API | Must | FR-PROFILE-01; BR-06 | Aarav logged in | PATCH profile | CGPA `8.20`, branch `CSE`, year `2027`, backlogs `0` | Status `200`; response includes those exact values |
| TC-API-008 | Reject CGPA above 10 | API | Must | FR-PROFILE-01 | Aarav profile CGPA is `8.20` | PATCH profile | CGPA `10.50` | Status `400 cgpa-out-of-range`; stored CGPA remains `8.20` |
| TC-API-009 | Submit incomplete profile fails | API | Must | FR-PROFILE-01; BR-06 | Aarav has no 12th percentage | POST profile submit | Missing `twelfth_percentage` | Status `409 profile-incomplete`; profile remains `DRAFT` |
| TC-API-010 | TPO verifies submitted profile | API | Must | FR-PROFILE-01; BR-07 | Aarav profile is `SUBMITTED` | POST TPO verify | `stu-aarav` in CSE | Status `200`; verification status `VERIFIED`; verifier is Meera |
| TC-API-047 | Verified profile edit resets verification | API | Must | FR-PROFILE-01; BR-07 | Aarav is `VERIFIED` with CGPA `8.20` | PATCH profile, then POST apply | CGPA `9.00`; Navira posting | Profile becomes `DRAFT`; verifier fields clear; apply returns `403 profile-not-verified` |
| TC-API-011 | Upload valid PDF resume | API | Must | FR-PROFILE-02; BR-08; upload target NFR-SEC-03 | Aarav logged in | POST resume multipart | `aarav-resume.pdf`, 1.4 MB, PDF signature | Status `201`; content type `application/pdf`; `is_current=true` |
| TC-API-012 | Reject fake PDF content | API | Must | FR-PROFILE-02; BR-08; content target NFR-SEC-03 | Aarav has current resume | POST resume multipart | `notes.pdf` with plain text | Status `400 resume-not-pdf`; current resume id is unchanged |
| TC-API-013 | Reject oversized resume | API | Must | FR-PROFILE-02; BR-08; size target NFR-SEC-03 | Aarav has current resume | POST resume multipart | PDF size `2,100,000` bytes | Status `400 resume-too-large`; no new ResumeDocument row |
| TC-SEC-003 | Block other-company resume download | SEC | Must | FR-PROFILE-02; FR-REC-01; BR-09; resume target NFR-SEC-04 | Rohan owns Navira; Prava owns application | GET resume as Rohan | `app-diya-prava` | Status `403 resume-access-denied`; response omits storage key |
| TC-API-014 | Create company profile | API | Must | FR-POST-01 | Rohan approved | PATCH recruiter company | Navira Systems details | Status `200`; company linked to Rohan profile |
| TC-API-015 | Create full-time draft posting | API | Must | FR-POST-01; BR-10; BR-11 | Rohan has company | POST recruiter postings | CTC `8.50`, one round, deadline future | Status `201`; state `DRAFT`; title saved |
| TC-API-016 | Reject internship without stipend | API | Must | FR-POST-01; BR-10 | Rohan has company | POST recruiter postings | Type `INTERNSHIP`, stipend blank | Status `400 stipend-required`; no Posting row |
| TC-API-017 | Submit complete draft | API | Must | FR-POST-02; BR-12 | Posting `post-nav-ft-2027` is `DRAFT` | POST submit | Complete posting fields | Status `200`; state `PENDING_APPROVAL` |
| TC-API-018 | TPO approves future posting | API | Must | FR-POST-02; BR-12 | Posting pending with future deadline | POST approve | Deadline `2026-11-20T17:00:00+05:30` | Status `200`; state `PUBLISHED`; approved_by Meera |
| TC-API-019 | TPO rejection reason length enforced | API | Must | FR-POST-02; BR-13 | Posting pending | POST reject | Reason `Bad` | Status `400 reason-required`; posting `post-nav-ft-2027` still has `PENDING_APPROVAL` |
| TC-IT-001 | Close posting after deadline | IT | Must | FR-POST-02; BR-14; deadline target NFR-REL-01 | Published posting deadline is past | Run close-postings command | Current time `2026-11-20T17:05:00+05:30` | State becomes `CLOSED`; `closed_at` is set |
| TC-UT-001 | Eligibility returns eligible | UT | Must | FR-ELIG-01; BR-16 | Aarav verified and no application | Evaluate eligibility | CGPA `8.20`, CSE, year 2027, no backlogs | Label `Eligible`; reason list is empty |
| TC-UT-002 | Eligibility incomplete draft reasons | UT | Must | FR-ELIG-01; BR-15 | Aarav profile is `DRAFT` and lacks phone | Evaluate eligibility | Navira published posting | Reason list is exactly [`PROFILE_INCOMPLETE`, `PROFILE_NOT_VERIFIED`] in that order |
| TC-UT-003 | Eligibility profile not verified reason | UT | Must | FR-ELIG-01; BR-15 | Profile is `SUBMITTED` | Evaluate eligibility | Complete profile | Reason list contains `PROFILE_NOT_VERIFIED` and label is `Not eligible` |
| TC-UT-004 | Eligibility posting not open reason | UT | Must | FR-ELIG-01; BR-15 | Profile verified | Evaluate draft posting | Posting state `DRAFT` | Reason list contains `POSTING_NOT_OPEN` and apply is false |
| TC-UT-005 | Eligibility deadline passed reason | UT | Must | FR-ELIG-01; BR-15 | Profile verified | Evaluate expired published posting | Now equals deadline | Reason list contains `DEADLINE_PASSED` and deadline flag is false |
| TC-UT-006 | Eligibility already applied reason | UT | Must | FR-ELIG-01; BR-15; BR-17 | Aarav already applied | Evaluate same posting | Existing application `APPLIED` | Reason list contains `ALREADY_APPLIED` and duplicate flag is true |
| TC-UT-007 | Eligibility academic mismatch reasons | UT | Must | FR-ELIG-01; BR-15 | Diya verified | Evaluate Navira posting | CGPA `6.40`, branch `EEE`, backlogs `1` | Reason list exactly contains `CGPA_BELOW_MINIMUM`, `BRANCH_NOT_ALLOWED`, and `ACTIVE_BACKLOGS_EXCEEDED` |
| TC-UT-008 | Eligibility graduation year reason | UT | Must | FR-ELIG-01; BR-15 | Farhan verified | Evaluate Navira posting | Year `2026`; allowed years `[2027]` | Reason list contains `GRAD_YEAR_NOT_ALLOWED` and excludes branch failure |
| TC-UT-009 | Dream policy block reason | UT | Should | FR-POLICY-01; BR-28 | Aarav accepted full-time `7.00` LPA | Evaluate full-time posting | New CTC `9.00` LPA | Reason list contains `DREAM_POLICY_BLOCKED` and threshold is `10.50` |
| TC-API-020 | Apply creates application and event | API | Must | FR-APP-01; BR-18; transaction target NFR-REL-02 | Aarav eligible for Navira | POST apply | Posting `post-nav-ft-2027` | Status `201`; state `APPLIED`; one timeline event exists |
| TC-API-048 | Ineligible apply returns reason details | API | Must | FR-APP-01; FR-ELIG-01; BR-15; BR-16 | Diya fails Navira eligibility | POST apply | Diya profile, Navira posting | Status `403 student-not-eligible`; `details.reason_ids` includes all failed rule codes |
| TC-API-021 | Duplicate apply is blocked | API | Must | FR-APP-01; BR-17 | Aarav has application | POST apply again | Same posting | Status `409 application-already-exists`; application count stays `1` |
| TC-API-022 | Apply after deadline is blocked | API | Must | FR-APP-01; BR-18 | Posting deadline passed | POST apply | Now `2026-11-20T17:00:01+05:30` | Status `409 deadline-passed`; no Application row |
| TC-API-023 | Withdraw from APPLIED | API | Must | FR-APP-01; BR-19 | Aarav application is `APPLIED` | POST withdraw | `app-aarav-nav-ft` | Status `200`; state `WITHDRAWN`; event `student_withdraw` |
| TC-API-024 | Withdraw after shortlist fails | API | Must | FR-APP-01; BR-19 | Aarav application is `SHORTLISTED` | POST withdraw | `app-aarav-nav-ft` | Status `409 withdrawal-not-allowed`; `app-aarav-nav-ft` still has `SHORTLISTED` |
| TC-API-025 | Recruiter shortlists one applicant | API | Must | FR-APP-02; BR-20; BR-21 | Rohan owns application in `APPLIED` | PATCH status | `to_state=SHORTLISTED` | Status `200`; timeline actor is Rohan; one event added |
| TC-API-026 | Reject after offered fails | API | Must | FR-APP-02; BR-20 | Application is `OFFERED` | PATCH status | `to_state=REJECTED` | Status `409 transition-not-allowed`; timeline count unchanged |
| TC-API-027 | Student accepts offer | API | Must | FR-APP-02; BR-20 | Application is `OFFERED` | POST accept | Offer CTC `8.50` | Status `200`; state `ACCEPTED`; accepted timestamp exists |
| TC-API-028 | Recruiter rejects during interview | API | Must | FR-APP-02; BR-20 | Application is `IN_INTERVIEW` | PATCH status | `to_state=REJECTED`, reason `Panel rejected` | Status `200`; state `REJECTED`; reason stored |
| TC-API-049 | Full-time offer requires amount | API | Must | FR-APP-02; BR-20 | Application is `IN_INTERVIEW` for full-time posting | PATCH status | `to_state=OFFERED` without `offer_ctc_lpa` | Status `400 offer-amount-required`; `app-aarav-nav-ft` still has `IN_INTERVIEW` |
| TC-API-050 | Full-time offer stores amount | API | Must | FR-APP-02; BR-20 | Application is `IN_INTERVIEW` for full-time posting | PATCH status | `to_state=OFFERED`, `offer_ctc_lpa=8.50` | Status `200`; state `OFFERED`; `offer_made_at` and CTC are visible to student |
| TC-API-029 | Recruiter sees own applicants only | API | Must | FR-REC-01; BR-22 | Navira and Prava applications exist | GET Navira applicants as Rohan | posting `post-nav-ft-2027` | Status `200`; every row company is Navira |
| TC-API-030 | Applicant branch filter | API | Must | FR-REC-01 | Rohan has CSE and ECE applicants | GET applicants with `branch=CSE` | Navira posting | Status `200`; all rows have branch `CSE`; ECE absent |
| TC-API-031 | Applicant minimum CGPA filter | API | Must | FR-REC-01 | Applicants have CGPA `7.99` and `8.20` | GET applicants with `min_cgpa=8.00` | Navira posting | Status `200`; `7.99` applicant absent; `8.20` present |
| TC-SEC-004 | Cross-company applicant list denied | SEC | Must | FR-REC-01; BR-22; ownership target NFR-SEC-04 | Rohan logged in | GET Prava posting applicants | Prava posting id | Status `403 posting-not-owned`; no applicant rows |
| TC-API-032 | Keyword job search | API | Must | FR-JOB-01 | Three published jobs exist | GET jobs with `q=software` | One title and one description contain software | Status `200`; only matching jobs returned |
| TC-API-033 | Job pagination page three | API | Must | FR-JOB-01; BR-23 | 23 jobs match | GET jobs `page=3&page_size=10` | Published jobs | Status `200`; page `3`; results length `3` |
| TC-API-034 | Eligible-only excludes failed jobs | API | Must | FR-JOB-01 | Aarav has one eligible and one ineligible job | GET jobs `eligible_only=true` | Ineligible reason `BRANCH_NOT_ALLOWED` | Status `200`; ineligible posting id absent |
| TC-API-035 | CTC descending sort | API | Must | FR-JOB-01; BR-23 | Two full-time jobs exist | GET jobs `sort=ctc_desc` | CTC `8.50` and `6.25` | First result has CTC `8.50` |
| TC-API-036 | Placement report student counts | API | Must | FR-TPO-01 | Seed 100 students, 72 verified | GET `/api/tpo/reports/summary/` | TPO session | Status `200`; counts are `100` and `72` |
| TC-UT-010 | Placement CTC even median | UT | Must | FR-TPO-01; BR-24 | Accepted full-time offers exist | Calculate metrics | Registered `4`; accepted full-time CTC `6.00`, `8.00`, `10.00`, `12.00` for four distinct students | Placed students `4`; placed percentage `100.00`; average `9.00`; median `9.00`; highest `12.00`; accepted offer count `4` |
| TC-IT-002 | Branch report ranks companies | IT | Must | FR-TPO-01; BR-25; query target NFR-OBS-02 | CSE offers by Navira and Prava | GET branch report | CSE registered `40`; Navira 7 accepted offers, Prava 3 accepted offers | CSE placed `10`; placed percentage `25.00`; Navira dense rank `1`; query-plan evidence saved |
| TC-API-037 | Empty offer aggregates are null | API | Must | FR-TPO-01; BR-24 | Three registered students but no accepted offers | GET report summary | Meera's TPO cookie jar | Status `200`; offer count `0`; placed `0`; percentage `0.00`; all CTC fields `null` |
| TC-API-053 | Student cannot call recruiter endpoints | API | Must | FR-API-01; BR-26 | Aarav has an authenticated student session | GET `/api/recruiter/postings/` | role `STUDENT` | Status `403 role-not-allowed`; no posting data or redirect header |
| TC-API-054 | Empty job search is a valid response | API | Must | FR-API-01 | Published Navira posting exists | GET jobs with `q=unmatched` | Search term absent from all jobs | Status `200`; count `0`; page `1`; page size `10`; results `[]` |
| TC-API-055 | Expired session has a stable error | API | Must | FR-API-01; BR-02 | Aarav's session expires under controlled security clock | GET `/api/me/applications/` with old cookie | Expired session, not wrong role | Status `403 authentication-required`; error envelope has no private data; `Location` absent |
| TC-API-056 | OpenAPI matches real API fixtures | API | Must | FR-API-01 | Generated schema and recorded request/response fixtures exist | Compare profile, search, apply, and failure responses to schema | Decimal CGPA, pagination, 403 and 409 errors | Required fields/types/statuses match; invalid field names fail the contract test |
| TC-API-057 | Summary CSV exact values | API | Must | FR-TPO-01; NFR-DATA-01 | Four verified students each accept one full-time offer | GET summary JSON and `?format=csv` | CTC `6.00,8.00,10.00,12.00` | CSV header/row exactly match document 08; parsed CSV equals JSON aggregates |
| TC-API-058 | Zero registered denominator | API | Must | FR-TPO-01; BR-24 | Isolated DB has zero StudentProfile rows | GET summary in both formats | Empty report dataset | Registered/verified/placed/offers `0`; percentage `0.00`; JSON CTC null and CSV CTC empty |
| TC-API-059 | Distinct students and internship exclusion | API | Must | FR-TPO-01; BR-24 | Four registered; Aarav accepts two full-time offers, Diya one internship and one PPO | GET summary | Aarav CTC `6.00,12.00`; internship stipend `25000`; PPO CTC `20.00` | Placed `1`; percentage `25.00`; accepted offer count `2`; highest `12.00`; average/median `9.00` |
| TC-UT-011 | Odd CTC median | UT | Must | FR-TPO-01; BR-24 | Three distinct accepted full-time offers | Calculate median and average | CTC `6.00,8.00,10.00` | Highest `10.00`; average `8.00`; median `8.00`; no float precision loss |
| TC-API-060 | Branch and company CSV equivalence | API | Must | FR-TPO-01; NFR-DATA-01 | Navira has two CSE offers and Prava one ECE offer | GET both reports as JSON and CSV, then branch filter CSE | Three distinct students with CTC `6.00,8.00,10.00` | Headers/order follow document 08; matching aggregate rows; CSE export has no ECE row |
| TC-API-061 | Report access and format validation | API | Must | FR-API-01; FR-TPO-01 | Student and TPO sessions exist | Request summary as student; invalid format as TPO | `format=pdf`; branch `UNKNOWN` on branch report | Student denied `403 role-not-allowed`; TPO gets `400 invalid-report-format` or `400 branch-invalid` |
| TC-IT-005 | Dense rank ties preserve branch counts | IT | Must | FR-TPO-01; BR-25 | CSE has four registered students; two companies each have two offers | GET branch report | Navira and Prava each offer twice to the same two students | Both ranks `1`; branch placed `2`; percentage `50.00`; registered count stays `4` on each row |
| TC-API-038 | Admin branch appears in choices | API | Must | FR-ADMIN-01; BR-27 | Admin creates active CSE branch | GET master branches | Branch `CSE` | Response includes `CSE` and full branch name |
| TC-API-039 | Admin domain setting controls registration | API | Must | FR-ADMIN-01; BR-01 | Admin setting is `sitm.example.in` | POST student register | `diya@example.in` | Status `400 college-email-required` |
| TC-API-040 | Disabled skill cannot be newly selected | API | Must | FR-ADMIN-01; BR-27 | Skill `Blockchain` is inactive | PATCH profile skills | Add `Blockchain` | Status `400 skill-inactive`; skill not linked |
| TC-API-041 | Dream policy allows 1.5x offer | API | Should | FR-POLICY-01; BR-28 | Aarav accepted `7.00` LPA | POST apply | New CTC `10.50` LPA | Status `201`; no `DREAM_POLICY_BLOCKED` |
| TC-API-042 | Bulk status rejects mixed company | API | Should | FR-REC-02; BR-29 | Rohan selects Navira and Prava applications | PATCH bulk status | Two application ids | Status `403 mixed-company-selection`; no states change |
| TC-IT-003 | Notification de-duplication | IT | Should | FR-NOTIF-01; BR-30 | Deadline reminder job retries twice | Run reminder twice | Key `deadline:NAV-FT-2027:Aarav` | One Notification row exists for Aarav |
| TC-IT-004 | Offer expires after seven days | IT | Should | FR-OFFER-01; BR-31 | Offer made `2026-11-01T10:00:00+05:30` | Run expiry at `2026-11-08T10:01:00+05:30` | Application `OFFERED` | State becomes `DECLINED`; reason `offer-expired` |
| TC-API-043 | Audit log records status change | API | Should | FR-AUDIT-01; BR-32 | Application APPLIED | PATCH status to SHORTLISTED | Reason `Meets criteria` | Audit row has actor, old state, new state, reason, timestamp |
| TC-API-044 | Applicant CSV uses active filters | API | Should | FR-REPORT-01; BR-33 | Navira has CSE and ECE applicants | GET applicant CSV with `branch=CSE` as Rohan and Meera | Fictional CGPA and branches | Header is document 08 applicant header; ECE rows and private storage fields absent |
| TC-SQL-001 | Bounded report query counts | IT | Should | FR-SQL-01; BR-34 | Isolated performance fixture has 5,000 applications | Count SQL queries per summary/branch request | Standard-profile database | Each endpoint uses at most `5` SQL queries; count does not grow with result rows |
| TC-API-045 | Interview invite timezone | API | Could | FR-SCHED-01; BR-35 | Scheduling feature enabled | Create interview slot | `2026-11-22T09:30:00+05:30` | `.ics` download uses Asia/Kolkata or UTC equivalent |
| TC-API-046 | Resume keywords capped at 20 | API | Could | FR-RESUME-02; BR-37 | Keyword extraction enabled | Upload resume with 25 detected terms | Aarav resume | Response stores 20 suggestion keywords |
| TC-PERF-001 | API p95 under 700 ms | PERF | Must | List latency NFR-PERF-01; seeded scale NFR-SCALE-01 | Standard profile has 2,000 students, 80 postings, 5,000 applications | Run authenticated list endpoint mix for 10 minutes | Warm database | 95th percentile response time is below 700 ms |
| TC-PERF-002 | Report under 2 seconds | PERF | Must | NFR-PERF-02 | Standard profile seeded | Request TPO summary 30 times | 5,000 applications | Each measured response is below 2 seconds after warm-up |
| TC-PERF-003 | Upload 2 MB PDF under 5 seconds | PERF | Must | NFR-PERF-03 | Loopback Django API is running | Upload max valid PDF | 2,097,152 bytes | Request completes below 5 seconds with status `201` |
| TC-SEC-005 | Logs omit secrets and resume content | SEC | Must | NFR-PRIV-01 | Login and resume upload executed | Inspect JSON logs | Password, CSRF token, resume text | None of those values appear in logs |
| TC-DOC-001 | Privacy note covers DPDP basics | DQ | Must | NFR-PRIV-02 | Student README drafted | Review privacy note | Purpose, retention, correction, deletion contact | All four items are present |
| TC-INFRA-002 | Backend quality gates pass | INFRA | Must | NFR-MAINT-01 | Backend app exists | Run Ruff and mypy | Application packages | Both commands pass with zero errors |
| TC-INFRA-003 | Report contracts survive serialization | INFRA | Must | NFR-DATA-01 | API provides JSON and CSV reports | Run Python report-schema checks | Empty CTC, Decimal `8.50`, comma in company name | JSON null/decimal strings become empty/two-place CSV cells; quoted names parse without extra columns |
| TC-OBS-001 | JSON logs include request id | DQ | Must | NFR-OBS-01 | API request succeeds | Inspect one log event | GET jobs request | Log has timestamp, level, request ID, user ID, path, status, duration |
| TC-SQL-002 | Large tie-ranking regression | IT | Should | FR-SQL-01; BR-34 | Performance fixture has tied company totals in CSE and ECE | Compare handwritten SQL output to Python reference aggregates | Offer counts `7,7,3` in each branch | Ranks are `1,1,2` per branch; no rank leaks across partitions |
| TC-INFRA-004 | App runs on supported OS targets | INFRA | Must | NFR-PORT-01 | OS prerequisites in document 06 are installed | Run the same start/health/stop commands on WSL2, macOS, Linux | Docker Compose lite profile | Both exact health JSON responses pass on each OS; data is retained after stop |
| TC-DQ-002 | Licence review has no paid Must tool | DQ | Must | NFR-LIC-01 | Dependency list generated | Review required Python and container dependencies | Approved backend packages | Every Must tool has a free licence suitable for public portfolio use |
| TC-COV-001 | Backend and eligibility coverage gates | DQ | Must | NFR-TEST-01 | Backend tests complete | Run coverage | Application packages | Backend line ≥85%, branch ≥75%, eligibility branch 100% |
| TC-COV-002 | Meaningful Python test floor | DQ | Must | NFR-TEST-02 | Implemented tests map to catalog IDs | Collect/run pytest and review case mapping | Positive, negative, boundary, contract, and local cases | At least `55` distinct meaningful cases pass; duplicated assertions do not inflate the count |
| TC-FLOW-001 | Student completes profile and applies | IT | Must | FR-AUTH-01; FR-PROFILE-01; FR-PROFILE-02; FR-APP-01; NFR-TEST-03 | Isolated Aarav starts DRAFT; TPO and published Navira exist | Python client login, PATCH profile, POST PDF, submit, TPO verify, GET eligibility, POST apply | Recorded PDF and valid academics | Apply `201 APPLIED`; one timeline event; profile VERIFIED; JUnit evidence saved |
| TC-FLOW-002 | Recruiter shortlists authorized applicant | IT | Must | Applicant scope FR-REC-01; pipeline FR-APP-02; integration NFR-TEST-03 | Aarav applied; Rohan approved | Login Rohan, GET filtered applicants, GET PDF, PATCH SHORTLISTED, GET timeline | Navira CSE applicant | PDF status `200`; final state `SHORTLISTED`; exactly one shortlist event with actor Rohan |
| TC-FLOW-003 | TPO approves posting and reads reports | IT | Must | Approval FR-POST-02; reporting FR-TPO-01; multi-request NFR-TEST-03 | Complete Navira draft has future deadline under controlled clock | Recruiter submits; Meera approves; student GET job; TPO GET summary JSON/CSV | Isolated three-student seed, zero accepted offers | State `PUBLISHED`; registered `3`; offer count `0`; CTC null/empty representations agree |
| TC-FLOW-004 | Unauthorized multi-role access stays blocked | IT | Must | Role contract FR-API-01; company scope FR-REC-01; denial flow NFR-TEST-03 | Student and Navira recruiter sessions exist | Student GET recruiter postings; Rohan GET Prava applicants/PDF; repeat protected request anonymously | Separate company application | `403 role-not-allowed`, `posting-not-owned`, `resume-access-denied`, `authentication-required`; no private data or redirect |
| TC-FLOW-005 | Local notification reminder is API-readable | IT | Should | FR-NOTIF-01 | Local notifications enabled; deadline due in 24 hours | Run reminder twice; login student; GET notifications; inspect Mailpit | One due Navira reminder | One notification row and one logical delivery de-dup key; unread `read_at=null` |
| TC-FLOW-006 | Bulk shortlist is atomic | IT | Should | FR-REC-02 | Twelve Navira APPLIED applications exist | Python recruiter client PATCH bulk SHORTLISTED, then GET applicants | Twelve IDs, all owned by Rohan | All twelve states `SHORTLISTED`; exactly twelve events added |
| TC-FLOW-007 | Filtered applicant export respects ownership | IT | Should | FR-REPORT-01 | CSE/ECE Navira applicants exist | Login Rohan; request CSE CSV; try Prava export | Approved recruiter session | CSV contains only CSE applicants; Prava export `403 posting-not-owned` |
| TC-FLOW-008 | Offer expiry appears in timeline API | IT | Should | FR-OFFER-01 | OFFERED application passes seven-day deadline | Run local expiry command; GET own applications and timeline | Controlled offer clock | State `DECLINED`; last event reason `offer-expired` |
| TC-API-051 | Second accepted offer withdraws active applications | API | Should | FR-POLICY-01; BR-28 | Aarav has one accepted full-time offer and active applications in `APPLIED`, `SHORTLISTED`, `IN_INTERVIEW`, and `OFFERED` | Accept second full-time offer | Second offer CTC `12.00` | Status `200`; second state `ACCEPTED`; all other active applications become `WITHDRAWN` with `policy_withdraw` in one transaction |
| TC-API-052 | New applications blocked after second offer | API | Should | FR-POLICY-01; BR-28 | Aarav has two accepted full-time offers | POST apply to three postings | Full-time `15.00` LPA, internship stipend `25000`, and internship-with-PPO stipend `30000` | All three attempts return `403 policy-limit-reached`; no Application rows are created |

## Required local acceptance cases

| ID | Title | Type | Priority | Linked requirement IDs | Preconditions | Steps | Test data | Expected result |
|---|---|---|---|---|---|---|---|---|
| TC-LOCAL-001 | Clean-clone startup initializes everything | INFRA | Must | Startup FR-LOCAL-01; local contract BR-38; portability NFR-PORT-01 | Clean implementation clone; cached pinned images/uv dependencies; empty isolated test volumes | Run document 06 start; GET live/ready; login seeded Aarav; run start again | Three-student seed and recorded PDFs | Start prints exact ready line; live `200 ok`; ready `200 ready/database ok/media ok`; second start changes no seed IDs or row counts |
| TC-LOCAL-002 | Deterministic eligibility without internet | API | Must | Fixture demonstration FR-LOCAL-01; offline runtime NFR-LOCAL-01 | Initial downloads complete; loopback/Compose retained but external egress disabled | Start cached lite profile; GET CSRF; login Aarav; GET fixed Navira UUID eligibility; run runtime pytest | `LOCAL_DEMO_NOW=2026-11-01T10:00:00+05:30`; document 06 fixtures | Eligibility `200` with exact `{"label":"Eligible","reason_ids":[],"messages":[]}`; runtime tests pass without external fetches or live SMTP |
| TC-LOCAL-003 | Persist application, session, timeline, and resume | IT | Must | Restart FR-LOCAL-01; retained volumes BR-38; durable state NFR-REL-03 | Isolated seeded stack, no Navira application | POST apply; POST recorded PDF; save IDs/hash/cookie; stop; start within session lifetime; GET session with old cookie; GET application/timeline/PDF | Aarav APPLIED record and private PDF | Old session remains authenticated; application ID/state remain; timeline count stays `1`; current PDF ID/SHA-256 remain; restart does not reverify the edited profile or replace credentials |
| TC-LOCAL-004 | Invalid input and local failures are explicit | API | Must | Startup failure FR-LOCAL-01; actionable diagnostics NFR-LOCAL-02 | Seeded stack and current PDF exist | PATCH CGPA `10.50`; stop DB and probe readiness/API; restore DB; inject media write failure; attempt startup with occupied API port | Existing CGPA `8.20`, old resume hash | `400 cgpa-out-of-range`; ready `503 database unavailable`; protected API `503 database-unavailable`; upload `503 resume-storage-unavailable` retains old PDF; occupied-port start nonzero with document 06 diagnostic |
| TC-LOCAL-005 | Published ports are loopback only | INFRA | Must | FR-LOCAL-01; BR-38 | Lite stack running; optional Mailpit profile may be enabled | Inspect resolved Compose bindings and connection checks | Host ports `8000,15432,11025,18025` | Every published binding is `127.0.0.1`; no `0.0.0.0` exposure; internal DB remains `5432` |
| TC-LOCAL-006 | Reset requires confirmation and is project-scoped | INFRA | Must | Reset entry point FR-LOCAL-01; explicit scope BR-38; retention guard NFR-REL-03 | Disposable acceptance volumes contain an application; separate sentinel volume exists | Run reset without flag; inspect records; then reset with exact confirmation flag | Isolated acceptance data only | Unconfirmed reset nonzero with exact confirmation diagnostic, application intact; confirmed reset removes only acceptance volumes/credentials and leaves sentinel untouched |

Trainer pre-check `LOCAL-PRECHECK` is the checklist in document 06. Save the OS/hardware profile, fixture hashes, test output, and measured resource use; proposed budgets are not claimed measurements.

## Named demo steps

| Demo step ID | Requirement links | Steps | Expected result |
|---|---|---|---|
| DEMO-ADMIN-001 | BR-05 | Create staff account Meera Nair with the built-in local Django admin, then authenticate Meera through the API and GET `/api/tpo/reports/summary/`. | API returns `200`; no self-service TPO registration exists; no custom admin interface is required. |

## Traceability matrix

| Requirement ID | Verification |
|---|---|
| FR-AUTH-01 | Auth coverage: TC-API-001; TC-API-002; TC-SEC-001; TC-SEC-006; TC-SEC-007; TC-SEC-008; TC-SEC-009; TC-SEC-010; TC-SEC-011; TC-API-003; TC-FLOW-001 |
| FR-AUTH-02 | Recruiter approval coverage: TC-API-004; TC-API-005; TC-API-006 |
| FR-PROFILE-01 | Profile coverage: TC-API-007; TC-API-008; TC-API-009; TC-API-010; TC-API-047; TC-FLOW-001 |
| FR-PROFILE-02 | Resume coverage: TC-API-011; TC-API-012; TC-API-013; TC-SEC-003 |
| FR-POST-01 | Draft posting coverage: TC-API-014; TC-API-015; TC-API-016 |
| FR-POST-02 | Posting lifecycle coverage: TC-API-017; TC-API-018; TC-API-019; TC-IT-001; TC-FLOW-003 |
| FR-ELIG-01 | Eligibility coverage: TC-UT-001; TC-UT-002; TC-UT-003; TC-UT-004; TC-UT-005; TC-UT-006; TC-UT-007; TC-UT-008; TC-UT-009 |
| FR-APP-01 | Apply coverage: TC-API-020; TC-API-021; TC-API-022; TC-API-023; TC-API-024; TC-API-048; TC-FLOW-001 |
| FR-APP-02 | Pipeline coverage: TC-API-025; TC-API-026; TC-API-027; TC-API-028; TC-API-049; TC-API-050; TC-FLOW-002 |
| FR-REC-01 | TC-SEC-003; TC-API-029; TC-API-030; TC-API-031; TC-SEC-004 |
| FR-JOB-01 | Job search coverage: TC-API-032; TC-API-033; TC-API-034; TC-API-035 |
| FR-TPO-01 | Reports: TC-API-036; TC-UT-010; TC-UT-011; TC-IT-002; TC-IT-005; TC-API-037; TC-API-057; TC-API-058; TC-API-059; TC-API-060; TC-API-061; TC-FLOW-003 |
| FR-API-01 | API contracts/permissions: TC-API-053; TC-API-054; TC-API-055; TC-API-056; TC-API-061; TC-FLOW-004 |
| FR-ADMIN-01 | Admin data coverage: TC-API-038; TC-API-039; TC-API-040 |
| FR-POLICY-01 | Policy coverage: TC-UT-009; TC-API-041; TC-API-051; TC-API-052 |
| FR-REC-02 | TC-API-042, TC-FLOW-006 |
| FR-NOTIF-01 | TC-IT-003, TC-FLOW-005 |
| FR-OFFER-01 | TC-IT-004, TC-FLOW-008 |
| FR-AUDIT-01 | TC-API-043 |
| FR-REPORT-01 | TC-API-044, TC-FLOW-007 |
| FR-SQL-01 | TC-SQL-001; TC-SQL-002 |
| FR-SCHED-01 | TC-API-045 |
| FR-RESUME-02 | TC-API-046 |
| FR-LOCAL-01 | TC-LOCAL-001; TC-LOCAL-002; TC-LOCAL-003; TC-LOCAL-004; TC-LOCAL-005; TC-LOCAL-006 |
| Auth and profile rules: BR-01; BR-02; BR-03; BR-04; BR-05; BR-06; BR-07; BR-08; BR-09 | Early rule coverage: TC-API-001; TC-API-003; TC-SEC-001; TC-API-005; DEMO-ADMIN-001; TC-API-009; TC-API-010; TC-API-011; TC-SEC-003 |
| Posting, eligibility, and application rules: BR-10; BR-11; BR-12; BR-13; BR-14; BR-15; BR-16; BR-17; BR-18; BR-19 | Draft/state/deadline/reason/uniqueness checks: TC-API-015; TC-API-017; TC-API-018; TC-API-019; TC-IT-001; TC-UT-002; TC-UT-001; TC-API-021; TC-API-022; TC-API-024 |
| BR-20; BR-21; BR-22; BR-23; BR-24; BR-25; BR-26; BR-27 | State and guard coverage: TC-API-025; TC-SEC-004; TC-API-033; TC-UT-010; TC-IT-002; TC-API-053; TC-API-038 |
| Policy, exports, scheduling, and local lifecycle: BR-28; BR-29; BR-30; BR-31; BR-32; BR-33; BR-34; BR-35; BR-37; BR-38 | TC-UT-009; TC-API-042; TC-IT-003; TC-IT-004; TC-API-043; TC-API-044; TC-SQL-001; TC-API-045; TC-API-046; TC-LOCAL-001; TC-LOCAL-005; TC-LOCAL-006 |
| NFR-PERF-01 | TC-PERF-001 |
| NFR-PERF-02 | TC-PERF-002 |
| NFR-PERF-03 | TC-PERF-003 |
| NFR-SCALE-01 | TC-PERF-001 |
| NFR-REL-01 | TC-IT-001 |
| NFR-REL-02 | TC-API-020 |
| NFR-REL-03 | Saved session/data state and confirmed reset: TC-LOCAL-003; TC-LOCAL-006 |
| NFR-SEC-01 | TC-SEC-002 |
| NFR-SEC-02 | CSRF coverage: TC-SEC-001; TC-SEC-006; TC-SEC-007; TC-SEC-008; TC-SEC-009; TC-SEC-010; TC-SEC-011 |
| NFR-SEC-03 | Upload security coverage: TC-API-011; TC-API-012; TC-API-013 |
| NFR-SEC-04 | TC-SEC-003; TC-SEC-004 |
| NFR-PRIV-01 | TC-SEC-005 |
| NFR-PRIV-02 | TC-DOC-001 |
| NFR-MAINT-01 | TC-INFRA-002 |
| NFR-DATA-01 | Serialization and aggregate equivalence: TC-INFRA-003; TC-API-057; TC-API-060 |
| NFR-OBS-01 | TC-OBS-001 |
| NFR-OBS-02 | TC-IT-002 |
| NFR-LOCAL-01 | TC-LOCAL-002 |
| NFR-LOCAL-02 | TC-API-008; TC-LOCAL-004 |
| NFR-PORT-01 | OS entry-point verification: TC-INFRA-004; TC-LOCAL-001 |
| NFR-LIC-01 | TC-DQ-002 |
| NFR-TEST-01 | TC-COV-001 |
| NFR-TEST-02 | TC-COV-002 |
| NFR-TEST-03 | Four Python/API flows: TC-FLOW-001; TC-FLOW-002; TC-FLOW-003; TC-FLOW-004 |
| NFR-HW-01 | LOCAL-PRECHECK: trainer executes document 06 lite pre-check on 8 GB/four-core laptop before week 1 |

## Performance test conditions

| Condition | Exact requirement |
|---|---|
| Reference hardware | 16 GB RAM standard profile with Docker memory 8 GB and swap 4 GB |
| Lite check | 8 GB RAM with Docker memory 4 GB and swap 4 GB |
| Seed data | 2,000 students, 100 recruiters, 80 postings, 5,000 applications |
| Warm-up | 2 minutes before measurements |
| Duration | 10 minutes for API mix; 30 repeated summary requests |
| Request mix | 45% job search, 20% applicant list, 15% TPO reports, 10% eligibility, 10% login/session |
| Pass criteria | API p95 below 700 ms; summary below 2 seconds; 2 MB upload below 5 seconds |
| Evidence | Summary table, run date, hardware profile, database row counts, query-plan note |

## Defect report fields

| Field | Required value |
|---|---|
| Title | Short failing behavior, for example `Recruiter can download Prava resume` |
| Requirement links | FR, BR, or NFR IDs affected |
| Environment | OS, Python version, backend commit, Docker profile |
| Steps to reproduce | Numbered steps with exact test account and input |
| Expected result | Exact status, JSON/CSV fields, database state, or persisted hash |
| Actual result | Observed status/body, row/event count, or hash |
| Evidence | Request ID, sanitized response, JUnit output, or query-plan note |
| Severity | Blocker, High, Medium, or Low |
| Owner | Student assignee |
| Fix verification | Test case ID or demo step that proves the fix |

## Entry criteria

1. The data model migrations exist for the feature under test.
2. Factories or seed data create the required fictional records.
3. The endpoint or local command has documented expected errors.
4. The developer can run the relevant test command locally.
5. No test uses real student, recruiter, or company data.

## Exit criteria

1. All Must test cases pass locally.
2. Backend coverage is at least 85% line and 75% branch.
3. The eligibility module has 100% branch coverage.
4. At least 55 meaningful Python cases pass, including report representations and role/ownership checks.
5. Four Must API integration flows and all six local acceptance cases pass with saved evidence.
6. Security tests for CSRF, cross-company resume access, and cross-student application access pass.
7. Query-plan evidence exists for branch-wise report and job search.
8. No Blocker or High defect remains open for a Must requirement.

[Back to README](../README.md)
