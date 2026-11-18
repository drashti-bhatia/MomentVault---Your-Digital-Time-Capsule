# FEATURE_TRACKER.md

# Visitor Photo Slideshow (MomentVault): Feature Tracker

| Item | Detail |
|---|---|
| Product | MomentVault, implementing Assignment 1: Visitor Photo Slideshow |
| Built from | `PRD.md` (every FR and NFR requirement) and `IMPLEMENTATION_PLAN.md` (the five-day schedule) |
| Tracker created | 18 November 2026 |
| Requirements tracked | 90 (57 Must, 28 Should, 5 Could) |
| Current state | **Nothing is built or verified.** Every row starts as `Not Started` / `Not Run`. |

> **Who changes this file.** Only you, the developer, change a **Status**, a **Testing status** or a **Verification evidence** cell, and only after you have really done the work. Neither Claude nor any other AI tool should mark a row as finished for you. If an AI says "this works", that is a claim to check, not evidence.

---

## 1. How to Use This Tracker (Beginner Guide)

**Each work session:**

1. Pick the next task from the daily plan (Section 4).
2. Change the **Status** of the rows it affects to `In Progress`.
3. When the code runs once for you, change it to `Implemented`.
4. When you start writing or running its tests, change it to `Testing`.
5. When the tests pass **and** you have written down proof, change it to `Verified`.
6. At the end of the day, update the progress snapshot (Section 3) and the daily log (Section 4).

**One rule above all:** a row is `Verified` only if the **Verification evidence** cell is filled in with something another person could check.

### 1.1 The Six Statuses

| Status | Meaning | Move to it when |
|---|---|---|
| **Not Started** | No work has begun | (Starting value) |
| **In Progress** | You are writing it now | You begin work |
| **Implemented** | The code exists and ran at least once, but is not yet properly tested | You saw it work once, but no test has passed yet |
| **Testing** | Tests are being written or run, or some have failed | You start the tests, or a test fails and you are fixing it |
| **Verified** | Implemented, tested, and proven with evidence | Tests pass **and** evidence is recorded |
| **Deferred** | A conscious decision not to build it now | You decide, and write the reason in the notes |

Typical path: `Not Started` → `In Progress` → `Implemented` → `Testing` → `Verified`.
A feature can go back (for example `Testing` → `In Progress`) if a test finds a bug. That is normal.

### 1.2 Testing Status Values

| Value | Meaning |
|---|---|
| **Not Run** | No test has been run (starting value) |
| **Pass** | Tests were run and passed |
| **Fail** | A test was run and failed |
| **Manual** | Checked by hand on a real browser or device, and recorded |
| **N/A** | Not tested because the feature is `Deferred` |

### 1.3 What Counts as Verification Evidence

Write short, checkable evidence such as:

| Kind | Example of the format (these are not real results) |
|---|---|
| Automated test | `test_upload.py::test_rejects_fake_image: passed, 18 Nov, commit abc1234` |
| Manual test | `TC-DSP-03 passed on Chrome 1920x1080, screenshot docs/evidence/dsp-03.png` |
| Document or file check | `.env.example reviewed, no secrets; git log search found none` |
| Measurement | `5 MB upload took 1.8 s (3 runs)` |

Never write "works", "done" or "OK" alone. Never write evidence for a test you did not run.

### 1.4 How to Defer a Feature

Change **Status** to `Deferred`, set **Testing status** to `N/A`, and write the reason and date in **Implementation notes**. Several rows below are planned for deferral by the plan (Tier B and "not scheduled"), but they still start as `Not Started` because you have not yet made that decision. The decision point is Gate G4 at the end of Day 4.

### 1.5 Reading the Columns

| Column | What it holds |
|---|---|
| **Feature ID** | The requirement ID from the PRD |
| **Feature name** | Short name taken from the PRD |
| **Priority** | `Must`, `Should` or `Could` (MoSCoW, as in the PRD) |
| **Responsible module** | Where the work lives. See the module key below. |
| **Dependencies** | Requirements that should work first |
| **Status** | One of the six statuses above |
| **Implementation notes** | The planned task (for example `D2-04`) plus a place for your own notes |
| **Testing status** | One of the five testing values above |
| **Verification evidence** | Proof, as in Section 1.3. `—` means none yet. |

**Module key** (folder names follow `BACKEND_DESIGN.md` Section 5 and `FRONTEND_DESIGN.md`):

| Module | Meaning |
|---|---|
| `api` | Backend routes (E1 to E10) |
| `services` | Backend business rules (visitor, consent, photo, slideshow services) |
| `image_processor` | Pillow checking and cleaning of images |
| `storage` | Writing, reading and deleting photo files |
| `db` | Models, repositories, database settings |
| `core` | Settings, security helpers, errors, middleware, consent notice |
| `FE` | React frontend: pages, components, hooks, lib (plain JavaScript helpers) |
| `Repo` / `Docs` / `Tests` | Repository hygiene, documents, test suite |

