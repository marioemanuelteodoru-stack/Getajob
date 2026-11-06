# Desk MVP Final Migration and Refactoring Plan

## Scope
This plan is derived directly from the strategy in desk-mvp-refactorying-strategy.md and is execution-ready for a two-person team.

Assumptions:
- Team size: 2 (You + Copilot pair-programming support).
- Current app is not usable in production due to the HTTP runtime architecture, but core business logic and domain code remain reusable.
- Production build must not include mock email or dummy data tools.
- Development workflow is local and air-gapped (no Git or internet dependency required for day-to-day work).

## Target Outcome
Ship a professional desktop-first Windows app that:
1. Runs without local HTTP API exposure.
2. Creates and starts with an empty local SQLite database on first launch.
3. Supports secure real email connection (Gmail OAuth and/or SMTP/IMAP).
4. Preserves status workflow logic between contacts and emails.
5. Includes map integration.

## Stack Lock (Approved)
1. Runtime stack:
- .NET 8 (LTS)
- C# 12
- WPF desktop UI using MVVM
- SQLite local database
- Single-process desktop runtime (no localhost HTTP server)

2. Production dependencies (allowlist):
- Microsoft.Data.Sqlite (DB connectivity)
- Dapper (lightweight data access)
- MailKit (SMTP/IMAP transport)
- MimeKit (message parsing/composition)
- Mapsui (OpenStreetMap rendering)

3. Conditional dependency:
- Google.Apis.Auth only if Gmail OAuth is included in MVP.
- If Gmail OAuth is postponed, Gmail can be supported later while shipping SMTP/IMAP first.

4. Explicitly excluded from production runtime:
- FastAPI runtime
- Vite/React/Electron runtime
- Any localhost API bridge

5. Security controls bound to this stack:
- Windows OS-protected credential storage (DPAPI/Credential Locker wrapper)
- TLS-only SMTP/IMAP connections
- No plaintext secrets in config, DB, logs, or crash dumps

## Dependency Policy (Locked)
1. NuGet lock file required and stored in the project.
2. Exact version pinning for all production packages.
3. New package requires manual approval entry in a dependency allowlist file.
4. Build/release gate scripts fail when:
- Unapproved package appears.
- Lockfile changes without approval.
- SBOM generation fails.
5. Package update cadence:
- Security updates: immediate patch window.
- Non-security updates: batched and regression-tested.

## Air-Gapped Development Policy (Locked)
1. No Git is required for development or release.
2. Development machine can run fully offline after initial tool/package transfer.
3. NuGet source configuration must use local offline feed only.
4. Public package sources are disabled in the development environment.
5. Mock email provider is mandatory in development profile to avoid external email traffic.
6. Production profile is built from the same source tree with dev-only modules excluded by gate checks.
7. Package intake process:
- Download and verify approved packages on a separate connected machine.
- Record hashes/signatures.
- Transfer by removable media.
- Import into offline local NuGet feed.

## Reuse Strategy (What We Keep)
1. Keep and port as-is in behavior:
- Contact-email status transition rules
- Message/contact linking rules
- Template behavior
- Attachment/default attachment behavior

2. Keep concept, rebuild implementation:
- Services become in-process application services (no route layer)
- UI screens are rebuilt natively in WPF while preserving flows and validations

3. Do not carry forward:
- HTTP endpoint contracts as runtime boundary
- Browser-style frontend/backend split

## Solution Structure (Locked)
1. src/GetaJob.Desktop
- WPF app shell, views, view models, navigation

2. src/GetaJob.Application
- Use cases and orchestrators
- Service interfaces (IContactService, IMessageService, IStatusService, IEmailProvider)

3. src/GetaJob.Domain
- Entities, value objects, status rules, domain policies

4. src/GetaJob.Infrastructure
- SQLite repositories (Dapper)
- Email providers (Mock in dev-only build profile, Real in production)
- Secret storage adapter
- Map provider adapter

5. src/GetaJob.Bootstrap
- First-run DB initialization
- Schema migrations
- Environment/build profile checks

6. tests/GetaJob.Tests.Unit
- Domain and service rule tests

7. tests/GetaJob.Tests.Integration
- DB bootstrap, repository, provider integration tests

8. build/
- Installer scripts
- Signing and SBOM scripts
- Artifact purity checks (no dev-only modules)
- Offline dependency gate scripts (allowlist + lockfile + hash verification)

