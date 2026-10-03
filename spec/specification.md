# Feature Specification: Weekly Executive Status Reporting

**Status**: Draft  
**Source**: `project_spec.md` and `spec/constitution.md`  
**Primary user**: Manager responsible for a 100-person T&M team  
**Report audience**: Senior management

## 1. Summary

Provide a repeatable way to collect team updates, combine them with Jira delivery data, and produce a concise weekly executive status report in Markdown. The manager reviews and may edit the report before sharing it. The system refreshes source data daily so a weekly report can use recent information without manually re-aggregating every source.

## 2. Goals

- Reduce the effort required to prepare a weekly report to under 10 minutes in the normal case.
- Present progress, completed work, in-flight work, risks, next-week priorities, and requested decisions in a consistent format.
- Combine Jira information with structured manual updates and project context.
- Persist generated reports so they can be reviewed and retrieved later.
- Make missing, stale, or unavailable source data visible rather than silently presenting an incomplete report as current.

## 3. Scope

### In scope

- Weekly executive status report generation.
- Jira issue data for configured projects or board filters, including summaries, status, assignments, milestones, overdue work, and blocker indicators where available.
- Structured manual inputs for team-level progress, project commentary, risks, blockers, next-week priorities, and decisions needed.
- Daily refresh of Jira data and configured team updates, with the latest completed refresh used for report generation.
- Report review and editing before the manager shares it outside the application.
- Markdown report generation and persistence.
- A manager-facing browser workflow consistent with the project constitution.

### Out of scope for this specification

- Real-time dashboards, advanced analytics, or forecasting.
- Automated approval workflows.
- Sending or publishing reports by email or to external destinations.
- Confluence data retrieval or publishing until its role and required workflows are confirmed.
- Integrations with other non-Jira systems.
- Replacing Jira as the source of truth for issue status.

## 4. Users and Roles

### Manager

- Configures the project/report period and Jira filter.
- Reviews data freshness and missing-input warnings.
- Starts report generation, reviews and edits the result, and retrieves saved reports.

### Team lead or update contributor

- Provides structured weekly updates for their team or area, including progress, risks, blockers, and upcoming work.
- Can identify the reporting period and project to which an update applies.

### Senior manager or stakeholder

- Reads the final concise report. Direct access to the application is not required by this specification.

Authentication, authorization details, and whether contributors need individual application accounts remain to be confirmed.

## 5. User Scenarios and Testing

### Scenario 1: Collect and maintain weekly updates (Priority: P1)

A team lead submits a structured update for the selected project and reporting week. The manager can see which expected updates have been received and identify missing updates before generating the report.

**Independent test**: Submit an update for a project and week, then retrieve it and confirm its fields and reporting period are preserved. Verify that an absent update is shown as missing rather than treated as a zero-risk or no-progress update.

**Acceptance scenarios**:

1. Given an authorized contributor and an open reporting period, when they submit an update with required fields, then the update is saved against the selected project and period.
2. Given a contributor submits an update missing a required field, when they attempt to save it, then the system identifies the missing field and does not accept an incomplete update as complete.
3. Given a reporting period with expected contributors, when the manager views collection status, then submitted and missing updates are distinguishable.
4. Given an update has already been submitted, when its author or an authorized manager corrects it, then the latest saved version is used and the change is not silently lost.

### Scenario 2: Refresh and inspect Jira data (Priority: P1)

The system retrieves Jira data for a configured project or board filter on a daily cadence. The manager can tell when the last successful refresh occurred and whether a refresh failed or returned incomplete data.

**Independent test**: Use a controlled Jira fixture to refresh a configured filter and verify that issue fields, refresh time, and failure state are represented correctly without live Jira credentials.

**Acceptance scenarios**:

1. Given valid Jira configuration, when a scheduled refresh runs, then matching issue data is retrieved and its refresh time is recorded.
2. Given Jira pagination is required, when a refresh completes, then all pages within the configured filter are processed or the refresh is marked incomplete.
3. Given Jira is unavailable or rejects authentication, when a refresh is attempted, then the failure is recorded and a clear, non-sensitive status is available to the manager.
4. Given Jira returns a rate-limit or transient error, when a refresh is attempted, then retries are bounded and the final outcome is visible.
5. Given no successful refresh exists for the expected freshness window, when a report is requested, then the manager is warned and the report is not represented as using current Jira data.

### Scenario 3: Generate and review a weekly report (Priority: P1)

The manager selects a project and week, reviews input readiness, and generates a concise Markdown report combining Jira information and manual updates. The manager can edit the generated content and save the final artifact.

**Independent test**: Provide deterministic Jira and manual-update fixtures, generate a report, and compare its required sections and content to expected output. Verify that the saved report can be retrieved unchanged.