### 1.6 Notes About the Source Documents

- Your message called the PRD and plan "approved". The documents themselves still say **draft for approval**, and no per-decision approval is on record. This tracker follows them as written. If you change a decision, update the affected rows.
- Task IDs in **Implementation notes** (`D1-03`, `D4-07` and so on) come from `IMPLEMENTATION_PLAN.md`. The plan does not list the requirement IDs for each task, so I matched them by reading the task descriptions. Where I had to guess, the note says **“My suggestion”**. Check these.
- `ASSUMPTIONS.md` Part 6.2 lists PRD updates still to be approved (for example, the meaning of “plays next”, and clearing the name on withdrawal). These could change some rows.

---

## 2. Open Decisions That Affect This Tracker

| ID | Decision | Rows affected |
|---|---|---|
| A-20 | Does “specific duration” mean seconds per photo, or time in the slideshow? | FR-DSP-02 |
| A-26 | Which screen will you demo on? | FR-DSP-01, NFR-COMP-02, NFR-USA-02 |
| P-1 | Keep or drop thumbnails? (Plan default: drop.) | Not a PRD requirement. Affects FR-STO-02 and API notes. |
| P-2 | Build withdrawal only, or deletion as well, or reword the notice? | FR-CON-01, FR-WDR-01 to FR-WDR-04, FR-DEL-01 to FR-DEL-05 |
| P-3 | Leave out `/admin` and E10? (Plan default: leave out.) | FR-OPR-01, FR-OPR-02 |
| SP-1 to SP-5 | Security proposals in `SECURITY_PRIVACY.md` (rate limit, cache header, operator endpoint, response headers, key length) | FR-VAL-08, FR-STO-05, FR-OPR-01, FR-OPR-02, NFR-SEC-04 |

---

## 3. Progress Snapshot

Update this table at the end of each day by counting the rows in Section 5. Only `Verified` counts as done.

| Status | Must | Should | Could | Total |
|---|---|---|---|---|
| Not Started | 57 | 28 | 5 | 90 |
| In Progress | 0 | 0 | 0 | 0 |
| Implemented | 0 | 0 | 0 | 0 |
| Testing | 0 | 0 | 0 | 0 |
| Verified | 0 | 0 | 0 | 0 |
| Deferred | 0 | 0 | 0 | 0 |
| **Total** | 57 | 28 | 5 | 90 |

*Counts shown are the starting values at creation. They are not progress.*

---

## 4. Daily Progress Summary (Five-Day Schedule)

Planned hours and tasks come from `IMPLEMENTATION_PLAN.md`. The **Actual hours** and **Done** columns are blank for you to fill. Mark **Done** only when the task’s completion criteria in the plan are met. “Task done” is not the same as “requirement Verified”, so also update the rows in Section 5.

**Daily rules from the plan:** one task, one commit; tests are written with the code; record AI mistakes in `AI_LOG.md` as they happen; run the day’s decision gate before moving on.

### Day 1: Environment, Project Setup and Foundations (4.0 h planned)

**Goal:** both servers start, the page can read settings from the backend, and the image processor is proven by tests.

| Task | Description | Planned h | Actual h | Done | Requirements it feeds |
|---|---|---|---|---|---|
| D1-01 | Install tools, pin versions | 0.40 | | ☐ | (Foundation) |
| D1-02 | Repository, folders, `.gitignore`, `.env.example` | 0.25 | | ☐ | NFR-SEC-02, NFR-DOC-01, NFR-DOC-02 |
| D1-03 | Backend skeleton, settings, consent notice, E1 | 0.75 | | ☐ | FR-CON-01, NFR-MNT-01 |
| D1-04 | Frontend skeleton | 0.75 | | ☐ | (Foundation) |
| D1-05 | Image processor and its tests (highest backend risk) | 1.25 | | ☐ | FR-VAL-01, FR-VAL-02, FR-VAL-05, FR-STO-03 |
| D1-B | Buffer | 0.45 | | n/a | |
| D1-C | Checkpoint, Gate G1 | 0.15 | | ☐ | |

**Gate G1:** is the image processor passing its tests, and is Day 1 within 4.5 h?  Result: ______
**Progress notes:** 
**Blockers:** 
**AI_LOG entries added today:** 

### Day 2: Database and Backend APIs (4.0 h planned)

**Goal:** a visitor can register and upload through the API, and the stored photo can be fetched back.

