# API specification

Purpose: This document defines the CampusHire REST API and the React SPA screens and routes.

## Part 1: REST API

The API base path is `/api/`. Responses use JSON except resume downloads. Authenticated endpoints use Django session authentication. Unsafe methods MUST include `X-CSRFToken`. In endpoint tables, role `TPO` also allows `ADMIN` unless the row explicitly says otherwise.

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

Protected endpoints return `403 authentication-required` when a session is missing or expired. The SPA MUST redirect to `/login` only for that key. It MUST keep the normal forbidden-page behavior for other `403` keys.

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
| GET | /api/tpo/dashboard/summary/ | TPO | Dashboard metrics | 200, 503 |
| GET | /api/tpo/dashboard/branch-report/ | TPO | Branch placement report | 200, 503 |
| GET | /api/tpo/dashboard/company-report/ | TPO | Company offers report | 200 |
| GET | /api/master/branches/ | Authenticated | Active branches | 200 |
| GET | /api/master/skills/ | Authenticated | Active skills | 200 |
| GET | /api/notifications/ | Authenticated | Should notifications | 200 |
| PATCH | /api/notifications/{notification_id}/read/ | Authenticated | Mark notification read | 200 |

### Authentication endpoints

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

Request fields: `email`, `password`.

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

Errors: 400 invalid-credentials; 403 recruiter-rejected.

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
  "skills": ["Python", "React"],
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
  "skills": ["Python", "React", "SQL"],
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

#### Dashboard

Dashboard definitions:

- Placed student means a distinct student with an `ACCEPTED` application for a `FULL_TIME` posting.
- Placed percentage is placed students divided by registered students, multiplied by 100, and rounded to 2 decimals.
- Placed percentage is `0.00` when no students are registered.
- `accepted_offer_count` is the number of accepted full-time applications.
- Highest, average, and median CTC use accepted applications' `offer_ctc_lpa`.
- Median for an even count is the mean of the two middle CTC values.
- Branch rows include registered students, placed students, and placed percentage.
- Company ranking uses dense rank on accepted full-time offers inside each branch.

`GET /api/tpo/dashboard/summary/` returns:

```json
{
  "registered_students": 100,
  "verified_students": 72,
  "placed_percentage": "18.00",
  "accepted_offer_count": 21,
  "highest_ctc_lpa": "10.00",
  "average_ctc_lpa": "8.00",
  "median_ctc_lpa": "8.00"
}
```

When no accepted full-time offers exist, the CTC fields return `"No offers yet"`.

`GET /api/tpo/dashboard/branch-report/` returns branch rows with a company rank from handwritten SQL.

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

`GET /api/tpo/dashboard/company-report/` returns company-wise offers. Errors: `503 report-timeout`.

### Master and notification endpoints

`GET /api/master/branches/` returns active branch identifiers.

```json
{"results": [{"branch": "CSE", "name": "Computer Science and Engineering"}]}
```

`GET /api/master/skills/` returns active skills.

`GET /api/notifications/` is a Should endpoint. It returns unread and recent read notifications with a `dedup_key` hidden from normal users.

## Part 2: UI screens and routes

The SPA uses React Router. It MUST show loading, empty, and error states. It MUST not render raw HTML from recruiter posting descriptions. The Vite development server proxies `/api` and `/admin` to Django at `http://localhost:8000`, so the browser uses one origin, `http://localhost:5173`. Session cookies and CSRF work without CORS. In the production-like profile, Nginx serves the SPA and proxies `/api`.

### Route summary

