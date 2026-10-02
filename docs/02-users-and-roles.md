# Users and roles

Purpose: This document defines CampusHire users, permissions, journeys, and user stories.

## Roles

| Role | Description | Account creation |
|---|---|---|
| Student | A final-year or pre-final-year student who maintains a profile and applies to postings. | Self-registers with `@sitm.example.in`. |
| Recruiter | A company HR user who manages company data and reviews applicants. | Registers, then waits for TPO approval. |
| Placement Officer | TPO staff member who verifies profiles, approves postings, and reads dashboards. | Created by System Admin. |
| System Admin | Django admin user who manages settings, branches, skills, and TPO accounts. | Created through Django superuser flow. |

## Personas

| Persona | Role | Goals | Pain points | Technical comfort |
|---|---|---|---|---|
| Aarav Rao | Student, CSE 2027 | Apply before deadlines and understand eligibility. | Does not know why Google Forms reject him. | Comfortable with mobile web forms. |
| Meera Nair | Placement Officer | Verify student profiles and publish approved drives. | Tracks CGPA corrections across many sheets. | Comfortable with admin panels and exports. |
| Rohan Bedi | Recruiter at Navira Systems | Collect eligible applicants for a Bengaluru role. | Receives mixed resumes from many branches. | Comfortable with applicant filters. |
| Kavya Menon | System Admin | Keep college settings accurate. | Wants minimal data fixes during drives. | Comfortable with Django admin. |

## Permission matrix

| Action | Student | Recruiter | Placement Officer | System Admin |
|---|---|---|---|---|
| Register with college e-mail | Yes | No | No | No |
| Register as recruiter | No | Yes | No | No |
| Approve recruiter | No | No | Yes | Yes |
| Login and logout | Yes | Yes, including pending approval | Yes | Yes |
| Edit own student profile | Yes | No | No | No |
| Verify student profile | No | No | Yes | Yes |
| Upload own resume | Yes | No | No | No |
| Download applicant resume | Own applications only | Own postings only | All postings | Yes |
| Create company profile | No | Yes | No | No |
| Create posting | No | Yes for own company | No | No |
| Submit posting for approval | No | Yes | No | No |
| Approve or reject posting | No | No | Yes | Yes |
| Search published jobs | Yes | No | Yes | Yes |
| Apply to posting | Yes | No | No | No |
| Withdraw application before shortlist | Yes | No | No | No |
| Update application status | No | One application at a time | No | No |
| View TPO dashboard | No | No | Yes | Yes |
| Manage branches and skills | No | No | No | Yes |

## Key user journeys

### Student applies to an eligible posting

```mermaid
flowchart TD
    A["Aarav logs in"] --> B["Completes profile and uploads PDF resume"]
    B --> C["TPO verifies profile"]
    C --> D["Aarav opens PUBLISHED posting"]
    D --> E["Eligibility engine checks all rules"]
    E --> F["Portal shows Eligible"]
    F --> G["Aarav clicks Apply"]
    G --> H["Application is APPLIED with timestamp"]
```

### Recruiter publishes a campus drive

```mermaid
flowchart TD
    A["Rohan registers recruiter account"] --> B["TPO approves recruiter"]
    B --> C["Rohan completes Navira Systems profile"]
    C --> D["Rohan creates DRAFT posting"]
    D --> E["Rohan submits for approval"]
    E --> F["Meera reviews eligibility and deadline"]
    F --> G["Meera approves posting"]
    G --> H["Posting becomes PUBLISHED"]
```

### TPO reviews placement performance

```mermaid
flowchart TD
    A["Meera opens dashboard"] --> B["System counts registered and verified students"]
    B --> C["System calculates accepted offer and placed metrics"]
    C --> D["SQL report ranks companies by branch offers"]
    D --> E["Meera checks highest, average, and median CTC"]
```

## User stories

| ID | Story | Priority | Linked FR IDs |
|---|---|---|---|
| US-01 | As a student, I want to register with my college e-mail and log out safely, so that only college students use the portal. | Must | FR-AUTH-01 |
| US-02 | As a recruiter, I want to log in and see my pending approval status, so that I know when the TPO unlocks company access. | Must | FR-AUTH-02 |
| US-03 | As a student, I want to complete my academic profile and upload a valid PDF resume, so that recruiters see verified data. | Must | FR-PROFILE-01, FR-PROFILE-02 |
| US-04 | As a recruiter, I want to maintain my company profile and submit postings, so that students can apply to current drives. | Must | FR-POST-01, FR-POST-02 |
| US-05 | As a placement officer, I want to approve postings with rejection reasons, so that invalid drives are not published. | Must | FR-POST-02 |
| US-06 | As a student, I want to see every eligibility reason, so that I know what blocks my application. | Must | FR-ELIG-01 |
| US-07 | As a student, I want to apply once and withdraw before shortlisting, so that my application record stays correct. | Must | FR-APP-01 |
| US-08 | As a recruiter, I want to move one applicant through the pipeline, so that each status change is deliberate. | Must | FR-APP-02 |
| US-09 | As a recruiter, I want to filter applicants and download authorized resumes, so that I review only my own posting data. | Must | FR-REC-01 |
| US-10 | As a student, I want to search jobs by keyword, type, location, CTC, and eligibility, so that I find relevant postings. | Must | FR-JOB-01 |
| US-11 | As a placement officer, I want placement metrics and branch-wise reports, so that I can brief college leadership. | Must | FR-TPO-01 |
| US-12 | As a user, I want a responsive role-based React SPA, so that each role sees only useful routes. | Must | FR-UI-01 |
| US-13 | As a system admin, I want to manage branches, skills, and college settings, so that master data is accurate. | Must | FR-ADMIN-01 |
| US-14 | As a placement officer, I want dream-offer rules, so that high offers do not block valid higher opportunities. | Should | FR-POLICY-01 |
| US-15 | As a recruiter, I want bulk status changes after shortlisting, so that I can process a campus drive faster. | Should | FR-REC-02 |
| US-16 | As a student, I want notifications and reminders, so that I do not miss deadlines and offer actions. | Should | FR-NOTIF-01, FR-OFFER-01 |
| US-17 | As a placement officer, I want audit logs, charts, and CSV exports, so that decisions are traceable. | Should | FR-AUDIT-01, FR-REPORT-01 |
| US-18 | As a student, I want optional schedule invites, dark mode, and resume keywords, so that the portal is more convenient. | Could | FR-SCHED-01, FR-THEME-01, FR-RESUME-02 |

## Route ownership summary

| Route area | Main role | Guard requirement |
|---|---|---|
| `/student/*` | Student | Authenticated student session |
| `/recruiter/*` | Recruiter | Approved recruiter session |
| `/tpo/*` | Placement Officer | TPO or System Admin session |
| `/admin/` | System Admin | Django admin permissions |
| `/login` and `/logout` | All users | Session and CSRF rules |

[Back to README](../README.md)