| Task | Description | Planned h | Actual h | Done | Requirements it feeds |
|---|---|---|---|---|---|
| D2-01 | Models, database set-up, repositories | 1.00 | | ☐ | FR-REG-04, FR-STO-02, FR-CON-03, FR-CNC-03 |
| D2-02 | Photo storage | 0.40 | | ☐ | FR-STO-01, FR-CNC-02 |
| D2-03 | Security helpers, visitor and consent services, E2 | 0.70 | | ☐ | FR-ID-01, FR-ID-02, FR-ID-03, FR-ID-03a, FR-ID-05, FR-CON-02, FR-CON-03 |
| D2-04 | Upload workflow E3 (central feature) | 1.10 | | ☐ | FR-REG-02, FR-REG-03, FR-UPL-02, FR-UPL-03, FR-UPL-04, FR-VAL-03, FR-VAL-04, FR-VAL-06, FR-STO-04, FR-CNC-01 |
| D2-05 | Image serving E8 | 0.20 | | ☐ | FR-STO-05 |
| D2-B | Buffer | 0.45 | | n/a | |
| D2-C | Checkpoint, Gate G2 | 0.15 | | ☐ | |

**Gate G2:** can a photo be uploaded through the API documentation page and fetched back?  Result: ______
**Progress notes:** 
**Blockers:** 
**AI_LOG entries added today:** 

### Day 3: React Frontend and API Integration (3.5 h planned)

**Goal:** a visitor on a real phone can complete the full upload journey.

| Task | Description | Planned h | Actual h | Done | Requirements it feeds |
|---|---|---|---|---|---|
| D3-01 | Shared UI parts and component styles | 0.60 | | ☐ | NFR-USA-01 |
| D3-02 | Helpers and the API client for upload | 0.50 | | ☐ | FR-VAL-07 (suggested) |
| D3-03 | Upload page for a new visitor | 1.40 | | ☐ | FR-REG-01, FR-UPL-01, FR-UPL-04, FR-UPL-05, FR-CON-01, FR-CON-02, FR-ID-01, NFR-SEC-03 |
| D3-04 | Integration on a real phone | 0.60 | | ☐ | NFR-USA-01, NFR-COMP-01 |
| D3-B | Buffer | 0.30 | | n/a | |
| D3-C | Checkpoint, Gate G3 | 0.10 | | ☐ | |

**Gate G3:** can a real phone upload a photo through the UI?  Result: ______
**Progress notes:** 
**Blockers:** 
**AI_LOG entries added today:** 

### Day 4: Slideshow, Returning Visitors and Essential Security (4.7 h planned)

**Goal:** a phone upload appears on the TV page, returning visitors are recognised, withdrawal works, and the basic security rules are checked.

| Task | Description | Planned h | Actual h | Done | Requirements it feeds |
|---|---|---|---|---|---|
| D4-01 | Slideshow feed E7 | 0.55 | | ☐ | FR-DSP-07, FR-AUT-01, FR-PRV-01, FR-PRV-02, FR-PRV-04 |
| D4-02 | Queue rules as plain functions | 0.50 | | ☐ | FR-DSP-04, FR-AUT-02, FR-AUT-04, FR-PRV-02, FR-PRV-03 |
| D4-03 | Display page | 1.00 | | ☐ | FR-DSP-01, FR-DSP-02, FR-DSP-03, FR-DSP-05, FR-DSP-06, FR-DSP-08, FR-AUT-01, FR-AUT-03 |
| D4-04 | Returning visitor (“Welcome back”, “Not you?”) | 0.40 | | ☐ | FR-ID-02, FR-ID-04 |
| D4-05 | Standard errors and request size guard | 0.40 | | ☐ | NFR-SEC-04, NFR-USA-03, FR-VAL-03, FR-VAL-04 |
| D4-06 | Essential security pass (review, not a new feature) | 0.20 | | ☐ | NFR-SEC-01, NFR-SEC-04 |
| D4-07 | Consent withdrawal (E6) and its screen | 1.25 | | ☐ | FR-WDR-01 to FR-WDR-04 |
| D4-B | Buffer | 0.25 | | n/a | |
| D4-C | Checkpoint, Gate G4 | 0.15 | | ☐ | |

**Gate G4:** does a phone upload appear on the TV page, and does withdrawal work?  Result: ______
**Decision at G4:** Tier B work (B1 to B6) starts only if G4 passes and the remaining buffer is at least the size of the item. Record each Tier B or “not scheduled” row as started or `Deferred` here: 
**Progress notes:** 
**Blockers:** 
**AI_LOG entries added today:** 

### Day 5: Testing, Documentation and Local Demo (3.8 h planned)

**Goal:** everything is proven, documented, rehearsed and submitted. **No new features today.**

| Task | Description | Planned h | Actual h | Done | Requirements it feeds |
|---|---|---|---|---|---|
| D5-01 | Contract and concurrency tests | 0.90 | | ☐ | FR-CNC-01, FR-CNC-02, FR-CNC-04, NFR-REL-01, NFR-TST-01 |
| D5-02 | Manual test run with real devices | 0.60 | | ☐ | NFR-USA-01, NFR-USA-02, NFR-COMP-01, NFR-COMP-02, NFR-PERF-01 to NFR-PERF-03 |
| D5-03 | Documentation | 0.90 | | ☐ | NFR-DOC-01 |
| D5-05 | Local demo set-up and restart test | 0.40 | | ☐ | NFR-REL-02 |
| D5-04 | Demo rehearsal, explain every component | 0.40 | | ☐ | (Assignment requirement F-10) |
| D5-06 | Submission | 0.30 | | ☐ | NFR-SEC-02, NFR-DOC-02 |
| D5-B | Buffer | 0.30 | | n/a | |

