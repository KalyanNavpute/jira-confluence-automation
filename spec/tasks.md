# Implementation Tasks: Weekly Executive Status Reporting

**Status**: Proposed; implementation tasks are gated on Phase 0 approval  
**Plan**: `spec/plan.md`  
**Specification**: `spec/specification.md`  
**Constitution**: `spec/constitution.md`

## Execution Rules

- Complete Phase 0 and obtain the M0 approval before starting implementation tasks T011 and later.
- Tasks marked conditional are performed only if their scope is approved in Phase 0.
- A task is complete only when all acceptance criteria pass and relevant checks are recorded.
- Preserve the constitution's stack and boundaries: React 18/Vite, Node.js/Express, PostgreSQL 15 via Docker, server-side Jira/Confluence integration, backend authorization, and versioned migrations.
- Do not implement Confluence or a separate CLI merely because they appear in unresolved decisions. Add tasks only if Phase 0 approves that scope.

## Phase 0: Scope and Requirements (Milestone M0)

### T001: Resolve product and integration scope

**Status**: Complete (decision D-001 recorded in the specification)  
**Dependencies**: None  
**Traceability**: C-01 (resolved as D-001)

**Acceptance criteria**:

- The product owner records whether this reporting feature is Jira-only within a broader Jira/Confluence application or includes a defined Confluence workflow.
- Any approved Confluence workflow has user scenarios, data boundaries, and acceptance criteria; otherwise, the MVP explicitly excludes it.
- `spec/specification.md` and `spec/plan.md` agree on the resulting scope.

### T002: Decide supported user interfaces

**Dependencies**: None  
**Traceability**: C-02, OD-001

**Acceptance criteria**:

- The required interface is recorded as browser, CLI, or both.
- If CLI support is approved, its operations and relationship to the Node.js backend are specified.
- Any decision to retain a Python CLI includes an explicit constitutional exception; no separate CLI is implemented by default.
- The specification, plan, and acceptance scenarios reflect the decision.

### T003: Define report trigger and refresh responsibilities

**Dependencies**: None  
**Traceability**: C-03, G-01, G-14, OD-006

**Acceptance criteria**:

- The weekly report trigger is defined as manual, scheduled, or both, including behavior when required inputs are missing.
- The sources covered by daily refresh are explicitly listed.
- The business timezone, schedule, weekend/holiday behavior, and whether manual generation remains available are recorded.
- The specification distinguishes refresh cadence from report-generation cadence.

### T004: Define Jira query and reporting-period semantics

**Dependencies**: T001, T003  
**Traceability**: G-02, G-05, G-06, OD-003

**Acceptance criteria**:

- The initial supported Jira filter mechanism and required issue fields are documented.
- Status mappings, due-date/overdue rules, blocker conventions, milestone data, and dependency handling are specified or explicitly deferred.
- The specification states whether reports use current status, period-end status, or changes during the reporting period.
- The required Jira history/snapshot retention behavior is testable from the documented rules.

### T005: Define manual input and completeness rules

**Dependencies**: None  
**Traceability**: G-03, G-08, FR-004 to FR-008

**Acceptance criteria**:

- Every update field is classified as required or optional.
- The specification defines how a contributor records “none,” leaves a field unanswered, or indicates unavailable/not applicable.
- Late, duplicate, corrected, and closed-period updates have explicit expected behavior.
- Update validation and report-readiness criteria are consistent.

### T006: Define report content ownership and conflict rules

**Dependencies**: T004, T005  
**Traceability**: G-04, G-06, FR-013, FR-014, FR-017

**Acceptance criteria**:

- Each report field is identified as manual, Jira-derived, or system-assembled.
- The rules for conflicting Jira and manual content are documented.
- Any calculations or summarization behavior is deterministic and has examples; AI-generated content is not assumed.
- Source attribution requirements are specific enough to verify in report tests.

### T007: Approve identity, roles, and contributor model

**Dependencies**: T001  
**Traceability**: G-10, G-11, NFR-005, OD-004, OD-009

**Acceptance criteria**:

- Authentication method and authorized roles are recorded.
- A role/action matrix covers project configuration, contributor updates, Jira refresh visibility, report generation/editing, and retrieval.
- The source of expected contributor/team membership and project access boundaries are defined.
- Jira authentication method, minimum scopes, and applicable data-handling policy are recorded.

