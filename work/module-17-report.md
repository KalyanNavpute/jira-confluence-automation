# Module 17 Completion Report

## Specification Contents
````markdown
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
- Confluence data retrieval and publishing are outside this reporting feature's MVP; any Confluence workflow requires a separately approved feature scope.
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
- **D-001 (resolved)**: This specification covers Jira-based weekly status reporting as a feature within the broader Jira/Confluence automation project. Confluence retrieval and publishing are excluded from this feature's MVP; a future Confluence workflow requires a separately approved feature specification with user scenarios and acceptance criteria.
- The source specification proposes a Python CLI, while the constitution constrains the application stack and this feature assumes a manager-facing browser workflow. Whether a separate CLI is also required is unresolved; implementation planning MUST record the decision before dropping or adding CLI support.
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
- **OD-008**: What report retention period and export/download behavior are required?
- **OD-009**: Should team leads submit updates directly, or should the manager enter/approve all updates?

## 13. Out of Scope for the MVP

- Automated email distribution or Confluence publishing.
- Real-time issue synchronization or live dashboards.
- Automatic approvals or workflow transitions in Jira.
- Predictive analytics, delivery forecasting, or inferred risk scoring.
- Integrations with systems other than Jira until explicitly approved.
````

## Commit History
```
f72979b (HEAD -> main, origin/main, origin/HEAD) updated task
1b31055 module 17 updated
d4fab79 module 17 spec
e34d49a module 16
6e75fe1 update
05b561f update for module 15
f29e88f module 14
512ad36 docs: link backlog task to issue #1
bbf1f14 (source/main, source/HEAD) add github mcp
00ec81c module 13 update
fa8d691 Module 13
462c08d module 12 report
04e3b13 module 12 update2
347358f Module 12
9fc9174 module 10 report
7d30eb3 module 10
0a94884 module 9
71f1c46 initial
96f880b updated
46d5c7c Latest changes
2f0a179 initial commit
```

## Commit Count
21

## Project Files