**Acceptance scenarios**:

1. Given valid project, reporting period, Jira data, and required manual inputs, when the manager generates a report, then the report contains every required section and is saved as Markdown.
2. Given required manual inputs are missing, when the manager attempts generation, then the system identifies the missing inputs and does not silently fabricate their content.
3. Given Jira data is stale or unavailable, when the manager generates or attempts to generate a report, then the system clearly identifies the data limitation and follows the configured policy for blocking or explicitly marking the report incomplete.
4. Given a generated report, when the manager edits and saves it, then the edited artifact is persisted and subsequent retrieval returns the saved version.
5. Given a prior report exists for the same project and period, when another report is generated, then the system avoids silently overwriting the prior artifact and makes the saved versions distinguishable.

### Scenario 4: Retrieve a prior report (Priority: P2)

The manager locates a previously generated report by project and reporting period and can inspect or download its Markdown content.

**Independent test**: Save reports for two periods and confirm that querying either period returns the matching report and metadata.

**Acceptance scenarios**:

1. Given a saved report, when the manager searches by project and reporting period, then the matching report and its generation metadata are returned.
2. Given no report exists for the selected period, when the manager searches, then the system returns an empty state rather than a misleading error.

## 6. Functional Requirements

### Project and reporting configuration

- **FR-001**: The system MUST allow an authorized manager to identify a project, owner/team, reporting period, and configured Jira project or board filter.
- **FR-002**: The system MUST support a weekly reporting period with explicit start and end dates.
- **FR-003**: The system MUST support configured expected contributors or teams so missing manual updates can be identified.

### Manual updates

- **FR-004**: The system MUST collect the project summary, completed items, in-progress items, blockers or risks, next-week priorities, and decisions needed.
- **FR-005**: The system MUST associate every manual update with a project and reporting period and retain its author and last-updated time.
- **FR-006**: The system MUST distinguish a deliberately empty response (for example, no known blockers) from an update that has not been submitted.
- **FR-007**: The system MUST support optional team-specific notes, stakeholder comments, and approval or escalation notes.
- **FR-008**: The system MUST validate required fields and provide actionable validation feedback before accepting a complete update.

### Jira refresh and aggregation

- **FR-009**: The system MUST retrieve Jira issue status, summary, assignment, milestone, overdue, and blocker/impediment information available through the configured Jira filter and permissions.
- **FR-010**: The system MUST refresh Jira data daily and record the outcome and completion time of each refresh.
- **FR-011**: The system MUST use the latest successful refresh from the prior day or newer for report generation, and MUST disclose when data is older or incomplete.
- **FR-012**: The system MUST handle paginated Jira results and surface incomplete refreshes.
- **FR-013**: The system MUST combine Jira facts and manual commentary without treating either source as a substitute for the other.
- **FR-014**: The system MUST preserve source attribution sufficiently for a manager to distinguish Jira-derived facts from manually supplied commentary.

### Report generation and persistence

- **FR-015**: The system MUST generate a Markdown report for a selected project and reporting period.
- **FR-016**: The report MUST contain project, owner, and reporting-week metadata, followed by Executive Summary, Completed, In Flight, Risks / Blockers, Next Week, and Decisions Needed sections.
- **FR-017**: The system MUST produce concise executive-facing content and MUST NOT invent values when source data is missing.
- **FR-018**: The manager MUST be able to review and edit generated content before treating it as final.
- **FR-019**: The system MUST persist generated and manager-edited reports and make them retrievable by project and reporting period.
- **FR-020**: Saving a new report for an existing project and period MUST preserve prior versions or request an explicit replacement; it MUST NOT silently overwrite them.
- **FR-021**: The system MUST provide a clear status for generation failures and identify missing or stale inputs.

## 7. Report Format

The generated Markdown MUST follow this logical structure. Sections with no applicable items remain present and state that no items were reported; the system must not infer that a blank or missing source means there are none.

```markdown
# Executive Weekly Status

- Project: <project>
- Owner: <owner/team>
- Week: <start date> to <end date>
- Data refreshed: <timestamp and status>

## Executive Summary
- <concise summary>

## Completed
- <completed item, with Jira reference where applicable>

## In Flight
- <in-progress item and expected completion state>

## Risks / Blockers
- <risk or blocker, impact, and owner where known>

## Next Week
- <planned item>

## Decisions Needed
- <decision, requested owner, or due date where known>
```

## 8. Data Entities

- **Project configuration**: Project name, owner/team, Jira filter, expected contributors, and configuration state.
- **Reporting period**: Project association, start and end dates, and collection status.
- **Team update**: Project and period association, contributor, structured update fields, submission state, and timestamps.
- **Jira refresh**: Filter, start and completion times, outcome, freshness, and non-sensitive error information.
- **Issue snapshot**: Jira issue identifier, summary, status, assignee, relevant milestone/due information, blocker indicator, and source refresh reference.
- **Status report**: Project and period association, generated content, generation metadata, saved version, and review/edit state.