### T008: Approve report artifact lifecycle

**Dependencies**: None  
**Traceability**: G-09, FR-018 to FR-020, NFR-010, OD-008

**Acceptance criteria**:

- The authoritative report storage location and download/export behavior are defined.
- Draft/final state, edit permissions, regeneration behavior, and version/replacement rules are specified.
- Required report metadata and retention period are recorded.
- Requirements prevent silent loss of manager edits and prior report versions.

### T009: Set operational, performance, and release criteria

**Dependencies**: T003  
**Traceability**: G-12 to G-16, NFR-002, OD-006, OD-008

**Acceptance criteria**:

- The under-10-minute target has a measurement boundary and representative team/Jira data volume.
- Required browser support, accessibility expectations, deployment boundary, and operational recovery requirements are documented.
- Required security, test, migration, backup/restore, and release checks are identified.
- Implementation-specific choices remain assigned to the technical design where they are not product constraints.

### T010: Update and approve the MVP contract

**Dependencies**: T001 to T009  
**Traceability**: M0 exit criteria

**Acceptance criteria**:

- `spec/specification.md` incorporates all approved decisions and contains no contradictory MVP requirements.
- `spec/plan.md` reflects the approved scope, interface, schedule, and acceptance gates.
- `spec/clarify.md` marks resolved questions as decided and retains only genuinely open items.
- Product and technical owners approve the updated specification; M0 is recorded complete.

## Phase 1: Application Foundation (Milestone M1)

### T011: Confirm runtime and repository conventions

**Dependencies**: T010  
**Traceability**: Phase 1; NFR-007, NFR-009

**Acceptance criteria**:

- Supported Node.js version, package manager, repository layout, and standard scripts are documented.
- The choices are compatible with the constitution and approved interface scope.
- Clean-checkout setup commands are identified for implementation and CI.

### T012: Scaffold the React frontend

**Dependencies**: T011  
**Traceability**: Constitution Technical Constraints; NFR-001, NFR-009

**Acceptance criteria**:

- A React 18/Vite frontend starts and builds using documented commands.
- Environment-specific API configuration is not hard-coded into source.
- The initial application shell handles loading and backend-unavailable states.
- Frontend build checks pass.

### T013: Scaffold the Express backend

**Dependencies**: T011  
**Traceability**: Constitution Technical Constraints; NFR-007, NFR-009

**Acceptance criteria**:

- A Node.js/Express service starts using documented commands.
- Configuration is read from environment without exposing secrets in logs or responses.
- A consistent client-safe error response format is established.
- Backend checks pass.

### T014: Add Docker PostgreSQL and migration tooling

**Dependencies**: T011  
**Traceability**: Constitution Core Principles 5 and 7; NFR-009

**Acceptance criteria**:

- PostgreSQL 15 can be started locally using the documented Docker workflow.
- Database credentials are supplied outside committed source and a safe example configuration is available.
- A versioned migration can be applied to a clean database and migration state can be inspected.
- Database startup and migration checks pass.

### T015: Connect application services and health checks

**Dependencies**: T012, T013, T014  
**Traceability**: Phase 1; NFR-003, NFR-009

**Acceptance criteria**:

- The frontend can call a backend endpoint and the backend can connect to PostgreSQL.
- Health/readiness responses distinguish service availability from database readiness.
- No frontend-to-database or privileged frontend-to-Jira connection exists.
- A service/database integration check passes.

### T016: Add foundation CI checks and setup documentation

**Dependencies**: T012, T013, T014, T015  
**Traceability**: Constitution Core Principles 6 and 7; M1 exit criteria

**Acceptance criteria**:

- Automated checks run frontend build, backend checks, and database migration/integration checks.
- Startup, shutdown, clean reset, and test commands are documented and verified from a clean checkout.
- M1 exit criteria in `spec/plan.md` pass.

## Phase 2: Project and Manual Updates (Milestone M2)

### T017: Implement project configuration

**Dependencies**: T016, T007  
**Traceability**: FR-001, FR-003

**Acceptance criteria**:

- An authorized manager can create, view, and update project name, owner/team, and approved Jira filter configuration.
- Invalid or unauthorized changes are rejected with actionable, non-sensitive errors.
- API and persistence tests cover valid, invalid, and unauthorized operations.

### T018: Implement reporting-period lifecycle

