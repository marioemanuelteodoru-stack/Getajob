# GetaJob — Project Documentation

**Last Updated: 2026-05-01**

## Project Overview

**GetaJob** is a privacy-first, local-first desktop application that helps users systematically apply for jobs at nearby businesses. It combines location data (from Google My Maps/KML exports), Gmail integration, and local contact management into a single, offline-capable application that the user fully controls.

### Vision

- **Disintermediation**: Direct access to job opportunities without recruitment agency gatekeeping
- **User data ownership**: All data (SQLite, Gmail tokens, notes) stays on the user's machine
- **Privacy by design**: No server, no data collection, no GDPR compliance burden
- **Simplicity**: Reuse existing tools (Gmail, Google Maps) rather than duplicating functionality
- **Proximity-first**: Focus on local neighborhoods for blue-collar, temp, gig, and local business jobs

### Architecture

- **Frontend**: Electron (Windows MVP) + React + Vite; Android/iOS via React Native (phase 2)
- **Database**: SQLite (local) with optional encryption via passkeys or passphrases
- **OAuth**: Desktop App OAuth Flow (no server-side keys exposed)
- **Backend**: None for production; local Python FastAPI for optional offline sync
- **Distribution**: Direct download (code-signed executable)

---

## Documentation Structure

This folder contains the canonical documentation for GetaJob architecture, API contracts, feature specifications, and migration plans.

### Key Files (Root)

- **README.md** (this file) — project overview and contributing guide
- **CHANGELOG.md** — dated record of significant changes and decisions
- **PROJECT_STATUS.md** — current build/release status and platform readiness
- **summary.md** — brief operational notes and memory for developers

### Organization by Topic

- **`api/`** — API contracts (OpenAPI YAML) and mock server guide
- **`web-mvp/`** — Web client architecture, module specs, migration plans, local-first implementation
- **`android-mvp/`** — Android app architecture and runtime notes
- **`vision/`** — Strategic direction, distribution strategy, feature vision
- **`design/`** — UX wireframes, color schemes, status system definitions
- **`migration-local-first/`** — Local-first migration plan and implementation roadmap
- **`archive/`** — Previous versions and historical notes
- **`error_log/`** — Build issues, debugging notes, and fixes

---

## Current Project Status (MVP)

### Release Candidate Features

✅ **Completed:**

- Encrypted IndexedDB with WebAuthn passkey unlock
- Windows Electron app building (NSIS installer + portable exe)
- Passkey registration and local unlock flow
- Settings page (logout, database preview, backup download)
- Dev server (Vite) with hot reload
- API contract defined (OpenAPI YAML)
- Data migration scripts ready

⚠️ **In Progress:**

- Passphrase fallback (Argon2/PBKDF2 KDF)
- Contacts list and detail pages UI
- Messages list and compose UI
- Gmail OAuth Desktop App Flow integration

❌ **Post-MVP:**

- Background job processing (search crawler, notifications)
- Multi-device sync (manual backups for MVP; cloud sync in v1.5+)
- Android/iOS versions
- Push notifications and GCM integration

### Platform Readiness

| Platform            | Status | Notes                                                         |
| ------------------- | ------ | ------------------------------------------------------------- |
| **Windows Desktop** | 70%    | App builds; UI partially wired; needs Windows machine testing |
| **Linux Desktop**   | 60%    | AppImage built; Electron config complete                      |
| **Android**         | 20%    | React Native scaffolded; not active until Web MVP complete    |
| **Backend API**     | 60%    | Endpoints defined; OAuth needs refactor for local-first       |
| **Database**        | 80%    | Schema complete; encryption working; migrations incomplete    |

---

## Canonical Data Models

### Status Keys (Shared Across Platforms)

| Key               | Color     | Meaning                         |
| ----------------- | --------- | ------------------------------- |
| `none`            | 🟢 Green  | Not yet contacted               |
| `initial_sent`    | 🟡 Yellow | First outreach sent             |
| `initial_replied` | 🔴 Red    | Rejected or no response         |
| `interview`       | 🔵 Blue   | Active conversation / interview |
| `hired`           | ⚪ Grey   | Hired or no longer operating    |

### Place (Contact) Model

```json
{
  "id": "uuid",
  "name": "Business Name",
  "address": "123 Main St",
  "email": "contact@business.com",
  "phone": "+1-555-0000",
  "website": "https://business.com",
  "coordinates": { "lat": 51.5, "lng": -0.1 },
  "status": "none",
  "notes": "User notes",
  "created_at": "2026-05-01T00:00:00Z",
  "updated_at": "2026-05-01T00:00:00Z"
}
```

---

## Quick Start for Development

### Prerequisites

- Node.js 18+ and npm
- Python 3.9+ (optional, for backend dev)
- Git

### Web MVP Dev

```bash
cd frontend/web-mvp
npm install
npm run dev
# Open http://localhost:5174/unlock
```

### Build Windows App

```bash
cd frontend/web-mvp
npm run build
npx electron-builder --win --x64
# Output: dist/GetaJob Setup 0.1.0.exe (installer)
```

### Run Mock API Server

```bash
cd docs/web-mvp/api
npx @stoplight/prism-cli mock openapi.yaml --port 4010
# Frontend can point at http://localhost:4010
```

---

## Contributing Guidelines

### Adding or Updating Documentation

1. Update the relevant file in `docs/` (e.g., `docs/web-mvp/CONTACTS.md` for Contacts module)
2. Add `Last Updated: YYYY-MM-DD` at the top of the file
3. For significant changes, add a line to `docs/CHANGELOG.md`
4. Commit with a clear message (e.g., "[docs] Update Contacts module spec")

### File Naming & Structure

- Use descriptive names: `CONTACTS.md`, `STATUS_SYSTEM.md`, etc.
- Use `docs/MODULE_TEMPLATE.md` as a template for new specs
- Keep one file per module or cross-cutting system
- Archive old versions in `docs/archive/`

### Code Documentation

- Inline comments in code should reference doc files: `see docs/web-mvp/CONTACTS.md`
- Keep API documentation in `docs/web-mvp/api/openapi.yaml` (source of truth)
- Use TypeScript/Python type hints; document non-obvious logic

---

## Key Decision Log

**2026-05-01:** Finalized local-first architecture (no cloud server). Updated all base documentation to reflect current state.
**2026-04-29:** Approved web-mvp migration plan. Scaffolded encrypted IndexedDB and passkey unlock flow.
**2026-04-28:** Set MVP feature priority: Contacts, Messages, Gmail OAuth.
**2026-04-20:** Approved Windows Electron as primary desktop target.
**2026-04-15:** Decided on local-first encryption + optional passphrase fallback.

_For complete decision history, see `CHANGELOG.md`._

---

## Links to Key Artifacts

- API Contract: [docs/web-mvp/api/openapi.yaml](web-mvp/api/openapi.yaml)
- Web MVP Overview: [docs/web-mvp/OVERVIEW.md](web-mvp/OVERVIEW.md)
- Android Feature Map: [docs/web-mvp/ANDROID_FEATURE_MAP.md](web-mvp/ANDROID_FEATURE_MAP.md)
- Local-First Implementation: [docs/web-mvp/LOCAL-FIRST-README.md](web-mvp/LOCAL-FIRST-README.md)
- Status Indicator System: [docs/web-mvp/level-2-foundations/STATUS_INDICATOR_SYSTEM.md](web-mvp/level-2-foundations/STATUS_INDICATOR_SYSTEM.md)

---

_GetaJob is a privacy-first job application tracker. Contribute improvements via commits to `docs/CHANGELOG.md`._
