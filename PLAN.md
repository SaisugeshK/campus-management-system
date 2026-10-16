# College Management Application — Development Plan

Status: Approved scope. Implementation specification for Claude.
Date: 2026-10-13

## 1. Instructions to Claude

Build this application incrementally using this document as the source of truth. Implement working end-to-end features, not placeholder pages. All modules below are committed scope; phases determine sequence rather than excluding later modules.

Before coding, inspect the repository, record compatible supported tool versions, and create a task checklist mapped to this plan. Preserve existing useful work. Ask only when a missing institutional policy materially changes implementation; otherwise use the stated defaults and document assumptions. Do not purchase services, activate live payments, or publish externally without owner authorization.

For each phase deliver database migrations, Java APIs, React screens, permission checks, tests, demo data, and setup documentation. Report completed, incomplete, blocked, and simulated features honestly. Maintain `docs/IMPLEMENTATION_STATUS.md` with a requirement-to-code-and-test matrix. Never claim the application is complete while agreed modules remain placeholders.

## 2. Product goals and defaults

One college with multiple departments initially. Model campus and institutional configuration so future expansion is possible; full multi-tenant SaaS is outside the initial design. Semester-based academics are the default, with configurable academic rules. Faculty, non-teaching staff, students, guardians, and administrators use email accounts.

Deliver a responsive React web application first. Serve it and Java REST APIs from one Spring Boot deployment and one URL. PostgreSQL is the system of record. Build reusable backend APIs so a later React Native mobile application uses the same business logic and database.

Default institutional timezone: Asia/Kolkata, configurable. Store timestamps as UTC and render in institutional/user timezone. Store birthdays and academic calendar dates as date values. Default currency: INR, configurable. Use synthetic data in demos. Manual attendance is the first implementation; biometric, GPS hardware, and other integrations use adapters later.

## 3. Required stack and architecture

| Layer | Required choice |
|---|---|
| Backend | Java, Spring Boot, Maven wrapper |
| Security | Spring Security, server-enforced permissions |
| Persistence | PostgreSQL, Spring Data JPA/Hibernate |
| Migrations | Flyway; no automatic production schema updates |
| Frontend | React, TypeScript, Vite, React Router |
| UI | Accessible reusable components; consistent design tokens |
| API | REST JSON under `/api/v1`, OpenAPI contract |
| Backend tests | JUnit, Spring Boot integration tests, Testcontainers PostgreSQL |
| Performance | Apache JMeter `.jmx` plans |
| Browser tests | Playwright TypeScript `.spec.ts` files |
| Mobile | React Native and TypeScript, later phase |
| Deployment | Executable Spring Boot JAR containing React build |

Select supported Java/Spring versions and compatible library versions at project initialization, verify official documentation, and pin them. Do not use Node.js as the application backend. Node is used only for frontend/mobile tooling and browser tests.

Use a modular monolith with packages per domain. Separate controllers, request/response DTOs, services, repositories, entities, and permission policies. Do not expose JPA entities directly. Keep controllers thin and business rules in transactional services. Avoid microservices in the initial deployment.

Use a persisted notification outbox and retry worker for emails and integration events. Do not hold database transactions open while contacting providers. Redis or a separate message broker is optional if measured needs justify it; the demo must work without them.

Suggested repository layout:

```text
college-management/
  PLAN.md
  backend/
    pom.xml
    mvnw, mvnw.cmd
    src/main/java/.../college/
      identity/ organization/ admissions/ students/ staff/
      academics/ timetable/ attendance/ approvals/ appointments/
      exams/ finance/ transport/ activities/ content/
      library/ hostel/ placements/ research/ resources/
      procurement/ services/ alumni/ reporting/ shared/
    src/main/resources/
      application.yml
      db/migration/
      static/                 # Generated frontend contents; not handwritten source
    src/test/java/
  frontend/
    package.json
    package-lock.json
    src/{app,features,components,api,auth,styles}/
    tests/e2e/
    playwright.config.ts
  mobile/                     # Created during mobile phase
  contracts/openapi/
  tests/jmeter/{plans,data,README.md}
  scripts/{build,release,security,backup,restore}/
  deploy/
  docs/
```