**Dependencies**: T016, T005  
**Traceability**: FR-002; Scenario 1

**Acceptance criteria**:

- A reporting period stores explicit start/end dates and follows the approved lifecycle states.
- Invalid date ranges and disallowed edits after period close are rejected.
- Period boundaries use the approved timezone/date rules.
- Lifecycle and boundary tests pass.

### T019: Implement expected contributor/team configuration

**Dependencies**: T017, T007  
**Traceability**: FR-003; G-10

**Acceptance criteria**:

- Authorized managers can configure expected contributors or teams using the approved membership source.
- The system identifies the expected update unit for a project and period.
- Duplicate membership and membership changes follow documented rules.
- Authorization and persistence tests pass.

### T020: Implement manual update submission and correction

**Dependencies**: T018, T019, T005  
**Traceability**: FR-004, FR-005, FR-007, FR-008; Scenario 1

**Acceptance criteria**:

- An authorized contributor can submit an update for an eligible project and reporting period.
- The approved fields, author, and created/updated timestamps are persisted.
- Corrections follow the approved edit/close-period rules and do not silently lose prior saved content.
- API and database tests cover create, read, correction, invalid input, and unauthorized access.

### T021: Implement update completeness and missing-input view

**Dependencies**: T019, T020, T005  
**Traceability**: FR-006, FR-008; Scenario 1

**Acceptance criteria**:

- Managers can distinguish received, missing, incomplete, explicitly empty, and other approved states.
- Validation feedback identifies the fields that prevent completion.
- The view does not represent missing data as “no risks” or “no work.”
- UI and API tests cover every approved completeness state.

### T022: Verify manual workflow authorization and integration

**Dependencies**: T017 to T021  
**Traceability**: NFR-005; M2 exit criteria

**Acceptance criteria**:

- Backend role checks are tested for project setup, update submission, correction, and completeness visibility.
- End-to-end tests cover contributor submission and manager review for one project and period.
- Scenario 1 and M2 exit criteria pass.

## Phase 3: Jira Integration and Refresh (Milestone M3)

### T023: Implement Jira credential and connection configuration

**Dependencies**: T016, T007  
**Traceability**: FR-009; NFR-004, NFR-006

**Acceptance criteria**:

- The backend connects using the approved Jira authentication method and least-privilege scopes.
- Credentials are not returned to the frontend, committed, or written to logs.
- Authentication failures produce a non-sensitive refresh status.
- Connector tests use controlled fixtures for routine checks.

### T024: Implement Jira query, pagination, and field mapping

**Dependencies**: T023, T004  
**Traceability**: FR-009, FR-012

**Acceptance criteria**:

- The approved project/filter configuration selects the expected Jira issues.
- All approved fields are mapped, including configured status, blocker, due-date, milestone, and dependency data.
- Paginated responses are fully processed or the refresh is marked incomplete.
- Tests cover mapping, missing/unavailable fields, empty results, and multi-page responses.

### T025: Persist Jira refresh records and reporting-period history

**Dependencies**: T014, T024, T004  
**Traceability**: FR-010, FR-011; G-02

**Acceptance criteria**:

- Each refresh records start/completion time, outcome, filter, and non-sensitive failure information.
- Issue state/history is persisted according to the approved temporal semantics and retention rules.
- A test can derive the required Jira facts for a reporting period from saved records.
- Migrations and tests protect referential integrity and prevent accidental history overwrite.

### T026: Implement refresh execution, retry, and idempotency behavior

**Dependencies**: T023, T024, T025, T003  
**Traceability**: FR-010, FR-012; Constitution Core Principle 4

**Acceptance criteria**:

- A refresh can be invoked by its approved trigger and records a terminal success, partial, or failure outcome.
- Retry limits and handling for authentication, rate-limit, timeout, and transient errors follow approved rules.
- Repeating a refresh does not duplicate or corrupt stored issue history.
- Tests cover retry exhaustion, repeat invocation, and partial failure.

### T027: Schedule daily refresh and support recovery

**Dependencies**: T026, T003  
**Traceability**: FR-010, OD-006, G-14

**Acceptance criteria**:

- Refresh runs on the approved schedule and timezone.
- Overlapping runs, missed schedules, restart recovery, and manual retry follow documented behavior.
- Scheduler failures are visible to the designated role/operator.
- Schedule behavior is tested without waiting for a real-time daily run.

