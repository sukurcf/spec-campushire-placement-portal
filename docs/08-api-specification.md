# API specification

Purpose: This document defines the CampusHire REST contracts, permissions, JSON/CSV reports, and API-client integration flows.

## REST API

The API base path is `/api/`. Responses use JSON except authorized PDF downloads and CSV reports. Local clients call `http://127.0.0.1:8000` directly; no custom business interface is implemented. Authenticated endpoints use Django session authentication. Unsafe methods MUST include `X-CSRFToken`. In endpoint tables, role `TPO` also allows `ADMIN` unless the row explicitly says otherwise.

### Shared rules

| Item | Exact value |
|---|---|
| CSRF cookie | `csrftoken` |
| CSRF header | `X-CSRFToken` |
| Unsafe methods | `POST`, `PUT`, `PATCH`, `DELETE` |
| Session idle lifetime | 8 hours |
| Session absolute lifetime | 14 days |
| Cookie flags | `HttpOnly`, `SameSite=Lax`, `Secure` outside local HTTP |
| Default page size | 10 |
| Maximum page size | 50 |
| Timestamp format | ISO 8601 with offset, for example `2026-11-20T17:00:00+05:30` |

Protected endpoints return `403 authentication-required` when a session is missing or expired. Wrong roles return `403 role-not-allowed`; object-ownership failures keep their specific keys. These responses contain no redirects or `Location` header. Session/CSRF clients retain cookies and reread the CSRF cookie after login because Django rotates it.

### Error format

All JSON errors MUST use this shape.

```json
{
  "error": {
    "key": "college-email-required",
    "message": "Use your sitm.example.in college e-mail address.",
    "field": "email",
    "details": {}
  }
}
```

### Pagination format

List endpoints return this shape.

```json
{
  "count": 23,
  "page": 3,
  "page_size": 10,
  "results": []
}
```

### Endpoint summary

