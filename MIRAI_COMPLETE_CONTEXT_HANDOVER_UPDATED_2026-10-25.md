# MIRAI BY DAILOQA — COMPLETE PROJECT & CHAT HANDOVER

**Snapshot assembled:** 25 October 2026 (India/IST).  
**Repository:** [krisshhjain/Mirai-Dashboard](https://github.com/krisshhjain/Mirai-Dashboard) (private; GitHub connector access may be needed).  
**Project lead / architect / sole ordinary PR approver-merger:** Krish Jain.  
**Purpose:** Give a *new ChatGPT conversation* sufficient context to continue the user's MIRAI project-management, architecture, requirements, GitHub PR-review, and developer-instruction workflows without starting over. This is a handover, not a mandate to implement or merge code.

> **NEW CHAT — READ THIS FIRST.** The user manages MIRAI by Dailoqa and wants an extremely careful AI engineering/review partner. Understand Sachin Sir's vision, technical conventions, all roles, merged work, current feature branches, outstanding integrations, and latest handoffs. **Use GitHub connector to verify *current* state before making any fresh PR approval**, since the facts below are a snapshot as of 25 October 2026. Never conflate documentation, a green CI check, a local result, an open PR, a merged PR, and stakeholder sign-off. Krish alone reviews/merges unless explicitly delegated. Distinguish user-approved business decisions from historical handbook proposals.

## 0. Fast resume / where we are now

- **Mission:** Build a modern, credible, Japanese-stakeholder-friendly, outsider-readable showcase and authorized internal management experience for **MIRAI by Dailoqa**: program/cohort/student journeys, Japanese/technical learning, assessments, marks, projects/internships, achievements/evidence, timelines, aggregates, and appropriate administration. Prioritize real, traceable evidence over impressive-but-fake dashboards.
- **Stakeholder:** **Sachin Sir** drives vision, UX feedback, and content priorities. No single overloaded dashboard; progressive drill-down, independent cohort journeys, no cohort-vs-cohort competition, at most **7 graphical elements per page**; Japanese audience should find the interface elegant and clear.
- **Actual code stack:** Existing **React 19 + TypeScript + Vite/Tailwind** frontend; **Python 3.12, Django 5.2, Django REST Framework** backend; **PostgreSQL 16** with **Django ORM migrations for `public`**, **SQLAlchemy 2.x + Alembic for `mirai`**. Docker Compose, Ruff, Pytest, frontend Vitest/lint/typecheck/build and CI. **Not Next.js or FastAPI** for this repo; historical speculative drafts used other stacks and are superseded by the actual code/approved HLD.
- **Mainline:** `develop` is integration, `main` stable release. Everyone works on task branches, PR → `develop`; **Krish reviews/merges**, no direct push/self-merge.
- **Backend completed and merged:** **Siddhesh Assessment Management PR #34** and **Akshat Marks Management PR #35** (Akshat filled in for Nikhil after an illness). Marks support **incremental per-student publication**, so 20 students' results may publish now, 10 later, while published individual records cannot be ordinarily edited (Super Admin audited corrections only). Validated with 706 backend tests on PR #35's final commit.
- **Reporting not finished:** **Anil**'s `feature/dashboard-reporting-api` branch is **8 behind `develop`, 1 commit ahead** at last review; implemented cohort counts, headline available metrics and placeholders for external/unintegrated KPIs. He must sync `develop`, accept/report the Marks read contract, integrate published assessment/marks data where suitable, define aggregation denominators and filters, security test, submit PR. Do not call his endpoints merged just because they exist on his branch.
- **Frontend audit open:** **Akansha's PR #36** `feature/akansha-four-phase-frontend-audit` at `be3124539333f0e6d162a1cc5f2b8f88cca4652c` open, unmerged, mergeable; CI green, but an acceptance pass requested after finding remaining hardcoded `Data Checks (9)` in `Header.tsx`, static Cohort 1/2 selectors and `Active Cohort` claims, and no visual verification. Her fixture removal generally strong; task is final *targeted* corrections, not rewrites. Depends on Jasleen/Tanvi shared fixtures; don't delete without coordination. **Do not assert PR #36 is merged.**
- **NEW — Jasleen PRs #37/#38 (verified 25 Oct):** **PR #37** (`jasleen/student-cohort-api-integration` → `develop`, head `da4c0f37bbd95ade1b93c4154201965876addec0`) is **open/unmerged/mergeable, 1 ahead and 0 behind**, with 4/4 reported green checks. It implements the live Student/Cohort roster, UUID navigation, permission-scoped current-view CSV export and tests (developer-reported 296 frontend tests). **PR #38** (`jasleen/profile-navigation-exports` → `jasleen/student-cohort-api-integration`, head `b97f36fcdf220487b31c3f38f3581c4fae23bfa8`) is **open/unmerged/mergeable** but **has zero checks because it targets the feature branch**. It adds real profile editing, scoped allowlists, fail-closed deep links and error handling, developer-reported 303 frontend tests. **Merge/review #37 first, then rebase/retarget #38 to `develop` and require its CI and review.** Neither is approved or merged by mere green checks. See §6.7.
- **QA:** **Anshuman Mathur** assigned end-to-end/integration test coverage. Closing the user's working “Phase 1” requires Anil integrated/merged, frontend truthfulness/integration and E2E user-journey tests passing, plus final demo/signoff; not merely team members saying “done”.
- **Next probable conversations:** PR #36 correction review; Jasleen PR #37/#38 stacked-PR review/merge sequence; Anil reporting branch review/contract negotiation; Anshuman tests; Phase 1 integrated acceptance; concise developer prompts/messages. The user strongly prefers code-grounded checks (files, tests, CI, authorization, races, migration chain), crisp copy/paste prompts, and a **clear merge / do-not-merge** verdict.

## 1. Sachin Sir — why MIRAI exists / original meetings

### 1.1 Problem, audiences and success

Sachin Sir wants a **clear view of what MIRAI actually does, has achieved and is achieving**, intelligible to outsiders, potential customers, Japanese stakeholders and reviewers. Internally, teachers, mentors, BR/TL, staff and students should be able to maintain or view appropriate data. The project should not be another cluttered internal KPI page, nor should an outsider need to understand each source application. It should tell a truthful, progressive story:

**MIRAI Program → Cohort → Individual learner/internship journey → Japanese/technical learning, attendance/performance, assessments, projects → milestones, activities, evidence, documents (BRD/FRD/HLD/LLD), achievements, approved aggregate outcomes.**

Stakeholder views are primarily **read-only, curated and privacy-safe**. Authorized employee/teacher/admin workflows can write, manage or publish source records. Never make a public/Reviewer view synonymous with unrestricted student detail. Show only enough information to understand performance and outcomes; allow permission-checked deeper drill-down when useful.

### 1.2 Meeting content and priorities

- Brainstorming originally covered dashboard purpose for **outsiders/customers**; scopes sometimes described as scope 1/scope 2 (not a finalized, signed functional-spec split).
- Wanted examples of student names/grades/progress reports, daily or weekly achievements, presentations, current accomplishments, technical and nontechnical work, functional deliverables, meaningful sample evidence, and engagement rather than shallow marketing counts.
- **No more than 7 graphics per page** (Sachin Sir's UX feedback); separate pages/modules for **Progress Timeline, Technical Learning, Mentors, BR, TL, Attendance, Engagement**, as well as Students, Projects, Japanese Learning, Reports and admin as applicable. Don't jam all visualizations on one page.
- Modern, elegant, clean, restrained Japanese-oriented design; understandable hierarchy, navigation and whitespace. Preserve approved existing design investment; improve it rather than rebuilding gratuitously.
- Separate cohort journeys; **no unapproved rankings or cohort competition**.
- Show BRD/HLD/LLD and project-stage evidence where access is approved. Let a reviewer explore “what did this cohort/person/team build?” and see source provenance, not invented placeholder content.
- Integrate/learn from existing systems (legacy PM, Performance, Ticket, Japanese learning website, Intern Hub). **Do not blindly clone legacy HR performance management**: MIRAI's new performance functionality is student assessments + marks, not HR appraisal. Some old source systems were discontinued or their integration is conditional.
- Sachin Sir initially provided an overcrowded dashboard mock; feedback was to split routes, improve hierarchy, and make things attractive to Japanese stakeholders. This informed frontend modularization and demo-data cleanup.
- Team also discussed documentation with track changes/history/contribution visibility for mentors; this is related to document governance and should not be treated as an implemented MIRAI feature without a separate task/decision.

### 1.3 Governance/document culture

BRD, FRD, HLD and LLD templates were prepared in **cover-only and full-document** variants with project branding. Minutes of Meetings (MOM) and structured plans were also drafted. The user distributed templates to the team for consistent requirements/architecture work. The templates and meeting ideas are **context**, not proof that every listed feature was implemented or approved. Project files and specs should trace BRD → FRD → HLD → LLD → API/tasks/tests, and changes must be attributable to owners.

## 2. Product scope and boundaries (do not accidentally expand)

| Area | Intended use | Verified status / caveats as of snapshot |
|---|---|---|
| Landing / MIRAI Program Journey | Stakeholder-friendly explanation, program goals, milestones, achievements/evidence | Prototype/frontend work exists; verify factual claims before publishing. |
| Authentication and persona RBAC | Session login; backend-enforced ability/scope; single cohesive persona-aware product | Implemented on `develop`, with scoped permissions; six personas below. |
| Students & Cohorts | Canonical cohort, batch/sub-batch, enrollments, profiles, student roster | Implemented backend API/core registry; frontend live integration progressing. |
| Japanese Learning | N5/N4/N3, lessons, coverage, progress, teacher workflow | **Status/source ownership must be verified**. Existing Japanese site was discussed. Handbook and HLD conflict on “maintain in MIRAI” vs “existing site adapter”; never invent a sync contract. |
| Academic Assessment | Cohort-based Japanese/technical assessment titles, dates, lifecycle, grading maxima | Siddhesh PR #34 **merged**. |
| Marks Management | Score entry, private unpublished results, per-student publication, audited correction | Akshat PR #35 **merged**. Incremental publication approved. |
| Dashboard reporting | Approved executive/cohort metrics, reviewer-safe aggregates | Anil branch in progress, not yet merged. Only verified sources; current marks contract awaiting his acceptance. |
| Projects/Internships | Portfolio, workstreams, stages, permitted docs/records, source attribution | PM adapter/API feature branches exist; do not assert source integration is production complete. |
| Technical Learning | Skills and technical progress | Source/definition needs approval; no fabricated metrics. |
| Attendance / Engagement | Authorized attendance/participation summaries | Conditional/source decision. |
| Mentors / BR / TL | Names, responsibilities, assigned relationships | Conditional; project interest exists, not necessarily operational backend. |
| Notifications, risks, finance, audit/log view, tickets | Appropriate future modules with validated source/privacy | Some frontend screens currently show honest unavailable states; not safe to fake counts. |

**Crucial architecture principle:** MIRAI is a **single front door / curated aggregation and management experience**, not six apps and not a complete copy of the PM/Japanese/Ticket tools. New academic marks are MIRAI-owned. External source data must have an agreed authoritative owner, explicit identity mapping, permitted scope and freshness/availability semantics.

## 3. Technical architecture — currently real

### 3.1 Runtime

- Repository: [`krisshhjain/Mirai-Dashboard`](https://github.com/krisshhjain/Mirai-Dashboard).
- Frontend: `frontend/` **React + TypeScript + Vite + Tailwind**, app shell, routes, design system, authored screens, typed API client and Vitest. It is **not** the user's unrelated SenaDrive Next.js app.
- Backend: `backend/` **Django 5.2 / Django REST Framework** (Python 3.12); session auth, serializers, permissions, routes, canonical identity, source adapters, SQLAlchemy services/repositories.
- PostgreSQL 16: `public` schema holds Django users/groups/sessions/framework migrations; `mirai` schema holds business tables via SQLAlchemy/Alembic. **Never create duplicate user models in SQLAlchemy**, migrate business tables using Django, or assume Django and SQLAlchemy transactions are automatically shared.
- Docker Compose dev baseline; root/backend instructions in README, `backend/README.md`, guides. No production hosting/security posture was conclusively approved in this conversation.
- `backend/config/urls.py` is top-level Django REST route authority; DRF schema `/api/v1/schema/`, Swagger UI `/api/v1/docs/`.
- API auth: Django **sessions**, HTTP cookies, **CSRF** on authenticated unsafe requests. **No JWT** for this MIRAI design. Cookie settings/CORS origins depend on deployment topology.
- Error envelope generally `{ "error": { ... } }`; status 400 validation, 401 no auth, 403 no scope or CSRF, 404 nonexistent/non-disclosable, 409 duplicate/concurrent conflict. PageNumberPagination typically `{count,next,previous,results}` with default page size 50. Check individual views and exact codes.

### 3.2 Authorized six personas and important checks

1. **Super Admin** — application-wide admin/manage privileges and `marks.correct_published` with mandatory reason + immutable audit. Not automatically Django infrastructure superuser.
2. **Japanese Teacher** — assigned cohort **Japanese subject** assessment/marks entry and publication; no arbitrary technical cohort access.
3. **Technical Admin** — assigned cohort **technical subject** assessment/marks entry and publication; not arbitrary Japanese subject access.
4. **Functional Admin** — authorized cohorts, profiles/program operations; approved summary and assigned detailed reporting rights, not global private data or marks publish by default.
5. **MIRAI Intern** — own profile/enrollments and **own published** assessment results only; drafts/unpublished/other intern results forbidden.
6. **Reviewer** — approved/published aggregates; no student PII, draft marks or detailed private records.

The capabilities, scopes (`NONE`, `SUMMARY`, `OWN`, `ASSIGNED`, `ALL`) and multi-role grants are enforced **server-side**. Staff cohort assignments reside in `mirai.staff_cohort_assignments`; never trust a frontend menu as sufficient access control. Sensitive operations need real permission, cohort, subject and record checks. Multi-role grants must not combine a cohort from one persona with a subject capability from another.

### 3.3 Canonical MIRAI business relationships

- `mirai.cohorts` → `mirai.cohort_enrollments` → custom Django UUID user.
- Cohort batch/sub-batch structure, student profile metadata, staff-to-cohort assignments.
- `mirai.assessments` (UUID; cohort FK, Japanese or technical category, lifecycle, max_marks and passing_marks, dates, creator, audit timestamps).
- `mirai.marks` unique `(assessment_id, student_id)`; score, is_published, published_at/by, optimistic version, created/updated metadata.
- `mirai.published_mark_corrections` append-only PostgreSQL-protected correction history with prior/new score, actor, reason and timestamp.
- App data is joined to Django UUID user identity by a deliberately controlled boundary; no casual cross-schema ORM ownership.

### 3.4 Alembic history / prior blocker resolved

Assessment migration was initially forked off the same predecessor as staff assignments causing **two Alembic heads**. Siddhesh synced `develop`, chained it after staff assignments, tested disposable PostgreSQL upgrades and concurrency. Current integrated tail as verified with Akshat PR #35:

```text
c4f1b2a9d3e8
  -> 80c3af6640c0    staff-to-cohort assignments
  -> 64b8d7a1e0f3    assessment management
  -> b91c7d2e4f5a    marks management (current head as of 25 Oct)
```

Always recheck `alembic heads`, full `alembic upgrade head` fresh DB and baseline upgrade, schema ownership, migration check in new PR. Never create a sibling Alembic head against stale `develop`.

## 4. Confirmed backend HTTP contracts in `develop` (25 Oct snapshot)

These were checked against root and module URL files during the preceding conversation; new chats should re-fetch for latest changes.

| Method(s) | Route | Notes |
|---|---|---|
| GET | `/api/v1/auth/csrf/` | CSRF token bootstrap |
| POST | `/api/v1/auth/login/` | `email` (or `username` email alias), `password`, optional `remember_me` |
| POST | `/api/v1/auth/logout/` | Ends session |
| GET | `/api/v1/auth/me/` and `/api/v1/me/` | Current user, roles, capabilities, capability_scopes |
| GET | `/api/v1/health/live` | Liveness |
| GET | `/api/v1/health/ready` | Database readiness |
| GET, POST | `/api/v1/cohorts/` | Create requires scope; list filtered |
| GET, PATCH | `/api/v1/cohorts/{cohort_id}/` | Canonical metadata; object-level auth |
| POST | `/api/v1/cohorts/{cohort_id}/lifecycle/` | Explicit lifecycle target |
| GET, POST | `/api/v1/cohorts/{cohort_id}/enrollments/` | `user_id` for enroll |
| PATCH | `/api/v1/cohorts/{cohort_id}/enrollments/{user_id}/` | Placement/activation |
| GET | `/api/v1/me/cohorts/` | Own enrollments |
| GET, PATCH | `/api/v1/me/profile/` | Allowed own profile fields only |
| GET | `/api/v1/students/` | Authorized roster with filters |
| GET, PATCH | `/api/v1/students/{user_id}/` | Scoped profile view/admin edit |
| GET, POST | `/api/v1/cohorts/{cohort_id}/batches/` | Batches |
| GET, PATCH, DELETE | `/api/v1/batches/{batch_id}/` | Batch detail |
| GET, POST | `/api/v1/batches/{batch_id}/sub-batches/` | Sub-batches |
| GET, PATCH, DELETE | `/api/v1/sub-batches/{sub_batch_id}/` | Sub-batch detail |
| GET, POST | `/api/v1/assessments/` | Assessment list/create |
| GET, PATCH | `/api/v1/assessments/{assessment_id}/` | Assessment read/update |
| GET, POST | `/api/v1/marks/` | List or create individual score |
| GET, PATCH | `/api/v1/marks/{mark_id}/` | Individual mark/read or unpublished update |
| POST | `/api/v1/marks/assessments/{assessment_id}/publish/` | Requires `{ "confirm": true }`; incremental |
| GET, POST | `/api/v1/marks/{mark_id}/corrections/` | History and Super Admin audited corrections |
| GET | `/api/v1/schema/` | OpenAPI source |
| GET | `/api/v1/docs/` | Swagger UI |

Assessment create: `cohort_id`, `title`, `category` required (`japanese` / `technical`); optional `status` (`draft` or `scheduled` initially), `max_marks`, `passing_marks`, `assessment_date`, `due_date`, `description`. Lifecycle `draft → scheduled → active → completed → archived` (with allowed cancellation transitions); intern draft not visible.

Marks create: `{assessment_id, student_id, score}`; score in `0..assessment.max_marks`, active enrollment, assessment `active` or `completed`. Unpublished update `{score, expected_version}`. Publish `{confirm:true}` selects only **currently unpublished** results, can happen repeatedly for late students, returns validation when none pending. Only Super Admin correction `{score, expected_version, reason}` on already-published record. Read list supports `published=true|false`, `assessment_id`, `student_id` with authorization filtering; unpublished hidden from interns. Publication row locks and optimistic versioning must remain intact.

**Do not mark these as implemented on `develop` until merged:** Anil's proposed `/api/v1/dashboard/overview/`, `/api/v1/dashboard/cohort-summary/` (feature branch only at snapshot), global search, generalized notification service, audit-log API, unsupported KPI aggregate endpoints.

## 5. Developer ownership / who is doing what

The authoritative historical member matrix is in the included HLD. **Assignments changed since 21 October**; the more recent specific owner/PR information below takes precedence over old draft names.

| Member | Assignment / scope | Status as of 25 Oct / notes |
|---|---|---|
| **Krish Jain** | Lead architect / integration planner / requirement & permission decisions / final PR reviewer-merger | Active across all; gives precise prompts and review gates. |
| **Sachin Sir** | Stakeholder/business vision, expected external narrative and design feedback | External stakeholder approval required for major claims/UX. |
| **Siddhesh** | Assessment Management: model, migration, CRUD/lifecycle, permissions, PostgreSQL tests | **Completed and PR #34 merged**. |
| **Nikhil** | Originally assigned Marks Management | Unavailable due to illness; **Akshat Awasthi assumed this assignment**. Do not attribute the merged marks code to Nikhil. |
| **Akshat Awasthi (AkshatAwa)** | **Actual recent assignment** Marks Management, originally Nikhil's scope; publication/correction/audit, handoff to Anil | **Completed and PR #35 merged**. Historical HLD also assigned him the Japanese source/enhancement work; do not assume that separate assignment is complete merely because Marks is merged. |
| **Anil** | Executive Overview + Cohort Reporting API, consume assessment and published marks where approved | **Feature branch present, 8 behind `develop`, 1 ahead**. Existing cohort/active counts; integration into merged marks/assessment remains. Needs PR + acceptance. |
| **Anshuman Mathur** | Test/quality workstream, recently **real end-to-end backend journeys and endpoint integration tests** | Work described as ongoing. Existing backend tests cover auth/session expiry, persona/cohort/subject/marks auth, PM membership resolver. Exact current test PR not verified. |
| **Akansha Attri** | Frontend shell/routes/auth UX, then **four-phase removal of fabricated operational fixture data** | **PR #36 open**, passes 4 checks, awaiting targeted final corrections + visual review, not yet merged. |
| **Jasleen (Jasleen2401)** | Dashboard modularization; live Students & Cohorts roster/CSV/export and profile navigation/editing | **PR #37 open against `develop`** (4 checks green); **PR #38 open as stacked child targeting #37 branch** (0 checks as of inspection). Review in order #37 → #38; neither merged. Residual shared demo consumers and pagination UX need handoff. |
| **Tanvi** | Landing modularization, responsive/accessibility/localization, design quality | Multiple frontend PRs merged historically; owns pages/consumers that Akansha deliberately did not break. Current latest tasks not independently reverified. |
| **Sriram** | PM source adapter and schema/identity mapping | Branch `feature/project-management-adapter` exists; current integration/PR completion not verified. |
| **Chaitanya** | PM read-only APIs, RBAC, OpenAPI | Branch `feature/project-management-api` exists; completion not verified. |
| **Ashish** | Django session login/logout/CSRF/current-user APIs | Related functionality is merged; do not assume entire individual task acceptance beyond reviewed PRs. |
| **Anant Sharma** | Six-persona RBAC and object-scope enforcement | RBAC merged; assignment integration branch `feature/rbac-assignments-api` exists, completion not verified. |
| **Anant Vaibhav** | Student profiles, cohort batches/sub-batches and roster | Related backend APIs exist in `develop`; precise personal signoff not verified. |
| **Sameer** | Typed frontend API client/CSRF/errors and/or parallel Ticket workstream in earlier handbook | Frontend API foundations integrated; original docs conflict on Sameer's exact workstream. Verify current issue before assigning more. |
| **Vaibhav Sharma** | Historical core backend collaborator | Specific current task not established in this chat. |

### 5.1 Team process and previous patterns

- The user assigns precise **one-owner, one-scope** tasks and often asks for ready-to-paste instructions to developer AI agents; prefers concise, unambiguous descriptions of what/how/tests/branch/PR/governance.
- Required PRs contain changed files, exact SHA, migration chain, tests/CI, risks, dependencies, API contracts, screenshots for frontend, open issues, reviewer signoff.
- If someone is ill/unavailable, responsibility may be reassigned; track assignment changes instead of treating old role matrix as current.
- For source-linked modules, ask developers for explicit **handoff contracts** and names of downstream consumers, rather than assuming their feature branches are integrated.

## 6. Recent PR narrative and decisions (key chronology)

### 6.1 Baseline → auth/UI modernization

- PR #1: Django/DRF/SQLAlchemy/Alembic/PostgreSQL/Docker/CI foundation.
- PR #2: Core cohort registry + enrollment API.
- PRs #9–#14: frontend quality CI, dashboard baseline modularization, shared UI extraction and interaction fixes.
- PRs #22–#26: frontend modular architecture, deep links/routes, session authentication integration, persona capability authorization and live Django API data.
- PRs #29–#33: landing/UI standardization, responsive accessibility/localization, hardening, security, performance and frontend final QA. **PR #33 merged** (SHA `04cb77aa...` noted in previous review); `develop` received ten-phase modernization.
- PR #19 was an obsolete integration attempt and was explicitly closed without merge. Never resurrect or treat it as deployed.

### 6.2 PR #34 — Siddhesh / Assessments: DONE AND MERGED

- PR: https://github.com/krisshhjain/Mirai-Dashboard/pull/34
- Feature branch: `feature/assessment-management`, final reviewed head `bf3713d795563598f94081d8cd45b093599184c1`.
- Merge commit: `1577317616641cf5d854c44cb70774104dbed135`.
- Files: Django assessment module + serializers/views/URLs, SQLAlchemy model/repository/service, migration `64b8d7a1e0f3`, domain registry imports, permissions/tests/docs.
- Original blocker: competing Alembic heads (`80c3af6640c0` staff assignments versus assessment migration) resolved by linearizing ancestry.
- Reviewer found assessment lifecycle race: sequential state checking could allow archived record revert. Siddhesh changed `update_assessment()` to `SELECT ... FOR UPDATE` before lifecycle validation, added real PostgreSQL concurrent archive-vs-update test, fixed migration-doc parent; **680 tests passed**, all 3 backend CI jobs green. Final verdict approved for merge and merged.

### 6.3 PR #35 — Akshat / Marks: DONE AND MERGED

- PR: https://github.com/krisshhjain/Mirai-Dashboard/pull/35
- Feature branch: `feature/marks-management-akshat`.
- Final reviewed head: `ea73b58c298bbe89d2d421e67e76f87f2f5b07b7`; merge commit `6d189338a4fc7aa5d1aff7d26e7d76c24a66604a`.
- Adds `mirai.marks` and `mirai.published_mark_corrections`, Alembic `b91c7d2e4f5a`, DRF list/create/detail/update/publish/corrections; strict RBAC, cohort/category rights, enrollment, score validation, optimistic versions, PostgreSQL append-only correction audit trigger.
- First version: potential post-publication late mark stranded; **later initial fix** made whole-assessment publication final. **User rejected that business rule** after explaining 30-student real-world scenario: 20 take exam now and 10 take exam later; teacher must publish early and late students incrementally without reopening the first 20.
- **Explicit final user decision: incremental publication**, with per-record immutability after publication, not an assessment-wide permanent lock. Akshat final third commit changed publication query to `Mark.is_published == False` and removed assessment-wide finality check. Creation/publication both lock assessment row, only unpublished rows publish, empty round validation, 20+10 regression and race regression.
- Correction to published mark: **Super Admin only**, mandatory nonblank reason, version control, durable history, no ordinary teacher edit of published record. Unpublished drafts can be edited by authorized user.
- **706 full backend tests passed** (26 dedicated Marks tests), Ruff, Alembic, backend 3/3 CI green; merged.
- **Exact reporting handoff doc:** [`docs/development/marks-reporting-read-contract.md`](https://github.com/krisshhjain/Mirai-Dashboard/blob/develop/docs/development/marks-reporting-read-contract.md). Anil's explicit acceptance pending as of snapshot.

### 6.4 PR #36 — Akansha / four-phase frontend operational fixture audit: OPEN

- PR: https://github.com/krisshhjain/Mirai-Dashboard/pull/36
- Branch `feature/akansha-four-phase-frontend-audit` / head `be3124539333f0e6d162a1cc5f2b8f88cca4652c`.
- At last check: **open, unmerged, mergeable, 1 commit ahead and 1 behind `develop`** (Marks was merged since branch base). 14 changed files, +1559/-804. **4/4 GitHub checks passed**; GitHub `Backend CI` and `Frontend CI` successful.
- Her 4 phases: (1) dependency map, (2) shell and search, (3) dashboard/reporting/notifications/admin fixture removal, (4) regressions + audit doc.
- Changes include removal of bogus `Sarah Tan` profile default, fake notifications/risk badge, fixture `people/projects` in global search, bogus shortcut metrics, executive KPI fallback, cohort period/trend/activity and audit log data. Live server-scoped student roster search preserved. No backend global search/notifications/audit log API invented; honest unavailable states are correct.
- New tests/updated tests; local `317/317` Vitest green, TS/lint/build passed, pre-existing 42 carousel lint warnings; Playwright screenshot download was blocked, so **manual or later Playwright visual check still needed**.
- Remaining specific reviewer findings from actual source: `frontend/src/app/shell/Header.tsx` shows hardcoded **`⚑ Data Checks (9)`**; header/ExecutiveOverview use static `Cohort 1`, `Cohort 2`, `All Cohorts` config; ExecutiveOverview displays **`Active Cohort`** for those cards independent of verified backend status. Ask Akansha to remove/neutralize unverified claims and avoid implying real cohort identity/status. Verify no other fabricated operational names/numbers/dates in owned screens; preserve working navigation, student roster search, RBAC, empty/error states, and existing contracts; add regression assertions.
- Shared `dashboardFixtures.ts` and `detailResolvers.ts` are consumed in **Jasleen-/Tanvi-owned** pages. Akansha must **document** remaining consumers and coordinate, **not delete** them or modify other owners' pages without approval.
- Final correction prompt was sent to user; requires sync with latest `develop`, visual browser checks, tests/build/CI, updated `Akansha_Four_Phase_Frontend_Audit.md`, new head SHA; leave existing PR open for Krish review.
- Akansha’s full audit file exists **on this feature branch, not `develop` yet**: [Akansha_Four_Phase_Frontend_Audit.md](https://github.com/krisshhjain/Mirai-Dashboard/blob/feature/akansha-four-phase-frontend-audit/Akansha_Four_Phase_Frontend_Audit.md).

### 6.5 Anil / Dashboard Reporting API: FEATURE IN PROGRESS

- Branch: [`feature/dashboard-reporting-api`](https://github.com/krisshhjain/Mirai-Dashboard/tree/feature/dashboard-reporting-api); latest known comparison: **1 commit ahead, 8 commits behind `develop`** (checked 25 Oct).
- Feature branch introduces (not merged) `backend/domain/repositories/reporting.py`, `backend/domain/services/reporting.py`, dashboard reporting DRF serializers/views/URLs via `modules/core_registry`, `backend/tests/test_dashboard_reporting.py`.
- Intended routes (feature branch only): `GET /api/v1/dashboard/overview/` and `GET /api/v1/dashboard/cohort-summary/` (optional `cohort_id`).
- Executive metrics: `total_students` = distinct enrolled user UUID count across scoped cohorts; `active_interns` = enabled Django users with active enrollment in active cohort; `active_cohorts` = scoped active cohorts. Other executive slots `japanese_progress`, `technical_progress`, `projects_on_track`, `key_risks` currently explicitly pending/deferred. Per-cohort summary contains total/active/inactive students and placeholder `n5_cleared` and `projects`. Tests include unauthenticated/intern denials, distinct counting, Reviewer aggregate PII protections, cohort assigned scope, etc.
- User informally described Anil as around **80% complete**; this is a team self-reported estimate, not independently measured acceptance. For his work to reach signoff, sync `develop`, integrate actual published marks+assessments where policy supports, approve or refine Marks reporting read contract, settle denominator semantics/date filters/privacy/subject scope, maintain fallback as `pending` for unsupported systems; add integration/security tests, run full CI, submit PR, wait for Krish review. Do not transform random Japanese test marks into **`N5 cleared`** without an approved mastery/certification rule. Do not invent tech/PM metrics.

### 6.6 Anshuman / E2E tests: IN PROGRESS

- Recent message: developer stated local PostgreSQL full backend suite and Ruff checks run, existing tests cover **auth/session expiry, persona access, cohort & subject scoping, privacy, marks authorization and PM membership resolver**, and asked Krish to confirm real endpoint contracts for missing end-to-end tests.
- Assistant verified root API routes and gave a response covering auth, core cohorts/roster, assessments, marks, incremental publication, session cookies/CSRF, error envelope, and the distinction between current public HTTP endpoints versus internal PM membership resolver. The source of truth is code under `backend/config/urls.py`, `accounts/urls.py`, `modules/*/urls.py` and serializers.
- Requested E2E tests: CSRF → login → current user → logout; Super Admin cohort/enrollment → Japanese Teacher assessment → enter 20 results → publish → ten late results → publish again → intern own published data only; unauthorized attempts, cross-cohort/subject/multi-role, no PII leakage; Super Admin correction audit, stale version, rollback, pagination/errors; reporting Reviewer aggregate only after Anil merges, with actual fixture data.
- Need a proper E2E PR, verified PostgreSQL tests, correct use of real HTTP API, CI, final integration demonstration. Do not fabricate endpoints or consider PM resolver a public route.

### 6.7 PRs #37 and #38 — Jasleen / live Students & Cohorts + Profiles: STACKED, OPEN, NOT YET SIGNED OFF

**Verified through the GitHub connector 25 Oct 2026.** The developer descriptions report passing local tests; those totals are not a substitute for the exact required CI after rebasing/retargeting.

**PR #37 — Base integration — review/merge FIRST**

- URL: https://github.com/krisshhjain/Mirai-Dashboard/pull/37
- Title: `feat(students-cohorts): live student roster, canonical UUID navigation & scoped CSV export`.
- Branch: `jasleen/student-cohort-api-integration` → **`develop`**.
- Status: **open, unmerged, mergeable**; last inspected head `da4c0f37bbd95ade1b93c4154201965876addec0`; `1 ahead / 0 behind develop` on inspection. GitHub user supplied **4/4 successful checks**; GitHub CI workflows `Backend CI` and `Frontend CI` reported success at the head.
- Size: **2 files**, 749 additions and 90 deletions (PR metadata); `frontend/src/pages/students/StudentsCohortsPage.tsx` and `frontend/src/test/integration/liveApiIntegration.test.tsx`.
- Contract: Uses existing `GET /api/v1/students/` paginated scoped roster plus `GET /api/v1/cohorts/`. Avoids presenting invented authenticated student records/KPIs. Canonical backend `user_id`/UUID for row navigation (`/students/{uuid}`), fail-closed invalid navigation, honest load/error/retry/empty states. CSV export is labeled **`Export Current View (.csv)`**, uses RFC 4180-safe escaping, and must export only currently loaded permission-scoped records (not the entire database).
- Developer local verification claim: typecheck and build pass, lint with **42 existing carousel warnings**, live API integration **35** tests, full frontend **296** tests.
- **Known limitation documented by Jasleen:** roster has **no pagination navigation UI** even though DRF results can be paginated, so only one loaded page is visible/exportable; communicate current-view scope accurately. No XLSX/PDF backend export service exists.
- **Review issues still to confirm before approval:** ensure no runtime authenticated fallback to `DEMO_STUDENTS_LIST` in surrounding derived cards/charts or interaction paths; verify actual filters/pagination, RBAC scope and field projection; confirm scope of `GET /api/v1/students/` and `GET /api/v1/cohorts/` under assigned/own grants; avoid silent exposure of test fixtures. Code inspection found the file still retains `const studentsList = students ?? DEMO_STUDENTS_LIST` for some presentation paths; assess all authenticated uses before issuing approval rather than assuming zero fixture leak. Require `develop` integration smoke tests and screenshot/manual visual check.

**PR #38 — Stacked profile/editing PR — review/merge SECOND**

- URL: https://github.com/krisshhjain/Mirai-Dashboard/pull/38
- Title: `feat(profiles): live profile editing, navigation contracts & scoped exports`.
- Branch: `jasleen/profile-navigation-exports` → **`jasleen/student-cohort-api-integration`** (deliberately stacked on PR #37), not directly into `develop` yet.
- Status: **open, unmerged, mergeable**; 2 commits on this PR, last head `b97f36fcdf220487b31c3f38f3581c4fae23bfa8`; `2 ahead / 0 behind` parent branch on inspection; **0 GitHub checks on the stacked PR**, so do not treat it as CI accepted.
- Size: **4 files**, 608 additions / 5 deletions: `frontend/src/features/cohorts/CohortDetailRoute.tsx`, `frontend/src/features/students/ProfileDetailView.tsx`, `frontend/src/features/students/StudentDetailRoute.tsx`, `frontend/src/test/integration/liveApiIntegration.test.tsx`.
- Trainee/self edit: `PATCH /api/v1/me/profile/` **only** `phone`, `linkedin_url`, `github_url`, `biography`.
- Staff/academic edit: `PATCH /api/v1/students/{user_id}/` **only** `institution`, `academic_program`, `academic_year`. These are backed by Django serializers/RBAC, not client-supplied privilege.
- Live profile detail synchronization; modal closes and success notification only after confirmed server success; explicit 400 validation field messages, 401 session expiry, 403 forbidden, 404 missing/out-of-scope, network/5xx/retry; fail-closed canonical UUID student/cohort deep links. Preserves current-view CSV capability from parent PR.
- Developer local verification claim: 42 live API integration tests / 303 total frontend tests across 17 files, typecheck/build/lint pass (42 inherited carousel warnings).
- **Must merge #37 first**. Then have Jasleen **retarget #38 to `develop`**, update against the latest integration head, verify the PR diff contains only intended Phase-3 delta, run Frontend CI and manual smoke tests, and request review. Do not merge directly into the temporary parent feature branch as a shortcut to bypass `develop` reviewer gate unless Krish expressly instructs a different integration strategy.

**Coordinating with Akansha:** PR #36 deliberately left shared fixtures for Jasleen/Tanvi. Review fixture-retirement overlap before merging overlapping frontend PRs; re-run both branches' CI after final `develop` synchronization. The currently unmerged PR #37/#38 are not proof that all live frontend/student data paths are already in `develop`.

## 7. Akshat → Anil handoff / reporting contract (exact ideas)

When Anil integrates, he should be told explicitly:

1. Assessment and Marks PRs **#34/#35 are merged into `develop`**; pull latest `develop` *first* and preserve Alembic linear head `b91c7d2e4f5a`.
2. Authoritative sources: `mirai.assessments` (`id`, `cohort_id`, `category`, `status`, `max_marks`, `passing_marks`, dates) joined to `mirai.marks` (`assessment_id`, `student_id`, `score`, `is_published`, timestamps) and canonical cohorts/enrollments.
3. **Published-only reporting:** use `is_published = true`, current corrected score; each `(assessment_id, student_id)` counted once; **do not join correction-history rows as extra scores**; no published record for not-yet-taken students. **Missing/unpublished ≠ zero or failure.** Multiple rounds represent a legitimate data state.
4. RBAC: `reporting.view_summary` only approved aggregates, `reporting.view_detail` only if explicitly approved and scoped; Reviewer aggregate-only, interns never see another's result. No correction actor/reason leaked. Functional Admin/Teacher/Technical Admin cohort/subject limits must follow matrix, with no unauthorized count leakage.
5. Define/report exact aggregation formulas and denominators (mean and pass rate of **published eligible results** versus registered/enrolled cohort; explain both instead of pretending identical), null/empty semantics, cohort/subject/assessment filtering, date-window, pagination. Must get product approval where metric meaning is material.
6. **Japanese N5 cleared is not defined as arbitrary average/pass of any Japanese assessment.** Needs explicit approved certification/level rules and source, otherwise `pending_integration`/`pending_source` is correct.
7. PM/Japanese other-source integration only through approved adapters and identity mapping, not direct unsanctioned copying or optimistic zero fallback.
8. Coordinate with Akansha/Jasleen frontend typed contracts. Add tests for incremental rounds, corrected marks, draft isolation, null/no data, cross-cohort/subject, Reviewer privacy, multi-role, repeated publications. Submit existing reporting PR/branch against `develop`; Krish alone merges.

## 8. Frontend integrity and the “no fake data” rule

The original React dashboard/Figma prototype included hardcoded cohort labels, fictional student/project profiles, Unsplash image links, fake notifications and audit events, time/percentage/KPI values, progress charts and other demo data. It was helpful to demonstrate layout but cannot appear as actual live program evidence.

**Production UI requirements:**

- Never replace missing results with `0` unless **true authoritative zero**. Distinguish **pending integration**, **no records yet**, **unpublished**, **permission denied**, **network failure**, **not collected**, **stale source** and **measured zero**.
- A static example must be unmistakably marked “demonstration/synthetic” if kept for demos. Prefer real authorized API data in production; no ambiguous fake names, dates, grades, progress, risk badges.
- Server RBAC provides security; frontend capability gates are additional UX only. Search must not expose cross-student data via fallback fixture, unauthorized global aggregation or other cohort names/counts.
- Preserve modern clean design and interaction behavior, accessibility/responsiveness/localization. Japanese stakeholders, no more than 7 graphics per page; avoid bulky single-screen dashboards.
- Only remove fixtures in own scope where safe; prepare handoff to owners of shared dependencies. Do not replace a fake KPI with a suspicious hardcoded “unavailable = 0”.
- Current UI may include screens with honest neutral/unavailable panels pending APIs; those panels are acceptable under a defined phase boundary, but **cannot be counted as fully integrated functionality**.

## 9. Repo governance, PR review and acceptance protocol

1. Fetch current `origin/develop` / verify latest GitHub state and user’s exact requested PR. Feature work starts from current `develop`; `main` stable release.
2. **No direct pushes to `develop` or `main`, no force push, no self-merges.** Contributor works in `feature/<task>` (or fix/docs/chore), opens PR to `develop`. Krish reviews/tests/merges; explicit delegation needed otherwise.
3. Review actual diff, changed files, scope creep, API and RBAC behavior, migrations and CI **at the exact head SHA**, not merely author PR description. Green CI is necessary but not sufficient. Check unresolved review threads and compare behind/ahead to `develop`.
4. For backend, Ruff lint/format, Django system and migration checks, Alembic `heads` and `check`, full/targeted Pytest and disposable PostgreSQL, concurrency and rollback tests, schema ownership, Backend CI (`backend-quality`, `backend-tests`, `backend-docker`). No silent model drift, sibling heads or weaker tests.
5. For frontend, TypeScript typecheck, lint, full Vitest (bounded concurrency when Windows default workers time out), production build, CI, UI screenshot/manual visual checks, auth/RBAC and route behavior, cross-owner integration tests. Existing warnings should be disclosed not misrepresented as new failures.
6. **Review outcomes:** APPROVE MERGE / DO NOT MERGE / TARGETED CHANGES REQUIRED / BLOCKED BY OTHER OWNER. Provide evidence and exact rationale, distinguish blocker vs follow-up.
7. Never merge through tool merely because the user asks “check/review”; user must explicitly request the action. PR approval should have a precise SHA and no outstanding regressions.
8. For assistant-generated instructions, preserve governance: **“Work on existing PR, do not merge/deploy; run checks; share commit SHA/results; return for Krish final review.”**
9. PR #19 was closed without merging; do not reactivate. Avoid reintroducing eliminated dependencies or dummy production data.

## 10. Milestone framing, “Phase 1”, acceptance gate and remaining work

**Important historical terminology:** The original 21 October HLD called **Phase 0** foundational code, **Phase 1** a 21–22 October *developer review/PR submission gate*, **Phase 2** authenticated joined journeys and integrations, **Phase 3** hardening. In later chats the user naturally used **“Phase 1 of development”** to mean the working first integrated MIRAI MVP slice. Likewise Akansha's **four internal audit phases** are not the product roadmap phases. When responding, say which meaning is intended and **do not imply a full production launch or stakeholder signoff merely because the current development slice is almost done**.

### 10.1 First integrated MVP (“Phase 1 development”) demonstration

```text
1. Real user/session → CSRF + login; backend identifies persona and assignment.
2. Authorized cohort roster / assigned teacher scope / real student enrollments.
3. Teacher creates Japanese/technical cohort assessment (no unauthorized writes).
4. Teacher enters marks for 20 students out of 30 and publishes them.
5. The 20 interns see only their own published scores, not another intern's or drafts.
6. Other 10 students finish later → their marks entered/published separately.
7. Teacher cannot modify published scores; Super Admin can correct with reason & audit.
8. Reviewer sees only approved aggregate metrics from published data; no PII leakage.
9. Frontend joins the journey to real API; honest unavailability for missing modules.
10. E2E tests + CI + mobile/browser/accessibility smoke check + Krish demo signoff.
```

### 10.2 Immediate sequence as of snapshot

1. **Akansha:** Fix hardcoded `Data Checks (9)` and other unsourced cohort/activity claims in owned shell/ExecutiveOverview; synchronize with latest `develop`, preserve navigation/permissions/real roster search; visual check, tests/build/CI, update audit ledger, keep PR #36 open pending reviewer. Review/merge when correct.
2. **Anil:** Sync `feature/dashboard-reporting-api` with merged Assessment & Marks; review and explicitly accept/amend `docs/development/marks-reporting-read-contract.md`; integrate only defensible published scores/assessment counts into metrics, define denominator/scoping rules with Krish, add tests, submit PR and CI. Do not count “Japanese progress/N5 cleared” without approved logic.
3. **Anshuman:** Build real E2E scenarios against disposable PostgreSQL/backend HTTP routes; negative RBAC, incremental marks, publish/correct, Reviewer reporting *after* that endpoint merges, regression. Submit PR.
4. **Jasleen:** Review/merge **PR #37 first** into `develop` only after fixing any remaining authenticated fixture fallbacks and agreeing the lack-of-pagination UI limitation. Then retarget the stacked **PR #38** to `develop`, re-run CI and review; preserve self/admin edit allowlists, success/error semantics and canonical UUID navigation. **Tanvi:** coordinate owned shared fixture consumers and other frontend sections; do not break unrelated routes.
5. **PM owners/Sriram/Chaitanya:** Verify adapter/API branches and contractual availability, source data privacy and ID mapping. Scope may be deferred if authoritative source not ready and Krish approves MVP cutline.
6. **Final integration:** Ensure all accepted PRs merged into `develop`, updated migrations fresh DB, backend/frontend green, no fixture leakage, reviewer privacy and user journey demo. Decide whether to release `develop` → `main` in a separate reviewed PR only when ready.
7. Stakeholder review with Sachin Sir: confirm layouts/navigation/progressive program story, relevant approved evidence, under-seven-graphics constraint, accessibility, localization and claims.

### 10.3 Known unapproved/pending policy questions

- Approved definition of “N5 cleared”/Japanese proficiency vs test scores and other graded metrics.
- Reporting count/mean/pass-rate denominators, retention, time windows and cohort changes; whether summary vs detail routes are needed and exactly who can see them.
- Japanese Learning source ownership conflict in older handbook/HLD; PM project data source, read authorization, identities and refresh model.
- Reviewer exact publication scope and external-vs-internal visibility; no raw intern PII unless specifically authorized.
- Real cohort labels and states: `Cohort 1/2` must not be mistaken for verified production records.
- Attendance, technical, mentor, BR/TL, risk, finance, notification/audit log and document workflow APIs/sources.
- Production hosting/CORS/CSRF cookie topology, secrets, backup, retention, RPO/RTO, audit export and deployment approval.
- Signed requirements/ADR changelog and final MVP acceptance matrix; previous handbook/HLD contain historical proposal statuses.

## 11. Document/source index — important Markdown and original references

**Self-contained source documents packaged alongside this handover** in `reference_docs/` (available when using the ZIP; the standalone MD still works by itself):

- `MIRAI_Project_Handbook.md` — original (dated 20 Oct 2026) project context, stakeholders, MVP and governance; historically proposed/open items must be reconciled with newer PR decisions.
- `MIRAI_High_Level_Design_v1.0.md` — original (dated 21 Oct 2026) complete design/architecture/ADR/team/phase matrix. Has historical planning dates and provisional decisions.
- `GIT_Document_Original_Extract.txt` — text extraction of the GIT Document's one-page governance policy.
- `Akansha_Four_Phase_Frontend_Audit_ORIGINAL_TASK_BRIEF.md` — original audit task/assignment reference recovered from Library; it is not the complete current feature-branch PR report.
- `JASLEEN_PR37_PR38_HANDOFF.md` — newly written stacked-PR/handoff reference with exact hashes, changed files, tests, integration order and known limitations.
- `TEAM_DOCUMENT_INDEX.md` — links to everyone's relevant GitHub Markdown files, authoritative source locations and branch status. None of those linked GitHub contents have been silently fabricated or treated as physical attachments.

**Additional team documentation in GitHub `develop` (links; not physically embedded in this portable package).** They remain readable to the same authorized GitHub account or future connector. These are helpful when reviewing the named owner/module:

### Architecture, security and DB

- [Assessment schema/contract — Siddhesh](https://github.com/krisshhjain/Mirai-Dashboard/blob/develop/docs/architecture/assessment-schema-contract.md)
- [Assessment Management detailed implementation plan — Siddhesh](https://github.com/krisshhjain/Mirai-Dashboard/blob/develop/docs/development/MIRAI_Assessment_Management_Implementation_Plan_v2.md)
- [Assessment API test guide](https://github.com/krisshhjain/Mirai-Dashboard/blob/develop/docs/development/assessment-api-test-guide.md)
- [Marks integration status — Akshat](https://github.com/krisshhjain/Mirai-Dashboard/blob/develop/docs/architecture/marks-management-integration-status.md)
- [Marks API guide — Akshat](https://github.com/krisshhjain/Mirai-Dashboard/blob/develop/docs/development/marks-management-api.md)
- [Marks → reporting read contract — Akshat / Anil pending](https://github.com/krisshhjain/Mirai-Dashboard/blob/develop/docs/development/marks-reporting-read-contract.md)
- [RBAC permission matrix](https://github.com/krisshhjain/Mirai-Dashboard/blob/develop/docs/architecture/rbac-permission-matrix.md)
- [RBAC usage / object-scope patterns](https://github.com/krisshhjain/Mirai-Dashboard/blob/develop/docs/architecture/rbac-usage.md)
- [Auth me / RBAC response contract](https://github.com/krisshhjain/Mirai-Dashboard/blob/develop/docs/architecture/auth-me-rbac-contract.md)
- [PM membership / resolver contract](https://github.com/krisshhjain/Mirai-Dashboard/blob/develop/docs/architecture/pm-membership-contract.md)
- [Database ownership](https://github.com/krisshhjain/Mirai-Dashboard/blob/develop/docs/architecture/database-ownership.md)
- [Database/migration conventions](https://github.com/krisshhjain/Mirai-Dashboard/blob/develop/docs/database/migrations.md)
- [Auth/session guide](https://github.com/krisshhjain/Mirai-Dashboard/blob/develop/docs/development/auth-session-guide.md)
- [Backend database guide](https://github.com/krisshhjain/Mirai-Dashboard/blob/develop/docs/development/backend-database-guide.md)
- [Core registry reference](https://github.com/krisshhjain/Mirai-Dashboard/blob/develop/docs/development/core-registry-reference.md)
- [Student/cohort API contracts](https://github.com/krisshhjain/Mirai-Dashboard/blob/develop/docs/development/students-cohorts-api.md)

### Frontend team and QA handovers

- [Akansha 4-phase detailed audit report — on PR #36 feature branch, NOT yet merged](https://github.com/krisshhjain/Mirai-Dashboard/blob/feature/akansha-four-phase-frontend-audit/Akansha_Four_Phase_Frontend_Audit.md)
- [Frontend architecture](https://github.com/krisshhjain/Mirai-Dashboard/blob/develop/docs/frontend/architecture.md)
- [Frontend authorization](https://github.com/krisshhjain/Mirai-Dashboard/blob/develop/docs/frontend/authorization.md)
- [Frontend auth integration](https://github.com/krisshhjain/Mirai-Dashboard/blob/develop/docs/frontend/auth-integration.md)
- [Frontend API integration](https://github.com/krisshhjain/Mirai-Dashboard/blob/develop/docs/frontend/api-integration.md)
- [Frontend app state contracts](https://github.com/krisshhjain/Mirai-Dashboard/blob/develop/docs/frontend/application-state-contracts.md)
- [Frontend design system](https://github.com/krisshhjain/Mirai-Dashboard/blob/develop/docs/frontend/design-system.md)
- [Frontend accessibility/i18n audit](https://github.com/krisshhjain/Mirai-Dashboard/blob/develop/docs/frontend/accessibility-i18n-audit.md)
- [Frontend accessibility guidelines](https://github.com/krisshhjain/Mirai-Dashboard/blob/develop/docs/frontend/accessibility-guidelines.md)
- [Frontend localization guidelines](https://github.com/krisshhjain/Mirai-Dashboard/blob/develop/docs/frontend/localization-guidelines.md)
- [Responsive design](https://github.com/krisshhjain/Mirai-Dashboard/blob/develop/docs/frontend/responsive-guidelines.md)
- [Frontend testing](https://github.com/krisshhjain/Mirai-Dashboard/blob/develop/docs/development/frontend-testing.md)
- [Final frontend QA report](https://github.com/krisshhjain/Mirai-Dashboard/blob/develop/docs/frontend/final-qa-report.md)
- [Frontend production hardening audit](https://github.com/krisshhjain/Mirai-Dashboard/blob/develop/docs/frontend/production-hardening-audit.md)
- [Frontend security validation](https://github.com/krisshhjain/Mirai-Dashboard/blob/develop/docs/frontend/security-validation.md)
- [Frontend performance baselines](https://github.com/krisshhjain/Mirai-Dashboard/blob/develop/docs/frontend/performance-baselines.md)
- [Frontend production readiness](https://github.com/krisshhjain/Mirai-Dashboard/blob/develop/docs/frontend/production-readiness.md)
- [Frontend handover](https://github.com/krisshhjain/Mirai-Dashboard/blob/develop/docs/frontend/handover.md)
- [Landing modularization](https://github.com/krisshhjain/Mirai-Dashboard/blob/develop/docs/frontend/landing-modularization.md)
- [Dashboard modularization plan — Jasleen / frontend](https://github.com/krisshhjain/Mirai-Dashboard/blob/develop/docs/development/dashboard-modularization-plan.md)
- [Dashboard interaction checklist](https://github.com/krisshhjain/Mirai-Dashboard/blob/develop/docs/development/dashboard-interaction-checklist.md)
- [Day 1 handover](https://github.com/krisshhjain/Mirai-Dashboard/blob/develop/docs/development/day-1-handover.md)

### Executable code and PR links

- [Backend route registry](https://github.com/krisshhjain/Mirai-Dashboard/blob/develop/backend/config/urls.py)
- [Django auth views](https://github.com/krisshhjain/Mirai-Dashboard/blob/develop/backend/accounts/views.py)
- [Core registry module URLs](https://github.com/krisshhjain/Mirai-Dashboard/blob/develop/backend/modules/core_registry/urls.py)
- [Assessment URLs](https://github.com/krisshhjain/Mirai-Dashboard/blob/develop/backend/modules/assessments/urls.py)
- [Marks URLs](https://github.com/krisshhjain/Mirai-Dashboard/blob/develop/backend/modules/marks/urls.py)
- [Marks service](https://github.com/krisshhjain/Mirai-Dashboard/blob/develop/backend/domain/services/marks.py)
- [Marks PostgreSQL tests](https://github.com/krisshhjain/Mirai-Dashboard/blob/develop/backend/tests/test_marks_management.py)
- [Anil feature branch reporting service — NOT MERGED](https://github.com/krisshhjain/Mirai-Dashboard/blob/feature/dashboard-reporting-api/backend/domain/services/reporting.py)
- [Anil feature branch reporting tests — NOT MERGED](https://github.com/krisshhjain/Mirai-Dashboard/blob/feature/dashboard-reporting-api/backend/tests/test_dashboard_reporting.py)
- [PR #34 merged](https://github.com/krisshhjain/Mirai-Dashboard/pull/34)
- [PR #35 merged](https://github.com/krisshhjain/Mirai-Dashboard/pull/35)
- [PR #36 pending Akansha final pass](https://github.com/krisshhjain/Mirai-Dashboard/pull/36)
- [PR #37 Jasleen — base roster integration, pending](https://github.com/krisshhjain/Mirai-Dashboard/pull/37)
- [PR #38 Jasleen — stacked profile integration, pending after #37](https://github.com/krisshhjain/Mirai-Dashboard/pull/38)

**Source-availability caveat:** This ZIP package physically includes the original Handbook and HLD Markdown, the original GIT policy extraction, and Akansha’s historical original task/assignment brief (NOT the 599-line final PR audit). The many repo Markdown files above are **indexed by exact paths/links, not silently claimed to be bundled**. Use GitHub connector in a new chat to read their current contents before making file-specific claims. Linked docs may change after this snapshot.

## 12. How the next assistant should work

1. Start by treating this MD and its reference documents as the project's historical context and decision ledger; ask for an up-to-date GitHub check when a user references a new PR or commit. **Do not repeat old resolved blockers after a developer fixes them.**
2. The user's immediate requests often are “review this PR” and “write exact developer prompt”. For PR reviews: retrieve PR info, compare branch and `develop`, inspect complete changed files and impacted services/test behavior, verify actual CI jobs at exact SHA, migration chain/security/edge cases, pending review comments; then issue merge verdict and concise copyable patch instructions if needed.
3. Use **actual current code** to settle API behavior; docs/historical drafts are lower precedence than Krish's latest decision and approved source code. Where docs and code disagree, flag both and propose targeted reconciliation.
4. Never assume an external Japanese/PM source, reporting metric, notification, audit-log, fully public reviewer view or production deployment exists solely because the prototype displays it.
5. If an expected feature has been reassigned from an original contributor to another, honor the actual new owner and don't reopen their old assignment.
6. Share ready-to-send developer instructions that name existing PR/branch, precise behaviors, tests, security, no scope creep, no self-merge, deliver SHA and CI.
7. For requested documents, make polished artifacts and give a real downloadable path. For status report, use confidence labels **VERIFIED IN CODE / APPROVED USER DECISION / DEVELOPER CLAIM / HISTORICAL PLAN / PENDING VALIDATION**.
8. Keep writing direct, helpful, technically rigorous. Explain material business implications in clear examples (e.g., 30 students, 20 now/10 later), not only abstractions.
9. The main decision-maker is Krish; he wants a proper end-to-end solution, not just a superficial scaffold and inflated “100%” progress. Never promise Phase 1 completed until data, authorization, frontend, tests and user-journey acceptance are verified.

## 13. Proposed opening message to paste into a new ChatGPT conversation

> I'm continuing MIRAI by Dailoqa. Read the attached `MIRAI_COMPLETE_CONTEXT_HANDOVER.md` thoroughly, plus its accompanying Handbook, HLD and GIT policy. It is the complete context from my previous ChatGPT engineering-management conversation. I'm the lead architect and PR reviewer; Sachin Sir is the stakeholder. Use the latest approved decisions and current GitHub repo `krisshhjain/Mirai-Dashboard`; `develop` is integration, `main` stable, and only I merge unless I delegate. Do not assume open PRs have merged; verify GitHub before answering any current status or reviewing changes. Phase 1's immediate remaining items are Akansha PR #36 final fixture-audit correction/visual check, Jasleen stacked PR #37 then #38 review/CI/merges, Anil reporting branch integration/contract and PR, Anshuman end-to-end integration tests, then integrated frontend/backend/user-journey demo/signoff. My next request follows below.

---

**End of context snapshot — 25 October 2026, updated to include Jasleen PRs #37/#38.**
