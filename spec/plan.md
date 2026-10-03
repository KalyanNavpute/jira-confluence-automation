# Implementation Plan: Weekly Executive Status Reporting

**Status**: Proposed; implementation is gated on Phase 0 decisions  
**Inputs**: `spec/constitution.md`, `spec/specification.md`, `spec/clarify.md`  
**Delivery approach**: Incremental vertical slices with independently verifiable milestones

## 1. Plan Summary

Deliver a browser-based weekly reporting application using React 18 with Vite, Node.js with Express, and PostgreSQL 15 via Docker. The planned MVP collects manual team updates, refreshes configured Jira data daily, creates and persists reviewable Markdown reports, and lets managers retrieve prior reports.

This plan does not silently decide unresolved product questions. Phase 0 must resolve the blocking conflicts in `spec/clarify.md` before implementation scope is committed. Until then, the phases below are a dependency-ordered proposal; Confluence integration and a separate CLI are not assumed to be MVP deliverables.

## 2. Planning Assumptions and Gates

The following are provisional planning assumptions, not approved requirements:

- The manager-facing browser workflow is the primary interface; a separate CLI is not included unless OD-001 is resolved in its favor.
- Jira is the only external data integration in the reporting MVP; Confluence remains out of scope until C-01/OD-007 is resolved.
- Managers trigger report generation; daily Jira refresh is a separate recurring operation, pending C-03 confirmation.
- Jira integration, credentials, and authorization remain server-side.
- Data model and implementation details will follow approved reporting semantics, freshness policy, artifact lifecycle, access roles, and retention decisions.

**Do not pass Phase 0** while C-01, C-02, or C-03 remain unresolved, or while the decisions in G-02, G-03, G-04, G-07, G-09, and G-11 are too ambiguous to define the MVP contract.

## 3. Phases and Milestones

### Phase 0: Resolve Scope and Freeze MVP Requirements

**Purpose**: Convert the draft specification and review findings into an implementable, testable MVP contract.

**Work**:

- Resolve C-01: confirm whether this feature is Jira-only within a broader Jira/Confluence product, or define Confluence workflows for this release.
- Resolve C-02: choose browser-only, CLI-only under an approved architecture exception, or browser plus CLI.
- Resolve C-03 and G-01: decide how reports are triggered and exactly which data is refreshed daily.
- Decide Jira reporting-period semantics, field/status mappings, and the stale/partial-data policy (G-02, G-05, G-07).
- Enumerate required versus optional manual fields; define explicit “none,” missing, unavailable, and not-applicable states (G-03, G-08).
- Define content ownership and source-conflict behavior, including whether summaries are formatted, calculated, or manually authored (G-04, G-06).
- Define authentication, role permissions, contributor roster, artifact storage/versioning/export, and retention (G-09 to G-11).
- Specify timezone, schedule behavior, performance measurement boundary, and required test/release gates (G-12 to G-16 as applicable).
- Update `spec/specification.md` and, if necessary, `spec/constitution.md`; remove resolved questions from `spec/clarify.md` or mark them decided.

**Milestone M0: Approved MVP Contract**

- Blocking decisions are recorded and reflected in the specification.
- Every MVP requirement has an observable acceptance criterion.
- Scope, interfaces, user roles, source data semantics, and report lifecycle are agreed.
- A feature plan can be created without relying on contradictory assumptions.

**Exit gate**: Product/technical owner approval of the updated specification. No production implementation begins before this milestone.

### Phase 1: Application Foundation and Reproducible Development

**Purpose**: Establish the approved runtime structure and a working end-to-end application skeleton.

**Work**:

- Confirm the supported Node.js version, package manager, repository layout, and local run/test/build commands.
- Set up the React 18/Vite frontend and Node.js/Express backend according to the approved interface decision.
- Add Docker-based PostgreSQL 15 development setup, environment configuration examples, and secret exclusions.
- Establish versioned database migrations and a minimal initial schema.
- Add frontend-to-backend connectivity, backend health/readiness checks, and consistent non-sensitive error handling.
- Add automated checks for frontend build, backend checks, migrations, and database-backed tests.
- Document startup, shutdown, reset, and test procedures.

**Milestone M1: Reproducible Application Skeleton**

- A developer can start the approved services and PostgreSQL 15 from documented commands.
- The frontend reaches the backend; the backend can connect to PostgreSQL.
- Database migrations run from a clean database and are repeatable.
- Secrets are not committed or exposed to browser code.
- Automated build and basic service/database checks pass.

