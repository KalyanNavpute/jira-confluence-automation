# Specification Review: Gaps and Clarifications

**Documents reviewed**: `spec/constitution.md`, `spec/specification.md`  
**Review stance**: Senior developer review; findings are ordered by implementation impact.  
**Overall assessment**: The reporting goal and high-level workflow are understandable, but several product and data decisions remain unresolved. The specification is not yet sufficiently precise to plan an implementation without making material assumptions.

## Findings

### Blocking Decisions

#### C-01: Product scope decision recorded

**References**: Constitution Purpose and Technical Constraints; specification Sections 3, 10, 12, and 13.

**Status**: Resolved in `spec/specification.md` as D-001.

This specification covers Jira-based weekly status reporting within the broader Jira/Confluence automation project. Confluence retrieval and publishing are excluded from this feature's MVP; any future Confluence workflow requires a separately approved feature specification with user scenarios and acceptance criteria.

#### C-02: CLI requirement conflicts with the browser workflow and technical constitution

**References**: Constitution Technical Constraints; specification Sections 3, 5, 10, and 12; source project specification Sections 5, 16, and 18.

The source specification calls for a Python CLI and says command-line report generation is an acceptance criterion. The constitution requires a React/Vite browser frontend and Node/Express backend; the feature spec assumes a browser workflow but leaves a separate CLI unresolved. The current requirements do not establish which interface is mandatory or whether both are required.

**Clarify**: Is the browser workflow the sole supported interface, must a CLI also be provided, or is the CLI requirement superseded? If CLI remains required, define its supported operations and whether it uses the Node backend or is a separate client. Do not implement a Python CLI without an explicit constitutional exception.

#### C-03: Weekly scheduling and manual generation are not reconciled

**References**: Specification Sections 1, 3, 5, 10, and 12; source project specification Sections 6, 13, 14, and 17.

The source requires one-click generation on a weekly schedule and leaves manual versus scheduled execution open. The feature spec describes a manager-triggered workflow and calls daily refresh a separate recurring operation, but does not explicitly decide whether reports are generated automatically, manually, or both.

**Clarify**: Does the weekly schedule automatically generate and save a report, remind a manager to generate one, or merely define the reporting period? Specify the trigger, timezone, behavior when inputs are incomplete, and whether manual generation remains available.

### High-Impact Requirement Gaps

#### G-01: “Daily refresh” has inconsistent data coverage

**References**: Specification Section 3; FR-010; source project specification Sections 8 and 13.

The in-scope list says daily refresh covers Jira data and configured team updates. FR-010 requires only Jira refresh. Manual updates are entered by contributors, but the document does not identify a separate source or process from which they would be refreshed.

**Clarify**: Does daily refresh apply only to Jira, or are manual updates imported from another defined source? If manual updates are only submitted in the application, define their submission cadence and how the system considers them current.

#### G-02: Weekly report semantics require historical Jira data

**References**: Specification Sections 1, 6, 7, and 8; FR-009 to FR-014; source project specification Sections 6 and 10.

The report must describe work completed and in flight for a specific week, but Jira requirements describe issue fields and a latest successful refresh. A current status does not establish whether an issue was completed during the selected week or what its status was at the period end. The Issue Snapshot entity does not say whether daily history is retained or overwritten.

**Clarify**: Should the report show status as of report generation, status as of period end, or changes that occurred during the reporting period? Define how completed work, overdue work, and milestone progress are determined, and whether Jira changelog/history or retained daily snapshots are required.

#### G-03: Required manual fields are not enumerated consistently

**References**: Specification Scenarios 1 and 3; FR-004, FR-006, and FR-008; source project specification Section 9.

Scenarios refer to “required fields,” but FR-004 says only that fields must be collected and FR-008 does not identify which are required. The source specification marks executive summary, completed items, in-progress items, blockers/risks, next-week plan, and decisions as required; the new specification does not say whether this requirement is retained. FR-006 also allows deliberately empty responses without defining which fields may be empty.

**Clarify**: List required fields per update and per report, define whether a valid “none” response satisfies each field, and distinguish an optional field, an unanswered field, and a confirmed empty value.

#### G-04: Report synthesis and ownership of content are unspecified

**References**: Specification Sections 1, 2, 5, 6, and 7; FR-013, FR-017, and FR-018.