These are logical entities, not a prescribed physical database schema. Schema and ownership details belong in the implementation plan and must follow the constitution's data-integrity principles.

## 9. Non-Functional Requirements

- **NFR-001 Usability**: A manager without advanced technical knowledge can complete the normal weekly workflow with clear progress and validation feedback.
- **NFR-002 Performance**: In the normal case, report preparation and generation can be completed in under 10 minutes for the stated team size of approximately 100 people.
- **NFR-003 Reliability**: Missing required input, Jira failures, and partial refreshes produce clear recoverable states rather than misleading successful output.
- **NFR-004 Security**: Jira credentials and other secrets are never exposed to browser code, committed source, logs, report content, or client-visible error details.
- **NFR-005 Authorization**: Protected operations are authorized by the backend; exact roles and identity provider are an open decision.
- **NFR-006 Privacy**: Jira and manual data are limited to the configured project/filter and permissions needed for the report.
- **NFR-007 Maintainability**: Jira access, report aggregation, persistence, and presentation remain independently testable boundaries.
- **NFR-008 Testability**: Routine automated tests use controlled fixtures or mocks and do not require live Jira credentials.
- **NFR-009 Reproducibility**: Development and tests use the stack constrained by the constitution: React 18 with Vite, Node.js with Express, and PostgreSQL 15 via Docker.
- **NFR-010 Auditability**: The system records report generation and refresh timestamps and retains distinguishable report versions; detailed retention duration is to be decided.

## 10. Constraints and Alignment

- The implementation MUST follow `spec/constitution.md`: React 18/Vite frontend, Node.js/Express backend, and PostgreSQL 15 via Docker.
- Jira and any future Confluence integration MUST be performed by the backend.
- Jira and manual updates are the data sources defined by the source specification.
- The source specification proposes a Python CLI, while the constitution constrains the application stack and this feature assumes a manager-facing browser workflow. Whether a separate CLI is also required is unresolved; implementation planning MUST record the decision before dropping or adding CLI support.
- The source specification excludes non-Jira integrations, while the project constitution names Jira and Confluence automation. Confluence workflows remain outside this feature until their use cases and acceptance criteria are approved.
- The source specification describes both a daily refresh and one-click weekly generation. This document treats daily refresh as a separate recurring operation and report generation as a manager-triggered weekly operation.

## 11. Success Criteria

- **SC-001**: A manager can produce a report for a project and week in under 10 minutes in the normal case, including review of required inputs.
- **SC-002**: Every generated report contains all six required content sections and the project, owner, and reporting-week metadata.
- **SC-003**: In controlled test data, Jira facts and manual updates appear in the correct report sections with distinguishable source attribution.
- **SC-004**: Missing manual updates, stale Jira data, and failed refreshes are visible before a report is treated as current.
- **SC-005**: A saved report can be retrieved by project and reporting period, and a new generation does not silently destroy an earlier version.
- **SC-006**: The weekly workflow is repeatable without custom code changes between reporting periods.

## 12. Assumptions and Open Decisions

- **A-001**: Jira is the only external data integration in the initial release; manual team updates provide non-Jira commentary.
- **A-002**: Reports are saved in the application and reviewed by the manager before being shared externally.
- **A-003**: Daily refresh runs in the early morning in a configured business timezone; the timezone and exact schedule are not yet specified.
- **OD-001**: Is a command-line interface required in addition to the manager-facing browser workflow?
- **OD-002**: Is Jira data mandatory to generate a report, or may a clearly marked manual-only report be generated when Jira is unavailable?
- **OD-003**: What Jira project/board filters, fields, status mappings, and blocker conventions should be supported initially?
- **OD-004**: Which identity provider, application roles, and contributor access model are required?
- **OD-005**: Should a report with stale or partially unavailable Jira data be blocked, or allowed with a prominent incomplete-data warning?
- **OD-006**: What business timezone and exact time should the daily refresh use, and how should weekends/holidays be handled?
- **OD-007**: Should Confluence be used as an input source, a publishing destination, or neither for the initial release?
- **OD-008**: What report retention period and export/download behavior are required?
- **OD-009**: Should team leads submit updates directly, or should the manager enter/approve all updates?

## 13. Out of Scope for the MVP

- Automated email distribution or Confluence publishing.
- Real-time issue synchronization or live dashboards.
- Automatic approvals or workflow transitions in Jira.
- Predictive analytics, delivery forecasting, or inferred risk scoring.
- Integrations with systems other than Jira until explicitly approved.