## 4. Identity, roles, and authorization

| Role | Responsibility and scope |
|---|---|
| Super Admin | Configuration, permission administration, technical operations |
| College Admin / Principal | College oversight, final approvals, operational reports |
| HOD | Own department, faculty allocations, departmental approvals |
| Faculty | Assigned classes, attendance, marks, mentoring, own requests |
| Student | Own records, registration, payments, requests, activities |
| Guardian | Explicitly linked student's permitted records |
| HR / Office Staff | Staff records, leave, admissions according to assigned permissions |
| Accountant | Fees, receipts, expenses, reconciliation |
| Exam Officer | Exams, eligibility, marks review, publication |
| Transport Manager | Vehicles, routes, drivers, allocations |
| Driver | Own route/trips and necessary passenger information |
| Librarian | Catalog, loans, fines |
| Hostel Warden | Hostel allocations and hostel operations |
| Placement Officer | Companies, internships, placements |
| Resource / Inventory Manager | Bookings, assets, maintenance, procurement |
| Research / Quality Coordinator | Research records, surveys, accreditation evidence |
| Content Reviewer / Publisher | Assigned publication approval and audience |
| Authority Assistant | Explicitly delegated appointment/calendar management |

Support multiple roles per user, effective-dated department assignments, and permission keys such as `attendance.mark`, `marks.publish`, `leave.approve`, and `content.publish_public`. Role is not sufficient: check record ownership, department, assigned class, guardian linkage, and request stage on every API.

Default deny. No self-approval. Delegations are time-limited, scoped, and audited. Protect privileged role assignment separately. Prevent removal of the last active system administrator. Use anonymized demo role-switching only in a clearly marked isolated demo environment; never bypass production authentication.

Email/password login, email verification, invitations, password reset, logout, session expiry, rate limiting, and account suspension are required. Use adaptive password hashing, single-use expiring reset tokens, generic account recovery responses, and no credentials in logs. Optional email OTP and privileged-user MFA must be supported through a documented integration path.

Web authentication: secure HttpOnly cookie sessions with CSRF protection on mutations. Mobile: standards-based OAuth2/OIDC authorization code with PKCE using a compatible self-hosted or approved identity provider when mobile ships. Do not invent a homegrown token protocol. Validate mobile token claims server-side and use the same user/permission service. Identity-provider decision is recorded before mobile implementation; paid identity services are not required for the demo.

## 5. Complete functional scope