**Progress notes:** 
**Blockers:** 
**AI_LOG entries added today (final count and at least two real AI mistakes):** 

### Weekly Totals

| | Planned h | Actual h |
|---|---|---|
| Day 1 | 4.00 | |
| Day 2 | 4.00 | |
| Day 3 | 3.50 | |
| Day 4 | 4.70 | |
| Day 5 | 3.80 | |
| **Total** | **20.00** | |

---

## 5. Feature Tracking Table

Every functional requirement (FR) and non-functional requirement (NFR) in `PRD.md` appears once. Update only the **Status**, **Implementation notes** (add your own notes after the plan reference), **Testing status** and **Verification evidence** columns.

### 3.1 New Visitor Registration (FR-REG)

| Feature ID | Feature name | Priority | Responsible module | Dependencies | Status | Implementation notes | Testing status | Verification evidence |
|---|---|---|---|---|---|---|---|---|
| FR-REG-01 | Ask for name and photos | Must | FE components (UploadPage, PhotoPicker) | — | Not Started | Plan: D3-03. Empty name or no photo must block the upload. | Not Run | — |
| FR-REG-02 | Validate the name (1–50 characters, trimmed, shown as plain text) | Must | services (visitor_service); FE form | FR-REG-01 | Not Started | Plan: D2-04 (server), D3-03 (form). Test with the script-tag name (see NFR-SEC-03). | Not Run | — |
| FR-REG-03 | Create a visitor on first valid submission | Must | services; db; core (security) | FR-REG-02, FR-CON-02, FR-ID-05 | Not Started | Plan: D2-03, D2-04. Visitor is created even if every photo is rejected. Token is returned once. | Not Run | — |
| FR-REG-04 | Names are not unique | Must | db (visitors table) | — | Not Started | Plan: D2-01. No uniqueness rule on name. Test two visitors both named “Sam”. | Not Run | — |

### 3.2 Returning Visitor Identification (FR-ID)

| Feature ID | Feature name | Priority | Responsible module | Dependencies | Status | Implementation notes | Testing status | Verification evidence |
|---|---|---|---|---|---|---|---|---|
| FR-ID-01 | Store a visitor token in the browser | Must | FE hooks and lib; core (security) | FR-REG-03 | Not Started | Plan: D2-03, D3-03 (store the returned token immediately). | Not Run | — |
| FR-ID-02 | Recognise a returning visitor | Must | api (E2); FE (useVisitor, WelcomeBack) | FR-ID-01, FR-ID-03 | Not Started | Plan: D2-03 (E2), D4-04 (page). “Welcome back”, no name field, no consent form. | Not Run | — |
| FR-ID-03 | Reject an unknown or invalid token | Must | core (dependencies, security) | FR-ID-01 | Not Started | Plan: D2-03. Frontend clears the token on a 401 (D4-04). | Not Run | — |
| FR-ID-03a | Never identify by name alone | Must | services; api | FR-REG-04 | Not Started | Plan: D2-03. Design rule: nothing looks a visitor up by name. Re-check in D4-06. | Not Run | — |
| FR-ID-04 | Switch visitor (“Not you?”) | Should | FE components (WelcomeBack) | FR-ID-02 | Not Started | Plan: included in D4-04 (forgets the token on this device only). | Not Run | — |
| FR-ID-05 | Protect the token (only a hash is stored) | Must | core (security); db | — | Not Started | Plan: D2-03. Token never in a URL, never logged. | Not Run | — |

### 3.3 Public Display Consent (FR-CON)

| Feature ID | Feature name | Priority | Responsible module | Dependencies | Status | Implementation notes | Testing status | Verification evidence |
|---|---|---|---|---|---|---|---|---|
| FR-CON-01 | Show a consent notice before first upload | Must | core (consent_notice); api (E1); FE (ConsentNotice) | — | Not Started | Plan: D1-03 (text and version), D3-03 (display). Notice item 4 depends on decision P-2 (withdrawal built, or notice reworded). | Not Run | — |
| FR-CON-02 | Consent is explicit and required | Must | FE; services (consent_service) | FR-CON-01 | Not Started | Plan: D2-03, D3-03. Unticked box. The server must also refuse a submission without consent. | Not Run | — |
| FR-CON-03 | Record consent (who, when, notice version) | Must | db (ConsentRecord); services | FR-CON-02 | Not Started | Plan: D2-01, D2-03. | Not Run | — |
| FR-CON-04 | Remind returning visitors | Should | FE components | FR-ID-02 | Not Started | Plan: Tier B, B2 (about 0.2 h). Not scheduled by default. | Not Run | — |

