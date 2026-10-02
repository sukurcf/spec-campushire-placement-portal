# System architecture

Purpose: This document describes the CampusHire architecture, key flows, data movement, local deployment, repository structure, design principles, and required ADRs.

## Architecture goals

CampusHire MUST expose safe, role-authorized placement workflows through REST. Django services enforce every business rule. Python tests or an API client are the only business demonstration clients; Django admin is a built-in local master-data tool.

## Context diagram

```mermaid
flowchart LR
    Student["Student"] --> Client["Python tests or API client"]
    Recruiter["Recruiter"] --> Client
    TPO["Placement Officer"] --> Client
    Admin["System Admin"] --> AdminSite["Built-in local Django admin"]
    Client --> API["Django REST API"]
    AdminSite --> DB
    API --> DB["PostgreSQL database"]
    API --> Media["Private resume media"]
    API --> Mailpit["Mailpit e-mail sink"]
    API --> Logs["JSON logs"]
```

## Container diagram

```mermaid
flowchart TB
    Client["Python tests or API client with cookie jar"] --> DRF["Django REST Framework API"]
    Operator["Local System Admin"] --> DjangoAdmin["Built-in Django admin"]
    DjangoAdmin --> ORM["Django ORM for master data"]
    DRF --> Services["Domain services"]
    Services --> Eligibility["Eligibility service"]
    Services --> Pipeline["Application pipeline service"]
    Services --> Reports["JSON and CSV report service"]
    Services --> Storage["Resume storage service"]
    Eligibility --> ORM["Django ORM"]
    Pipeline --> ORM
    Reports --> ORM
    Reports --> SQL["Handwritten SQL reports"]
    Storage --> PrivateMedia["Private media directory"]
    ORM --> Postgres["PostgreSQL 18.x"]
    SQL --> Postgres
    Services --> Notifications["Notification service"]
    Notifications --> Mailpit["Mailpit 1.31.x"]
```

## Key sequence diagrams

### Student registration and CSRF session

```mermaid
sequenceDiagram
    actor Student
    participant Client as Python tests or API client
    participant API as Django API
    participant DB as PostgreSQL
    Student->>Client: Register aarav.rao@sitm.example.in
    Client->>API: GET /api/csrf/
    API-->>Client: 204 and csrftoken cookie
    Client->>API: POST /api/auth/student-register/ with X-CSRFToken
    API->>DB: Create User and StudentProfile draft
    DB-->>API: Created
    API-->>Client: 201 with role STUDENT
```

### Profile completion, resume upload, and verification

```mermaid
sequenceDiagram
    actor Student
    actor TPO
    participant Client as API client
    participant API as Django API
    participant Media as Private media
    participant DB as PostgreSQL
    Student->>Client: Supply profile fields and recorded PDF
    Client->>API: PATCH /api/me/profile/ with CSRF
    Client->>API: POST /api/me/resume/ with CSRF
    API->>Media: Store PDF after content check
    API->>DB: Mark resume current
    Client->>API: POST /api/me/profile/submit/ with CSRF
    API->>DB: Validate required fields and set SUBMITTED
    TPO->>Client: Use TPO session for verification
    Client->>API: POST /api/tpo/students/{id}/verify/ with CSRF
    API->>DB: Set profile VERIFIED with verifier
    API-->>Client: 200 with verification_status VERIFIED
```

### Posting approval and deadline closure

```mermaid
sequenceDiagram
    actor Recruiter
    actor TPO
    participant Client as API client
    participant API as Django API
    participant DB as PostgreSQL
    participant Command as close_postings command
    Recruiter->>Client: Submit draft posting
    Client->>API: POST /api/recruiter/postings/{id}/submit/ with CSRF
    API->>DB: Set PENDING_APPROVAL
    TPO->>Client: Approve NAV-FT-2027 using TPO session
    Client->>API: POST /api/tpo/postings/{id}/approve/ with CSRF
    API->>DB: Set PUBLISHED
    Command->>API: Run within 5 minutes of deadline
    API->>DB: Set CLOSED when now >= deadline_at
```

### Eligibility check and application

```mermaid
sequenceDiagram
    actor Student
    participant Client as Python tests or API client
    participant API as Django API
    participant Eligibility as Eligibility service
    participant DB as PostgreSQL
    Student->>Client: Check Associate Software Engineer eligibility
    Client->>API: GET /api/jobs/{id}/eligibility/
    API->>Eligibility: Evaluate all reason codes
    Eligibility->>DB: Read profile, posting, applications, offers
    Eligibility-->>API: Eligible with empty reasons
    API-->>Client: 200 Eligible with empty reasons
    Student->>Client: Request application
    Client->>API: POST /api/jobs/{id}/apply/ with CSRF
    API->>DB: Create Application and ApplicationEvent in one transaction
    API-->>Client: 201 with state APPLIED
```

### Recruiter status update and authorized resume download

```mermaid
sequenceDiagram
    actor Recruiter
    participant Client as API client
    participant API as Django API
    participant DB as PostgreSQL
    participant Media as Private media
    Recruiter->>Client: Filter CSE applicants for NAV-FT-2027
    Client->>API: GET /api/recruiter/postings/{id}/applications/?branch=CSE
    API->>DB: Check company ownership and list rows
    API-->>Client: 200 with CSE applicants
    Recruiter->>Client: Request Aarav resume
    Client->>API: GET /api/applications/{id}/resume/
    API->>DB: Verify recruiter owns posting
    API->>Media: Stream current PDF
    API-->>Client: 200 application/pdf
```

