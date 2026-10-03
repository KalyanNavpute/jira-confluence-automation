# Implementation Backlog

Checkboxes reflect the current workspace as inspected on 2026-10-03. Complete tasks in phase order unless a dependency allows work to proceed in parallel.

## Setup

- [x] Establish the project repository and `.gitignore`.
- [ ] Set up a reproducible Python development environment and document the supported Python version. (#1)
- [ ] Add dependency management for runtime and development/test dependencies; keep the Jira client choice explicit and minimal.
- [ ] Define the initial configuration contract for project name, owner/team, Jira project or board filter, output directory, and refresh settings.
- [ ] Define where secrets are read from the organization's approved credential mechanism; do not store Jira credentials in source, config files, or logs.
- [ ] Confirm the supported manual-update input format and document a small example fixture for local development.

## Core Features

- [x] Provide a CLI for project, owner, summary, completed work, in-flight work, blockers, next-week priorities, decisions/notes, reporting dates, and output path.
- [x] Generate Markdown with the Executive Summary, Completed, In Flight, Risks / Blockers, Next Week, and Decisions Needed sections.
- [x] Default the reporting week to the current Monday-through-Sunday period and allow explicit date overrides.
- [x] Create the output directory when needed and save the generated Markdown report to the configured path.
- [ ] Replace placeholder CLI defaults for required report data with clear required-input validation; report missing fields without silently inventing project updates.
- [ ] Validate reporting dates, including format, ordering, and week boundaries; return a concise actionable error for invalid values.
- [ ] Normalize manual team updates into a documented internal report structure, preserving ownership, status, milestone/expected completion, risk impact, mitigation, and decision details when supplied.
- [ ] Keep generated bullets concise and distinguish completed items from work still in flight; use explicit `None` text when a section has no entries.
- [ ] Add a refresh workflow that can be invoked independently of report generation, records the last successful refresh time, and makes the latest prior-day data available to report generation.
- [ ] Define and enforce stale-data behavior: show the source/freshness timestamp and warn or fail clearly when refreshed data is older than the agreed threshold.

## Integration

- [ ] Implement Jira authentication using the approved credential source and ensure requests, errors, and debug output never reveal secrets or sensitive issue content unnecessarily.
- [ ] Load the configured Jira project/board filter and retrieve only the issue fields needed for reporting: key, summary, status, assignee/team assignment, due date, milestone, and blocker/impediment indicators.
- [ ] Map Jira responses into the internal report structure and handle pagination, empty results, API errors, and rate limits with useful messages.
- [ ] Aggregate Jira delivery status, completed and in-flight work, milestone progress, overdue issues, and blocker counts without duplicating manual team updates.
- [ ] Merge Jira-derived information with manual project summary, risks, exceptions, next-week priorities, and decision requests; retain manual context where Jira cannot provide it.
- [ ] Persist or cache the daily refreshed Jira snapshot with its retrieval timestamp so weekly report generation uses the latest successful prior-day data.
- [ ] Add a repeatable daily refresh command and document an early-morning local-business-time schedule using the host's scheduler; make refresh failures visible without exposing credentials.
- [ ] Add a repeatable weekly report command that consumes the latest refreshed snapshot plus current manual updates and writes to the designated output folder.
- [ ] Keep email-ready output, real-time dashboards, non-Jira integrations, approvals, and advanced forecasting out of this release.

## Testing

- [ ] Add unit tests for CLI parsing, required-field validation, date defaults/overrides, and invalid date handling.
- [ ] Add formatter tests asserting the required headings, metadata, concise list rendering, empty-section text, and stable Markdown output.
- [ ] Add tests for output persistence, directory creation, and clear handling of unwritable paths.
- [ ] Add manual-input parsing and normalization tests, including incomplete updates and risks/decisions with ownership details.
- [ ] Add Jira client tests using mocked responses for normal results, pagination, empty results, overdue/blocker classification, rate limits, and API/authentication failures.
- [ ] Add aggregation tests proving Jira and manual updates merge without losing manual context or duplicating work items.
- [ ] Add refresh tests for successful snapshots, prior-day freshness, stale cache, failed refresh, and report behavior when Jira is unavailable.
- [ ] Add security-focused tests or checks confirming credentials and sensitive response data are not emitted in command output or logs.
- [ ] Add an end-to-end CLI test that generates a stakeholder-ready Markdown report from representative Jira and manual fixtures.
- [ ] Measure generation with a representative large-team dataset and verify the report workflow completes within the 10-minute target.
- [ ] Run the full test suite and confirm a clean run from the documented development setup.

## Documentation

- [ ] Write a README covering prerequisites, environment setup, configuration, secure Jira credential setup, refresh, and report-generation commands.
- [ ] Document the manual-update schema with a realistic example and instructions for team leads to provide structured, verified inputs.
- [ ] Document Jira filter setup, required permissions/fields, refresh cadence, snapshot freshness, and recovery from common API failures.
- [ ] Document how to configure the daily early-morning refresh and weekly report run on supported host schedulers, including how to inspect failures.
- [ ] Provide a sample generated report and explain the output location and reporting-week/date behavior.
- [ ] Document operational limitations, data-quality expectations, sensitive-data handling, and the deferred email/dashboard scope.
- [ ] Update the existing project task tracking to reflect completed backlog items and remaining work.