### 3.4 Photo Upload (FR-UPL)

| Feature ID | Feature name | Priority | Responsible module | Dependencies | Status | Implementation notes | Testing status | Verification evidence |
|---|---|---|---|---|---|---|---|---|
| FR-UPL-01 | Select multiple photos | Must | FE components (PhotoPicker, PhotoPreviews) | FR-REG-01 | Not Started | Plan: D3-03. | Not Run | — |
| FR-UPL-02 | Define when an upload is complete | Must | services (photo_service); api (E3) | FR-STO-04 | Not Started | Plan: D2-04. A photo is complete once its file is stored and recorded. | Not Run | — |
| FR-UPL-03 | Process files independently | Must | services (photo_service) | FR-VAL-01 to FR-VAL-05 | Not Started | Plan: D2-04. Each photo committed separately. One bad file must not stop the others. | Not Run | — |
| FR-UPL-04 | Per-photo feedback | Must | api (E3 result list); FE (UploadResults) | FR-UPL-03 | Not Started | Plan: D2-04, D3-03. | Not Run | — |
| FR-UPL-05 | Upload progress | Should | FE hooks (useUpload) | FR-UPL-01 | Not Started | Plan: D3-03 builds an indeterminate bar. A real percentage is Tier B, B4. Check the PRD acceptance criteria to see which satisfies it. | Not Run | — |

### 3.5 File Validation and Upload Limits (FR-VAL)

| Feature ID | Feature name | Priority | Responsible module | Dependencies | Status | Implementation notes | Testing status | Verification evidence |
|---|---|---|---|---|---|---|---|---|
| FR-VAL-01 | Allowed types only (JPEG and PNG) | Must | image_processor; services | — | Not Started | Plan: D1-05, D2-04. Assumption A-12. | Not Run | — |
| FR-VAL-02 | Verify the real content, not the name | Must | image_processor | FR-VAL-01 | Not Started | Plan: D1-05. Fully decode with Pillow. Test fake, empty and corrupt files. | Not Run | — |
| FR-VAL-03 | File size limit (5 MB) | Must | core (middleware); services | — | Not Started | Plan: D2-04, D4-05 (413 guard). | Not Run | — |
| FR-VAL-04 | File count limit (5 per submission) | Must | services; core (middleware) | — | Not Started | Plan: D2-04, D4-05. | Not Run | — |
| FR-VAL-05 | Image dimension limit (25 megapixels) | Must | image_processor | — | Not Started | Plan: D1-05. Check from the header before decoding (AD-24, TF-3). | Not Run | — |
| FR-VAL-06 | Per-visitor photo limit (20) | Should | services (photo_service) | FR-STO-02 | Not Started | Plan: D2-04 includes it. | Not Run | — |
| FR-VAL-07 | Early checks in the browser | Should | FE lib (photoChecks) | FR-VAL-01, FR-VAL-03, FR-VAL-04 | Not Started | My suggestion: D3-02 (helpers). Not named in the plan; confirm. | Not Run | — |
| FR-VAL-08 | Basic rate limiting | Could | core (middleware) | — | Not Started | Plan: not scheduled (Section 6.2). SECURITY_PRIVACY finding F-1 proposes a minimal limit. Decision needed. | Not Run | — |

### 3.6 Photo Storage (FR-STO)

| Feature ID | Feature name | Priority | Responsible module | Dependencies | Status | Implementation notes | Testing status | Verification evidence |
|---|---|---|---|---|---|---|---|---|
| FR-STO-01 | Safe storage names (random, never the visitor’s file name) | Must | storage (photo_storage) | — | Not Started | Plan: D2-02. Include path-traversal tests. | Not Run | — |
| FR-STO-02 | Store photo information | Must | db (Photo model, repository) | — | Not Started | Plan: D2-01. | Not Run | — |
| FR-STO-03 | Strip hidden photo data and re-encode | Should | image_processor | FR-VAL-02 | Not Started | Plan: D1-05 builds it. Test with a GPS image. Also covers NFR-PRV-01. | Not Run | — |
| FR-STO-04 | Store and record together (no orphans) | Must | services; storage; db | FR-STO-01, FR-STO-02 | Not Started | Plan: D2-04. File first, row after. Undo both if either step fails. | Not Run | — |
| FR-STO-05 | Serve photos safely | Must | api (E8) | FR-STO-01 | Not Started | Plan: D2-05. By photo id only. Review the Cache-Control value (SECURITY_PRIVACY F-2). | Not Run | — |

### 3.7 Slideshow Display (FR-DSP)

