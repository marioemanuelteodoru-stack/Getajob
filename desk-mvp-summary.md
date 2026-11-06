# Desk MVP Summary

## Direction
GetaJob will move from a web-style architecture (frontend + HTTP backend) to a true desktop-first architecture.

The new Desk MVP will be a local Windows application with:
- No local HTTP API between UI and business logic.
- A local SQLite database stored on the user machine.
- Secure email integration that connects only when sending/receiving.
- Development-only test tooling (dummy data and dummy email) that is not shipped in production.

## Product Goals
1. Contacts app behavior: local personal database on user disk.
2. Email workflow: send and receive using user mail accounts (later production mode).
3. Status workflow: maintain contact status based on email actions/events.
4. Map support: OpenStreetMap or Google Maps integration for contact locations.

## Security Position
- Default to offline/local operation.
- Remove localhost endpoints and IP-exposed app surfaces.
- Use minimal, reviewed, pinned dependencies.
- Use OS-protected credential storage for real email mode.

## Build Modes
1. Development/QA Mode (internal only)
- Empty database on first launch.
- Optional dummy data generation.
- Dummy email inbox/outbox simulation.
- No real provider connection unless explicitly enabled for test scenarios.
- Not included in production builds.

2. Production User Mode
- Secure SMTP/IMAP or OAuth-enabled provider integration.
- TLS-only network communication.
- Least-privilege sync behavior.
- End users connect only their real email account (for example Gmail OAuth or SMTP/IMAP).
- No dummy email or dummy data controls available.

## Success Criteria for Desk MVP
- Installer deploys app and first run creates an empty DB reliably.
- App starts successfully with zero records.
- Development builds provide dummy data and dummy email flows for internal validation.
- Production builds provide only real email account integration.