| Method | Path | Roles | Purpose | Main statuses |
|---|---|---|---|---|
| GET | /health/live/ | Anonymous | Process liveness | 200 |
| GET | /health/ready/ | Anonymous | Local DB and private-media readiness | 200, 503 |
| GET | /api/schema/ | Anonymous | OpenAPI contract without credentials | 200 |
| GET | /api/csrf/ | Anonymous | Set CSRF cookie | 204 |
| GET | /api/session/ | Anonymous | Read current session | 200 |
| POST | /api/auth/student-register/ | Anonymous | Register student | 201, 400 |
| POST | /api/auth/recruiter-register/ | Anonymous | Register recruiter | 201, 400 |
| POST | /api/auth/login/ | Anonymous | Login | 200, 400, 403 |
| POST | /api/auth/logout/ | Authenticated | Logout | 204, 403 |
| GET/PATCH | /api/me/profile/ | Student | Read or update own profile | 200, 400, 403 |
| POST | /api/me/profile/submit/ | Student | Submit profile for TPO verification | 200, 409 |
| POST | /api/me/resume/ | Student | Upload current resume | 201, 400, 403 |
| GET | /api/me/applications/ | Student | List own applications | 200 |
| GET | /api/jobs/ | Student | Search published jobs | 200, 400 |
| GET | /api/jobs/{posting_id}/ | Student | Read job detail | 200, 404 |
| GET | /api/jobs/{posting_id}/eligibility/ | Student | Check eligibility | 200 |
| POST | /api/jobs/{posting_id}/apply/ | Student | Apply to job | 201, 403, 409 |
| POST | /api/applications/{application_id}/withdraw/ | Student | Withdraw own application | 200, 409 |
| POST | /api/applications/{application_id}/accept/ | Student | Accept offer | 200, 409 |
| POST | /api/applications/{application_id}/decline/ | Student | Decline offer | 200, 409 |
| GET | /api/applications/{application_id}/timeline/ | Student, Recruiter, TPO | Timeline events | 200, 403 |
| GET | /api/applications/{application_id}/resume/ | Student, Recruiter, TPO, Admin | Download resume | 200, 403, 404 |
| GET/PATCH | /api/recruiter/company/ | Recruiter | Manage company profile | 200, 400, 403 |
| GET/POST | /api/recruiter/postings/ | Recruiter | List or create postings | 200, 201, 400 |
| GET/PATCH | /api/recruiter/postings/{posting_id}/ | Recruiter | Read or update own posting | 200, 400, 403 |
| POST | /api/recruiter/postings/{posting_id}/submit/ | Recruiter | Submit posting for approval | 200, 409 |
| GET | /api/recruiter/postings/{posting_id}/applications/ | Recruiter | List own applicants | 200, 403 |
| PATCH | /api/recruiter/applications/{application_id}/status/ | Recruiter | Update one applicant status | 200, 400, 403, 409 |
| PATCH | /api/recruiter/applications/bulk-status/ | Recruiter | Should bulk status update | 200, 403, 409 |
| GET | /api/tpo/recruiters/ | TPO | Approval queue | 200 |
| POST | /api/tpo/recruiters/{recruiter_id}/approve/ | TPO | Approve recruiter | 200 |
| POST | /api/tpo/recruiters/{recruiter_id}/reject/ | TPO | Reject recruiter | 200, 400 |
| GET | /api/tpo/students/ | TPO | Student verification queue | 200 |
| POST | /api/tpo/students/{student_id}/verify/ | TPO | Verify profile | 200 |
| POST | /api/tpo/students/{student_id}/reject/ | TPO | Reject profile | 200, 400 |
| GET | /api/tpo/postings/ | TPO | Posting approval queue | 200 |
| POST | /api/tpo/postings/{posting_id}/approve/ | TPO | Approve posting | 200, 409 |
| POST | /api/tpo/postings/{posting_id}/reject/ | TPO | Reject posting | 200, 400 |
| GET | /api/tpo/reports/summary/ | TPO | Placement metrics as JSON or CSV | 200, 400, 503 |
| GET | /api/tpo/reports/branch-report/ | TPO | Branch placement rows as JSON or CSV | 200, 400, 503 |
| GET | /api/tpo/reports/company-report/ | TPO | Company offers as JSON or CSV | 200, 400, 503 |
| GET | /api/tpo/postings/{posting_id}/applications/ | TPO | Should applicant JSON/CSV export with branch/CGPA filters | 200, 400, 404 |
| GET | /api/master/branches/ | Authenticated | Active branches | 200 |
| GET | /api/master/skills/ | Authenticated | Active skills | 200 |
| GET | /api/notifications/ | Authenticated | Should notifications | 200 |
| PATCH | /api/notifications/{notification_id}/read/ | Authenticated | Mark notification read | 200 |

### Authentication endpoints

`GET /health/live/` and `GET /health/ready/` use the exact status/body contracts in document 06. They reveal no credentials, SQL errors, storage paths, or personal data. A protected operation encountering unavailable PostgreSQL returns `503 database-unavailable` in the standard error envelope. Resume storage failure returns `503 resume-storage-unavailable` and retains the old current resume.

`GET /api/schema/` returns the generated OpenAPI document. It describes all Must request/response schemas, role requirements, pagination, decimal strings, JSON/CSV representations, and error keys. Python contract tests check recorded requests/responses against it.

#### GET /api/csrf/

Sets the `csrftoken` cookie. It returns no body.

HTTP outcomes: `204`.

#### GET /api/session/

Returns the current authenticated user or an anonymous session.

```json
{
  "authenticated": true,
  "user": {
    "id": "user-aarav",
    "email": "aarav.rao@sitm.example.in",
    "role": "STUDENT"
  }
}
```

HTTP outcomes: `200`.

#### POST /api/auth/student-register/

Roles: Anonymous. CSRF required. These views need explicit CSRF protection because DRF session authentication checks CSRF only for authenticated users.

Request fields: `email` string, `password` string min 12 chars, `full_name` string.

```json
{
  "email": "aarav.rao@sitm.example.in",
  "password": "<password>",
  "full_name": "Aarav Rao"
}
```