The system is expected to generate a concise report, but the specification does not define whether it formats submitted text, derives content from Jira, calculates summaries, or uses another synthesis method. The executive summary is a required source input in the original spec but is also presented as generated report content. There are no rules for resolving conflicts between Jira facts and manual commentary.

**Clarify**: Identify which fields are authored by contributors, which are derived from Jira, and which are assembled or summarized by the system. Define how conflicting sources are represented, whether manual content may override an imported fact, and what the manager must review before finalizing. Do not imply AI-generated summaries unless that behavior is explicitly required.

#### G-05: Jira query and field mapping are not defined

**References**: Specification Sections 3, 6, 8, and 12; FR-001 and FR-009; OD-003.

“Project or board filter,” “milestone,” “blocker/impediment,” and “overdue” can depend on Jira configuration and custom fields. No initial JQL/filter rules, supported Jira project types, field mappings, status categories, or permission assumptions are stated.

**Clarify**: Specify the initial supported Jira filter mechanism, required fields and custom-field mapping, status-to-report-section mapping, blocker conventions, due-date rules, and behavior when a field is unavailable or inaccessible.

#### G-06: Risks, dependencies, and milestone progress lack explicit reporting rules

**References**: Specification Sections 2, 6, 7, and 8; FR-004, FR-009, and FR-016; source project specification Sections 3, 6, and 10.

The source requirements call out dependencies, milestone progress, blocker counts, and overdue/at-risk work. The new spec mentions some source fields but does not require dependency capture or define how milestone progress, blocker counts, or at-risk status appears in the report. The Team Update and Issue Snapshot entities also lack defined fields for these concepts.

**Clarify**: Confirm which of these are required for the MVP; define their source, calculation or entry method, and report placement. If they are not required, remove or explicitly defer the corresponding source requirements.

#### G-07: Stale, failed, and partial Jira data has no single policy

**References**: Specification Scenarios 2 and 3; FR-011, FR-012, and FR-021; OD-002 and OD-005; SC-004.

The report must disclose stale or incomplete data, but Scenario 2 says a request with no successful refresh is warned and not represented as current. Scenario 3 allows either blocking generation or marking the report incomplete. OD-002 and OD-005 overlap without resolving the policy. The phrase “from the prior day or newer” does not define a freshness window, weekend behavior, or how independent source timestamps are assessed.

**Clarify**: Define a maximum Jira-data age, separate behavior for unavailable versus stale versus partial data, whether generation is blocked in each case, and the exact warning/metadata rendered. Consolidate OD-002 and OD-005 after deciding whether manual-only reports are allowed.

#### G-08: Empty report sections can misstate missing information

**References**: Specification Sections 6 and 7; FR-006 and FR-017.

Section 7 says an empty section must state “no items were reported,” but also says missing data must not be interpreted as none. It is unclear how the report distinguishes a confirmed zero-item response from missing contributor updates, unavailable Jira fields, or a failed refresh.

**Clarify**: Define distinct report states or wording for confirmed none, not submitted, unavailable, and not applicable. Specify whether incomplete sections may be finalized and how the incompleteness is shown.

### Medium-Impact Gaps

#### G-09: Report artifact storage, export, and version behavior are ambiguous

**References**: Specification Sections 5, 6, 8, 9, and 12; FR-018 to FR-020; OD-008; source project specification FR-10 and Section 11.

The source specification says to save a Markdown file to the project root or a designated folder. The new specification requires persistence and retrieval but does not state whether Markdown is stored in PostgreSQL, written to a file, or both. Download/export behavior is open. FR-020 permits either preserving versions or explicit replacement, while Scenario 3 acceptance case 5 expects distinguishable prior reports; draft/final states and regeneration after edits are not defined.

**Clarify**: Define the authoritative artifact location, whether users can download a `.md` file, version creation/replacement rules, how a regenerated report relates to manager edits, and what metadata identifies a final report.

#### G-10: Contributor coverage and reporting-period lifecycle are incomplete

**References**: Specification Sections 4, 5, 6, and 8; FR-003, FR-005, and FR-006; OD-004 and OD-009.

The manager configures expected contributors or teams, but the spec does not define how that roster is created or updated, how multiple contributors map to teams, or how duplicate and late submissions are handled. It also refers to an “open reporting period” without defining who opens/closes it, whether contributors can edit after close, or how reopened periods work.

**Clarify**: Define contributor/team membership source, update ownership, submission uniqueness, late-update behavior, and reporting-period states and permissions.