| Module | Required features |
|---|---|
| College setup | Departments, campuses, courses, batches, semesters, sections, subjects, rooms, calendars, holidays, policy settings |
| Admissions | Applications, document review, decisions, offers, enrollment, IDs, roll numbers, invitations |
| Students | Profiles, guardians, documents, enrollment history, promotions, transfers, graduation |
| Staff / HR | Profiles, qualifications, department assignments, workload, recruitment, onboarding, leave, exit clearance |
| Payroll | Configurable salary components, pay runs, approval, payslips, adjustments; jurisdiction-specific rules require verification before production |
| Academic registration | Electives, prerequisites, credits, subject selection, approvals, degree progress |
| Curriculum | Versioned syllabus, credits, outcomes, proposals, approval, batch mappings, lesson plans, completion |
| Timetable | Faculty/room allocation, conflict detection, substitutions, swaps, rescheduling, cancellations |
| Attendance | Subject/session student attendance, staff daily attendance, late/absent/excused, corrections, percentages, shortage alerts |
| Leave and substitution | Leave balances, date/session coverage, approvals, substitute recommendations, acknowledgment, handover, uncovered-class reports |
| Appointments | Authority calendars, booking, review, reschedule, cancellations, group/recurring meetings, notes and actions |
| Permissions | Events, trips, exam/conference participation, purchases, reimbursements, student requests, resource requests |
| Exams | Internal/semester exams, question-bank permissions, schedules, eligibility, hall tickets, marks, moderation, grades, results, backlogs, supplementary exams, revaluation |
| Learning materials | Private materials, assignments, deadlines, submissions, feedback, authorized downloads |
| Finance | Fee plans, installments, scholarships, concessions, dues, payment sandbox/live adapter, receipts, refunds, expenses, reconciliation |
| Transport | Buses, drivers, routes, stops, capacity, allocation, fees, trips, maintenance, vehicle document expiry, simulated then integrated GPS |
| Activities | Clubs, sports, cultural events, registrations, attendance, achievements, certificates |
| News / circulars | Draft/review/publish, targeted audiences, attachments, scheduling, expiry, acknowledgment, public publishing permission |
| Faculty achievements | Exams passed, certifications, qualifications, FDPs, conferences, publications, patents, grants, awards, verification |
| Library | Catalog, copies, borrowing, returns, renewals, reservations, fines |
| Hostel | Rooms/beds, capacity, allocations, fees, visitor rules, maintenance |
| Placements | Company records, eligibility, applications, interviews, offers, internships |
| Research | Projects, publications, grants, collaborators, evidence, ethics approvals when required |
| Mentoring | Adviser allocation, appointments, progress follow-up, confidential counselling access |
| Certificates | Bonafide, transcript and transfer requests, approvals, serial numbers, verification, issue/revocation |
| Resources | Room/lab/auditorium/equipment booking, approval, conflict detection |
| Procurement / assets | Vendors, requests, quotes, purchase orders, goods receipt, stock, assets, maintenance |
| Student services | Grievances, support tickets, lost-and-found, disciplinary records, clearance |
| Visitors / gate passes | Visitor appointments, approvals, entry/exit, student/staff gate passes |
| Alumni | Records, events, mentoring, engagement with consent |
| Quality | Course/faculty feedback, surveys, restricted results, accreditation evidence |
| Reports | Role dashboards, operational reports, filtered exports, audit history |

Sensitive counselling, grievance, disciplinary, payroll, and exam material require dedicated permissions, not ordinary department-wide access.

## 6. Core workflows and business rules

### 6.1 Admissions
Draft application → submitted → documents reviewed → accepted/rejected/waitlisted → offer accepted → enrolled → student identifier generated → invitation sent. Support returned-for-correction and withdrawn states. Enrollment is transactional and duplicate identifiers are prevented by database constraints.

### 6.2 Configurable approvals
Request draft → submitted → pending current stage → approved/rejected/returned → next stage or completed. Returned requests may be revised and resubmitted with history. Cancellation and authorized revocation have explicit effects.

Approval templates define type, department, amount thresholds, sequential/parallel stages, approver role, quorum, required attachments, due interval, escalation, and delegation. Snapshot the applicable template when submitted so later configuration changes do not silently change active requests. Preserve request versions, stage decisions, comments, timestamps, and actor/delegate identity.

| Request | Default route |
|---|---|
| Leave | Faculty/staff → HOD/manager → HR or Principal if policy requires |
| Event / guest lecture | Organizer → HOD → Principal |
| Industrial visit | Faculty → HOD → Principal; Finance/Transport for required clearances |
| Conference / external exam permission | Faculty → HOD → Principal |
| Purchase / reimbursement | Requester → HOD → Finance → threshold-based authority |
| Student permission | Student → adviser → HOD if required |
| Resource booking | Requester → resource manager |
| Circular / achievement | Author → reviewer → publisher |

Prevent repeated decisions and race conditions using optimistic locking and stage checks. Reminders do not equal approval. Escalation reassigns or alerts according to policy; never auto-approve silently. Deny self-approval even for users holding multiple roles. A completed approval creates its domain effect exactly once, with audit/outbox records in the same transaction.