`<password>` is a generated local test password. Do not commit reusable real passwords.

Success response `201`:

```json
{
  "id": "user-aarav",
  "email": "aarav.rao@sitm.example.in",
  "role": "STUDENT",
  "profile_status": "DRAFT"
}
```

Errors: 400 college-email-required; 400 password-too-short; 403 csrf-failed.

#### POST /api/auth/recruiter-register/

Roles: Anonymous. CSRF required. Registration creates a linked company stub from `company_name`; the recruiter completes the company before the first posting.

Request fields: `email`, `password`, `full_name`, `designation`, `company_name`.

Success response `201`:

```json
{
  "id": "user-rohan",
  "email": "rohan@navira.example.com",
  "role": "RECRUITER",
  "approval_status": "PENDING"
}
```

Errors: 400 company-name-required; 400 password-too-short.

#### POST /api/auth/login/

Request fields: `email`, `password`. Explicit CSRF protection is required even for an anonymous request. Retain the returned session and newly rotated CSRF cookie.

Success response `200` for an approved recruiter:

```json
{
  "authenticated": true,
  "role": "RECRUITER",
  "approval_status": "APPROVED"
}
```

Pending recruiter login also returns `200`:

```json
{
  "authenticated": true,
  "role": "RECRUITER",
  "approval_status": "PENDING"
}
```

Errors: 400 invalid-credentials; 403 recruiter-rejected with `error.details.approval_reason`; 403 csrf-failed.

#### POST /api/auth/logout/

Invalidates the server session. HTTP outcomes are `204` and `403 csrf-failed`.

### Student profile and resume endpoints

#### GET /api/me/profile/

Returns the current student profile.

```json
{
  "full_name": "Aarav Rao",
  "roll_number": "SITM-2027-0142",
  "phone": "+919876543210",
  "branch": "CSE",
  "graduation_year": 2027,
  "cgpa": "8.20",
  "active_backlogs": 0,
  "tenth_percentage": "91.20",
  "twelfth_percentage": "88.40",
  "skills": ["Python", "SQL"],
  "links": {"github": "https://github.com/aarav-example"},
  "verification_status": "VERIFIED",
  "profile_completeness_percent": 100,
  "has_current_resume": true
}
```

#### PATCH /api/me/profile/

Updates allowed profile fields. CSRF required. If a `VERIFIED` profile changes any BR-06 field, the server resets it to `DRAFT` and clears `verified_by_id` and `verified_at`.

Request example:

```json
{
  "roll_number": "SITM-2027-0142",
  "branch": "CSE",
  "graduation_year": 2027,
  "cgpa": "8.20",
  "active_backlogs": 0,
  "tenth_percentage": "91.20",
  "twelfth_percentage": "88.40",
  "skills": ["Python", "SQL", "Docker"],
  "phone": "+919876543210"
}
```

Success: `200` with the profile. Errors: 400 cgpa-out-of-range; 400 branch-invalid; 400 graduation-year-invalid.

#### POST /api/me/profile/submit/

Moves a complete profile to `SUBMITTED`.

Success response `200`:

```json
{
  "verification_status": "SUBMITTED",
  "message": "Profile submitted for TPO verification."
}
```

Errors: `409 profile-incomplete`.

#### POST /api/me/resume/

Uploads one PDF in multipart form field `file`. Maximum size is `2,097,152` bytes. The backend checks that the file starts with bytes `%PDF-`, checks the `.pdf` extension, and stores it privately. Avoid `python-magic` because native `libmagic` often fails on Windows. If a `VERIFIED` profile changes its current resume, the server resets it to `DRAFT`.

Success response `201`:

```json
{
  "id": "resume-aarav-2026-10",
  "original_filename": "aarav-resume.pdf",
  "content_type": "application/pdf",
  "size_bytes": 1468006,
  "is_current": true
}
```

Errors: 400 resume-not-pdf; 400 resume-too-large; 503 resume-storage-unavailable.

### Job search and application endpoints

#### GET /api/jobs/