#### G-11: Authorization and sensitive-data handling need project decisions

**References**: Constitution Core Principles 3 and 4; specification Sections 4 and 9; NFR-004 to NFR-006; OD-004.

The documents require backend authorization and protection of Jira credentials, but do not define identity provider, role permissions, project-level visibility, Jira authentication method, required scopes, or organizational data classification/retention rules. The manager, team lead, and stakeholder roles are descriptive rather than an enforceable permission model.

**Clarify**: Define authentication, role/action matrix, project access boundaries, Jira credential handling and minimum scopes, audit events, and applicable data-handling/retention policy. Confirm whether stakeholders can access reports in the application or only receive exported files.

#### G-12: Report content and formatting are not testable enough

**References**: Specification Sections 2, 6, 7, and 11; FR-016 and FR-017; SC-002 and SC-003.

“Concise,” “executive-facing,” “short bullet points,” and “expected completion state” lack measurable rules. The template does not specify date format, ordering, maximum length, Jira reference format, source attribution format, or whether a section may contain multiple team/issue entries. SC-002 says “all six required content sections,” but there are six named content sections plus report metadata and the refresh metadata line.

**Clarify**: Define a minimal report schema and testable formatting rules, including date/timezone format, issue key/link behavior, sorting/grouping, bullet length or other concise-content criteria, and precisely which sections count toward SC-002.

#### G-13: Performance target lacks a measurement boundary

**References**: Specification Goals; NFR-002; SC-001.

“Under 10 minutes in the normal case” does not define when timing starts and ends, whether contributor collection and Jira refresh are included, the expected Jira issue volume, or how much time is reserved for human review. The stated team size alone is not a workload profile.

**Clarify**: Define the measured workflow boundary, expected issue/update volume, test conditions, and whether the target is system response time or total manager effort.

#### G-14: Operational behavior and recovery are underspecified

**References**: Constitution Core Principles 4 and 7; specification Scenarios 2 and 3; FR-010 to FR-012 and FR-021; OD-006 and OD-008.

The spec requires daily scheduling and bounded integration retries but does not define retry limits/backoff, refresh overlap behavior, manual retry, failure notifications, scheduler ownership, or recovery after a backend/database restart. It also leaves retention duration and production deployment/backup expectations unspecified.

**Clarify**: Decide which operational details are product requirements for the MVP: schedule and timezone, retry/recovery behavior, operator visibility/notification, retention, and backup/restore expectations. Implementation-specific mechanisms can remain for the plan.

#### G-15: Test and completion criteria omit several required boundaries

**References**: Constitution Core Principles 6; specification Sections 5, 9, and 11.

The spec gives useful fixture-based scenario tests but does not explicitly require tests for authorization enforcement, data-conflict handling, report versioning, manual-update empty states, daily scheduler behavior, or database migration/persistence. The constitution requires these boundaries to be verified where applicable.

**Clarify**: Add acceptance tests for decisions made above, especially the selected access model, historical/status semantics, stale-data policy, and artifact-version rules. Define the required validation gates for MVP completion; a numeric coverage target is optional unless the team needs one.

#### G-16: Architecture and deployment boundary are only partially defined

**References**: Constitution Core Principles 2 and 7; specification Sections 3, 8, 9, and 10.

The constitution sets React/Vite, Node/Express, and PostgreSQL 15 via Docker, but the specification does not say whether Docker is a development-only database setup or also the deployment model, how the browser frontend is served, or what environments are supported. These may belong in the plan, but the intended runtime boundary should be known before implementation tasks are divided.

**Clarify**: Confirm supported deployment environment(s) and whether Docker/PostgreSQL constraints apply only to local development or also to production. Leave container orchestration and detailed service design to the implementation plan unless they are product constraints.

## Clarification Order

Resolve these decisions before creating implementation tasks:

1. C-02: required interface(s), including whether CLI is needed alongside the constitution's browser frontend.
2. C-03 and G-01: report trigger and daily-refresh responsibilities.
3. G-02, G-05, and G-07: Jira time semantics, mappings, and stale/partial-data policy.
4. G-03, G-04, G-06, and G-08: required inputs, synthesis/ownership, and report behavior for missing or empty data.
5. G-09 to G-11: artifact lifecycle, contributors, permissions, and data handling.
6. G-12 to G-16: measurable output, performance, operations, verification, and deployment assumptions.