# CI/CD Pipeline – Comprehensive Guide

> This document summarizes a Jenkins declarative pipeline powered by a shared library (e.g., `sparkPipeline`). It explains **what it includes**, **how it is set up**, and **how it operates** across CI and CD phases, using **generic naming** throughout.

---

## 1) High‑Level Overview

- **Purpose:** End‑to‑end **CI/CD** for a large front‑end application.
- **CI (Pull Requests):** Install → Static Analysis → Build & Checks → Unit Tests → E2E Gating → Reports & Commit Statuses.
- **CD (Default/Hotfix branches):** Build & Package → Version/Metadata updates → Deploy to Integration → Staging (aka Load Test) → Production (manual gate) with **post‑deploy TAP tests** and notifications.
- **Key Integrations:**
  - **Commit Statuses:** Pending / Success / Failure contexts per stage.
  - **Notifications:** Messages to team chat rooms on success/failure.
  - **Quality Gates:** Unit coverage, lint reports, E2E reports, code quality scanning.
  - **Artifacts:** Distributables, raw project tar for scanners, source maps, webpack stats, test reports.

---

## 2) Setup & Foundations

### Shared Library
- Uses a shared Jenkins library (e.g., `sparkPipeline`) that provides helpers like **branch detection**, **PR metadata**, **status setters**, **archive/publish**, and **parallel orchestration**.

### Secrets & Credentials
- Credentials are injected via `withCredentials([...])`, exposing **string env vars** only for the duration of each stage.
- Examples: OAuth client IDs/secrets, API keys, browser testing credentials, analytics keys, and multiple environment‑specific service credentials (e.g., `*_PROD`, `*_INT`, `*_FEDRAMP`, `*_CONTROLLED`, `*_CUSTOM_ORG`).

### Branch / Mode Detection
- **Pull Request (PR) builds** trigger **CI only**.
- **Default branch / Hotfix / Special test branch** trigger build + package and **CD** with deploy steps.
- Environment flags (e.g., `SERVICE_ENV=fedramp|controlled|customOrg`) toggle feature gates and test suites.

### Status & Notifications
- On PR start, all stage contexts are set to **pending** (Static Analysis, Unit tests, E2E tests, Build & Checks, Distributable Artifact).
- On completion, each stage updates its **commit status** with deep links to artifacts.
- Pipeline posts **notifications** to chat rooms (configurable room IDs) for key events and failures.

---

## 3) CI Stages (Pull Requests)

### A. Setup
- Install dependencies (`./bin/install.sh`).
- Initialize Playwright suite for gating.
- Reset commit statuses to **pending** for all tracked contexts.

### B. Static Analysis (Parallel Group)
- **Static checks:** ESLint (source), Stylelint (SCSS), design‑token checks.
- **Reports:** HTML reports are published and archived.
- **Outcome:** Sets commit status to success/failure; notifies author on failure.

### C. Build & Checks (Parallel Group)
- **Build:** Webpack build + auxiliary checks.
- **Compatibility:** ECMAScript compatibility verification.
- **Integrity:** Yarn lock consistency, translations/trusted‑proxy parity, packaging checks (including device‑specific bundles).
- **Artifacts:** `dist/**`, `distributable` link in commit status, build stats archived.

### D. Tooling Prep (Parallel Group)
- Install **Playwright browsers** for E2E tests.

### E. Unit Tests
- Run Jest unit tests with coverage.
- On failure: convert JUnit to a readable HTML summary, publish reports, notify author, set failure status.
- On success: link to coverage dashboard in commit status.

### F. E2E (Gating) Tests
- Run Playwright gating suites in multiple modes:
  - **Standard gating**
  - **Feature gating**
  - **Compliance gating** (e.g., FedRAMP‑mode)
- Merge & archive reports, set commit status, notify on failure.

---

## 4) CD Stages (Default / Hotfix Branches)

### A. Build & Package
- Install + Build; capture auxiliary versions (e.g., web‑effects) if needed.
- Archive distributable tarball of built assets.

### B. Parallel Post‑Build Jobs
- **Raw Project Tar:** Full workspace tar for external security scanners.
- **Cookie Consent CSV:** Generate, archive.
- **Code Quality Scan:** Run quality scanner (e.g., SonarQube) on default branch.

### C. Finalize Build
- Archive **source maps** and **build stats**.
- **Stash** the workspace (excluding heavy outputs) for later post‑deploy steps.

### D. Metadata Rewrites
- Update version fields in deployment descriptors (e.g., `.webpack`, `.microservice`) to the pipeline’s **build version**.
- Inject captured asset versions into the container/microservice descriptor.