### T028: Display Jira freshness and refresh status

**Dependencies**: T025, T026, T007  
**Traceability**: Scenario 2; FR-011, FR-021

**Acceptance criteria**:

- Authorized managers can inspect last refresh time and current/partial/stale/failed status.
- Error details are actionable but do not reveal credentials or sensitive response data.
- Status uses the approved freshness threshold and policy.
- UI/API tests cover each status state.

### T029: Verify Jira integration behavior

**Dependencies**: T023 to T028  
**Traceability**: Scenario 2; FR-009 to FR-014; M3 exit criteria

**Acceptance criteria**:

- Deterministic tests cover pagination, rate limits, authentication failure, stale/partial data, and historical status behavior.
- No routine test requires live Jira credentials.
- Scenario 2 and M3 exit criteria pass.

## Phase 4: Report Generation and Artifact Lifecycle (Milestone M4)

### T030: Implement report readiness and input policy

**Dependencies**: T021, T028, T004, T005  
**Traceability**: FR-017, FR-021; G-03, G-07, G-08

**Acceptance criteria**:

- Readiness evaluates required manual inputs and Jira state using the approved rules.
- Missing, explicitly empty, unavailable, stale, and partial inputs remain distinguishable.
- The system blocks or permits generation exactly as the approved policy specifies.
- Tests cover each readiness outcome.

### T031: Implement deterministic report aggregation

**Dependencies**: T025, T020, T006, T030  
**Traceability**: FR-013, FR-014, FR-017; SC-003

**Acceptance criteria**:

- Jira facts and manual content are assigned to report sections according to approved ownership rules.
- Source conflicts and attribution follow documented behavior.
- Required calculations, counts, milestones, dependencies, and status history use approved rules.
- Tests verify representative normal, conflict, empty, and incomplete input sets.

### T032: Implement Markdown rendering and format validation

**Dependencies**: T031, T006  
**Traceability**: FR-015, FR-016, FR-017; SC-002

**Acceptance criteria**:

- Output matches the approved Markdown structure and metadata/date formats.
- All required sections appear, including approved wording for empty or incomplete sections.
- Jira references and source attribution use the approved format.
- Golden/fixture tests verify exact structure and representative content.

### T033: Implement report persistence and version rules

**Dependencies**: T014, T008, T032  
**Traceability**: FR-019, FR-020; NFR-010

**Acceptance criteria**:

- Reports persist in the approved authoritative location with project, period, version, generation time, freshness, and source metadata.
- Create, replacement, and version behavior follows the approved lifecycle.
- Prior content and manager edits cannot be silently overwritten.
- Database constraints and tests verify retrieval and version integrity.

### T034: Implement manager review and editing

**Dependencies**: T032, T033, T007  
**Traceability**: FR-018; Scenario 3

**Acceptance criteria**:

- An authorized manager can inspect and edit generated Markdown before finalization.
- Save, cancel, and stale-edit/concurrent-edit behavior follow approved rules.
- Saved content is returned unchanged on subsequent retrieval.
- UI/API tests cover edit authorization, validation, and persistence.

### T035: Verify report generation and lifecycle

**Dependencies**: T030 to T034  
**Traceability**: Scenario 3; FR-015 to FR-021; M4 exit criteria

**Acceptance criteria**:

- Deterministic tests verify report sections, attribution, incomplete-data handling, edits, and regeneration/version behavior.
- Missing content is never fabricated.
- Scenario 3 and M4 exit criteria pass against representative fixtures.

## Phase 5: History and End-to-End Workflow (Milestone M5)

### T036: Implement report history search and retrieval

**Dependencies**: T033, T007  
**Traceability**: Scenario 4; FR-019; SC-005

**Acceptance criteria**:

- An authorized manager can find reports by project and reporting period.
- The result identifies the selected version and its generation/freshness metadata.
- No matching report produces an explicit empty state.
- API, database, and UI tests cover retrieval and empty results.

### T037: Implement approved Markdown export

**Dependencies**: T008, T033, T036  
**Traceability**: Scenario 4; OD-008

**Acceptance criteria**:

- If export is approved, an authorized manager can download a valid `.md` artifact for the selected report version.
- Exported content matches the persisted version and uses a safe filename.
- If export is not approved, the task is marked not applicable and no export surface is added.