Query parameters: `q`, `type`, `location`, `min_ctc_lpa`, `eligible_only`, `sort`, `page`, `page_size`.

Allowed sort values: `deadline_asc`, `deadline_desc`, `ctc_asc`, `ctc_desc`.

Example response `200`:

```json
{
  "count": 2,
  "page": 1,
  "page_size": 10,
  "results": [
    {
      "id": "post-nav-ft-2027",
      "title": "Associate Software Engineer",
      "company": "Navira Systems Pvt Ltd",
      "posting_type": "FULL_TIME",
      "locations": ["Bengaluru"],
      "ctc_lpa": "8.50",
      "deadline_at": "2026-11-20T17:00:00+05:30",
      "eligibility_label": "Eligible"
    }
  ]
}
```

Errors: 400 page-size-out-of-range; 400 sort-invalid.

#### GET /api/jobs/{posting_id}/

Returns detail for a published posting.

```json
{
  "id": "post-nav-ft-2027",
  "title": "Associate Software Engineer",
  "company": "Navira Systems Pvt Ltd",
  "posting_type": "FULL_TIME",
  "description": "Build web features for placement operations.",
  "locations": ["Bengaluru"],
  "ctc_lpa": "8.50",
  "stipend_monthly": null,
  "eligibility": {
    "min_cgpa": "7.00",
    "allowed_branches": ["CSE", "ISE", "ECE"],
    "graduation_years": [2027],
    "max_active_backlogs": 0
  },
  "selection_rounds": [
    {"round_order": 1, "name": "Aptitude"},
    {"round_order": 2, "name": "Technical Interview"}
  ],
  "deadline_at": "2026-11-20T17:00:00+05:30",
  "state": "PUBLISHED"
}
```

#### GET /api/jobs/{posting_id}/eligibility/

Returns all failing eligibility reasons in fixed order.

For eligible seeded Aarav and Navira, the exact response is `{"label":"Eligible","reason_ids":[],"messages":[]}`. The arrays align by index; a client must not discard later failure reasons. The local UUID/clock/fixtures are defined in document 06.

```json
{
  "label": "Not eligible",
  "reason_ids": [
    "CGPA_BELOW_MINIMUM",
    "BRANCH_NOT_ALLOWED",
    "ACTIVE_BACKLOGS_EXCEEDED"
  ],
  "messages": [
    "Your CGPA is below the minimum required for this posting.",
    "Your branch is not eligible for this posting.",
    "Your active backlogs exceed the allowed limit."
  ]
}
```

#### POST /api/jobs/{posting_id}/apply/

Creates one application when eligible.

Success response `201`:

```json
{
  "id": "app-aarav-nav-ft",
  "posting_id": "post-nav-ft-2027",
  "current_state": "APPLIED",
  "applied_at": "2026-10-21T10:15:00+05:30"
}
```

Errors: 409 application-already-exists; 409 deadline-passed; 403 profile-not-verified; 403 student-not-eligible with `details.reason_ids` listing all failed eligibility rules; 403 policy-limit-reached for every posting type after the second accepted full-time offer.

#### Student application actions

`POST /api/applications/{application_id}/withdraw/` returns `200` with state `WITHDRAWN` or `409 withdrawal-not-allowed`.

`POST /api/applications/{application_id}/accept/` returns `200` with state `ACCEPTED` or `409 transition-not-allowed`.

`POST /api/applications/{application_id}/decline/` returns `200` with state `DECLINED`.

#### GET /api/applications/{application_id}/timeline/

Returns events visible to the student owner, owning recruiter, or TPO.

```json
{
  "application_id": "app-aarav-nav-ft",
  "events": [
    {
      "event_type": "apply",
      "from_state": null,
      "to_state": "APPLIED",
      "actor": "Aarav Rao",
      "created_at": "2026-10-21T10:15:00+05:30",
      "reason": ""
    }
  ]
}
```

#### GET /api/applications/{application_id}/resume/

Streams `application/pdf` for the authorized student owner, owning recruiter, TPO, or admin. It MUST NOT return the private storage key. Errors: 403 resume-access-denied; 404 resume-not-found.