### E. Deploy
- **Integration:** Automatic deployment, no consumer tests by default.
- **Staging (Load Test):** Typically manual input unless “hotlane”; runs **post‑deploy TAP tests** across multiple suites (standard/compliance/feature).
- **Production:** **Manual gate**; runs post‑deploy TAP tests (standard).

### F. Post‑Deploy TAP Tests
- Parameterized test runs against target URLs/sites.
- Exit codes are **merged** across suites to compute final outcome.
- On failure: merge & archive Playwright reports, notify rooms, mark build unstable/failure.

---

## 5) Auxiliary Pipelines

### External Tests Pipeline
- Runs **E2E against live endpoints** (production or integration) to validate media/signaling outside of the main deploy path.
- Parameterizes URL/site and environment flags; archives reports and notifies rooms.

### Ad‑Hoc TAP Pipeline
- Trigger TAP tests with custom URL/suite.
- Supports **custom org** mode (org/site/identity service parameters); archives reports and optionally notifies rooms.

---

## 6) Artifacts & Reporting

- **Lint Reports:** ESLint, Stylelint (HTML).
- **Unit Coverage:** Coverage summary + HTML dashboard.
- **E2E Reports:** Playwright merged HTML report.
- **Build Outputs:** Production assets (`dist/**`), source maps, build stats.
- **Distributables:** Packaged tarballs; **raw project tar** for external scanners.
- **Config Outputs:** Updated deployment descriptors and optional server config templates.

---

## 7) Exit Codes & Result Aggregation

- `determineExitCode(a, b, c)` selects the most severe exit code across suites (e.g., `0=success`, `3=automation failure`, `4=migration failure`), otherwise the first non‑zero.
- `determineFailedSuiteNames(...)` summarizes which suites failed (used in notifications).

> **Implementation note:** Ensure shell step return values are treated as **integers** (not strings) when switching on exit codes.

---

## 8) Failure Handling & Notifications

- **Commit Statuses:** Each stage updates its context with success/failure and a direct link to the relevant report.
- **Chat Notifications:** Failure messages (+ report links) are posted to team rooms; author notifications for unit‑test failures.
- **Fail‑Fast:** Parallel groups stop early on failures to conserve resources.

---

## 9) Recommendations / Hygiene

- Normalize `switch` cases to **integers** for exit codes.
- Fix minor typos in room variables; prefer central constants for room IDs.
- Reduce credential surface per stage (least privilege).
- Consolidate duplicated helpers (e.g., report merge, prod creds) in the shared library.
- Externalize hard‑coded URLs into configuration.
- Return empty string (not `0`) when no suite fails for string interpolation clarity.

---

## 10) Full CI/CD Flow (Mermaid)

```mermaid
flowchart TB
  subgraph PR_CI[Pull Request CI]
    A[Start: PR Opened] --> B[Setup: install, set pending statuses]
    B --> C{{Parallel: Static Analysis<br/>Build & Checks<br/>Install Browsers}}
    C --> D[Unit Tests]
    D --> E[E2E Gating (Std/Feature/Compliance)]
    E --> F[Publish Reports & Update Commit Statuses]
    F --> G{Success?}
    G -- Yes --> H[Notify Success (optional)]
    G -- No --> I[Notify Failure + Links]
  end

  subgraph BRANCH_CD[Default/Hotfix Branch CD]
    J[Start: Commit on default/hotfix] --> K[Build & Package]
    K --> L{{Parallel: Raw Tar for Scanners<br/>Cookie Consent CSV<br/>Quality Scan (default branch)}}
    L --> M[Finalize: archive maps & stats, stash workspace]
    M --> N[Rewrite Versions in Deploy Descriptors]
    N --> O[Deploy: Integration]
    O --> P[Deploy: Staging (Load Test)]
    P --> Q[Post‑Deploy TAP (Std/Compliance/Feature)]
    Q --> R{Proceed to Production?}
    R -- Manual Gate --> S[Deploy: Production]
    S --> T[Post‑Deploy TAP (Standard)]
    T --> U[Publish Reports & Notify Rooms]
  end

  A -. triggers CI statuses .-> F
  J -. triggers CD chain .-> U
```

---

## 11) TL;DR

- **CI:** PRs run setup → static checks → build → unit → E2E gating → publish reports → update commit statuses → notify.
- **CD:** Default/hotfix branches build & package → update versions → deploy to integration → staging (post‑deploy tests) → manual production (post‑deploy tests) → publish reports → notify.