### 6.3 Appointments and meetings
Authority publishes availability → requester selects slot, purpose, participants, attachments → pending request → accept/reject/propose new slot → confirmation → meeting completed/no-show → notes and action items.

Availability includes timezone, slot duration, buffers, office hours, holidays, blocked periods, and maximum bookings. Prevent simultaneous acceptance of overlapping bookings. A pending request must have a defined reservation expiry or be explicitly non-reserving; default is a temporary reservation with configurable expiry. Reschedules require participant confirmation. Recurring meetings support series and single-occurrence edits with finite recurrence limits.

Assistants manage only delegated calendars. Private purposes/attachments are visible only to authorized participants and approvers. Meeting minutes have separate visibility. Appointment acceptance does not approve an associated event, budget, or permission request.

### 6.4 Leave and substitute faculty
Request leave → identify affected teaching sessions → review leave → plan coverage → offer substitute → acknowledgment → publish timetable update → deliver class and mark attendance.

Recommend only available, eligible faculty within workload policy. Check leave, overlapping classes, appointments and other blocking duties. HOD can override subject eligibility only with explicit permission and reason. Store original teacher, assigned substitute, date/period, handover materials, decision history, and delivery status.

Separate leave decision from coverage status. Emergency leave may be approved with an uncovered-class warning; elective leave can require coverage according to policy. A declined or expired substitute offer returns the class to uncovered status. Substitute attendance access lasts only for the assigned session. Leave changes/cancellation trigger coverage re-evaluation and notifications without deleting history. Support swaps, reschedules, cancellation approval, and partial-day leave.

### 6.5 Attendance
Generate class session → assigned teacher/substitute marks attendance → submit → lock per policy → corrections require reason and approval. Enforce unique student/session records. Use enrolled roster effective on session date. Define scheduled/conducted/cancelled sessions and excused-absence denominator policy before percentage calculation. Do not count cancelled classes as absences. Preserve original and corrected values. Alerts are deduplicated.

### 6.6 Exams and curriculum
Exam draft → schedule → eligibility snapshot with reviewable overrides → hall tickets → marks entry → moderation/approval → publication. Students cannot access unpublished results. Lock published marks; corrections create an authorized revision. Version grading rules, rounding, credits, pass criteria, and revaluation decisions.

Curriculum proposal → academic review → approved version → effective batch mapping. Preserve existing batch versions. Distinguish faculty professional-exam achievements from student exams and curriculum proposals.

### 6.7 Finance
Assign fee plan → generate obligations → payment initiated → verified payment outcome → ledger/receipt → reconciliation. Use decimal amounts, currency codes, immutable financial entries, reversal entries, and unique receipt numbers. Never trust frontend payment success. Authenticate callbacks, verify with gateway, and handle duplicates/retries idempotently. Refunds require approval. Sandbox cannot create production payment records.

### 6.8 Content and achievements
Author creates draft and evidence → reviewer verifies → publisher approves audience/date → publish → notify → expire/archive. Faculty profile evidence verification and public achievement publication are distinct. Audience enforcement applies to APIs and files. Acknowledgment records user/time/version. Major edits to published official circulars require a new version and review. No paid advertising network is required.

### 6.9 Other operations
Transport/hostel allocation must enforce capacity concurrently. Vehicle expiry alerts do not falsely imply roadworthiness approval. Resource reservations check overlaps in a transaction. Library tracks physical copies and unique active loans. Certificates include revocation and a minimal-data verification endpoint. Procurement separates request, approval, order, receipt and payment. Academic rollover is previewable, idempotent, and does not overwrite historical records.

## 7. Data model blueprint

Use UUID identifiers, audit timestamps, explicit lifecycle states, foreign keys, indexes, unique constraints, and version columns where concurrent editing matters. Keep archived records readable under permission policy. Apply deletion/retention by record category; do not globally soft-delete everything or remove financial/audit history casually.