**Exit gate**: Constitution stack and boundary checks pass; environment setup is reproducible from a clean checkout.

### Phase 2: Project Configuration and Manual Update Workflow

**Purpose**: Support the human data-collection workflow and its completeness states.

**Work**:

- Implement project/reporting-period configuration and the approved contributor/team model.
- Implement role-protected manual update creation, retrieval, and correction.
- Validate the approved required/optional fields and preserve author and timestamps.
- Represent submitted, missing, explicitly empty, and other approved update states distinctly.
- Show managers expected versus received update status and actionable missing-input feedback.
- Add backend authorization and persistence constraints based on the Phase 0 role matrix.

**Milestone M2: Manual Inputs Ready**

- An authorized contributor can submit and correct an update for the correct project and period.
- A manager can identify missing submissions and inspect update completeness.
- Empty answers are not confused with missing answers.
- Validation, authorization, API, and database persistence tests pass.

**Exit gate**: Phase 0-defined input rules and the Scenario 1/FR-003 to FR-008 acceptance criteria pass.

### Phase 3: Jira Integration and Daily Refresh

**Purpose**: Import reliable Jira data with traceable freshness and failure states.

**Work**:

- Implement a backend-only Jira connector using the approved authentication method and least-privilege scopes.
- Support the approved project/filter configuration, fields, status mappings, blocker rules, and pagination behavior.
- Implement bounded retries for approved transient/rate-limit errors and record refresh start, completion, and outcome.
- Implement the approved daily schedule, timezone, overlap handling, and recovery/manual-retry behavior.
- Persist the approved issue history/snapshot model needed for reporting-period semantics.
- Expose refresh status and non-sensitive error details to authorized managers.
- Add deterministic mocked Jira tests for pagination, rate limits, authentication failure, partial refresh, stale data, and historical status behavior.

**Milestone M3: Jira Data Ready**

- A configured refresh imports all expected pages or records a clear incomplete result.
- The system reports last successful refresh and accurately distinguishes current, stale, failed, and partial data per policy.
- Required historical facts for a reporting period can be derived and tested.
- No routine automated test requires live Jira credentials; no secrets or sensitive details leak to client/logs.

**Exit gate**: Scenario 2 and FR-009 to FR-014 acceptance criteria pass, including the Phase 0 freshness and temporal semantics decisions.

### Phase 4: Report Assembly, Editing, and Versioned Persistence

**Purpose**: Produce a deterministic report from approved inputs and preserve the review lifecycle.

**Work**:

- Implement report readiness checks using the agreed rules for required, missing, empty, stale, and unavailable inputs.
- Aggregate Jira facts and manual updates according to approved ownership, conflict, and attribution rules.
- Generate the approved Markdown structure, including required metadata, sections, date formatting, and empty/incomplete-state wording.
- Support manager review and editing, with explicit draft/final behavior if adopted in Phase 0.
- Implement the approved artifact storage, versioning/replacement, retrieval, and export behavior.
- Preserve the source/report metadata needed to understand which inputs and refresh state informed each version.
- Test deterministic generation, section completeness, content attribution, edits, regeneration, version preservation, and failure paths.

**Milestone M4: Report Workflow Complete**

- A manager can generate a report for a selected project and period from valid test inputs.
- The output meets the approved Markdown contract and does not fabricate missing content.
- Stale, partial, or missing data is handled exactly as specified.
- Editing and saving works, and a new generation cannot silently destroy prior content.
- Scenario 3 and FR-015 to FR-021 acceptance criteria pass.

**Exit gate**: Report output and artifact lifecycle are approved against representative fixtures, including no-data and conflict cases.

### Phase 5: Report History and End-to-End User Workflow

**Purpose**: Complete the normal manager journey across setup, readiness, generation, review, and retrieval.

**Work**:

- Implement report history search by project and reporting period.
- Add the approved download/export action, if required.
- Provide clear empty, loading, error, and success states across the browser workflow.
- Verify frontend and backend authorization for project configuration, update, refresh, report, and retrieval operations.
- Validate that the manager can distinguish report versions and their generation/freshness metadata.
- Run end-to-end tests through the approved interface using controlled Jira and database fixtures.

**Milestone M5: MVP Workflow Accepted**

- A manager can complete the agreed reporting workflow without developer intervention.
- A prior report can be found and retrieved; missing reports produce an explicit empty state.
- End-to-end acceptance scenarios for manual updates, refresh, report generation, and retrieval pass.
- The approved interface decision (browser and/or CLI) is covered by acceptance tests.