| Feature ID | Feature name | Priority | Responsible module | Dependencies | Status | Implementation notes | Testing status | Verification evidence |
|---|---|---|---|---|---|---|---|---|
| FR-DSP-01 | Separate display page | Must | FE pages (DisplayPage) | — | Not Started | Plan: D4-03. Setup needs one manual full-screen step (assumption A-26). | Not Run | — |
| FR-DSP-02 | Show each photo for a set duration | Must | FE hooks (useSlideshow); core (settings) | FR-DSP-01 | Not Started | Plan: D4-03, D1-03. Duration value is still OPEN (A-20). | Not Run | — |
| FR-DSP-03 | Show the whole photo (no cropping) | Must | FE pages (DisplayPage) | FR-DSP-01 | Not Started | Plan: D4-03. | Not Run | — |
| FR-DSP-04 | Continuous loop | Must | FE lib (slideshowQueue) | FR-DSP-07 | Not Started | Plan: D4-02. | Not Run | — |
| FR-DSP-05 | Empty state | Must | FE pages (DisplayPage) | FR-DSP-01 | Not Started | Plan: D4-03 (shows the join address). | Not Run | — |
| FR-DSP-06 | Skip photos that fail to load | Must | FE hooks (useSlideshow) | FR-DSP-02 | Not Started | Plan: D4-03 (skip within 2 seconds). | Not Run | — |
| FR-DSP-07 | Show only active photos | Must | services (slideshow_service); db | FR-STO-02 | Not Started | Plan: D4-01. | Not Run | — |
| FR-DSP-08 | Recover from connection problems | Should | FE hooks (useSlideshow) | FR-AUT-01 | Not Started | Plan: D4-03 (keep playing if the server drops). | Not Run | — |
| FR-DSP-09 | Transition effect (fade) | Could | FE pages, css | FR-DSP-02 | Not Started | Plan: Tier B, B3. Instant swap already meets every Must Have. | Not Run | — |
| FR-DSP-10 | Name caption | Could | api (E7); FE (DisplayPage) | FR-CON-01 | Not Started | Plan: not scheduled. Needs an API change and a sixth consent point. | Not Run | — |

### 3.8 Automatic Slideshow Updates (FR-AUT)

| Feature ID | Feature name | Priority | Responsible module | Dependencies | Status | Implementation notes | Testing status | Verification evidence |
|---|---|---|---|---|---|---|---|---|
| FR-AUT-01 | Updates without refreshing | Must | api (E7); FE hooks (useSlideshow) | FR-DSP-01 | Not Started | Plan: D4-01, D4-03. Polling, interval 5 seconds (A-25). | Not Run | — |
| FR-AUT-02 | New photos play next | Must | FE lib (slideshowQueue) | FR-AUT-01 | Not Started | Plan: D4-02. “Next” means next in the queue (ASSUMPTIONS A-21). | Not Run | — |
| FR-AUT-03 | Do not interrupt the current photo | Must | FE hooks (useSlideshow) | FR-DSP-02 | Not Started | Plan: D4-03. | Not Run | — |
| FR-AUT-04 | Removal updates | Should | FE lib (slideshowQueue) | FR-AUT-01 | Not Started | Plan: D4-02 (drops photos that left the feed). | Not Run | — |

### 3.9 Previous Photo Retrieval (FR-PRV)

| Feature ID | Feature name | Priority | Responsible module | Dependencies | Status | Implementation notes | Testing status | Verification evidence |
|---|---|---|---|---|---|---|---|---|
| FR-PRV-01 | Play a returning visitor’s previous photos | Must | services (slideshow_service) | FR-ID-02, FR-DSP-07 | Not Started | Plan: D4-01, D4-04. | Not Run | — |
| FR-PRV-02 | Playback order (new first, then earlier, most recent first) | Must | services (slideshow_service); FE lib | FR-PRV-01 | Not Started | Plan: D4-01, D4-02. | Not Run | — |
| FR-PRV-03 | No duplicates in a sequence | Must | FE lib (slideshowQueue) | FR-PRV-02 | Not Started | Plan: D4-02. | Not Run | — |
| FR-PRV-04 | Only the right photos (owner’s, active) | Must | services (slideshow_service); db | FR-PRV-01 | Not Started | Plan: D4-01. | Not Run | — |
| FR-PRV-05 | “My photos” view | Should | api (E4); FE | FR-ID-02 | Not Started | Plan: Tier B, B1 (with deletion). | Not Run | — |

### 3.10 Consent Withdrawal (FR-WDR)

| Feature ID | Feature name | Priority | Responsible module | Dependencies | Status | Implementation notes | Testing status | Verification evidence |
|---|---|---|---|---|---|---|---|---|
| FR-WDR-01 | Withdraw consent at any time | Should | api (E6); FE (WithdrawConsent) | FR-ID-02 | Not Started | Plan: promoted into committed scope, D4-07 (decision P-2). | Not Run | — |
| FR-WDR-02 | Confirm before withdrawing | Should | FE components (ConfirmDialog) | FR-WDR-01 | Not Started | Plan: D4-07. | Not Run | — |
| FR-WDR-03 | Effects of withdrawal | Should | services (visitor_service); storage; db | FR-WDR-01 | Not Started | Plan: D4-07. Check files, database rows, feed and token together. Also test withdrawal during an upload (SECURITY_PRIVACY F-7). | Not Run | — |
| FR-WDR-04 | Uploading after withdrawal needs fresh consent | Should | services; core (dependencies) | FR-WDR-03, FR-ID-03 | Not Started | Plan: D4-07. | Not Run | — |