### T038: Complete end-to-end manager workflow states

**Dependencies**: T021, T028, T034, T036  
**Traceability**: NFR-001, NFR-003; Scenarios 1 to 4

**Acceptance criteria**:

- The browser workflow communicates loading, empty, validation, stale/partial, failure, and success states.
- Managers can navigate from input readiness through generation/review to prior report retrieval.
- The interface does not imply that missing data is confirmed empty.
- Usability checks pass for the approved supported browser set.

### T039: Run interface-level acceptance tests

**Dependencies**: T029, T035, T036, T038  
**Traceability**: M5 exit criteria; SC-001, SC-006

**Acceptance criteria**:

- End-to-end tests cover manual update, refresh status, report generation/edit, and retrieval using controlled fixtures.
- The approved browser and/or CLI interface is covered; no unapproved interface is required for a pass.
- The workflow is repeatable across two reporting periods without custom code changes.
- Product owner accepts the M5 scenarios and the performance measurement is recorded.

## Phase 6: Hardening and Release Readiness (Milestone M6)

### T040: Complete security and privacy review

**Dependencies**: T022, T029, T035, T039  
**Traceability**: Constitution Core Principle 3; NFR-004 to NFR-006

**Acceptance criteria**:

- Backend authorization is verified for every protected operation in the approved role/action matrix.
- Secret handling, Jira scopes, input validation, and sensitive logging are reviewed against approved policy.
- No credentials or restricted Jira content appear in browser bundles, client errors, or logs.
- Findings are fixed or recorded as explicitly accepted risks.

### T041: Verify operational failure and recovery paths

**Dependencies**: T027, T035  
**Traceability**: NFR-003; G-14

**Acceptance criteria**:

- Tests cover Jira outage/rate-limit behavior, database unavailability, restart, retry, and recovery according to approved policy.
- Failed or incomplete operations remain visible and do not create falsely successful reports.
- Recovery procedures are exercised and documented.

### T042: Validate migrations, retention, and restore behavior

**Dependencies**: T014, T025, T033, T008  
**Traceability**: Constitution Core Principles 5 and 7; NFR-010

**Acceptance criteria**:

- Migrations apply safely to a clean database and the supported upgrade path.
- Approved retention behavior preserves or removes reports and issue history as specified.
- Backup/restore expectations are tested for the supported environment.
- Restored report versions and source metadata pass integrity checks.

### T043: Validate performance and accessibility requirements

**Dependencies**: T039, T009  
**Traceability**: NFR-001, NFR-002; SC-001

**Acceptance criteria**:

- The approved performance scenario runs with representative project, contributor, and Jira issue volumes.
- Timing boundaries and results are recorded, and the approved target is met or a mitigation is approved.
- Supported-browser and accessibility checks meet the agreed criteria.

### T044: Finalize operational and user documentation

**Dependencies**: T041, T042  
**Traceability**: Constitution Core Principle 7; M6 exit criteria

**Acceptance criteria**:

- Setup, configuration, Jira connection, schedule, report workflow, troubleshooting, recovery, and retention instructions match the implemented product.
- Documentation contains no real secrets and includes safe example configuration.
- A reviewer can follow the documented clean setup and recovery procedure.

### T045: Release readiness review and sign-off

**Dependencies**: T040 to T044  
**Traceability**: M6 exit criteria; Constitution Core Principle 6

**Acceptance criteria**:

- All approved acceptance, integration, security, migration, performance, and accessibility gates pass.
- No blocking clarification remains; residual risks and deferred scope are documented.
- Product and technical owners approve release readiness.
- M6 is recorded complete.

## Dependency Summary

```text
T001-T009 -> T010 (M0 approval)
T010 -> T011-T016 (M1 foundation)
T016 -> T017-T022 (M2 manual workflow)
T016 -> T023-T029 (M3 Jira integration; may proceed alongside M2 after contracts stabilize)
T022 + T029 -> T030-T035 (M4 report generation)
T035 -> T036-T039 (M5 end-to-end acceptance)
T039 -> T040-T045 (M6 hardening and release)
```

## Scope Change Rule

If Phase 0 approves Confluence integration, a separate CLI, or another interface not represented in the MVP tasks, add a scoped task group with its own user scenarios, dependencies, tests, and acceptance criteria before implementation begins.