### Recruiter endpoints

#### GET/PATCH /api/recruiter/company/

Reads or updates the approved recruiter's company.

GET returns the company stub created at registration. PATCH completes `industry`, `headquarters`, and `description` before the first posting.

Patch request:

```json
{
  "name": "Navira Systems Pvt Ltd",
  "website": "https://navira.example.com",
  "industry": "Software products",
  "headquarters": "Bengaluru, Karnataka",
  "description": "Fictional software company for the CampusHire project."
}
```

Success: `200`. Errors: 403 recruiter-pending-approval; 400 company-name-required.

#### GET/POST /api/recruiter/postings/

Creates draft postings for the recruiter's company.

Create request:

```json
{
  "title": "Associate Software Engineer",
  "posting_type": "FULL_TIME",
  "description": "Build placement portal features.",
  "locations": ["Bengaluru"],
  "ctc_lpa": "8.50",
  "min_cgpa": "7.00",
  "allowed_branches": ["CSE", "ISE", "ECE"],
  "graduation_years": [2027],
  "max_active_backlogs": 0,
  "deadline_at": "2026-11-20T17:00:00+05:30",
  "selection_rounds": [
    {"round_order": 1, "name": "Aptitude"},
    {"round_order": 2, "name": "Technical Interview"}
  ]
}
```

Success `201` has state `DRAFT`. Errors: 400 stipend-required; 400 deadline-must-be-future; 400 selection-round-required.

#### PATCH /api/recruiter/postings/{posting_id}/

Allows updates only while state is `DRAFT`. Errors: 403 posting-not-owned; 409 posting-not-editable.

#### POST /api/recruiter/postings/{posting_id}/submit/

Moves a complete draft to `PENDING_APPROVAL`.

Success:

```json
{
  "id": "post-nav-ft-2027",
  "state": "PENDING_APPROVAL"
}
```

#### GET /api/recruiter/postings/{posting_id}/applications/

The Should `?format=csv` representation preserves the JSON list's branch and `min_cgpa` filters and ownership checks. CSV headers are `application_id,student_name,branch,cgpa,current_state,applied_at`; rows sort by `applied_at,application_id`. It exports the complete filtered set, not only the current page, and never includes storage keys or resume bytes. The Should TPO applicant endpoint uses the same columns/filters, permits TPO/ADMIN only, and can read any posting.

Filters: `branch`, `min_cgpa`, `state`, `page`, `page_size`.

```json
{
  "count": 1,
  "page": 1,
  "page_size": 10,
  "results": [
    {
      "application_id": "app-aarav-nav-ft",
      "student_name": "Aarav Rao",
      "branch": "CSE",
      "cgpa": "8.20",
      "current_state": "APPLIED",
      "applied_at": "2026-10-21T10:15:00+05:30",
      "resume_download_url": "/api/applications/app-aarav-nav-ft/resume/"
    }
  ]
}
```

Errors: `403 posting-not-owned`.

#### PATCH /api/recruiter/applications/{application_id}/status/

Updates one application per request in Must scope.

Request:

```json
{
  "to_state": "OFFERED",
  "offer_ctc_lpa": "8.50",
  "reason": "Meets CGPA and branch criteria."
}
```

Success:

```json
{
  "application_id": "app-aarav-nav-ft",
  "from_state": "IN_INTERVIEW",
  "to_state": "OFFERED",
  "offer_ctc_lpa": "8.50",
  "offer_stipend_monthly": null,
  "offer_made_at": "2026-10-25T15:30:00+05:30",
  "event_created": true
}
```

For `OFFERED`, full-time postings require `offer_ctc_lpa` from `0.50` to `100.00`. Internship postings require `offer_stipend_monthly` from `0` to `300000`. `INTERNSHIP_WITH_PPO` can also include an optional PPO CTC. The student timeline and application detail show these offer values. Errors: 403 application-not-owned; 409 transition-not-allowed; 400 offer-amount-required; 400 reason-too-long.