| Domain | Principal entities |
|---|---|
| Identity | User, Role, Permission, UserRole, DepartmentScope, Invitation, Session/IdentityLink, GuardianStudentLink |
| Organization | College, Campus, Department, AcademicYear, Term, Program, Batch, Section, Room, CalendarDay |
| People | Student, Staff, Guardian, Document, AdmissionApplication, Enrollment, StaffAssignment |
| Academics | Subject, CurriculumVersion, CurriculumSubject, CourseOffering, SubjectRegistration, LessonPlan, Assignment, Submission, AdviserAssignment |
| Scheduling | TimetableVersion, TeachingSession, FacultyAllocation, ResourceBooking |
| Attendance / leave | AttendanceRecord, AttendanceCorrection, StaffAttendance, LeaveType, LeaveBalance, LeaveRequest, CoverageAssignment |
| Approval | WorkflowTemplateVersion, Request, RequestVersion, ApprovalStage, Decision, Delegation, Escalation |
| Meetings | AvailabilityRule, CalendarBlock, Appointment, Participant, RecurrenceSeries, MeetingNote, ActionItem |
| Exams | Exam, ExamSession, EligibilitySnapshot, HallTicket, MarkEntry, GradeRuleVersion, ResultVersion, Revaluation |
| Finance | FeePlan, FeeObligation, ScholarshipApplication, LedgerEntry, PaymentAttempt, Receipt, Refund, Expense, PayrollRun, Payslip |
| Transport | Vehicle, DriverAssignment, Route, Stop, RouteStop, TransportAllocation, Trip, MaintenanceRecord |
| Content | Post, PostVersion, AudienceRule, Acknowledgment, Achievement, EvidenceVerification |
| Campus | Book, BookCopy, Loan, Reservation, Hostel, RoomBed, HostelAllocation, Asset, Vendor, PurchaseOrder, GoodsReceipt |
| Services | PlacementDrive, JobApplication, Internship, ResearchProject, Grant, CertificateRequest, Certificate, Ticket, Grievance, VisitorEntry, GatePass, AlumniProfile, Survey, SurveyResponse |
| Infrastructure | AuditEvent, Notification, OutboxEvent, DeliveryAttempt, ImportJob, ExportJob |

Files have metadata, size, MIME type, checksum, uploader, ownership, audience, storage key and retention category. Never expose raw private storage paths. Add migration-backed constraints for capacity and uniqueness where feasible; enforce overlap rules safely under concurrent transactions.

## 8. API and interface requirements

Endpoint families: `/auth`, `/users`, `/roles`, `/college`, `/departments`, `/admissions`, `/students`, `/staff`, `/curricula`, `/registrations`, `/timetables`, `/sessions`, `/attendance`, `/leave`, `/coverage`, `/appointments`, `/requests`, `/exams`, `/results`, `/fees`, `/payments`, `/transport`, `/activities`, `/posts`, `/achievements`, `/library`, `/hostel`, `/placements`, `/research`, `/resources`, `/procurement`, `/certificates`, `/tickets`, `/visitors`, `/alumni`, `/surveys`, `/reports`, `/files`, `/notifications`.

Use explicit action endpoints for submit/approve/reject/publish rather than unrestricted status editing. Example: `POST /api/v1/requests/{id}/decisions`, `POST /api/v1/leave/{id}/submit`, `POST /api/v1/coverage/{id}/acknowledge`. Define schemas and security for every endpoint in OpenAPI.

Paginate lists, allowlisted filters/sorts, consistent validation errors, UTC timestamp format, correlation IDs, and correct HTTP status codes. Use idempotency keys for payments and repeatable high-impact actions. Limit bulk imports/exports and process large work asynchronously. Version contracts without tying them to React screens.

Required shared screens: login, verification/recovery, role dashboard, global authorized search, notification inbox, my requests, approvals inbox, profile, document viewer, and settings. Feature screens need list/search/filter, detail/history, create/edit, and permitted actions. Calendar screens cover timetable, appointments, events and room bookings. Use accessible labels, keyboard navigation, readable contrast, responsive layouts, loading/empty/error states and confirmation for destructive actions. Hide inaccessible controls but still enforce permissions server-side.