```
.agent.md
.github/copilot-instructions.md
.mcp.json
.venv/bin/Activate.ps1
.venv/bin/activate
.venv/bin/activate.csh
.venv/bin/activate.fish
.venv/bin/pip
.venv/bin/pip3
.venv/bin/pip3.9
.venv/bin/python
.venv/bin/python3
.venv/bin/python3.9
.venv/lib/python3.9/site-packages/_distutils_hack/__init__.py
.venv/lib/python3.9/site-packages/_distutils_hack/override.py
.venv/lib/python3.9/site-packages/distutils-precedence.pth
.venv/lib/python3.9/site-packages/pip-21.2.4.dist-info/INSTALLER
.venv/lib/python3.9/site-packages/pip-21.2.4.dist-info/LICENSE.txt
.venv/lib/python3.9/site-packages/pip-21.2.4.dist-info/METADATA
.venv/lib/python3.9/site-packages/pip-21.2.4.dist-info/RECORD
.venv/lib/python3.9/site-packages/pip-21.2.4.dist-info/REQUESTED
.venv/lib/python3.9/site-packages/pip-21.2.4.dist-info/WHEEL
.venv/lib/python3.9/site-packages/pip-21.2.4.dist-info/entry_points.txt
.venv/lib/python3.9/site-packages/pip-21.2.4.dist-info/top_level.txt
.venv/lib/python3.9/site-packages/pip/__init__.py
.venv/lib/python3.9/site-packages/pip/__main__.py
.venv/lib/python3.9/site-packages/pip/_internal/__init__.py
.venv/lib/python3.9/site-packages/pip/_internal/build_env.py
.venv/lib/python3.9/site-packages/pip/_internal/cache.py
.venv/lib/python3.9/site-packages/pip/_internal/cli/__init__.py
.venv/lib/python3.9/site-packages/pip/_internal/cli/autocompletion.py
.venv/lib/python3.9/site-packages/pip/_internal/cli/base_command.py
.venv/lib/python3.9/site-packages/pip/_internal/cli/cmdoptions.py
.venv/lib/python3.9/site-packages/pip/_internal/cli/command_context.py
.venv/lib/python3.9/site-packages/pip/_internal/cli/main.py
.venv/lib/python3.9/site-packages/pip/_internal/cli/main_parser.py
.venv/lib/python3.9/site-packages/pip/_internal/cli/parser.py
.venv/lib/python3.9/site-packages/pip/_internal/cli/progress_bars.py
.venv/lib/python3.9/site-packages/pip/_internal/cli/req_command.py
.venv/lib/python3.9/site-packages/pip/_internal/cli/spinners.py
.venv/lib/python3.9/site-packages/pip/_internal/cli/status_codes.py
.venv/lib/python3.9/site-packages/pip/_internal/commands/__init__.py
.venv/lib/python3.9/site-packages/pip/_internal/commands/cache.py
.venv/lib/python3.9/site-packages/pip/_internal/commands/check.py
.venv/lib/python3.9/site-packages/pip/_internal/commands/completion.py
.venv/lib/python3.9/site-packages/pip/_internal/commands/configuration.py
.venv/lib/python3.9/site-packages/pip/_internal/commands/debug.py
.venv/lib/python3.9/site-packages/pip/_internal/commands/download.py
.venv/lib/python3.9/site-packages/pip/_internal/commands/freeze.py
.venv/lib/python3.9/site-packages/pip/_internal/commands/hash.py
.venv/lib/python3.9/site-packages/pip/_internal/commands/help.py
.venv/lib/python3.9/site-packages/pip/_internal/commands/index.py
.venv/lib/python3.9/site-packages/pip/_internal/commands/install.py
.venv/lib/python3.9/site-packages/pip/_internal/commands/list.py
.venv/lib/python3.9/site-packages/pip/_internal/commands/search.py
.venv/lib/python3.9/site-packages/pip/_internal/commands/show.py
.venv/lib/python3.9/site-packages/pip/_internal/commands/uninstall.py
.venv/lib/python3.9/site-packages/pip/_internal/commands/wheel.py
.venv/lib/python3.9/site-packages/pip/_internal/configuration.py
.venv/lib/python3.9/site-packages/pip/_internal/distributions/__init__.py
.venv/lib/python3.9/site-packages/pip/_internal/distributions/base.py
.venv/lib/python3.9/site-packages/pip/_internal/distributions/installed.py
.venv/lib/python3.9/site-packages/pip/_internal/distributions/sdist.py
.venv/lib/python3.9/site-packages/pip/_internal/distributions/wheel.py
.venv/lib/python3.9/site-packages/pip/_internal/exceptions.py
.venv/lib/python3.9/site-packages/pip/_internal/index/__init__.py
.venv/lib/python3.9/site-packages/pip/_internal/index/collector.py
.venv/lib/python3.9/site-packages/pip/_internal/index/package_finder.py
.venv/lib/python3.9/site-packages/pip/_internal/index/sources.py
.venv/lib/python3.9/site-packages/pip/_internal/locations/__init__.py
.venv/lib/python3.9/site-packages/pip/_internal/locations/_distutils.py
.venv/lib/python3.9/site-packages/pip/_internal/locations/_sysconfig.py
.venv/lib/python3.9/site-packages/pip/_internal/locations/base.py
.venv/lib/python3.9/site-packages/pip/_internal/main.py
.venv/lib/python3.9/site-packages/pip/_internal/metadata/__init__.py
.venv/lib/python3.9/site-packages/pip/_internal/metadata/base.py
.venv/lib/python3.9/site-packages/pip/_internal/metadata/pkg_resources.py
.venv/lib/python3.9/site-packages/pip/_internal/models/__init__.py
.venv/lib/python3.9/site-packages/pip/_internal/models/candidate.py
.venv/lib/python3.9/site-packages/pip/_internal/models/direct_url.py
.venv/lib/python3.9/site-packages/pip/_internal/models/format_control.py
.venv/lib/python3.9/site-packages/pip/_internal/models/index.py
.venv/lib/python3.9/site-packages/pip/_internal/models/link.py
.venv/lib/python3.9/site-packages/pip/_internal/models/scheme.py
.venv/lib/python3.9/site-packages/pip/_internal/models/search_scope.py
.venv/lib/python3.9/site-packages/pip/_internal/models/selection_prefs.py
.venv/lib/python3.9/site-packages/pip/_internal/models/target_python.py
.venv/lib/python3.9/site-packages/pip/_internal/models/wheel.py
.venv/lib/python3.9/site-packages/pip/_internal/network/__init__.py
.venv/lib/python3.9/site-packages/pip/_internal/network/auth.py
.venv/lib/python3.9/site-packages/pip/_internal/network/cache.py
.venv/lib/python3.9/site-packages/pip/_internal/network/download.py
.venv/lib/python3.9/site-packages/pip/_internal/network/lazy_wheel.py
.venv/lib/python3.9/site-packages/pip/_internal/network/session.py
.venv/lib/python3.9/site-packages/pip/_internal/network/utils.py
.venv/lib/python3.9/site-packages/pip/_internal/network/xmlrpc.py
.venv/lib/python3.9/site-packages/pip/_internal/operations/__init__.py
.venv/lib/python3.9/site-packages/pip/_internal/operations/build/__init__.py
.venv/lib/python3.9/site-packages/pip/_internal/operations/build/metadata.py
.venv/lib/python3.9/site-packages/pip/_internal/operations/build/metadata_legacy.py
.venv/lib/python3.9/site-packages/pip/_internal/operations/build/wheel.py
.venv/lib/python3.9/site-packages/pip/_internal/operations/build/wheel_legacy.py
.venv/lib/python3.9/site-packages/pip/_internal/operations/check.py
.venv/lib/python3.9/site-packages/pip/_internal/operations/freeze.py
.venv/lib/python3.9/site-packages/pip/_internal/operations/install/__init__.py
.venv/lib/python3.9/site-packages/pip/_internal/operations/install/editable_legacy.py
.venv/lib/python3.9/site-packages/pip/_internal/operations/install/legacy.py
.venv/lib/python3.9/site-packages/pip/_internal/operations/install/wheel.py
.venv/lib/python3.9/site-packages/pip/_internal/operations/prepare.py
.venv/lib/python3.9/site-packages/pip/_internal/pyproject.py
.venv/lib/python3.9/site-packages/pip/_internal/req/__init__.py
.venv/lib/python3.9/site-packages/pip/_internal/req/constructors.py
.venv/lib/python3.9/site-packages/pip/_internal/req/req_file.py
.venv/lib/python3.9/site-packages/pip/_internal/req/req_install.py
.venv/lib/python3.9/site-packages/pip/_internal/req/req_set.py
.venv/lib/python3.9/site-packages/pip/_internal/req/req_tracker.py
.venv/lib/python3.9/site-packages/pip/_internal/req/req_uninstall.py
.venv/lib/python3.9/site-packages/pip/_internal/resolution/__init__.py
.venv/lib/python3.9/site-packages/pip/_internal/resolution/base.py
.venv/lib/python3.9/site-packages/pip/_internal/resolution/legacy/__init__.py
.venv/lib/python3.9/site-packages/pip/_internal/resolution/legacy/resolver.py
.venv/lib/python3.9/site-packages/pip/_internal/resolution/resolvelib/__init__.py
.venv/lib/python3.9/site-packages/pip/_internal/resolution/resolvelib/base.py
.venv/lib/python3.9/site-packages/pip/_internal/resolution/resolvelib/candidates.py
.venv/lib/python3.9/site-packages/pip/_internal/resolution/resolvelib/factory.py
.venv/lib/python3.9/site-packages/pip/_internal/resolution/resolvelib/found_candidates.py
.venv/lib/python3.9/site-packages/pip/_internal/resolution/resolvelib/provider.py
.venv/lib/python3.9/site-packages/pip/_internal/resolution/resolvelib/reporter.py
.venv/lib/python3.9/site-packages/pip/_internal/resolution/resolvelib/requirements.py
.venv/lib/python3.9/site-packages/pip/_internal/resolution/resolvelib/resolver.py
.venv/lib/python3.9/site-packages/pip/_internal/self_outdated_check.py
.venv/lib/python3.9/site-packages/pip/_internal/utils/__init__.py
.venv/lib/python3.9/site-packages/pip/_internal/utils/_log.py
.venv/lib/python3.9/site-packages/pip/_internal/utils/appdirs.py
.venv/lib/python3.9/site-packages/pip/_internal/utils/compat.py
.venv/lib/python3.9/site-packages/pip/_internal/utils/compatibility_tags.py
.venv/lib/python3.9/site-packages/pip/_internal/utils/datetime.py
.venv/lib/python3.9/site-packages/pip/_internal/utils/deprecation.py
.venv/lib/python3.9/site-packages/pip/_internal/utils/direct_url_helpers.py
.venv/lib/python3.9/site-packages/pip/_internal/utils/distutils_args.py
.venv/lib/python3.9/site-packages/pip/_internal/utils/encoding.py
.venv/lib/python3.9/site-packages/pip/_internal/utils/entrypoints.py
.venv/lib/python3.9/site-packages/pip/_internal/utils/filesystem.py
.venv/lib/python3.9/site-packages/pip/_internal/utils/filetypes.py
.venv/lib/python3.9/site-packages/pip/_internal/utils/glibc.py
.venv/lib/python3.9/site-packages/pip/_internal/utils/hashes.py
.venv/lib/python3.9/site-packages/pip/_internal/utils/inject_securetransport.py
.venv/lib/python3.9/site-packages/pip/_internal/utils/logging.py
.venv/lib/python3.9/site-packages/pip/_internal/utils/misc.py
.venv/lib/python3.9/site-packages/pip/_internal/utils/models.py
.venv/lib/python3.9/site-packages/pip/_internal/utils/packaging.py
.venv/lib/python3.9/site-packages/pip/_internal/utils/parallel.py
.venv/lib/python3.9/site-packages/pip/_internal/utils/pkg_resources.py
.venv/lib/python3.9/site-packages/pip/_internal/utils/setuptools_build.py
.venv/lib/python3.9/site-packages/pip/_internal/utils/subprocess.py
.venv/lib/python3.9/site-packages/pip/_internal/utils/temp_dir.py
.venv/lib/python3.9/site-packages/pip/_internal/utils/unpacking.py
.venv/lib/python3.9/site-packages/pip/_internal/utils/urls.py
.venv/lib/python3.9/site-packages/pip/_internal/utils/virtualenv.py
.venv/lib/python3.9/site-packages/pip/_internal/utils/wheel.py
.venv/lib/python3.9/site-packages/pip/_internal/vcs/__init__.py
.venv/lib/python3.9/site-packages/pip/_internal/vcs/bazaar.py
.venv/lib/python3.9/site-packages/pip/_internal/vcs/git.py
.venv/lib/python3.9/site-packages/pip/_internal/vcs/mercurial.py
.venv/lib/python3.9/site-packages/pip/_internal/vcs/subversion.py
.venv/lib/python3.9/site-packages/pip/_internal/vcs/versioncontrol.py
.venv/lib/python3.9/site-packages/pip/_internal/wheel_builder.py
.venv/lib/python3.9/site-packages/pip/_vendor/__init__.py
.venv/lib/python3.9/site-packages/pip/_vendor/appdirs.py
.venv/lib/python3.9/site-packages/pip/_vendor/cachecontrol/__init__.py
.venv/lib/python3.9/site-packages/pip/_vendor/cachecontrol/_cmd.py
.venv/lib/python3.9/site-packages/pip/_vendor/cachecontrol/adapter.py
.venv/lib/python3.9/site-packages/pip/_vendor/cachecontrol/cache.py
.venv/lib/python3.9/site-packages/pip/_vendor/cachecontrol/caches/__init__.py
.venv/lib/python3.9/site-packages/pip/_vendor/cachecontrol/caches/file_cache.py
.venv/lib/python3.9/site-packages/pip/_vendor/cachecontrol/caches/redis_cache.py
.venv/lib/python3.9/site-packages/pip/_vendor/cachecontrol/compat.py
.venv/lib/python3.9/site-packages/pip/_vendor/cachecontrol/controller.py
.venv/lib/python3.9/site-packages/pip/_vendor/cachecontrol/filewrapper.py
.venv/lib/python3.9/site-packages/pip/_vendor/cachecontrol/heuristics.py
.venv/lib/python3.9/site-packages/pip/_vendor/cachecontrol/serialize.py
.venv/lib/python3.9/site-packages/pip/_vendor/cachecontrol/wrapper.py
.venv/lib/python3.9/site-packages/pip/_vendor/certifi/__init__.py
.venv/lib/python3.9/site-packages/pip/_vendor/certifi/__main__.py
.venv/lib/python3.9/site-packages/pip/_vendor/certifi/cacert.pem
.venv/lib/python3.9/site-packages/pip/_vendor/certifi/core.py
.venv/lib/python3.9/site-packages/pip/_vendor/chardet/__init__.py
.venv/lib/python3.9/site-packages/pip/_vendor/chardet/big5freq.py
.venv/lib/python3.9/site-packages/pip/_vendor/chardet/big5prober.py
.venv/lib/python3.9/site-packages/pip/_vendor/chardet/chardistribution.py
.venv/lib/python3.9/site-packages/pip/_vendor/chardet/charsetgroupprober.py
.venv/lib/python3.9/site-packages/pip/_vendor/chardet/charsetprober.py
.venv/lib/python3.9/site-packages/pip/_vendor/chardet/cli/__init__.py
.venv/lib/python3.9/site-packages/pip/_vendor/chardet/cli/chardetect.py
.venv/lib/python3.9/site-packages/pip/_vendor/chardet/codingstatemachine.py
.venv/lib/python3.9/site-packages/pip/_vendor/chardet/compat.py
.venv/lib/python3.9/site-packages/pip/_vendor/chardet/cp949prober.py
.venv/lib/python3.9/site-packages/pip/_vendor/chardet/enums.py
.venv/lib/python3.9/site-packages/pip/_vendor/chardet/escprober.py
.venv/lib/python3.9/site-packages/pip/_vendor/chardet/escsm.py
.venv/lib/python3.9/site-packages/pip/_vendor/chardet/eucjpprober.py
.venv/lib/python3.9/site-packages/pip/_vendor/chardet/euckrfreq.py
.venv/lib/python3.9/site-packages/pip/_vendor/chardet/euckrprober.py
.venv/lib/python3.9/site-packages/pip/_vendor/chardet/euctwfreq.py
.venv/lib/python3.9/site-packages/pip/_vendor/chardet/euctwprober.py
.venv/lib/python3.9/site-packages/pip/_vendor/chardet/gb2312freq.py
.venv/lib/python3.9/site-packages/pip/_vendor/chardet/gb2312prober.py
.venv/lib/python3.9/site-packages/pip/_vendor/chardet/hebrewprober.py
.venv/lib/python3.9/site-packages/pip/_vendor/chardet/jisfreq.py
.venv/lib/python3.9/site-packages/pip/_vendor/chardet/jpcntx.py
.venv/lib/python3.9/site-packages/pip/_vendor/chardet/langbulgarianmodel.py
.venv/lib/python3.9/site-packages/pip/_vendor/chardet/langgreekmodel.py
.venv/lib/python3.9/site-packages/pip/_vendor/chardet/langhebrewmodel.py
.venv/lib/python3.9/site-packages/pip/_vendor/chardet/langhungarianmodel.py
.venv/lib/python3.9/site-packages/pip/_vendor/chardet/langrussianmodel.py
.venv/lib/python3.9/site-packages/pip/_vendor/chardet/langthaimodel.py
.venv/lib/python3.9/site-packages/pip/_vendor/chardet/langturkishmodel.py
.venv/lib/python3.9/site-packages/pip/_vendor/chardet/latin1prober.py
.venv/lib/python3.9/site-packages/pip/_vendor/chardet/mbcharsetprober.py
.venv/lib/python3.9/site-packages/pip/_vendor/chardet/mbcsgroupprober.py
.venv/lib/python3.9/site-packages/pip/_vendor/chardet/mbcssm.py
.venv/lib/python3.9/site-packages/pip/_vendor/chardet/metadata/__init__.py
.venv/lib/python3.9/site-packages/pip/_vendor/chardet/metadata/languages.py
.venv/lib/python3.9/site-packages/pip/_vendor/chardet/sbcharsetprober.py
.venv/lib/python3.9/site-packages/pip/_vendor/chardet/sbcsgroupprober.py
.venv/lib/python3.9/site-packages/pip/_vendor/chardet/sjisprober.py
.venv/lib/python3.9/site-packages/pip/_vendor/chardet/universaldetector.py
.venv/lib/python3.9/site-packages/pip/_vendor/chardet/utf8prober.py
.venv/lib/python3.9/site-packages/pip/_vendor/chardet/version.py
.venv/lib/python3.9/site-packages/pip/_vendor/colorama/__init__.py
.venv/lib/python3.9/site-packages/pip/_vendor/colorama/ansi.py
.venv/lib/python3.9/site-packages/pip/_vendor/colorama/ansitowin32.py
.venv/lib/python3.9/site-packages/pip/_vendor/colorama/initialise.py
.venv/lib/python3.9/site-packages/pip/_vendor/colorama/win32.py
.venv/lib/python3.9/site-packages/pip/_vendor/colorama/winterm.py
.venv/lib/python3.9/site-packages/pip/_vendor/distlib/__init__.py
.venv/lib/python3.9/site-packages/pip/_vendor/distlib/_backport/__init__.py
.venv/lib/python3.9/site-packages/pip/_vendor/distlib/_backport/misc.py
.venv/lib/python3.9/site-packages/pip/_vendor/distlib/_backport/shutil.py
.venv/lib/python3.9/site-packages/pip/_vendor/distlib/_backport/sysconfig.cfg
.venv/lib/python3.9/site-packages/pip/_vendor/distlib/_backport/sysconfig.py
.venv/lib/python3.9/site-packages/pip/_vendor/distlib/_backport/tarfile.py
.venv/lib/python3.9/site-packages/pip/_vendor/distlib/compat.py
.venv/lib/python3.9/site-packages/pip/_vendor/distlib/database.py
.venv/lib/python3.9/site-packages/pip/_vendor/distlib/index.py
.venv/lib/python3.9/site-packages/pip/_vendor/distlib/locators.py
.venv/lib/python3.9/site-packages/pip/_vendor/distlib/manifest.py
.venv/lib/python3.9/site-packages/pip/_vendor/distlib/markers.py
.venv/lib/python3.9/site-packages/pip/_vendor/distlib/metadata.py
.venv/lib/python3.9/site-packages/pip/_vendor/distlib/resources.py
.venv/lib/python3.9/site-packages/pip/_vendor/distlib/scripts.py
.venv/lib/python3.9/site-packages/pip/_vendor/distlib/t32.exe
.venv/lib/python3.9/site-packages/pip/_vendor/distlib/t64.exe
.venv/lib/python3.9/site-packages/pip/_vendor/distlib/util.py
.venv/lib/python3.9/site-packages/pip/_vendor/distlib/version.py
.venv/lib/python3.9/site-packages/pip/_vendor/distlib/w32.exe
.venv/lib/python3.9/site-packages/pip/_vendor/distlib/w64.exe
.venv/lib/python3.9/site-packages/pip/_vendor/distlib/wheel.py
.venv/lib/python3.9/site-packages/pip/_vendor/distro.py
.venv/lib/python3.9/site-packages/pip/_vendor/html5lib/__init__.py
.venv/lib/python3.9/site-packages/pip/_vendor/html5lib/_ihatexml.py
.venv/lib/python3.9/site-packages/pip/_vendor/html5lib/_inputstream.py
.venv/lib/python3.9/site-packages/pip/_vendor/html5lib/_tokenizer.py
.venv/lib/python3.9/site-packages/pip/_vendor/html5lib/_trie/__init__.py
.venv/lib/python3.9/site-packages/pip/_vendor/html5lib/_trie/_base.py
.venv/lib/python3.9/site-packages/pip/_vendor/html5lib/_trie/py.py
.venv/lib/python3.9/site-packages/pip/_vendor/html5lib/_utils.py
.venv/lib/python3.9/site-packages/pip/_vendor/html5lib/constants.py
.venv/lib/python3.9/site-packages/pip/_vendor/html5lib/filters/__init__.py
.venv/lib/python3.9/site-packages/pip/_vendor/html5lib/filters/alphabeticalattributes.py
.venv/lib/python3.9/site-packages/pip/_vendor/html5lib/filters/base.py
.venv/lib/python3.9/site-packages/pip/_vendor/html5lib/filters/inject_meta_charset.py
.venv/lib/python3.9/site-packages/pip/_vendor/html5lib/filters/lint.py
.venv/lib/python3.9/site-packages/pip/_vendor/html5lib/filters/optionaltags.py
.venv/lib/python3.9/site-packages/pip/_vendor/html5lib/filters/sanitizer.py
.venv/lib/python3.9/site-packages/pip/_vendor/html5lib/filters/whitespace.py
.venv/lib/python3.9/site-packages/pip/_vendor/html5lib/html5parser.py
.venv/lib/python3.9/site-packages/pip/_vendor/html5lib/serializer.py
.venv/lib/python3.9/site-packages/pip/_vendor/html5lib/treeadapters/__init__.py
.venv/lib/python3.9/site-packages/pip/_vendor/html5lib/treeadapters/genshi.py
.venv/lib/python3.9/site-packages/pip/_vendor/html5lib/treeadapters/sax.py
.venv/lib/python3.9/site-packages/pip/_vendor/html5lib/treebuilders/__init__.py
.venv/lib/python3.9/site-packages/pip/_vendor/html5lib/treebuilders/base.py
.venv/lib/python3.9/site-packages/pip/_vendor/html5lib/treebuilders/dom.py
.venv/lib/python3.9/site-packages/pip/_vendor/html5lib/treebuilders/etree.py
.venv/lib/python3.9/site-packages/pip/_vendor/html5lib/treebuilders/etree_lxml.py
.venv/lib/python3.9/site-packages/pip/_vendor/html5lib/treewalkers/__init__.py
.venv/lib/python3.9/site-packages/pip/_vendor/html5lib/treewalkers/base.py
.venv/lib/python3.9/site-packages/pip/_vendor/html5lib/treewalkers/dom.py
.venv/lib/python3.9/site-packages/pip/_vendor/html5lib/treewalkers/etree.py
.venv/lib/python3.9/site-packages/pip/_vendor/html5lib/treewalkers/etree_lxml.py
.venv/lib/python3.9/site-packages/pip/_vendor/html5lib/treewalkers/genshi.py
.venv/lib/python3.9/site-packages/pip/_vendor/idna/__init__.py
.venv/lib/python3.9/site-packages/pip/_vendor/idna/codec.py
.venv/lib/python3.9/site-packages/pip/_vendor/idna/compat.py
.venv/lib/python3.9/site-packages/pip/_vendor/idna/core.py
.venv/lib/python3.9/site-packages/pip/_vendor/idna/idnadata.py
.venv/lib/python3.9/site-packages/pip/_vendor/idna/intranges.py
.venv/lib/python3.9/site-packages/pip/_vendor/idna/package_data.py
.venv/lib/python3.9/site-packages/pip/_vendor/idna/uts46data.py
.venv/lib/python3.9/site-packages/pip/_vendor/msgpack/__init__.py
.venv/lib/python3.9/site-packages/pip/_vendor/msgpack/_version.py
.venv/lib/python3.9/site-packages/pip/_vendor/msgpack/exceptions.py
.venv/lib/python3.9/site-packages/pip/_vendor/msgpack/ext.py
.venv/lib/python3.9/site-packages/pip/_vendor/msgpack/fallback.py
.venv/lib/python3.9/site-packages/pip/_vendor/packaging/__about__.py
.venv/lib/python3.9/site-packages/pip/_vendor/packaging/__init__.py
.venv/lib/python3.9/site-packages/pip/_vendor/packaging/_manylinux.py
.venv/lib/python3.9/site-packages/pip/_vendor/packaging/_musllinux.py
.venv/lib/python3.9/site-packages/pip/_vendor/packaging/_structures.py
.venv/lib/python3.9/site-packages/pip/_vendor/packaging/markers.py
.venv/lib/python3.9/site-packages/pip/_vendor/packaging/requirements.py
.venv/lib/python3.9/site-packages/pip/_vendor/packaging/specifiers.py
.venv/lib/python3.9/site-packages/pip/_vendor/packaging/tags.py
.venv/lib/python3.9/site-packages/pip/_vendor/packaging/utils.py
.venv/lib/python3.9/site-packages/pip/_vendor/packaging/version.py
.venv/lib/python3.9/site-packages/pip/_vendor/pep517/__init__.py
.venv/lib/python3.9/site-packages/pip/_vendor/pep517/build.py
.venv/lib/python3.9/site-packages/pip/_vendor/pep517/check.py
.venv/lib/python3.9/site-packages/pip/_vendor/pep517/colorlog.py
.venv/lib/python3.9/site-packages/pip/_vendor/pep517/compat.py
.venv/lib/python3.9/site-packages/pip/_vendor/pep517/dirtools.py
.venv/lib/python3.9/site-packages/pip/_vendor/pep517/envbuild.py
.venv/lib/python3.9/site-packages/pip/_vendor/pep517/in_process/__init__.py
.venv/lib/python3.9/site-packages/pip/_vendor/pep517/in_process/_in_process.py
.venv/lib/python3.9/site-packages/pip/_vendor/pep517/meta.py
.venv/lib/python3.9/site-packages/pip/_vendor/pep517/wrappers.py
.venv/lib/python3.9/site-packages/pip/_vendor/pkg_resources/__init__.py
.venv/lib/python3.9/site-packages/pip/_vendor/pkg_resources/py31compat.py
.venv/lib/python3.9/site-packages/pip/_vendor/progress/__init__.py
.venv/lib/python3.9/site-packages/pip/_vendor/progress/bar.py
.venv/lib/python3.9/site-packages/pip/_vendor/progress/counter.py
.venv/lib/python3.9/site-packages/pip/_vendor/progress/spinner.py
.venv/lib/python3.9/site-packages/pip/_vendor/pyparsing.py
.venv/lib/python3.9/site-packages/pip/_vendor/requests/__init__.py
.venv/lib/python3.9/site-packages/pip/_vendor/requests/__version__.py
.venv/lib/python3.9/site-packages/pip/_vendor/requests/_internal_utils.py
.venv/lib/python3.9/site-packages/pip/_vendor/requests/adapters.py
.venv/lib/python3.9/site-packages/pip/_vendor/requests/api.py
.venv/lib/python3.9/site-packages/pip/_vendor/requests/auth.py
.venv/lib/python3.9/site-packages/pip/_vendor/requests/certs.py
.venv/lib/python3.9/site-packages/pip/_vendor/requests/compat.py
.venv/lib/python3.9/site-packages/pip/_vendor/requests/cookies.py
.venv/lib/python3.9/site-packages/pip/_vendor/requests/exceptions.py
.venv/lib/python3.9/site-packages/pip/_vendor/requests/help.py
.venv/lib/python3.9/site-packages/pip/_vendor/requests/hooks.py
.venv/lib/python3.9/site-packages/pip/_vendor/requests/models.py
.venv/lib/python3.9/site-packages/pip/_vendor/requests/packages.py
.venv/lib/python3.9/site-packages/pip/_vendor/requests/sessions.py
.venv/lib/python3.9/site-packages/pip/_vendor/requests/status_codes.py
.venv/lib/python3.9/site-packages/pip/_vendor/requests/structures.py
.venv/lib/python3.9/site-packages/pip/_vendor/requests/utils.py
.venv/lib/python3.9/site-packages/pip/_vendor/resolvelib/__init__.py
.venv/lib/python3.9/site-packages/pip/_vendor/resolvelib/compat/__init__.py
.venv/lib/python3.9/site-packages/pip/_vendor/resolvelib/compat/collections_abc.py
.venv/lib/python3.9/site-packages/pip/_vendor/resolvelib/providers.py
.venv/lib/python3.9/site-packages/pip/_vendor/resolvelib/reporters.py
.venv/lib/python3.9/site-packages/pip/_vendor/resolvelib/resolvers.py
.venv/lib/python3.9/site-packages/pip/_vendor/resolvelib/structs.py
.venv/lib/python3.9/site-packages/pip/_vendor/six.py
.venv/lib/python3.9/site-packages/pip/_vendor/tenacity/__init__.py
.venv/lib/python3.9/site-packages/pip/_vendor/tenacity/_asyncio.py
.venv/lib/python3.9/site-packages/pip/_vendor/tenacity/_utils.py
.venv/lib/python3.9/site-packages/pip/_vendor/tenacity/after.py
.venv/lib/python3.9/site-packages/pip/_vendor/tenacity/before.py
.venv/lib/python3.9/site-packages/pip/_vendor/tenacity/before_sleep.py
.venv/lib/python3.9/site-packages/pip/_vendor/tenacity/nap.py
.venv/lib/python3.9/site-packages/pip/_vendor/tenacity/retry.py
.venv/lib/python3.9/site-packages/pip/_vendor/tenacity/stop.py
.venv/lib/python3.9/site-packages/pip/_vendor/tenacity/tornadoweb.py
.venv/lib/python3.9/site-packages/pip/_vendor/tenacity/wait.py
.venv/lib/python3.9/site-packages/pip/_vendor/tomli/__init__.py
.venv/lib/python3.9/site-packages/pip/_vendor/tomli/_parser.py
.venv/lib/python3.9/site-packages/pip/_vendor/tomli/_re.py
.venv/lib/python3.9/site-packages/pip/_vendor/urllib3/__init__.py
.venv/lib/python3.9/site-packages/pip/_vendor/urllib3/_collections.py
.venv/lib/python3.9/site-packages/pip/_vendor/urllib3/_version.py
.venv/lib/python3.9/site-packages/pip/_vendor/urllib3/connection.py
.venv/lib/python3.9/site-packages/pip/_vendor/urllib3/connectionpool.py
.venv/lib/python3.9/site-packages/pip/_vendor/urllib3/contrib/__init__.py
.venv/lib/python3.9/site-packages/pip/_vendor/urllib3/contrib/_appengine_environ.py
.venv/lib/python3.9/site-packages/pip/_vendor/urllib3/contrib/_securetransport/__init__.py
.venv/lib/python3.9/site-packages/pip/_vendor/urllib3/contrib/_securetransport/bindings.py
.venv/lib/python3.9/site-packages/pip/_vendor/urllib3/contrib/_securetransport/low_level.py
.venv/lib/python3.9/site-packages/pip/_vendor/urllib3/contrib/appengine.py
.venv/lib/python3.9/site-packages/pip/_vendor/urllib3/contrib/ntlmpool.py
.venv/lib/python3.9/site-packages/pip/_vendor/urllib3/contrib/pyopenssl.py
.venv/lib/python3.9/site-packages/pip/_vendor/urllib3/contrib/securetransport.py
.venv/lib/python3.9/site-packages/pip/_vendor/urllib3/contrib/socks.py
.venv/lib/python3.9/site-packages/pip/_vendor/urllib3/exceptions.py
.venv/lib/python3.9/site-packages/pip/_vendor/urllib3/fields.py
.venv/lib/python3.9/site-packages/pip/_vendor/urllib3/filepost.py
.venv/lib/python3.9/site-packages/pip/_vendor/urllib3/packages/__init__.py
.venv/lib/python3.9/site-packages/pip/_vendor/urllib3/packages/backports/__init__.py
.venv/lib/python3.9/site-packages/pip/_vendor/urllib3/packages/backports/makefile.py
.venv/lib/python3.9/site-packages/pip/_vendor/urllib3/packages/six.py
.venv/lib/python3.9/site-packages/pip/_vendor/urllib3/packages/ssl_match_hostname/__init__.py
.venv/lib/python3.9/site-packages/pip/_vendor/urllib3/packages/ssl_match_hostname/_implementation.py
.venv/lib/python3.9/site-packages/pip/_vendor/urllib3/poolmanager.py
.venv/lib/python3.9/site-packages/pip/_vendor/urllib3/request.py
.venv/lib/python3.9/site-packages/pip/_vendor/urllib3/response.py
.venv/lib/python3.9/site-packages/pip/_vendor/urllib3/util/__init__.py
.venv/lib/python3.9/site-packages/pip/_vendor/urllib3/util/connection.py
.venv/lib/python3.9/site-packages/pip/_vendor/urllib3/util/proxy.py
.venv/lib/python3.9/site-packages/pip/_vendor/urllib3/util/queue.py
.venv/lib/python3.9/site-packages/pip/_vendor/urllib3/util/request.py
.venv/lib/python3.9/site-packages/pip/_vendor/urllib3/util/response.py
.venv/lib/python3.9/site-packages/pip/_vendor/urllib3/util/retry.py
.venv/lib/python3.9/site-packages/pip/_vendor/urllib3/util/ssl_.py
.venv/lib/python3.9/site-packages/pip/_vendor/urllib3/util/ssltransport.py
.venv/lib/python3.9/site-packages/pip/_vendor/urllib3/util/timeout.py
.venv/lib/python3.9/site-packages/pip/_vendor/urllib3/util/url.py
.venv/lib/python3.9/site-packages/pip/_vendor/urllib3/util/wait.py
.venv/lib/python3.9/site-packages/pip/_vendor/vendor.txt
.venv/lib/python3.9/site-packages/pip/_vendor/webencodings/__init__.py
.venv/lib/python3.9/site-packages/pip/_vendor/webencodings/labels.py
.venv/lib/python3.9/site-packages/pip/_vendor/webencodings/mklabels.py
.venv/lib/python3.9/site-packages/pip/_vendor/webencodings/tests.py
.venv/lib/python3.9/site-packages/pip/_vendor/webencodings/x_user_defined.py
.venv/lib/python3.9/site-packages/pip/py.typed
.venv/lib/python3.9/site-packages/pkg_resources/__init__.py
.venv/lib/python3.9/site-packages/pkg_resources/_vendor/__init__.py
.venv/lib/python3.9/site-packages/pkg_resources/_vendor/appdirs.py
.venv/lib/python3.9/site-packages/pkg_resources/_vendor/packaging/__about__.py
.venv/lib/python3.9/site-packages/pkg_resources/_vendor/packaging/__init__.py
.venv/lib/python3.9/site-packages/pkg_resources/_vendor/packaging/_compat.py
.venv/lib/python3.9/site-packages/pkg_resources/_vendor/packaging/_structures.py
.venv/lib/python3.9/site-packages/pkg_resources/_vendor/packaging/_typing.py
.venv/lib/python3.9/site-packages/pkg_resources/_vendor/packaging/markers.py
.venv/lib/python3.9/site-packages/pkg_resources/_vendor/packaging/requirements.py
.venv/lib/python3.9/site-packages/pkg_resources/_vendor/packaging/specifiers.py
.venv/lib/python3.9/site-packages/pkg_resources/_vendor/packaging/tags.py
.venv/lib/python3.9/site-packages/pkg_resources/_vendor/packaging/utils.py
.venv/lib/python3.9/site-packages/pkg_resources/_vendor/packaging/version.py
.venv/lib/python3.9/site-packages/pkg_resources/_vendor/pyparsing.py
.venv/lib/python3.9/site-packages/pkg_resources/extern/__init__.py
.venv/lib/python3.9/site-packages/pkg_resources/tests/data/my-test-package-source/setup.py
.venv/lib/python3.9/site-packages/setuptools-58.0.4.dist-info/INSTALLER
.venv/lib/python3.9/site-packages/setuptools-58.0.4.dist-info/LICENSE
.venv/lib/python3.9/site-packages/setuptools-58.0.4.dist-info/METADATA
.venv/lib/python3.9/site-packages/setuptools-58.0.4.dist-info/RECORD
.venv/lib/python3.9/site-packages/setuptools-58.0.4.dist-info/REQUESTED
.venv/lib/python3.9/site-packages/setuptools-58.0.4.dist-info/WHEEL
.venv/lib/python3.9/site-packages/setuptools-58.0.4.dist-info/entry_points.txt
.venv/lib/python3.9/site-packages/setuptools-58.0.4.dist-info/top_level.txt
.venv/lib/python3.9/site-packages/setuptools/__init__.py
.venv/lib/python3.9/site-packages/setuptools/_deprecation_warning.py
.venv/lib/python3.9/site-packages/setuptools/_distutils/__init__.py
.venv/lib/python3.9/site-packages/setuptools/_distutils/_msvccompiler.py
.venv/lib/python3.9/site-packages/setuptools/_distutils/archive_util.py
.venv/lib/python3.9/site-packages/setuptools/_distutils/bcppcompiler.py
.venv/lib/python3.9/site-packages/setuptools/_distutils/ccompiler.py
.venv/lib/python3.9/site-packages/setuptools/_distutils/cmd.py
.venv/lib/python3.9/site-packages/setuptools/_distutils/command/__init__.py
.venv/lib/python3.9/site-packages/setuptools/_distutils/command/bdist.py
.venv/lib/python3.9/site-packages/setuptools/_distutils/command/bdist_dumb.py
.venv/lib/python3.9/site-packages/setuptools/_distutils/command/bdist_msi.py
.venv/lib/python3.9/site-packages/setuptools/_distutils/command/bdist_rpm.py
.venv/lib/python3.9/site-packages/setuptools/_distutils/command/bdist_wininst.py
.venv/lib/python3.9/site-packages/setuptools/_distutils/command/build.py
.venv/lib/python3.9/site-packages/setuptools/_distutils/command/build_clib.py
.venv/lib/python3.9/site-packages/setuptools/_distutils/command/build_ext.py
.venv/lib/python3.9/site-packages/setuptools/_distutils/command/build_py.py
.venv/lib/python3.9/site-packages/setuptools/_distutils/command/build_scripts.py
.venv/lib/python3.9/site-packages/setuptools/_distutils/command/check.py
.venv/lib/python3.9/site-packages/setuptools/_distutils/command/clean.py
.venv/lib/python3.9/site-packages/setuptools/_distutils/command/config.py
.venv/lib/python3.9/site-packages/setuptools/_distutils/command/install.py
.venv/lib/python3.9/site-packages/setuptools/_distutils/command/install_data.py
.venv/lib/python3.9/site-packages/setuptools/_distutils/command/install_egg_info.py
.venv/lib/python3.9/site-packages/setuptools/_distutils/command/install_headers.py
.venv/lib/python3.9/site-packages/setuptools/_distutils/command/install_lib.py
.venv/lib/python3.9/site-packages/setuptools/_distutils/command/install_scripts.py
.venv/lib/python3.9/site-packages/setuptools/_distutils/command/py37compat.py
.venv/lib/python3.9/site-packages/setuptools/_distutils/command/register.py
.venv/lib/python3.9/site-packages/setuptools/_distutils/command/sdist.py
.venv/lib/python3.9/site-packages/setuptools/_distutils/command/upload.py
.venv/lib/python3.9/site-packages/setuptools/_distutils/config.py
.venv/lib/python3.9/site-packages/setuptools/_distutils/core.py
.venv/lib/python3.9/site-packages/setuptools/_distutils/cygwinccompiler.py
.venv/lib/python3.9/site-packages/setuptools/_distutils/debug.py
.venv/lib/python3.9/site-packages/setuptools/_distutils/dep_util.py
.venv/lib/python3.9/site-packages/setuptools/_distutils/dir_util.py
.venv/lib/python3.9/site-packages/setuptools/_distutils/dist.py
.venv/lib/python3.9/site-packages/setuptools/_distutils/errors.py
.venv/lib/python3.9/site-packages/setuptools/_distutils/extension.py
.venv/lib/python3.9/site-packages/setuptools/_distutils/fancy_getopt.py
.venv/lib/python3.9/site-packages/setuptools/_distutils/file_util.py
.venv/lib/python3.9/site-packages/setuptools/_distutils/filelist.py
.venv/lib/python3.9/site-packages/setuptools/_distutils/log.py
.venv/lib/python3.9/site-packages/setuptools/_distutils/msvc9compiler.py
.venv/lib/python3.9/site-packages/setuptools/_distutils/msvccompiler.py
.venv/lib/python3.9/site-packages/setuptools/_distutils/py35compat.py
.venv/lib/python3.9/site-packages/setuptools/_distutils/py38compat.py
.venv/lib/python3.9/site-packages/setuptools/_distutils/spawn.py
.venv/lib/python3.9/site-packages/setuptools/_distutils/sysconfig.py
.venv/lib/python3.9/site-packages/setuptools/_distutils/text_file.py
.venv/lib/python3.9/site-packages/setuptools/_distutils/unixccompiler.py
.venv/lib/python3.9/site-packages/setuptools/_distutils/util.py
.venv/lib/python3.9/site-packages/setuptools/_distutils/version.py
.venv/lib/python3.9/site-packages/setuptools/_distutils/versionpredicate.py
.venv/lib/python3.9/site-packages/setuptools/_imp.py
.venv/lib/python3.9/site-packages/setuptools/_vendor/__init__.py
.venv/lib/python3.9/site-packages/setuptools/_vendor/more_itertools/__init__.py
.venv/lib/python3.9/site-packages/setuptools/_vendor/more_itertools/more.py
.venv/lib/python3.9/site-packages/setuptools/_vendor/more_itertools/recipes.py
.venv/lib/python3.9/site-packages/setuptools/_vendor/ordered_set.py
.venv/lib/python3.9/site-packages/setuptools/_vendor/packaging/__about__.py
.venv/lib/python3.9/site-packages/setuptools/_vendor/packaging/__init__.py
.venv/lib/python3.9/site-packages/setuptools/_vendor/packaging/_compat.py
.venv/lib/python3.9/site-packages/setuptools/_vendor/packaging/_structures.py
.venv/lib/python3.9/site-packages/setuptools/_vendor/packaging/_typing.py
.venv/lib/python3.9/site-packages/setuptools/_vendor/packaging/markers.py
.venv/lib/python3.9/site-packages/setuptools/_vendor/packaging/requirements.py
.venv/lib/python3.9/site-packages/setuptools/_vendor/packaging/specifiers.py
.venv/lib/python3.9/site-packages/setuptools/_vendor/packaging/tags.py
.venv/lib/python3.9/site-packages/setuptools/_vendor/packaging/utils.py
.venv/lib/python3.9/site-packages/setuptools/_vendor/packaging/version.py
.venv/lib/python3.9/site-packages/setuptools/_vendor/pyparsing.py
.venv/lib/python3.9/site-packages/setuptools/archive_util.py
.venv/lib/python3.9/site-packages/setuptools/build_meta.py
.venv/lib/python3.9/site-packages/setuptools/cli-32.exe
.venv/lib/python3.9/site-packages/setuptools/cli-64.exe
.venv/lib/python3.9/site-packages/setuptools/cli.exe
.venv/lib/python3.9/site-packages/setuptools/command/__init__.py
.venv/lib/python3.9/site-packages/setuptools/command/alias.py
.venv/lib/python3.9/site-packages/setuptools/command/bdist_egg.py
.venv/lib/python3.9/site-packages/setuptools/command/bdist_rpm.py
.venv/lib/python3.9/site-packages/setuptools/command/build_clib.py
.venv/lib/python3.9/site-packages/setuptools/command/build_ext.py
.venv/lib/python3.9/site-packages/setuptools/command/build_py.py
.venv/lib/python3.9/site-packages/setuptools/command/develop.py
.venv/lib/python3.9/site-packages/setuptools/command/dist_info.py
.venv/lib/python3.9/site-packages/setuptools/command/easy_install.py
.venv/lib/python3.9/site-packages/setuptools/command/egg_info.py
.venv/lib/python3.9/site-packages/setuptools/command/install.py
.venv/lib/python3.9/site-packages/setuptools/command/install_egg_info.py
.venv/lib/python3.9/site-packages/setuptools/command/install_lib.py
.venv/lib/python3.9/site-packages/setuptools/command/install_scripts.py
.venv/lib/python3.9/site-packages/setuptools/command/launcher manifest.xml
.venv/lib/python3.9/site-packages/setuptools/command/py36compat.py
.venv/lib/python3.9/site-packages/setuptools/command/register.py
.venv/lib/python3.9/site-packages/setuptools/command/rotate.py
.venv/lib/python3.9/site-packages/setuptools/command/saveopts.py
.venv/lib/python3.9/site-packages/setuptools/command/sdist.py
.venv/lib/python3.9/site-packages/setuptools/command/setopt.py
.venv/lib/python3.9/site-packages/setuptools/command/test.py
.venv/lib/python3.9/site-packages/setuptools/command/upload.py
.venv/lib/python3.9/site-packages/setuptools/command/upload_docs.py
.venv/lib/python3.9/site-packages/setuptools/config.py
.venv/lib/python3.9/site-packages/setuptools/dep_util.py
.venv/lib/python3.9/site-packages/setuptools/depends.py
.venv/lib/python3.9/site-packages/setuptools/dist.py
.venv/lib/python3.9/site-packages/setuptools/errors.py
.venv/lib/python3.9/site-packages/setuptools/extension.py
.venv/lib/python3.9/site-packages/setuptools/extern/__init__.py
.venv/lib/python3.9/site-packages/setuptools/glob.py
.venv/lib/python3.9/site-packages/setuptools/gui-32.exe
.venv/lib/python3.9/site-packages/setuptools/gui-64.exe
.venv/lib/python3.9/site-packages/setuptools/gui.exe
.venv/lib/python3.9/site-packages/setuptools/installer.py
.venv/lib/python3.9/site-packages/setuptools/launch.py
.venv/lib/python3.9/site-packages/setuptools/monkey.py
.venv/lib/python3.9/site-packages/setuptools/msvc.py
.venv/lib/python3.9/site-packages/setuptools/namespaces.py
.venv/lib/python3.9/site-packages/setuptools/package_index.py
.venv/lib/python3.9/site-packages/setuptools/py34compat.py
.venv/lib/python3.9/site-packages/setuptools/sandbox.py
.venv/lib/python3.9/site-packages/setuptools/script (dev).tmpl
.venv/lib/python3.9/site-packages/setuptools/script.tmpl
.venv/lib/python3.9/site-packages/setuptools/unicode_utils.py
.venv/lib/python3.9/site-packages/setuptools/version.py
.venv/lib/python3.9/site-packages/setuptools/wheel.py
.venv/lib/python3.9/site-packages/setuptools/windows_support.py
.venv/pyvenv.cfg
.venv/smoke-test-report.md
.vscode/mcp.json
TODO.md
backlog.md
bulk_validate_walkthroughs.py
compound_interest.py
generate_weekly_status_report.py
greeting_tool.py
instructions/calculate-compound-interest.agent.md
instructions/create-status-report.agent.md
instructions/creating-instructions.agent.md
instructions/generate-weekly-status-report.agent.md
instructions/main.agent.md
instructions/use-compound-interest.agent.md
instructions/use-simple-interest.agent.md
instructions/write-tests.agent.md
project_spec.md
reports/example.md
reports/instructions.md
reports/template.md
spec/clarify.md
spec/constitution.md
spec/plan.md
spec/specification.md
spec/tasks.md
tests/fixtures/walkthroughs/complete.md
tests/fixtures/walkthroughs/missing-quiz.md
tests/fixtures/walkthroughs/missing-summary.md
tools/compound_interest.py
tools/simple_interest.py
validation-rules.md
work/module-08-report.md
work/module-09-report.md
work/module-10-report.md
work/module-12-report.md
work/module-13-report.md
work/module-14-report.md
work/module-15-report.md
work/module-16-report.md
work/weekly-status-report.md
```