## Prepared Offline Artifacts (Created)
1. Root local package source config:
- nuget.config
2. Dependency allowlist and integrity artifacts:
- build/dependency-allowlist.json
- build/package-hashes.sha256
3. Offline setup and validation scripts:
- build/offline-bootstrap-checklist.md
- build/verify-offline-dependencies.ps1

## Delivery Timeline (10 Weeks)
1. Weeks 1-2: Phase 0 + Phase 1 (baseline lock + empty DB bootstrap).
2. Weeks 3-4: Phase 2 (in-process service extraction).
3. Weeks 5-6: Phase 3 + Phase 4 (dev-only mock email + dummy data).
4. Weeks 7-8: Phase 5 (real email provider integration + secure storage).
5. Weeks 9-10: Phase 6 (installer, hardening, release readiness).

## Ownership Model
1. You (Product/Architecture Owner):
- Final decisions on UX, stack selection, security policy acceptance.
- Provider/business rules validation.
- Release approvals.

2. Copilot (Engineering Execution Partner):
- Code implementation, refactors, tests, docs, and build gating setup.
- Security checks and dependency governance automation.
- Migration scripts, installer mechanics, and release verification scripts.

## Work Breakdown by Phase

## Phase 0 - Baseline and Safety Rails (Week 1)
Objectives:
- Lock current business behavior before architecture changes.

Tasks:
1. Extract and document status transition rules as a single source of truth.
2. Capture email workflow rules (send, receive, sync, message-linking to contacts).
3. Build regression tests for:
- Status transitions by event.
- Contact/email link consistency.
- Startup behavior assumptions.
4. Define dependency allowlist and exact version pinning rules.

Deliverables:
- Behavior spec document.
- Regression test suite v1.
- Dependency policy document.

Estimate:
- 4-5 working days.

Acceptance Criteria:
- Tests pass against current behavior.
- Rule coverage for core status/email flows is in place.

## Phase 1 - Empty DB Bootstrap (Week 2)
Objectives:
- Fresh install always boots with an empty DB.

Tasks:
1. Implement first-run bootstrap module:
- Resolve deterministic per-user app data directory.
- Create DB file when missing.
- Create schema only.
- Insert technical metadata only (for example schema_version table).
2. Remove startup seeding of business entities.
3. Harden startup for true-empty tables (no implicit required rows).
4. Add Reset Database to Empty action (developer/admin utility).
5. Add install/startup integration tests for empty DB path.

Deliverables:
- Bootstrap module.
- Empty-startup test suite.
- Updated startup docs.

Estimate:
- 5 working days.

Acceptance Criteria:
- Fresh install launches with zero records and no errors.
- All key screens load without seed data.

## Phase 2 - In-Process Service Extraction (Weeks 3-4)
Objectives:
- Remove local REST runtime dependency.

Tasks:
1. Create application service interfaces for contacts, messages, status, templates, attachments.
2. Move business logic out of endpoint/controller layer into in-process services.
3. Replace UI HTTP client calls with direct service invocations.
4. Keep compatibility facade temporarily for migration stability.
5. Remove runtime requirement for localhost backend process.

Deliverables:
- In-process service layer.
- UI wiring to service layer.
- Removal plan for deprecated API layer.

Estimate:
- 8-10 working days.

Acceptance Criteria:
- App runs with no local API server.
- Core flows (contacts/messages/status/templates) operational.

## Phase 3 - Dev-Only Mock Email Environment (Week 5)
Objectives:
- Safe full testing without real mailbox connections.

Tasks:
1. Define EmailProvider contract:
- sendMessage
- fetchNewMessages
- testConnection
2. Implement MockEmailProvider with local outbox and synthetic inbound replies.
3. Add developer-only controls for message simulation.
4. Add build flags that compile/enable mock provider only in development builds.

Deliverables:
- Email provider abstraction.
- Mock provider implementation.
- Dev-only tools panel.

Estimate:
- 4-5 working days.

Acceptance Criteria:
- End-to-end email/status tests pass using mock provider.
- Production build config excludes mock provider module and UI.

## Phase 4 - Dev-Only Dummy Data Framework (Week 6)
Objectives:
- Fast repeatable QA scenario setup.

Tasks:
1. Implement Generate Dummy Data action (development only).
2. Generate realistic contacts, statuses, templates, attachment metadata, and message histories.
3. Support Additive and Replace generation modes.
4. Ensure no dummy tooling appears in production build.

Deliverables:
- Dummy data generator.
- Dev-only command/UI entry point.
- Scenario presets for QA.