## 9. Single-URL deployment and build

Required workflow:

1. Install locked frontend dependencies using `npm ci`.
2. Run frontend checks and build using `npm run build`.
3. Clean only generated frontend files in `backend/src/main/resources/static`.
4. Copy the contents of `frontend/dist/` into that static directory.
5. Package using Maven wrapper, running required backend checks.
6. Verify the JAR contains `index.html` and frontend assets.
7. Run `java -jar` with the selected environment profile.
8. Smoke-test root, direct React route refresh, APIs, permissions and missing assets.

Automate with a cross-platform script or Maven-integrated build; document manual equivalents. Never require a separate frontend server in production. Development may use Vite with `/api` proxying to Spring Boot.

Example public paths: `/` frontend; `/students` frontend route; `/api/v1/students` API. Forward only eligible browser route GET requests to `index.html`. Never forward `/api/**`, management endpoints, file downloads, or missing assets. Missing APIs/assets must return appropriate errors, not HTML. Use relative web API URLs. Apply index-page revalidation and long-lived cache headers for hashed assets.

PostgreSQL remains a separate persistent service; single URL does not mean putting the database into the JAR. Store uploads outside the JAR and public static directory. Run production behind HTTPS, typically via a reverse proxy. Restrict health/management endpoints and database network exposure. Keep external secrets outside builds. Include JAR/system-service and optional container deployment instructions, migrations, health checks, logs, graceful shutdown and restart procedures.

Release frontend changes by rebuilding the JAR. Keep previous release artifacts. Back up before migrations; test restore. Reverting an application JAR may not reverse a database migration: document schema compatibility and recovery for each migration.

## 10. Testing deliverables

### Backend
JUnit tests for business rules; integration tests against real PostgreSQL via Testcontainers. Test authorization, guardian linkage, department boundaries, expired delegation, self-approval, duplicate actions, booking collisions, substitution conflicts, attendance denominator, published-mark locks, payment retries, and transaction rollback. Do not rely exclusively on an in-memory database.

### JMeter
Deliver executable `.jmx` files for authentication, dashboard/read workload, attendance submission, approval decisions, appointment booking, result publication/read spikes, and mixed workload. Include synthetic CSV fixtures, setup/cleanup instructions, environment properties, correlation/token extraction, CSRF handling where needed, assertions, think time, ramp-up, duration, and command-line execution instructions. GUI is for authoring; load execution is non-GUI. Keep heavy listeners disabled. Credentials are injected, not committed.

Example command contract: `jmeter -n -t tests/jmeter/plans/mixed-workload.jmx -JbaseUrl=http://localhost:8080 -Jusers=50 -Jduration=300 -l results.jtl -e -o report` (use a fresh output directory).

Initial demo targets, subject to documented hardware/workload approval: 50 concurrent virtual users, under 1% unexpected errors, p95 ordinary reads under 1 second and ordinary writes under 2 seconds. Exclude intentionally long exports from these thresholds and track them separately. Production capacity is determined by realistic tests, not these initial numbers. Run only against an authorized dedicated environment; never simulate load against live payment/email providers.

### Playwright
Deliver `playwright.config.ts`, fixtures, role accounts and `.spec.ts` files for login/recovery, role restrictions, admissions, attendance, leave/coverage, appointments, permission approval, exams/results, fees/sandbox payment, transport allocation, circulars/achievements and later modules as implemented. Test success, rejection, returned requests, and important conflicts. Test the packaged JAR, not only the development frontend.

Use isolated synthetic records, stable accessible locators, auto-waiting assertions, and screenshots/traces on failure. Include desktop Chromium, Firefox/WebKit smoke coverage and mobile viewport coverage. A mobile browser viewport does not replace native React Native tests.