## Data-flow description

| Flow | Data items | Stores touched | Controls |
|---|---|---|---|
| Account and session | E-mail, password hash, role, session ID, CSRF token | User, session table | College domain, secure cookies, CSRF header |
| Student profile | Roll number, branch, year, CGPA, backlogs, percentages, skills, links | StudentProfile, StudentSkill | Field validation and TPO verification |
| Resume upload | File name, size, PDF signature, hash, private key | ResumeDocument, private media | Check the file starts with `%PDF-`, enforce 2,097,152 byte limit, avoid native `libmagic` |
| Posting lifecycle | Company, compensation, eligibility, rounds, deadline, state | Company, Posting, PostingBranch, SelectionRound | Ownership, TPO approval, deadline closure |
| Eligibility | Profile, posting rules, existing application, accepted offers | StudentProfile, Posting, Application | All reason codes returned in fixed order |
| Application pipeline | Current state, actor, reason, timestamp | Application, ApplicationEvent | Transactional state and timeline update |
| REST reports | Student counts, offers, CTC metrics, branch ranks | StudentProfile, Application, Posting, Company | Accepted full-time offers only; JSON/CSV agreement and query-plan evidence |

Private resume bytes MUST NOT appear in logs, list responses, or CSV exports.

## Local deployment view

| Profile | Host resources | Running services | Notes |
|---|---|---|---|
| Standard | 16 GB RAM, four cores, 20 GB free disk, Docker memory 8 GB, swap 4 GB | Django, PostgreSQL, optional Mailpit | API integration flows and report performance runs. |
| Lite | 8 GB RAM, four cores, 20 GB free disk, Docker memory 4 GB, swap 4 GB | PostgreSQL 768 MB, Django 768 MB | Optional Mailpit is off unless e-mail is selected. |

All exposed ports bind to `127.0.0.1`: Django `8000`, PostgreSQL `15432`, optional Mailpit SMTP `11025` and built-in console `18025`. Internal PostgreSQL remains `db:5432`. Named volumes `campushire_pgdata` and `campushire_private_media` survive stop/start. Document 06 defines initialization and health checks; document 09 traces local acceptance. Resource figures are proposed budgets, not execution results.

Windows students MUST use WSL2 Ubuntu. The 8 GB `.wslconfig` values are `memory=4GB` and `swap=4GB`. The 16 GB values are `memory=8GB` and `swap=4GB`. The trainer MUST pre-check the lite profile before week 1.

## Expected student repository folder tree

```text
campushire/
├── backend/
│   ├── campus_hire/
│   ├── accounts/
│   ├── profiles/
│   ├── companies/
│   ├── postings/
│   ├── applications/
│   ├── reports/
│   ├── notifications/
│   └── tests/
├── docs/
│   ├── adr/
│   ├── query-plans/
│   └── test-reports/
├── fixtures/
│   ├── local-demo/
│   └── resumes/
├── scripts/
│   ├── local-start.sh
│   ├── local-stop.sh
│   └── local-reset.sh
└── README.md
```

The folder names are guidance for the implementation repository. The specification repository MUST contain Markdown documents only.

## Design principles

| Principle | CampusHire use |
|---|---|
| Server-side authorization | Every API checks role and ownership before it reads or writes data. |
| Layered backend | Views validate HTTP data. Services enforce placement rules. Models persist data. |
| One transaction for state changes | Application state and timeline event change together. |
| Private-by-default files | Resumes are streamed after authorization. Direct public media URLs are not used. |
| Contract-first API | Stable JSON errors, OpenAPI schemas, and CSV headers describe every business operation. |
| 12-factor configuration | Domain, cookie settings, database URL, media path, and mail settings come from environment variables. |
| Local-first development | All Must features run on a laptop without paid cloud services. |
| Measured performance | REST reports and job search require seed data and saved query-plan evidence. |

## ADRs the student MUST write

| ADR ID | Topic | Exact question to answer |
|---|---|---|
| ADR-001 | Session authentication and CSRF | How does a Python/API client retain cookies and send CSRF tokens, including on anonymous login and registration? |
| ADR-002 | Resume storage | How are PDF resumes stored privately, scanned by content type, and served only after authorization? |
| ADR-003 | Eligibility service boundary | Which module owns eligibility reason-code evaluation, and how does it stay fully branch-tested? |
| ADR-004 | Posting deadline closure | How does the system combine request-time checks with a scheduled command to close postings within 5 minutes? |
| ADR-005 | REST SQL reports | Which report queries use handwritten SQL and window functions, and how are indexes justified by `EXPLAIN ANALYZE`? |
| ADR-006 | API contracts and permissions | How do role/ownership checks, decimal serialization, empty results, and JSON/CSV representations stay consistent? |
| ADR-007 | Notification delivery | If notifications are built, how does at-least-once delivery use de-duplication keys? |
| ADR-008 | Local lifecycle | How do seed idempotency, loopback ports, private-media volumes, offline fixtures, and explicit reset confirmation work? |

[Back to README](../README.md)