Estimate:
- 3-4 working days.

Acceptance Criteria:
- Empty app can be populated with full test scenario in <2 minutes.
- Production artifact has no dummy entry points.

## Phase 5 - Real Email Provider Integration (Weeks 7-8)
Objectives:
- Secure real-world email operation for end users.

Tasks:
1. Implement RealEmailProvider:
- Gmail OAuth flow.
- SMTP/IMAP configuration flow for non-OAuth providers.
2. Enforce TLS and certificate validation.
3. Store secrets/tokens only using OS-protected mechanism.
4. Add sync rules:
- Least-privilege mailbox scope.
- Incremental sync by UID/Message-ID.
- Configurable sync frequency.
5. Add provider conformance tests and failure-recovery tests.

Deliverables:
- Real email provider module.
- Secure credential storage integration.
- Sync engine v1.

Estimate:
- 8-10 working days.

Acceptance Criteria:
- Send/receive works with supported providers.
- No plaintext credential storage.
- Recovery from network/auth failures is graceful.

## Phase 6 - Installer, Packaging, Release Hardening (Weeks 9-10)
Objectives:
- Professional release path with deterministic first-run behavior.

Tasks:
1. Build installer that ships binaries/config only (no populated DB).
2. Implement first-run experience:
- Create empty DB.
- Prompt user to connect real email account.
3. Add uninstall options for user-data retention/removal.
4. Add release gates:
- Verify no mock/dummy modules in production build.
- Dependency allowlist enforcement.
- SBOM generation.
- Build signing and artifact verification.
5. Run full install/upgrade/uninstall test passes across clean Windows environments.

Deliverables:
- Production installer.
- First-run wizard.
- Release checklist and automation.

Estimate:
- 8-10 working days.

Acceptance Criteria:
- Install -> launch -> empty DB -> connect real email works reliably.
- Production artifacts pass all release gates.

## Test Matrix

## A) Startup and Database
1. Fresh install, first run with no DB present.
2. Existing DB with current schema.
3. Existing DB with older schema (migration path).
4. Reset to empty and restart.

Expected:
- No crash, no mandatory seed assumptions, schema valid.

## B) Email and Status Logic
1. Send initial contact email.
2. Receive reply and auto/status update behavior.
3. Multiple threads for same contact.
4. Deleted/archived message handling.

Expected:
- Status transitions remain deterministic and correct.

## C) Security
1. No local listening server required.
2. Credentials not present in logs/files plaintext.
3. TLS required for provider connections.
4. Dependency gate blocks unauthorized packages.

Expected:
- Security controls enforced by tests and build pipeline.

## D) Build Profiles
1. Development build includes mock and dummy tools.
2. Production build excludes mock and dummy tools.

Expected:
- Build-time enforcement passes and artifacts verified.

## Release Gates (Go/No-Go)
1. Gate 1: Behavior lock
- Regression suite green for status/email core flows.

2. Gate 2: Empty DB reliability
- Fresh install startup test passes on all target environments.

3. Gate 3: No localhost runtime dependency
- App starts and functions without backend server process.

4. Gate 4: Security baseline
- Secrets storage validated, TLS enforced, SBOM generated, dependency policy green.

5. Gate 5: Production artifact purity
- No mock provider/dummy tooling included in release binaries.

6. Gate 6: End-to-end acceptance
- Install, connect real account, send/receive, status tracking, and map lookup validated.

## Risks and Response Plan
1. Hidden seed assumptions break empty startup.
- Response: exhaustive empty-DB tests and null-safe UI loading.

2. Logic regressions during service extraction.
- Response: phase-by-phase migration with regression suite as blocker.

3. Provider-specific edge cases.
- Response: provider conformance tests and retry/backoff policies.

4. Security drift via dependencies.
- Response: pinned lockfiles, allowlist gate, SBOM gate.

## Definition of Done
1. Production installer is stable and signed.
2. App launches with a truly empty DB on first run.
3. User connects real email account directly (Gmail OAuth or SMTP/IMAP).
4. Contact-email-status workflow works end-to-end.
5. No local API exposure and no dev-only tooling in production artifacts.

## Immediate Next Actions (This Week)
1. Create the .NET 8 solution skeleton using the locked project structure.
2. Start Phase 0 behavior spec and regression suite.
3. Start Phase 1 empty DB bootstrap implementation.
4. Implement local offline dependency allowlist and lockfile gate scripts.