### Release evidence
Store test commands, execution results, performance environment, security reports and failures with the release. No fabricated passing reports. A failed or unavailable required check is recorded as unresolved, not silently skipped.

## 11. Security and release gate

Automate source analysis for Java/TypeScript, Maven/npm dependency scanning, secret scanning, SBOM generation, container/infrastructure scans when present, and ZAP testing against demo/staging. Use maintained compatible tools; document scanner rules, advisory feed availability and suppressions. OWASP Dependency-Check is an option for Java dependencies; use a suitable npm scanner too. Local/open-source checks must be available without mandatory paid subscriptions.

Review all source modules, build scripts and configuration, not only recently edited files. Include manual review of access control, sensitive exports, uploads, payments, approval authority, injection, XSS, CSRF, path traversal, session handling and rate limits. Validate content type and size, restrict upload extensions, quarantine/scan when appropriate, sanitize displayed content, and prevent public access to private files. Redact logs and exports. Define retention and consent rules with the institution.

Release checklist:

- [ ] All phase acceptance criteria and required tests pass.
- [ ] Scans are current and completed; feeds and exclusions are recorded.
- [ ] No unresolved critical/high finding, except documented false-positive assessment or owner-approved time-limited risk exception.
- [ ] No committed secrets; production configuration validated.
- [ ] Authorization checks and payment callback verification tested.
- [ ] Performance targets met for documented environment.
- [ ] Backup restoration and migration/rollback strategy validated.
- [ ] Demo/sandbox credentials and test shortcuts absent from production.
- [ ] Release notes, support instructions and monitoring prepared.
- [ ] Owner approves production release and any costs.

Automated scanners cannot guarantee zero vulnerabilities. Continue dependency monitoring and patching after launch. Production payroll/legal/privacy rules must be verified for the applicable institution and jurisdiction.

## 12. Demo-first providers and paid transition

| Capability | Demo | Production after approval |
|---|---|---|
| Hosting | Local/demo server | Approved hosting/server |
| Database | Local PostgreSQL | Managed or operated PostgreSQL |
| Email | Local mail capture | Verified transactional email service |
| Payments | Mock adapter/gateway sandbox | Approved live gateway |
| Documents | Private persistent local directory | Private persistent storage/object storage |
| SMS/WhatsApp | Simulated delivery | Approved provider/templates |
| GPS | Clearly labeled simulated positions | Hardware/provider integration |
| Monitoring | Local logs/metrics | Operational monitoring and alerting |
| Identity/mobile | Local supported identity setup when needed | Approved self-hosted/managed provider |

Use interfaces for providers and explicit `demo`, `test`, `staging`, `prod` configuration. Simulations must show their status and never report real delivery/payment/location. Protect production startup from mock payment, demo login bypass, default passwords and publicly available seed accounts. Seed data is opt-in and forbidden in production.

Create `docs/PROVIDER_DECISIONS.md` listing purpose, alternatives, demo behavior, current verified free/trial limits, expiry, region, data handling, cost basis, required credentials, and migration steps. Verify current provider pricing before selection; do not promise a free tier will remain available. No paid dependency is mandatory to demonstrate core workflows. Owner approval is required for purchases, live account activation and real charges.

Transition: complete demo → user acceptance → budget/provider approval → production setup → sandbox-to-live integration validation → data migration rehearsal → release gate → launch. Do not carry synthetic financial/student data into production.

## 13. React Native roadmap

Keep backend business rules independent of UI and document all mobile-consumable APIs now. Mobile uses the same `/api/v1` services and database. Share generated API types/client contracts and compatible validation utilities; React DOM components are not directly reusable as native screens.

Mobile phase: authentication/secure OS credential storage, dashboard, timetable, attendance view, requests/approvals, appointments, notices, results, fee status, document uploads and transport information. Add native push registration, deep links, permission prompts, app version compatibility, accessibility and network-error handling. Offline writes require an explicit conflict/idempotency design; do not assume automatic synchronization. Add native test coverage and device acceptance tests. Store publishing accounts and paid push/provider decisions require approval.