#### PATCH /api/recruiter/applications/bulk-status/

Should endpoint. Request fields: `application_ids` array, `to_state`, `reason`. It MUST reject mixed-company selections with `403 mixed-company-selection`.

### TPO endpoints

#### Recruiter approvals

`GET /api/tpo/recruiters/?status=PENDING` returns pending recruiters.

`POST /api/tpo/recruiters/{recruiter_id}/approve/` returns:

```json
{
  "id": "rec-rohan",
  "approval_status": "APPROVED",
  "approved_by": "Meera Nair"
}
```

`POST /api/tpo/recruiters/{recruiter_id}/reject/` request:

```json
{
  "reason": "Company email could not be verified"
}
```

Errors: `400 reason-required`.

#### Student verification

`GET /api/tpo/students/?status=SUBMITTED&branch=CSE` returns submitted profiles.

`POST /api/tpo/students/{student_id}/verify/` sets `VERIFIED`.

`POST /api/tpo/students/{student_id}/reject/` requires a correction reason.

#### Posting approvals

`GET /api/tpo/postings/?state=PENDING_APPROVAL` returns pending postings.

`POST /api/tpo/postings/{posting_id}/approve/` returns state `PUBLISHED` when deadline is in the future.

`POST /api/tpo/postings/{posting_id}/reject/` request:

```json
{
  "reason": "CTC value missing"
}
```

Errors: 400 reason-required; 409 deadline-passed.

#### Placement reporting endpoints

FR-TPO-01 / BR-24 / BR-25 definitions:

- Placed student means a distinct student with an `ACCEPTED` application for a `FULL_TIME` posting.
- Placed percentage is placed students divided by registered students, multiplied by 100, and rounded to 2 decimals.
- Placed percentage is `0.00` when no students are registered.
- `accepted_offer_count` is the number of accepted full-time applications.
- Highest, average, and median CTC use accepted applications' `offer_ctc_lpa`.
- Only `current_state=ACCEPTED` joined to `posting_type=FULL_TIME` contributes offers or CTC. Internship and internship-with-PPO amounts are excluded.
- Median for an even count is the mean of the two middle CTC values.
- Branch rows include registered students, placed students, and placed percentage.
- Company ranking uses dense rank on accepted full-time offers inside each branch.
- Registered/verified/placed counts use distinct student IDs, never counts inflated by joins or multiple accepted offers.
- Percentages and non-empty CTC values are decimal strings rounded to two places using decimal arithmetic. Empty CTC aggregates are JSON `null`, never salary zero.

`GET /api/tpo/reports/summary/` returns:

```json
{
  "registered_students": 100,
  "verified_students": 72,
  "placed_students": 18,
  "placed_percentage": "18.00",
  "accepted_offer_count": 21,
  "highest_ctc_lpa": "10.00",
  "average_ctc_lpa": "8.00",
  "median_ctc_lpa": "8.00"
}
```

When no accepted full-time offers exist, CTC fields are `null`, `accepted_offer_count=0`, and `placed_students=0`. With zero registered students, `placed_percentage="0.00"`. These are data semantics, not display labels.

`GET /api/tpo/reports/branch-report/` returns branch/company rows with a company rank from handwritten SQL. Optional `branch=CSE` filters before aggregation. Branch registered/placed metrics are distinct counts within that branch; they are repeated on each company row and must not be summed across company rows. Companies with tied accepted offer counts share a dense rank.

```json
{
  "rows": [
    {
      "branch": "CSE",
      "registered_students": 40,
      "placed_students": 12,
      "placed_percentage": "30.00",
      "company": "Navira Systems Pvt Ltd",
      "company_offer_count": 7,
      "company_rank_in_branch": 1
    }
  ]
}
```

`GET /api/tpo/reports/company-report/` returns company-wise accepted full-time offers and distinct placed students:

```json
{
  "rows": [
    {
      "company": "Navira Systems Pvt Ltd",
      "accepted_offer_count": 7,
      "placed_students": 6,
      "highest_ctc_lpa": "10.00",
      "average_ctc_lpa": "8.00",
      "median_ctc_lpa": "8.00"
    }
  ]
}
```