| Route | Role | Purpose | Main fields | Actions | States and guards |
|---|---|---|---|---|---|
| `/login` | Anonymous | Login | E-mail, password | Login | Shows `invalid-credentials`; redirects by role after login |
| `/register/student` | Anonymous | Student registration | Name, college e-mail, password | Register | Rejects non-`sitm.example.in` e-mail |
| `/register/recruiter` | Anonymous | Recruiter registration | Name, e-mail, designation, company, password | Register | Shows pending approval after success |
| `/student/dashboard` | Student | Student landing page | Profile status, applications, notifications | Continue profile, view jobs | Guard blocks non-students |
| `/student/profile` | Student | Edit profile | Required fields from BR-06, skills, links | Save, submit | Field validation; loading saved profile; error banner on 400 |
| `/student/resume` | Student | Upload resume | PDF file | Upload | Shows `resume-not-pdf`, `resume-too-large`; success shows file name |
| `/student/jobs` | Student | Search jobs | Keyword, type, location, min CTC, eligible-only, sort | Search, paginate | Empty state says `No matching jobs found` |
| `/student/jobs/:postingId` | Student | Job detail and eligibility | Posting fields, rounds, eligibility reasons | Apply | Apply disabled for every eligibility reason |
| `/student/applications` | Student | Own applications | Posting, company, state, dates | View timeline, withdraw | Empty state for no applications |
| `/student/applications/:applicationId` | Student | Timeline | Events, actor, timestamp, reason | Accept, decline, withdraw when allowed | Guard checks owner |
| `/recruiter/pending` | Recruiter | Approval waiting page | Approval status and reason | Logout | Approved recruiters redirect to company page |
| `/recruiter/company` | Recruiter | Company profile | Name, website, industry, headquarters, description | Save | Guard requires `APPROVED` |
| `/recruiter/postings` | Recruiter | Posting list | Title, type, state, deadline | Create, edit, submit | Empty state guides first posting |
| `/recruiter/postings/new` | Recruiter | Create posting | Posting fields and rounds | Save draft | Validates compensation by type |
| `/recruiter/postings/:postingId` | Recruiter | Edit own posting | Draft fields, rejection reason | Save, submit | Non-owner shows access denied |
| `/recruiter/postings/:postingId/applicants` | Recruiter | Applicant list | Branch, CGPA, state filters | Download resume, update state | Shows 403 page for other company |
| `/tpo/dashboard` | TPO | Metrics and reports | Counts, CTC, branch and company tables | Refresh, export Should | Report timeout shows panel error |
| `/tpo/recruiters` | TPO | Recruiter approvals | Recruiter, company, status | Approve, reject | Rejection reason required |
| `/tpo/students` | TPO | Profile verification | Student, branch, status | Verify, reject | Missing profile values highlighted |
| `/tpo/postings` | TPO | Posting approvals | Posting, company, deadline | Approve, reject | Reject reason 10-500 chars |
| `/admin-help` | Admin | Django admin guidance | Admin URL and master data list | Open admin | Explains admin is server-rendered |

### Form validation messages

| Field | Route | Invalid input | Message |
|---|---|---|---|
| Student e-mail | `/register/student` | `aarav.rao@gmail.com` | Use your `sitm.example.in` college e-mail address. |
| Password | Registration and login | 11 characters | Password must be at least 12 characters. |
| CGPA | `/student/profile` | `10.50` | CGPA must be from 0.00 to 10.00. |
| Graduation year | `/student/profile` | `2035` | Graduation year must be from 2026 to 2030. |
| Resume | `/student/resume` | Plain text named `.pdf` | Upload a real PDF file. |
| Resume size | `/student/resume` | 2,100,000 bytes | Resume must be 2 MB or smaller. |
| Stipend | Posting form | Internship with blank stipend | Stipend is required for internship postings. |
| Rejection reason | TPO reject dialogs | `Bad` | Reason must be 10 to 500 characters. |
| Page size | Jobs URL | `page_size=100` | Page size must be between 1 and 50. |

### UI state requirements

| Screen group | Loading state | Empty state | Error state |
|---|---|---|---|
| Student jobs | Skeleton cards for search results | `No matching jobs found` | Shows API error and keeps filters |
| Profile | Disabled Save button and spinner | Not applicable | Field errors remain beside inputs |
| Resume | Progress text `Uploading resume` | `No current resume uploaded` | Shows exact upload error message |
| Recruiter applicants | Table skeleton | `No applicants match these filters` | 403 page for other-company posting |
| TPO dashboard | Panel-level spinners | `No offers yet` for CTC metrics | Failed panel shows retry button |
| Approval queues | Row skeletons | `No pending approvals` | Error banner with request ID |

### Route guard rules

| Guard | Exact behavior |
|---|---|
| Anonymous-only | Logged-in users redirect to their dashboard. |
| Authenticated | Anonymous users redirect to `/login` and return after login. |
| Student | Non-students see `You do not have access to this page`. |
| Recruiter approved | Pending recruiters go to `/recruiter/pending`; rejected recruiters see the TPO reason. |
| TPO | `TPO` and `ADMIN` sessions can open TPO routes. Other roles see `You do not have access to this page`. |
| Application owner | Students cannot open another student's application detail. |

[Back to README](../README.md)