**Exit gate**: Product owner accepts the end-to-end MVP scenarios and the measured workflow meets the approved performance target.

### Phase 6: Hardening and Release Readiness

**Purpose**: Validate security, operational recovery, maintainability, and release documentation.

**Work**:

- Review backend authorization, secret handling, Jira scopes, input validation, and sensitive logging.
- Test refresh and report behavior under transient external API failure, database failure, restart, and retry conditions.
- Validate migration safety, backup/restore expectations, retention behavior, and report-version integrity per approved operational requirements.
- Run the agreed performance test with representative team and Jira issue volumes; record timing boundaries and results.
- Verify accessibility/usability for the supported browsers and relevant user roles.
- Finalize configuration documentation, operational runbook, known limitations, and release checklist.

**Milestone M6: Release Candidate**

- All required acceptance, integration, security, migration, and performance checks pass.
- No unresolved blocking clarification remains; accepted residual risks are documented.
- Setup, configuration, recovery, and operational procedures are documented.
- Product and technical owners approve the release candidate.

## 4. Cross-Phase Quality Gates

- **Specification gate**: Every feature task traces to an approved requirement or is explicitly identified as enabling work.
- **Constitution gate**: React 18/Vite, Node.js/Express, PostgreSQL 15 via Docker, server-side integrations, backend authorization, and versioned migrations remain enforced.
- **Test gate**: Changed behavior has focused unit/integration coverage; database and Jira boundaries use deterministic fixtures where possible.
- **Security gate**: No credentials in source/browser/log output; protected operations enforce backend authorization; Jira scopes are least-privilege.
- **Data gate**: Refresh status, source attribution, temporal semantics, and report versioning are tested before reports are treated as authoritative.
- **Acceptance gate**: A milestone is complete only when its stated exit criteria pass or an explicit exception is approved and documented.

## 5. Dependency Order

```text
M0 Approved MVP Contract
  -> M1 Reproducible Application Skeleton
  -> M2 Manual Inputs Ready
  -> M3 Jira Data Ready
  -> M4 Report Workflow Complete
  -> M5 MVP Workflow Accepted
  -> M6 Release Candidate
```

M2 and M3 may be developed in parallel after M1 if the approved data contracts and access model are stable. M4 depends on both. M5 depends on M4. M6 depends on the complete MVP workflow.

## 6. Milestone-to-Spec Traceability

| Milestone | Primary specification coverage |
| --- | --- |
| M0 | C-01 to C-03; G-01 to G-16; OD-001 to OD-009 |
| M1 | Constitution Technical Constraints; NFR-007, NFR-009 |
| M2 | Scenario 1; FR-001 to FR-008; NFR-005 |
| M3 | Scenario 2; FR-009 to FR-014; NFR-003, NFR-004, NFR-006, NFR-008 |
| M4 | Scenario 3; FR-015 to FR-021; SC-002 to SC-005 |
| M5 | Scenario 4; NFR-001, NFR-002; SC-001 and SC-006 |
| M6 | Constitution Core Principles 3 to 7; NFR-002 to NFR-010; all approved release criteria |

## 7. Risks and Mitigations

| Risk | Impact | Mitigation / gate |
| --- | --- | --- |
| Scope/interface decisions remain open | Rework or incompatible architecture | Resolve C-01 to C-03 before Phase 1 implementation |
| Jira fields and status history vary by project | Incorrect weekly completion or risk reporting | Approve mappings and reporting-period semantics in Phase 0; test representative fixtures in Phase 3 |
| Missing or late team updates | Misleading report completeness | Define contributor roster, required fields, and incomplete-report policy before Phase 2 |
| Jira outages or rate limits | Stale or partial reporting inputs | Implement bounded retries, explicit refresh state, and approved stale-data behavior in Phase 3 |
| Generated and manually edited report versions diverge | Loss of manager changes or unclear source of truth | Approve persistence and regeneration/version rules before Phase 4 |
| Sensitive Jira data is exposed | Security or compliance incident | Enforce backend-only integration, least privilege, access controls, and log review throughout delivery |

## 8. Explicitly Deferred

- Confluence retrieval or publishing until C-01/OD-007 is resolved and a feature scenario is approved.
- A separate CLI until C-02/OD-001 is resolved.
- Email distribution, approval workflows, real-time dashboards, forecasting, and other integrations, consistent with the specification's MVP exclusions.
- Detailed API contracts, database schema, component boundaries, and deployment topology to the implementation design, once Phase 0 product decisions are settled.