All three endpoints MUST return JSON by default (`application/json`) and support `?format=csv` (`text/csv; charset=utf-8`). They use identical filters, permissions, aggregate definitions, and decimal precision in both formats. CSV has UTF-8 headers, standard quoting, LF record endings, and empty cells for JSON `null`. Responses include an attachment filename: `placement-summary.csv`, `branch-placement.csv`, or `company-offers.csv`.

| Endpoint suffix | Exact CSV header | Stable row order |
|---|---|---|
| `summary/` | `registered_students,verified_students,placed_students,placed_percentage,accepted_offer_count,highest_ctc_lpa,average_ctc_lpa,median_ctc_lpa` | Exactly one aggregate row, including an empty dataset |
| `branch-report/` | `branch,registered_students,placed_students,placed_percentage,company,company_offer_count,company_rank_in_branch` | Branch code ascending, rank ascending, company name ascending |
| `company-report/` | `company,accepted_offer_count,placed_students,highest_ctc_lpa,average_ctc_lpa,median_ctc_lpa` | Accepted offer count descending, company name ascending |

A registered branch with zero accepted full-time offers has one branch row with `company=null`, `company_offer_count=0`, and `company_rank_in_branch=null`; no registered students yields no branch rows. A company with zero accepted offers is omitted; no accepted offers yields `rows=[]` and a header-only company CSV. Unsupported formats return `400 invalid-report-format`, invalid branch filters return `400 branch-invalid`, unauthorized roles return `403 role-not-allowed`, and report timeouts return `503 report-timeout` without partial CSV data.

For the four-student fixture with accepted CTC `6.00,8.00,10.00,12.00`, the summary CSV is exactly:

```text
registered_students,verified_students,placed_students,placed_percentage,accepted_offer_count,highest_ctc_lpa,average_ctc_lpa,median_ctc_lpa
4,4,4,100.00,4,12.00,9.00,9.00
```

Document 09 tests even/odd medians, multiple accepted offers for one student, internship exclusion, zero denominators, ranking ties, and JSON/CSV equivalence.

### Master and notification endpoints

`GET /api/master/branches/` returns active branch identifiers.

```json
{"results": [{"branch": "CSE", "name": "Computer Science and Engineering"}]}
```

`GET /api/master/skills/` returns active skills.

`GET /api/notifications/` is a Should endpoint. It returns unread and recent read notifications with a `dedup_key` hidden from normal users.

## API-client integration flows

Python tests or an API client MUST use the contracts above without a custom interface. Django admin at `/admin/` is a built-in local tool for master data/staff accounts only; it is not a mandatory business interface.

| Flow | Exact requests | Expected evidence |
|---|---|---|
| Session lifecycle | GET CSRF; POST login with token; GET session; POST logout with rotated token; GET own applications with old cookie | Login `200`; logout `204`; final request `403 authentication-required`, no redirect |
| Student applies | PATCH profile; POST PDF; submit; TPO verifies in its own session; student GET eligibility; POST apply | `VERIFIED`; ordered eligibility arrays; apply `201 APPLIED`; one event |
| Recruiter reviews | GET own applicants with `branch=CSE&min_cgpa=8.00`; GET authorized PDF; PATCH one status | Filtered own data only; `application/pdf`; `SHORTLISTED` event with recruiter actor |
| TPO reports | GET each `/api/tpo/reports/` endpoint as JSON, then `?format=csv` | Equal aggregates, exact CSV headers, SQL rank/query-plan evidence |

For a missing session, the client must authenticate again before retrying unsafe requests; it must not retry a write blindly. For `recruiter-pending-approval`, the client may read session/approval state but cannot use recruiter feature endpoints. Validation and ownership errors retain their documented keys and leave persisted data unchanged.

The local operation and exact fixture demo in document 06, four Must Python/API flows in document 09, and local acceptance tests are the integration deliverables. API responses and test reports replace screen captures.

[Back to README](../README.md)