## 14. Delivery phases and acceptance

| Phase | Deliverables | Acceptance evidence |
|---|---|---|
| 0 | Repository, pinned stack, schema blueprint, UI shell, build scripts, test scaffolding | Fresh setup and packaged single-URL smoke test |
| 1 | College setup, identity, roles, audit, notifications, approval engine | Permissions and self-approval tests; invitation/login; request stage flow |
| 2 | Admissions, students, staff, timetable, attendance, leave, substitutes, appointments | End-to-end admission; absent faculty coverage; concurrent booking conflict tests |
| 3 | Curriculum, registration, materials, exams, results, fees, scholarships | Approved result publication; versioned curriculum; sandbox payment reconciliation |
| 4 | Transport, activities, events/permissions, circulars, achievements, library, resource bookings | Capacity checks; approval-to-publication flow; loans and booking conflicts |
| 5 | Hostel, payroll, procurement/assets, placements, research, mentoring, certificates, services, visitors, alumni, quality | Complete workflows with scoped permissions and module tests |
| 6 | Full demo acceptance, imports/rollover/reports, security/performance hardening, production runbooks | Requirement matrix complete; release evidence; restore rehearsal |
| 7 | React Native app on shared backend | Native role workflows, secure authentication, device testing |

Security, tests, responsive UI, audit and documentation are continuous requirements, not work deferred entirely to phase 6. Every phase ends with a runnable demo and a list of remaining scope. Native app can follow web approval while API compatibility remains mandatory throughout.

## 15. Final deliverables and definition of done

- Complete Java/React source and reproducible locked builds.
- PostgreSQL migrations, indexes, synthetic seeds and import templates.
- Role/permission matrix and documented institutional policies.
- OpenAPI contract, architecture notes and domain state machines.
- JUnit/integration tests, JMeter plans/data, Playwright tests/configuration.
- Security scripts/reports, SBOM, release checklist and exception register.
- Single-JAR build, deployment, backup, restore and upgrade instructions.
- Provider decision/cost document and demo-to-production runbook.
- User/admin operating guides and requirement traceability matrix.
- React Native implementation and native tests in its scheduled phase.

A module is done only when its workflow works through the UI and API, permissions are enforced server-side, migrations are reproducible, meaningful positive/negative tests pass, and the documentation describes actual behavior. A page with buttons or mocked responses alone is not implementation.

## 16. Owner decisions to gather during implementation

Do not block project foundation on these, but resolve each before its affected workflow goes live: actual college hierarchy; departments/programs; grading/promotion/attendance policies; leave and coverage policy; approval thresholds; expected population/concurrency; retention/guardian-access policies; payroll rules; existing data quality; production domain/hosting/budget; selected providers; public publication rules; mobile platform priorities.

## 17. Official references

Verify current versions and behavior when implementing:

- Spring Boot: https://docs.spring.io/spring-boot/reference/
- Spring Security: https://docs.spring.io/spring-security/reference/
- React: https://react.dev/
- Vite: https://vite.dev/guide/
- PostgreSQL: https://www.postgresql.org/docs/
- JMeter: https://jmeter.apache.org/usermanual/best-practices.html
- Playwright: https://playwright.dev/docs/writing-tests
- OWASP Dependency-Check: https://owasp.org/projects/dependency-check
- OWASP security guidance: https://owasp.org/secure-my-app
- React Native: https://reactnative.dev/docs/getting-started

## 18. Suggested first instruction to Claude

> Read PLAN.md completely. Inspect the existing repository and create a requirement checklist. Start with Phase 0 and Phase 1 using Java Spring Boot, React TypeScript/Vite and PostgreSQL. Implement the single-JAR frontend packaging workflow, secure identity and permissions, migrations, approval-engine foundation and working tests. Keep all later agreed modules tracked. Do not purchase providers or activate production services. After each phase, show the runnable result, actual test evidence and remaining work, then continue through the approved scope.
