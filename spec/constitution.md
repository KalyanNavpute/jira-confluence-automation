# Project Constitution

## Purpose

This constitution establishes the engineering principles and constraints for the Jira and Confluence automation project. Feature specifications, implementation plans, and code changes must follow these principles. When requirements conflict, resolve the conflict explicitly in the relevant feature specification rather than silently weakening this constitution.

## Core Principles

### 1. Specify Behavior Before Implementation

- Describe user needs, expected behavior, constraints, and acceptance criteria before implementation begins.
- Keep specifications focused on observable outcomes; defer implementation choices to the plan unless a technology constraint is stated here.
- Update the specification when approved scope or behavior changes.

### 2. Preserve Clear Application Boundaries

- Use React 18 with Vite for the browser-based frontend.
- Use Node.js with Express for the backend API and integration orchestration.
- Use PostgreSQL 15 for persistent application data, run through Docker for local development and repeatable environments.
- Keep presentation, API, business logic, persistence, and external Jira/Confluence integration responsibilities distinct.
- The frontend must access application data through the backend API; it must not connect directly to PostgreSQL or call privileged Jira/Confluence APIs with server credentials.

### 3. Protect Credentials and Data

- Store secrets outside source control and never expose them in browser code, logs, reports, or error responses.
- Enforce authorization on the backend for protected operations; do not rely on frontend-only checks.
- Request only the Jira and Confluence permissions needed for the specified workflows.
- Validate and constrain external input, and handle sensitive project content according to the organization's data-handling requirements.

### 4. Make Integrations Reliable and Respectful

- Encapsulate Jira and Confluence API access behind backend integration modules.
- Handle authentication failures, rate limits, timeouts, pagination, and transient errors explicitly.
- Make retries bounded and avoid repeating non-idempotent operations without safeguards.
- Keep external API behavior testable through controlled fixtures or mocks; do not require live credentials for routine automated tests.

### 5. Maintain Data Integrity

- Model persisted data and schema changes explicitly, using versioned migrations.
- Use database constraints and transactions where needed to protect invariants and multi-step updates.
- Define synchronization behavior, ownership, and freshness expectations for data copied from Jira or Confluence in the feature specification.
- Do not silently overwrite user-authored data with imported data.

### 6. Verify Changes at the Right Boundaries

- Add or update tests for changed behavior, including relevant API, integration, and persistence boundaries.
- Keep routine tests deterministic and independent of live Jira/Confluence services.
- Validate input, authorization, failure paths, and data integrity for features that cross trust or persistence boundaries.
- A feature is complete only when its acceptance criteria are met and the relevant checks pass, or any remaining gaps are documented.

### 7. Keep Development Reproducible

- Document commands and configuration required to run, test, and build the application.
- Use Docker to provide a repeatable PostgreSQL 15 development database; do not assume a developer has a separately configured local database.
- Keep environment-specific configuration outside committed source and provide safe example configuration where needed.
- Prefer the smallest maintainable solution that meets the specification; avoid unrelated refactoring.

## Technical Constraints

- Frontend: React 18 and Vite.
- Backend: Node.js and Express.
- Database: PostgreSQL 15, provisioned via Docker.
- Jira and Confluence integrations must be performed by the backend.
- Any change to these constraints requires an explicit amendment to this constitution.

## Governance

- This constitution applies to all feature specifications, plans, tasks, code, and tests in the project.
- Each feature plan must identify relevant principles and explain any necessary exception before implementation.
- Amendments must state the reason for the change and identify affected specifications, plans, tests, or documentation.
- Where a feature requirement appears to conflict with a principle, pause implementation until the conflict is resolved and recorded.