### 3.11 Photo Deletion (FR-DEL)

| Feature ID | Feature name | Priority | Responsible module | Dependencies | Status | Implementation notes | Testing status | Verification evidence |
|---|---|---|---|---|---|---|---|---|
| FR-DEL-01 | Delete a single photo | Should | api (E5); FE | FR-PRV-05 | Not Started | Plan: Tier B, B1. | Not Run | — |
| FR-DEL-02 | Only the owner can delete | Should | services (photo_service) | FR-ID-03 | Not Started | Plan: Tier B, B1. | Not Run | — |
| FR-DEL-03 | Full removal (file and record) | Should | services; storage; db | FR-DEL-01 | Not Started | Plan: Tier B, B1. | Not Run | — |
| FR-DEL-04 | Confirm before deleting | Should | FE components (ConfirmDialog) | FR-DEL-01 | Not Started | Plan: Tier B, B1 (reuses the D4-07 dialog). | Not Run | — |
| FR-DEL-05 | Handle repeat deletion | Should | services (photo_service) | FR-DEL-01 | Not Started | Plan: Tier B, B1. | Not Run | — |

### 3.12 Multiple Simultaneous Uploads (FR-CNC)

| Feature ID | Feature name | Priority | Responsible module | Dependencies | Status | Implementation notes | Testing status | Verification evidence |
|---|---|---|---|---|---|---|---|---|
| FR-CNC-01 | Independent concurrent uploads (10 at once) | Must | services; db; storage | FR-UPL-03 | Not Started | Plan: D2-04 (design), D5-01 (the test). Check that the test really runs requests at the same time (TF-7). | Not Run | — |
| FR-CNC-02 | No overwriting | Must | storage (photo_storage) | FR-STO-01 | Not Started | Plan: D2-02, D5-01. | Not Run | — |
| FR-CNC-03 | Database busy handling | Must | db (database settings); core (errors) | FR-CNC-01 | Not Started | Plan: D2-01 (WAL, busy timeout), D4-05. | Not Run | — |
| FR-CNC-04 | Display stays responsive during uploads | Must | api (E7); FE hooks | FR-AUT-01 | Not Started | Plan: D4-01, D5-01, D5-02. | Not Run | — |
| FR-CNC-05 | Duplicate submission protection | Should | FE hooks (useUpload) | FR-UPL-05 | Not Started | Not named in the plan. Suggested: disable the button while uploading (D3-03). Confirm. | Not Run | — |

### 3.13 Operator Removal (FR-OPR)

| Feature ID | Feature name | Priority | Responsible module | Dependencies | Status | Implementation notes | Testing status | Verification evidence |
|---|---|---|---|---|---|---|---|---|
| FR-OPR-01 | Operator removes any photo | Could | api (E10) | FR-DEL-03 | Not Started | Plan: not scheduled (Section 6.2). SECURITY_PRIVACY F-3 suggests building E10 only. Decision needed. | Not Run | — |
| FR-OPR-02 | Operator key is a secret | Could | core (settings); Repo | FR-OPR-01, NFR-SEC-02 | Not Started | Plan: not scheduled. Placeholder only in .env.example. | Not Run | — |

### 4. Non-Functional Requirements (NFR)

| Feature ID | Feature name | Priority | Responsible module | Dependencies | Status | Implementation notes | Testing status | Verification evidence |
|---|---|---|---|---|---|---|---|---|
| NFR-SEC-01 | Uploads are untrusted and pass all FR-VAL checks | Must | image_processor; services | FR-VAL-01 to FR-VAL-05 | Not Started | Run the file-validation tests with the Appendix B sample files. | Not Run | — |
| NFR-SEC-02 | No secrets committed; .env.example has placeholders only | Must | Repo | — | Not Started | Plan: D1-02, D5-06. Search the files and the Git history before submission. | Not Run | — |
| NFR-SEC-03 | Visitor text is always shown as plain text | Must | FE | FR-REG-02 | Not Started | Test with the script-tag name. | Not Run | — |
| NFR-SEC-04 | Errors reveal no paths, tokens, traces or database details | Must | core (errors, error_handlers) | — | Not Started | Plan: D4-05, D4-06. Force errors and read the messages. | Not Run | — |
| NFR-SEC-05 | Cross-origin requests only from the configured address | Should | core (settings, main) | — | Not Started | Not named in a plan task. Needed while frontend and backend run on different addresses. | Not Run | — |
| NFR-PRV-01 | Hidden image data (GPS) is not kept | Should | image_processor | FR-STO-03 | Not Started | Same check as FR-STO-03. | Not Run | — |
| NFR-PRV-02 | Collect only name, photos and consent records | Must | db | — | Not Started | Review the data model against DATABASE_DESIGN. | Not Run | — |
| NFR-PRV-03 | Deployed versions use HTTPS | Should | Deployment | — | Not Started | Only applies if you deploy (A-30 is OPEN). Local demo uses HTTP; say so in the README. | Not Run | — |
| NFR-PERF-01 | A 5 MB photo is validated and stored in 5 seconds or less | Should | image_processor; services | FR-UPL-02 | Not Started | Time the upload test and record the number. | Not Run | — |
| NFR-PERF-02 | Non-upload requests answer in 1 second or less (up to 200 photos) | Should | api; db | FR-DSP-07 | Not Started | Time the list or feed request. | Not Run | — |
| NFR-PERF-03 | Display shows its first photo in 3 seconds or less (local network) | Should | FE pages | FR-DSP-01 | Not Started | Time the page load. | Not Run | — |
| NFR-REL-01 | At least 10 simultaneous uploads without loss | Must | services; db; storage | FR-CNC-01 | Not Started | Same test as FR-CNC-01. | Not Run | — |
| NFR-REL-02 | A server restart loses no photos or records | Must | db; storage | FR-STO-02 | Not Started | Plan: D5-05 (restart test). | Not Run | — |
| NFR-USA-01 | Upload page works at about 360 px width without sideways scrolling | Must | FE pages (UploadPage), css | FR-REG-01 | Not Started | Plan: D3-03, D3-04, D5-02. | Not Run | — |
| NFR-USA-02 | Display text readable from a distance at 1920 × 1080 | Should | FE pages (DisplayPage), css | FR-DSP-01 | Not Started | Manual check on the real screen. | Not Run | — |
| NFR-USA-03 | Error messages are plain and say what to do next | Must | core (errors); FE | — | Not Started | Review the PRD Section 11 messages. | Not Run | — |
| NFR-COMP-01 | Upload page works on Chrome, Edge, Firefox and Safari (including phones) | Should | FE | FR-REG-01 | Not Started | Manual check; record which browsers you really tried. | Not Run | — |
| NFR-COMP-02 | Display works on Chrome or Edge at 1920 × 1080 (TV browser if available) | Must | FE pages (DisplayPage) | FR-DSP-01 | Not Started | Manual check. Which screen you demo on is still OPEN (A-26). | Not Run | — |
| NFR-MNT-01 | Settings P-01 to P-09 defined in one place | Should | core (settings) | — | Not Started | Plan: D1-03. Change P-01 and watch FR-DSP-02. | Not Run | — |
| NFR-MNT-02 | Consistent naming; follows the approved architecture | Must | All | — | Not Started | Review against CODING_STANDARDS at each checkpoint. | Not Run | — |
| NFR-TST-01 | Each Must Have has an automated test or a recorded manual test | Must | Tests; Docs | All Must items | Not Started | Check against TESTING_PLAN. Recorded at D5-01 and D5-02. | Not Run | — |
| NFR-DOC-01 | README, ASSUMPTIONS, AI_LOG, ARCHITECTURE, tests and .env.example exist | Must | Docs; Repo | — | Not Started | Plan: D1-02 (stubs), D5-03 (finish). | Not Run | — |
| NFR-DOC-02 | Commit history is regular and meaningful | Must | Repo | — | Not Started | One task, one commit (plan rule 1). Check with git log. | Not Run | — |
---

## 6. Items Outside the PRD (Not Tracked Until You Approve Them)

These come from other documents and are not PRD requirements. They are listed so they are not forgotten. If you decide to build one, add it as a new row in Section 5 with a clear ID.

| Item | Source | Note |
|---|---|---|
| Thumbnails and E9 | ASSUMPTIONS A-15, plan decision P-1 | Plan default is to leave them out |
| Photo cache header change (private, short lifetime) | SECURITY_PRIVACY F-2 (SP-2) | Protects withdrawal from browser caches |
| Security response headers | SECURITY_PRIVACY F-4 (SP-4) | Review at D4-06 |
| Global photo cap | SECURITY_PRIVACY F-1 (SP-1) | Pairs with rate limiting (FR-VAL-08) |
| Start-up orphan-file sweep | Plan Tier B, B5 | Cleans up after a crash |
| `AI_LOG.md` with 5 to 10 real entries, including at least 2 real AI mistakes | Assignment requirement | Not a PRD feature. Never invent entries. |

---

## 7. Change Log for This File

| Date | Change | By |
|---|---|---|
| 18 Nov 2026 | Tracker created. All 90 rows set to Not Started. | Generated from PRD and IMPLEMENTATION_PLAN |
| | | |
