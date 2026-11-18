# TESTING_PLAN.md

# Visitor Photo Slideshow (MomentVault): Testing Plan

| Item | Detail |
|---|---|
| Product | MomentVault, implementing Assignment 1: Visitor Photo Slideshow |
| Document version | 1.0 (draft for approval) |
| Date | 17 November 2026 |
| Based on | PRD (Sections 3, 7, 10, 11, Appendix B), API_SPECIFICATION (Sections 4, 5, 11), DATABASE_DESIGN (Section 15.4), SECURITY_PRIVACY (Sections 26, 28), plus ARCHITECTURE, BACKEND_DESIGN (Section 22), FRONTEND_DESIGN (Sections 16, 18) and IMPLEMENTATION_PLAN (Days 1 to 5) |
| Audience | An MCA beginner building and testing the project alone, with Claude and Gemini as coding assistants |
| Not included | Test code. This plan describes tests in words and tables. |

## Read This First

**No test in this document has been run.** The application does not exist yet. Every row in Section 14 has `—` in **Actual result** and `Not run` in **Status**. A row may be changed to `Pass` only by the person who ran it, only after running it, and only with the real observed result written in **Actual result**. Section 15 is the results section you complete during implementation.

**What is a plan, and what is evidence.** The expected results in this file come from the approved documents. They are **claims to be checked**, not facts about the code. Some expected results rely on framework behaviour that was not run in the planning sandbox (FastAPI, Starlette, SQLAlchemy). Those are marked **to verify**.

**Labels used in this file**

| Label | Meaning |
|---|---|
| **(A)** | Automated test, written in pytest (or a plain script for pure frontend helpers) and run by a command |
| **(M)** | Manual test, done by a person on a real browser, phone or screen, and recorded |
| **P1, P2, P3** | Priority, defined in Section 3.4 |
| **[if built]** | The test applies only if an optional or proposed feature is built (IMPLEMENTATION_PLAN Tier B, SECURITY_PRIVACY proposals SP-1 to SP-5). If the feature is not built, mark the row `Skipped` with the reason. Never mark it `Pass`. |

---

## Findings Found While Planning the Tests

| # | Finding | Effect on testing | Action |
|---|---|---|---|
| **TF-1** | **Test effort does not fit unless it is tiered.** IMPLEMENTATION_PLAN gives 0.9 h for final automated tests (D5-01) and 0.6 h for manual testing (D5-02), plus the tests written inside each build task. 215 cases are listed below (142 automated, 73 manual). They cannot all be run by hand in 0.6 h. | Cases are tiered **P1, P2, P3**. P1 is the minimum for submission. There are 107 automated P1 cases and 38 manual P1 cases. Counted by script, the 38 manual P1 cases are more than 0.6 h allows at a realistic 1 to 2 minutes each when grouped in a phone-plus-display session; plan about 1.0 h and take the extra 0.4 h from the Day 5 buffer (decision TP-1). Automated cases cost almost nothing to re-run once written. | Run all automated cases. Run manual **P1** first. Run manual P2 and P3 only if time remains, and record `Not run` honestly for the rest. |
| **TF-2** | **Several expected results depend on decisions that are still open.** Rate limit and global cap (SP-1), the photo cache header (SP-2), operator endpoint E10 (SP-3), response headers (SP-4), minimum operator key length (SP-5), and whether deletion (E5) and "My photos" are built (IMPLEMENTATION_PLAN Tier B). | Rows that depend on these are tagged **[if built]**. Their expected value for the cache header is written as "the decided value", not a guess. | Record each decision in Section 15.2 before running the related rows. |
| **TF-3** | **The per-file validation order in API_SPECIFICATION 5.5 lists the full decode (step f) before the header pixel check (step g), but the text says the pixel check happens "before decoding".** | A huge-pixel image must be rejected **without being fully decoded**. The order in the table could be read the wrong way. | Test case TC-CNT-07 checks the behaviour (rejected, and memory does not spike). If the build does the opposite, record it as a defect. |
| **TF-4** | **Which file code a given bad file gets depends on the order of the checks.** For example a `.pdf` file gets `FILE_TYPE_NOT_ALLOWED` (extension, step b), but a PDF renamed `.jpg` gets `FILE_NOT_AN_IMAGE` (step d), and a 6 MiB fake image gets `FILE_TOO_LARGE` (step a) before anything else. | Expected results below follow the documented order exactly. | If the real code differs, decide whether the code or the document is wrong, and record it. Do not edit the expected result to match. |
| **TF-5** | **"Within 10 seconds" in PRD FR-AUT-01 means "enters the display's playback queue", not "appears on screen"** (ARCHITECTURE 8.3, finding 1). On screen it appears after the current photo ends, up to about 15 seconds later. | The timing tests measure both moments separately and record them. Pass or fail uses the PRD wording for the queue and records the screen time as information. | Confirm this reading with your assessor if timing is graded. |
| **TF-6** | **The display page needs a browser, so it cannot be covered by the backend tests.** Automated browser testing tools exist but add a dependency and time. | The display and upload page are tested **manually** (Section 10). Only the pure ordering logic (`slideshowQueue`) is tested automatically. | Automated browser tests are listed as FUTURE (Section 3.3). |
| **TF-7** | **True parallelism in the 10-upload test is not guaranteed** by the in-process test client (BV-7 in earlier documents). A test that never runs two requests at the same time proves nothing. | TC-SIM-01 must be able to show that requests overlapped, or it must run against the real server over HTTP. | Decide on first use. Record which method you used in Section 15. |
| **TF-8** | **Cases that depend on the consent notice wording** (PRD FR-CON-01 item 4 promises deletion or withdrawal) are only meaningful once decision P-2 of IMPLEMENTATION_PLAN is settled. | TC-CON-08 checks the notice against the features actually built. | Run it **last**, after deciding what was built. |

---

## Coverage of the Requested Topics

| # | Topic | Section |
|---|---|---|
| 1 | Testing objectives | 1 |
| 2 | Testing scope | 2 |
| 3 | Testing types | 3 |
| 4 | Test environment | 4 |
| 5 | Test data | 5 |
| 6 | Unit testing | 6 |
| 7 | API testing | 7 |
| 8 | Database testing | 8 |
| 9 | Integration testing | 9 |
| 10 | Frontend manual testing | 10 |
| 11 | Security testing | 11 |
| 12 | Acceptance testing | 12 |
| Tools | Recommended tools | 13 |
| Test case table (all 19 requested areas and more) | 14 |
| Results section | 15 |

---

# 1. Testing Objectives

| # | Objective | Measured by |
|---|---|---|
| O-1 | Show that every **Must Have** requirement in the PRD works as written | Every Must Have FR has at least one passing test (traceability matrix, Section 15.6) |
| O-2 | Catch the bugs most likely in this project: image handling, upload results, token logic, consent, withdrawal, timers on the display | Automated tests first for these areas |
| O-3 | Prove the **privacy-critical behaviours**: consent is enforced by the server, withdrawn photos disappear, nobody can delete another person's photo, tokens are never leaked | Section 11 and the P1 security rows |
| O-4 | Prove the app **survives bad input**: wrong files, oversized files, broken uploads, missing tokens | The negative cases in Section 14 |
| O-5 | Prove the **display keeps running** when something goes wrong (empty feed, missing image, server stopped) | Manual display tests |
| O-6 | Give the developer **repeatable evidence** for the demo and the assessor | The results section (Section 15) with dated, real outcomes |
| O-7 | Stay **realistic**: a beginner, about 3 to 4 hours of total test work in a 20-hour project | Tiered priorities and a minimum set (Section 3.4) |

**Not an objective:** reaching a coverage percentage (BACKEND_DESIGN 22.6). The goal is that every Must Have acceptance criterion has a test or a recorded manual check.

---

# 2. Testing Scope

## 2.1 In Scope

| Area | Included |
|---|---|
| Backend API | All endpoints in API_SPECIFICATION: E1 config, E2 current visitor, E3 upload with registration, E4 my photos, E5 delete own photo, E6 withdraw consent, E7 slideshow feed, E8 image, E9 thumbnail (if built), E10 operator removal (if built) |
| Backend internals | Image processor, photo storage, security helpers, services, repositories, settings, error handling |
| Database | Constraints, relationships, withdrawal state, cursor safety, concurrency (DATABASE_DESIGN 15.4) |
| Frontend | Upload page (new and returning visitor), display page, admin page (if built) |
| Slideshow logic | Ordering rules in `slideshowQueue` |
| Security and privacy controls | Section 11 and SECURITY_PRIVACY Section 28 |
| Demo conditions | Real phone, real or nearest screen, local network, restart |

## 2.2 Out of Scope

| Not tested | Reason |
|---|---|
| Load or performance beyond the 10-simultaneous-upload test | PRD NFR-PERF targets are Should Have and the app is single-server |
| Formal accessibility audit | A keyboard walk-through and contrast check only (FRONTEND_DESIGN 12) |
| Penetration testing | Not part of the MVP. Security tests here are the design's own controls. |
| Every browser and phone | Chrome or Edge on a computer and one real phone are the target (A-26, NFR-COMP-02) |
| Hosting and HTTPS | Local demo only (A-30). Run these tests again if you ever deploy. |
| Legal or compliance assessment | Not tested, not claimed |
| Third-party libraries' own internals | Only how this project uses them |
| Features not built (Tier B and later) | Rows are tagged **[if built]** |

---

# 3. Testing Types

## 3.1 The Types Used

| Type | What it checks | Automated or manual | Section |
|---|---|---|---|
| **Unit tests** | One small function or class on its own (image processor, storage, token helpers, queue rules, early file checks) | Automated | 6 |
| **Repository and database tests** | Constraints and queries against a temporary database file | Automated | 8 |
| **Service tests** | Business rules with a real temporary database and storage, no web server | Automated | 6 |
| **API (contract) tests** | Every endpoint through FastAPI's test client, checking status, body shape and error codes | Automated | 7 |
| **Integration tests** | Several parts together: upload then feed then image then deletion; backend with frontend; display with server | Automated and manual | 9 |
| **Concurrency tests** | Ten uploads at once, feed during uploads | Automated | 9.3 |
| **Frontend manual tests** | Pages on phone, laptop and TV | Manual | 10 |
| **Security tests** | Hostile input, ownership, token and key handling, headers, secrets | Both | 11 |
| **Acceptance tests** | Each feature against the PRD's "done when" list | Mostly the above, signed off by hand | 12 |
| **Regression tests** | Re-running everything after a fix | Automated, plus the P1 manual set | 3.5 |
| **Exploratory testing** | Unscripted use to find surprises (30 minutes) | Manual | 3.6 |

## 3.2 Why This Mix

| Choice | Reason |
|---|---|
| Most tests are **automated at the API and unit level** | They are fast, repeatable and cheap to re-run after every change. The image processor and the upload workflow hold the hardest logic. |
| The **display and upload pages are tested by hand** | They need a real browser, real photos, real timing and a real screen (TECH_STACK: React tests optional) |
| Pure frontend logic (`slideshowQueue`, `photoChecks`) gets a small automated check | It is plain JavaScript with no browser, so it is cheap to test and is the trickiest frontend code |

## 3.3 Deliberately Not Used in the MVP

| Not used | Why | When it would make sense |
|---|---|---|
| Automated browser tests (end-to-end tools) | Extra dependency, setup time, flaky timing for a beginner | FUTURE, once the manual cases are stable |
| React component tests | Optional (TECH_STACK 4.16) | If more than 2 hours remain |
| Load testing tools | Not required | Production planning |
| Mocking everything | Hides integration bugs. The plan prefers a **real temporary database and real folder** to mocks. | Only to force a failure (for example a failing file write) |

## 3.4 Priorities

| Priority | Meaning | Run when |
|---|---|---|
| **P1** | Must Have requirement, privacy or security essential, or a demo-critical behaviour. **Must be run and must pass (or have a recorded known limitation) before submission.** | Always |
| **P2** | Should Have requirement or an important edge case | If time remains after P1 |
| **P3** | Could Have, optional, or a conditional feature | Only if built and time remains |

**Minimum set for submission:** every automated case (they run in seconds) and every **manual P1** case. This is the realistic target for the time budget (TF-1).

## 3.5 Regression Rule

After any bug fix: re-run the **whole automated suite** (one command), plus the manual P1 cases that touch the same area. A fix is not complete until the suite passes.

## 3.6 Exploratory Session

One timed 30-minute session on a phone and the display, with no script. Try odd things: rapid taps, back button, rotating the phone, a very long name, uploading while the TV runs, switching browsers. Record every surprise in the defect log (Section 15.4), even if it turns out not to be a bug.

---

# 4. Test Environment

## 4.1 Environments

| Environment | Purpose | Used for |
|---|---|---|
| **E-AUTO: automated** | Runs on the developer's computer, in-process, no network | All automated tests. Every test creates its **own temporary database file and photo folder** and deletes them afterwards. |
| **E-LOCAL: local running app** | The real server and the built frontend, one address | Manual tests, rehearsal, restart tests |
| **E-PHONE: real phone** | A real phone on the same network | Upload page, camera and library chooser, real photo files |
| **E-SCREEN: display** | The computer screen or TV that will show the slideshow at 1920 by 1080 if possible | Display tests, soak test |

## 4.2 Environment Rules

| Rule | Reason |
|---|---|
| Automated tests **never** touch the real `data/` folder or real database | Safety. A test must not delete real photos. |
| Use a database **file**, not an in-memory database | In-memory SQLite is per connection and breaks threads and WAL (BACKEND_DESIGN 22.2) |
| Each test starts from an empty database and folder | No test depends on another (NFR-TST-01) |
| Use the application factory with **test settings** (temporary folder, small limits where useful) | BACKEND_DESIGN 6.2 |
| No network access in automated tests | Self-contained |
| Test images are **generated in code**, not committed | Keeps the repository small and safe (BACKEND_DESIGN 22.4) |
| Manual tests use the **built** frontend through the backend (one address) | Same as the demo. The development server is only for coding. |
| Real phones use the computer's **network address**, not `localhost` | Otherwise the phone cannot reach it |
| Record the environment for every run (Section 15.1) | A result means nothing without it |

## 4.3 Configuration Used in Tests

| Setting | Test value | Why |
|---|---|---|
| Data folder | A temporary folder | Isolation |
| Slide seconds and poll seconds | Real defaults (10 and 5) for manual tests. Small values (for example 1 and 1) in tests of the display logic. | Speed |
| Per-file limit, files per request, pixels, photos per visitor | Defaults (5 MiB, 5, 25 MP, 20). Small values in some tests (for example per visitor limit 3) to avoid making 20 images. | Speed |
| Operator key | Empty (feature off) in most tests. A 32-character test value in the E10 tests. A placeholder, never a real key. | Safety |
| Rate limit and global cap [if built] | Very small values in their tests | Speed |

## 4.4 Hardware and Software Record

Fill in Section 15.1 with: computer operating system, Python version, Node version, browser and version, phone model and browser, screen resolution, and the Git commit being tested.

---

# 5. Test Data

## 5.1 Principle

Everything is generated by small helper functions (the "builders") from BACKEND_DESIGN 22.4. This makes each case readable and keeps dangerous or large files out of the repository.

## 5.2 Image Files (PRD Appendix B)

| Name | How it is made | Used by |
|---|---|---|
| `valid.jpg` (about 500 KB) | A picture drawn in code and saved as JPEG | Normal success |
| `valid.png` (about 500 KB) | Same, as PNG | Normal success |
| `portrait.jpg` and `landscape.jpg` | Tall and wide pictures | Display fit, orientation |
| `sideways.jpg` | A JPEG with a rotation tag | Orientation test |
| `gps_photo.jpg` | A JPEG with location data (**tag handling to check on first use**) | Metadata removal |
| `exactly_5mb.jpg` | A valid JPEG padded with zero bytes to exactly 5,242,880 bytes (verified: still decodes) | Size boundary accepted |
| `over_5mb.jpg` | The same plus one byte | Size boundary rejected |
| `fake_image.jpg` | A plain text file named `.jpg` | Wrong content |
| `fake_pdf.jpg` | A PDF-like text file named `.jpg` | Wrong content |
| `empty.jpg` | Zero bytes | Empty file |
| `corrupt.jpg` | A valid JPEG cut in half (verified: opens, then fails on full decode) | Damaged file |
| `appended.jpg` | A valid JPEG with text added to the end | Hidden data removal |
| `animation.gif` | A GIF | Disallowed type by name |
| `gif_as_jpg.jpg` | The same GIF named `.jpg` | Disallowed type by content |
| `document.pdf` | A small PDF-like file | Disallowed type |
| `drawing.svg` | A small SVG text file | Disallowed type |
| `photo.heic` | Any bytes with a `.heic` name (a real HEIC from an iPhone for manual tests) | HEIC message |
| `png_as_jpg.jpg` | A real PNG named `.jpg` | Content wins, accepted |
| `huge_pixels.png` | A solid colour image of 30 megapixels saved as PNG (verified: only about 37 KB) | Pixel limit |
| `cmyk.jpg` | A CMYK image saved as JPEG (verified) | Colour normalisation |
| `large_picture.jpg` | A 4000 by 3000 picture | Resize to 1920 |
| `small_picture.jpg` | A 200 by 100 picture | Not enlarged |
| `animated.png` | An animated PNG, if one can be made easily | Flattening to one frame (**to verify**) |
| `photo.jpg` ten times | The same builder, ten visitors | Concurrency (same file name) |

## 5.3 Real Photos for Manual Tests

| File | Source | Purpose |
|---|---|---|
| 3 or more real phone photos, 3 MB or larger, at least one **portrait** and one **landscape** | The tester's own phone, with no people other than the tester and with permission if anyone else is shown | Orientation, size, real conditions |
| One real **iPhone** photo in the phone's default format, if an iPhone is available | An iPhone | HEIC behaviour (A-12, unverified) |

**Privacy rule for test photos:** use only photos you own, avoid faces of others, and **delete the data folder and database after testing**. Never paste test photos into an AI assistant.

## 5.4 Names and Text

| Name | Purpose |
|---|---|
| `Asha` | Normal name |
| `   Asha   ` | Trimming |
| `Sam` (twice, as two visitors) | Duplicate names |
| 50 characters of `a` and 51 characters of `a` | Boundary |
| `Zoë 王 😀` | Accents and other scripts (confirm against FR-REG-02) |
| `<script>alert(1)</script>` | Plain-text display (NFR-SEC-03) |
| `'; DROP TABLE visitors;--` | SQL characters (text only, never run as SQL) |
| A name with a control character (character code 7) | `NAME_INVALID` |
| Empty and spaces only | `NAME_REQUIRED` |

## 5.5 Tokens and Keys

| Value | Purpose |
|---|---|
| A real token from a test registration | Returning visitor |
| 43 random characters not issued by the server | Unknown token |
| 129 characters | Over-long token |
| A token containing spaces or symbols | Malformed token |
| A 32-character test operator key | E10 tests |
| A wrong key and an empty header | E10 failures |

## 5.6 Fixtures (Shared Setup)

| Fixture | Gives |
|---|---|
| Settings | Test settings with a temporary folder |
| App and client | The application and its test client |
| Registered visitor | A visitor created through the real service, with the raw token |
| Two registered visitors | For ownership tests |
| Image builders | Functions for the files above |
| Feed helper | A function to call E7 and return its parts |

## 5.7 Manual Test Devices

| Device | Role |
|---|---|
| Computer running the server | Server and, in a second window, the display |
| Second browser window or second device | The display ("TV") |
| One real phone | The visitor |
| A second phone or a second browser profile | The second visitor (for same-name and ownership tests) |

---

# 6. Unit Testing

Unit tests check one part alone, using real objects and temporary folders where needed (the plan avoids mocks except to force a failure).

## 6.1 Backend Units

| Group | What is tested | File (BACKEND_DESIGN 5) | Cases in Section 14 |
|---|---|---|---|
| **Image processor** | Valid JPEG and PNG, fake, corrupt, empty, wrong format, pixel limit (not fully decoded), orientation, metadata removal, colour mode, resize, no enlargement, gate release after an error | `test_image_processor.py` | UPL, FMT, CNT |
| **Photo storage** | Stored name shape (32 hex plus extension), unique names, exclusive create, path tricks refused, no leftover temporary file after a failed write, deleting a missing file is not an error | `test_photo_storage.py` | SEC, UPL |
| **Security helpers** | Token is 43 characters and different each time. Hash is stable and is not the token. Operator key comparison accepts only the right key. | `test_security.py` | RET, SEC |
| **Settings** | Bad values stop start-up. Operator key off by default. Short operator key refused [if built]. | `test_settings.py` (optional) | ADM |
| **Consent and visitor services** | Consent must be exactly `true`. Version must match. Registration creates visitor and consent together or neither. Withdrawal changes all together. | `test_visitor_service.py` | REG, CON, WDR |
| **Photo service** | Limits, per-file results in order, ownership, repeat deletion, no orphan file | `test_photo_service.py` | UPL, MUL, DEL |
| **Slideshow service** | First request has no groups. New group. Previous photos ordering. Batch across two polls. Removal. | `test_slideshow_service.py` | SLD, NEW |

## 6.2 Frontend Units (Small, Automated If Possible)

| Module | What is tested | How |
|---|---|---|
| `slideshowQueue` | Four scenarios: first feed, new group arrives, photo removed, empty feed. Duplicates are skipped. The next five photos can be predicted. | Fixed input and expected output. A plain script run with Node's built-in test runner or a small hand-run script. **To verify** the runner is available in the installed Node version. No extra dependency. |
| `photoChecks` | Type, size and count checks use the limits from E1 and the PRD's messages | Same |
| `format` | Shortened file names and file sizes | Same |
| `tokenStorage` | Returns "none" instead of failing when storage is blocked | Manual, in the browser |

**Rule:** if these checks take more than about 30 minutes to set up, do them by hand with fixed data and record the result in Section 15.

## 6.3 What Unit Tests Cannot Show

They do not prove the pieces work together (that is Sections 7 and 9), and they do not prove the page looks right (Section 10).

---

# 7. API Testing

## 7.1 Approach

Every endpoint is called through **FastAPI's test client** in-process against a temporary database and folder. The 34 contract tests in API_SPECIFICATION Section 11 are the core. Both the frontend and backend developer could write them from the document alone. They are implemented in full and are extended by the cases in Section 14.

## 7.2 What Every API Test Checks

| Check | Rule |
|---|---|
| Status code | Matches API_SPECIFICATION exactly |
| Body shape | Only the documented fields. No `token_hash`, no internal ids, no file paths. |
| Error shape | `{"error": {code, message, details, request_id}}` for every error, including 404 and 405 (replacing the framework's default) |
| Request id | The `X-Request-ID` header equals `error.request_id` |
| Nothing stored on failure | Database rows and files unchanged after a refused request |
| No sensitive data | No token, hash, path or stack trace in any body |
| Cache headers | `no-store` on E2 to E6, E10 and the feed. Photo headers as decided (SP-2). |
| Order of results | E3 results are in the order the files were sent |

## 7.3 Endpoint Coverage Map

| Endpoint | Success | Main failures to test | Case area |
|---|---|---|---|
| **E1** config | 200 with limits, durations, types, notice version, points, label | None expected. Config unavailable is a frontend case. | API |
| **E2** current visitor | 200 name and count | 401 no token, bad token, over-long token | RET |
| **E3** upload | 200 new visitor, 200 returning, mixed results | 422 name, consent, files. 409 notice changed. 413 too large. 429 [if built]. 503 busy. File-level errors. | REG, UPL, FMT, CNT, SIZ, MUL, CON |
| **E4** my photos | 200 newest first, own only | 401 | DUP, DEL |
| **E5** delete own photo [if built] | 200 removed, 200 already removed | 401, 403 not owner, 404 unknown | DEL |
| **E6** withdraw | 200 | 422 no confirm, 401 old token | WDR |
| **E7** slideshow | 200 ids, groups, cursor | 422 bad `since_id` | SLD, NEW, EMP |
| **E8** image | 200 image | 404 removed, 404 unknown, 422 or 404 non-number | UPL, SEC |
| **E9** thumbnail [if built] | 200 image | 404 | UPL |
| **E10** operator remove [if built] | 200 | 401 no or wrong key, 404 feature off, 404 unknown photo | ADM |

## 7.4 Error Code Coverage

Every code in the API_SPECIFICATION catalogue should be triggered at least once. A test reads the catalogue and checks that no code appears in the code that is missing from the document, and that every documented code has a case (API_SPECIFICATION and BACKEND_DESIGN 22.5).

| Code | Case |
|---|---|
| `NAME_REQUIRED` | TC-REG-03 |
| `NAME_TOO_LONG` | TC-REG-04 |
| `NAME_INVALID` | TC-REG-05 |
| `CONSENT_REQUIRED` | TC-CON-01 |
| `CONSENT_NOTICE_CHANGED` | TC-CON-04 |
| `NO_FILES` | TC-MUL-04 |
| `TOO_MANY_FILES` | TC-MUL-02 |
| `REQUEST_TOO_LARGE` | TC-SIZ-03 |
| `MALFORMED_REQUEST` | TC-API-04 |
| `INVALID_PARAMETER` | TC-SLD-05 |
| `CONFIRMATION_REQUIRED` | TC-WDR-01 |
| `VISITOR_NOT_RECOGNISED` | TC-RET-03 |
| `NOT_PHOTO_OWNER` | TC-DEL-03 |
| `PHOTO_NOT_FOUND` | TC-DEL-05, TC-SEC-09 |
| `NOT_FOUND` | TC-API-02 |
| `METHOD_NOT_ALLOWED` | TC-API-03 |
| `OPERATOR_KEY_INVALID` | TC-ADM-02 |
| `TOO_MANY_REQUESTS` [if built] | TC-SIZ-07 |
| `SERVER_BUSY` | TC-API-08 |
| `INTERNAL_ERROR` | TC-API-06 |
| File: `FILE_TYPE_NOT_ALLOWED` | TC-FMT-01 to 05 |
| File: `FILE_NOT_AN_IMAGE` | TC-CNT-01 |
| File: `FILE_TOO_LARGE` | TC-SIZ-02 |
| File: `IMAGE_TOO_LARGE_PIXELS` | TC-CNT-07 |
| File: `FILE_EMPTY_OR_CORRUPT` | TC-CNT-02, TC-CNT-03 |
| File: `PHOTO_LIMIT_REACHED` | TC-SIZ-04 |
| File: `SAVE_FAILED` | TC-API-07 |

---

# 8. Database Testing

## 8.1 Approach

Database tests use a **fresh temporary SQLite file**, never the real one. They check that the database itself refuses bad data, so a bug in the application cannot silently corrupt it (DATABASE_DESIGN 15.4). They also prove the relationships between the three tables.

## 8.2 The Tables and What Must Hold

| Table | Key rules to prove |
|---|---|
| `visitors` | Name not unique. Token hash unique. An `active` visitor must have a name and a hash. A `withdrawn` visitor must have neither. Only the two statuses are allowed. |
| `consent_records` | Belongs to a visitor. At most one open consent per visitor. Withdrawn time not earlier than given time. |
| `photos` | Belongs to a visitor. Status only `active` or `removed`. Stored name unique. Numbers always increase and are never reused. |

## 8.3 Relationships to Prove

| Relationship | Test |
|---|---|
| Visitor has many photos | One visitor with 3 photos. Count and list return exactly 3. |
| Visitor has many consent records over time | A visitor can have an old withdrawn consent and one open consent. Two open consents are refused. |
| A photo cannot exist without a visitor | Inserting a photo with a missing visitor is refused (foreign keys must be on for **every** connection, including worker threads) |
| A visitor with photos cannot be deleted | Refused (restrict rule) |
| Withdrawal changes the three tables together | One transaction. Visitor becomes withdrawn with name and hash NULL. Photos removed. Consent withdrawn. All or nothing. |
| Photos never leak across visitors | Queries return only the owner's photos |

## 8.4 Critical Database Setting

SQLite does not enforce foreign keys unless switched on for each connection. TC-DB-01 and TC-DB-09 prove it is on, including for a connection created inside a worker thread. If this test fails, **every other relationship protection is silently absent.**

## 8.5 Consistency Between Database and Files

| Check | Test |
|---|---|
| After a failed upload, no photo row and no file remain | TC-DB-10 |
| After a delete, the row is `removed` and the file is gone | TC-DEL-06 |
| After withdrawal, all files are gone | TC-WDR-06 |
| Every active photo row has a file, and every file has a row | TC-DB-11 (a scan helper used at the end of several tests) |

---

# 9. Integration Testing

## 9.1 Purpose

Integration tests check that parts that work alone also work together. The flows are the ones a real visitor takes.

## 9.2 Flows

| Flow | Parts involved | Where |
|---|---|---|
| **New visitor, upload, see it on the display** | Upload page, E1, E3, image processor, storage, database, E7, E8, display | TC-INT-01 (M) |
| **Returning visitor adds photos** | E2, E3 with token, E7 group with `previous` photos | TC-INT-02 (A), TC-INT-03 (M) |
| **Upload, delete, gone from the display** | E3, E5 or E10, database, files, E7, display | TC-INT-04 (A), TC-INT-05 (M) |
| **Upload, withdraw, gone everywhere** | E3, E6, database, files, E7, display, token | TC-INT-06 (A), TC-INT-07 (M) |
| **Empty display, first upload** | E7 empty, display empty state, E3, display | TC-EMP-03 (M) |
| **Frontend error handling against real error codes** | Page and API | TC-FE-xx (M) |
| **Restart** | Data persists across a server restart | TC-BKI-05 (M) |

## 9.3 Concurrency Tests

| Case | Description |
|---|---|
| TC-SIM-01 | Ten different new visitors upload at the same moment, all using the file name `photo.jpg` |
| TC-SIM-02 | While those uploads run, the feed is requested repeatedly and must always answer |
| TC-SIM-03 | One visitor at 18 photos makes two simultaneous uploads of 5. The limit of 20 must not be exceeded. |
| TC-SIM-04 | Withdraw consent while an upload is in progress. No active photo may remain for the withdrawn visitor (SECURITY_PRIVACY F-7). |

## 9.4 How to Get Real Concurrency

| Method | Note |
|---|---|
| Several threads, each with its own test client, against a temporary database file | First choice. **To verify** that requests really overlap (TF-7), for example by recording start and end times. |
| The real server started in the background and a thread pool making real HTTP requests | Use if the first method never overlaps |
| Record total time and any "busy" responses | Reported in Section 15 |

## 9.5 Frontend and Backend Together

Manual, on the built app: the contract between the page and the server is exercised by every manual case. Pay special attention to: the multipart upload (never set the content type by hand), the token stored after the **first** upload even if every file was rejected, the page reaction to each error code, and the relative `/api` address working both in development and in the built app.

---

# 10. Frontend Manual Testing

## 10.1 Why Manual

The pages depend on a real browser, a real phone and a real screen. The plan records each result by hand (NFR-TST-01, TECH_STACK).

## 10.2 How to Run a Manual Case

| Step | Do |
|---|---|
| 1 | Start the server and open the built app on the stated device |
| 2 | Follow the steps exactly as written |
| 3 | Compare with the expected result |
| 4 | Write what **actually happened** in the Actual result cell (words, not "OK") |
| 5 | Set Status to Pass or Fail |
| 6 | On a Fail, add a row to the defect log (Section 15.4) and note a screenshot name |
| 7 | Never change the expected result |

## 10.3 Upload Page Checklist

| Area | What to look at (cases in Section 14) |
|---|---|
| Layout at 360 px | No sideways scrolling, buttons easy to tap (TC-FE-01) |
| Registration form | Labels, name field, validation messages in the PRD's words (TC-FE-02) |
| Dropzone and previews | Choose button, preview tiles, remove button, "not added" list (TC-FE-03, TC-FE-04) |
| Consent | Notice visible with no click, checkbox unticked on every load (TC-CON-06, TC-CON-07) |
| Progress and results | Busy state, no double submit, per-file results, summary banner (TC-FE-05 to TC-FE-07) |
| Returning visitor | Welcome back, no name or consent, Not you? (TC-RET-07, TC-RET-08) |
| Errors | Each error appears where it happened (TC-FE-08, TC-FE-09) |
| Keyboard | A full upload by keyboard alone (TC-FE-10) |
| Text safety | The script-tag name is plain text (TC-SEC-04) |

## 10.4 Display Page Checklist

| Area | What to look at |
|---|---|
| Starts without input | TC-SLD-06 |
| Whole photo, not cropped or stretched | TC-SLD-07 |
| Slide duration | TC-SLD-08 |
| Loop | TC-SLD-09 |
| Empty state | TC-EMP-02 |
| Missing image skipped | TC-SLD-10 |
| No controls, no cursor | TC-SLD-11 |
| New photo plays next and does not interrupt | TC-NEW-04 to TC-NEW-06 |
| Removal | TC-DEL-08, TC-WDR-09 |
| Server stopped and restarted | TC-BKI-01 to TC-BKI-04 |
| Long run | TC-SLD-13 |

## 10.5 Cross-Device and Accessibility Checks

| Check | Case |
|---|---|
| Phone and laptop both complete the journey | TC-FE-01, TC-FE-11 |
| Reduced motion setting | TC-FE-12 |
| Browser zoom at 200 percent | TC-FE-13 |
| Colour contrast (design values calculated, not re-tested here) | Reference only |

---

# 11. Security Testing

## 11.1 Approach

Security tests check that the design's own controls hold (SECURITY_PRIVACY Section 26 and Section 28). This is **not** a penetration test. The automated cases are in `test_security.py` and the others, and the manual checks in this plan.

## 11.2 Mapping to the Security Plan

| SECURITY_PRIVACY test | Case here |
|---|---|
| S-1 disguised and disallowed files | TC-FMT-xx, TC-CNT-xx |
| S-2 huge pixel image | TC-CNT-07 |
| S-3 GPS removed | TC-UPL-05 |
| S-4 appended data removed | TC-CNT-05 |
| S-5 animated PNG | TC-CNT-06 |
| S-6 traversal names in storage | TC-SEC-01 |
| S-7 traversal file name in upload | TC-SEC-02 |
| S-8 token and hash storage | TC-SEC-03 |
| S-9 consent bypass | TC-CON-01 to 05 |
| S-10 delete another visitor's photo | TC-DEL-03 |
| S-11 token failures | TC-RET-03, TC-RET-04 |
| S-12 withdrawal outcome | TC-WDR-02 to TC-WDR-08 |
| S-13 withdraw during upload | TC-SIM-04 |
| S-14 removed or unknown photos | TC-SEC-09 |
| S-15 image headers | TC-SEC-10 |
| S-16 oversized body | TC-SIZ-03, TC-SIZ-05 |
| S-17 error shapes | TC-API-01 to 06 |
| S-18 logs clean | TC-SEC-06 |
| S-19 rate limit and cap | TC-SIZ-06, TC-SIZ-07 |
| S-20 E10 | TC-ADM-01 to TC-ADM-07 |
| S-21 10 simultaneous uploads | TC-SIM-01 |
| S-22 SQL characters | TC-SEC-05 |
| M-1 to M-10 manual checks | TC-SEC-04, TC-SEC-07, TC-SEC-08, TC-SEC-11 to TC-SEC-15 |

## 11.3 Safety Rules for Security Testing

| Rule | Reason |
|---|---|
| Only test **your own** running copy | Never test anyone else's system |
| Use test data and test keys only | No real secrets in tests |
| Do not paste test output with tokens into AI assistants | Section 22 of SECURITY_PRIVACY |
| Delete the data folder after the tests | Privacy |

---

# 12. Acceptance Testing

## 12.1 What It Is

Acceptance testing asks: **is each feature done according to the PRD?** The PRD's Section 7 gives a "done when" rule for each feature. Acceptance is a **sign-off table** (Section 15.5), completed by the developer after the cases pass, and checkable by an assessor.

## 12.2 Feature Acceptance Map (From PRD Section 7)

| Feature | Done when | Cases |
|---|---|---|
| New visitor registration | FR-REG-01 to FR-REG-04 pass. A first-time submission creates exactly one visitor and one consent record. | TC-REG-xx, TC-DUP-xx |
| Returning visitor identification | FR-ID-01 to FR-ID-03a and FR-ID-05 pass. The browser keeps the visitor across restarts. Same-name people stay separate. | TC-RET-xx, TC-DUP-xx |
| Public display consent | FR-CON-01 to FR-CON-03 pass. The server refuses a no-consent request even when sent directly. | TC-CON-xx |
| Photo upload | FR-UPL-01 to FR-UPL-04 pass. A mixed submission gives correct per-file results. | TC-UPL-xx, TC-MUL-xx |
| File validation and limits | FR-VAL-01 to FR-VAL-05 pass using every sample file in Appendix B | TC-FMT-xx, TC-CNT-xx, TC-SIZ-xx |
| Photo storage | FR-STO-01, 02, 04 and 05 pass. No orphan files or records after forced failures. | TC-DB-xx, TC-SEC-xx, TC-API-07 |
| Slideshow display | FR-DSP-01 to FR-DSP-07 pass on a second device or window | TC-SLD-xx, TC-EMP-xx |
| Automatic updates | FR-AUT-01 to FR-AUT-03 pass. Timing is measured and recorded. | TC-NEW-xx |
| Previous photo retrieval | FR-PRV-01 to FR-PRV-04 pass. Order matches FR-PRV-02. | TC-NEW-02, TC-NEW-03, TC-DUP-04 |
| Multiple simultaneous uploads | FR-CNC-01 to FR-CNC-04 pass in an automated test with at least 10 parallel uploads | TC-SIM-xx |
| Consent withdrawal (Should) | FR-WDR-01 to FR-WDR-04 pass. Files confirmed gone. | TC-WDR-xx |
| Photo deletion (Should) | FR-DEL-01 to FR-DEL-05 pass. A different visitor cannot delete the photo. | TC-DEL-xx |

## 12.3 MVP Release Gate (PRD Section 7)

The MVP can be submitted when **every Must Have requirement passes**, all required documents exist, and the demo can be run from the README instructions on a clean setup.

## 12.4 Demo Acceptance

A final **clean-setup run**: clone the repository into a new folder, follow the README exactly, run the automated suite, start the app and complete the demo script (new visitor, upload, display, returning visitor, withdrawal). Record the result in Section 15.5.

---

# 13. Recommended Tools

All tools below are free. Install the exact versions you pin on Day 1 (IMPLEMENTATION_PLAN D1-01).

## 13.1 Automated Testing

| Tool | Use | Why | Notes |
|---|---|---|---|
| **pytest** | Runs every automated test and reports results | The standard Python tool. Plain functions, simple output, strong fixtures. TECH_STACK 4.12. | Essential |
| **FastAPI TestClient** | Calls the API in-process without starting a server | Fast, no network. Part of FastAPI (TECH_STACK 4.13). | Needs **httpx** installed. Use as a context manager so start-up and shutdown run (**to verify** against the installed version). |
| **httpx** | HTTP client used by the test client | Required by the test client | Essential |
| **pytest temporary folders and monkeypatch** | A fresh folder per test, and a safe way to force a failure (for example a failing file write) | Built into pytest | Use instead of mocking libraries |
| **Pillow** | Build test images in code | Already a project dependency | No extra install |
| **Python thread pool (standard library)** | The simultaneous upload tests | No extra dependency | See Section 9.4 |
| **Test markers** (for example `slow` and `concurrency`) | Run the fast tests first | Built into pytest | Optional |
| **pytest-cov** | Optional coverage report | Only to find untested lines. **Coverage percentage is not a goal.** | Optional, extra dependency |
| **Node built-in test runner or a hand-run script** | Pure frontend helpers | No extra dependency. **To verify** the runner exists in the installed Node. | Optional |

## 13.2 Manual Testing Aids

| Tool | Use |
|---|---|
| Browser developer tools | Device mode (360 px width), network tab (headers, status codes), application tab (local storage, token), console (errors) |
| A real phone on the same network | The visitor |
| `curl` (command line) | Send a raw request with no consent, a bad token, a wrong key, a traversal file name |
| SQLite command-line tool or a free database viewer | Read the database after a test (names, hashes, statuses) |
| File explorer | See the photo folder (random names, files gone after removal) |
| Screenshot tool | Evidence for fails and for the demo |
| A stopwatch | Timing cases |
| A simple checklist | Section 15 |

## 13.3 Not Recommended for the MVP

| Not used | Reason |
|---|---|
| Browser automation tools | Extra dependency and setup (TF-6). FUTURE. |
| Mocking libraries | Real temporary database and folder give more truth |
| Load testing tools | Not required |
| A test management service | A table in this file is enough |

## 13.4 How to Run (Words Only)

| Task | How |
|---|---|
| Run the whole automated suite | One pytest command from the backend folder. It must pass from a **fresh clone**. |
| Run one file or one case | Name it in the same command |
| Show detail | Use the verbose option |
| Save evidence | Copy the final pytest summary text into `docs/test-results/` with the date and commit |

---

# 14. Detailed Test Case Table

**How to read.** Columns: **ID** (with (A) automated or (M) manual), **Feature**, **Preconditions**, **Test steps**, **Test input**, **Expected result**, **Priority**, **Actual result**, **Status**. **Actual result** and **Status** are blank on purpose. All rows are `Not run`.

Notation: "E3" means the upload endpoint. "Standard error" means the standard error shape. "Nothing stored" means no visitor, consent, photo row or file was created.

## 14.1 Visitor Registration

| ID | Feature | Preconditions | Test steps | Test input | Expected result | Priority | Actual result | Status |
|---|---|---|---|---|---|---|---|---|
| TC-REG-01 (A) | Register a new visitor | Empty database. Notice version is `v1`. | 1) Send E3 with name, consent, version and one file. | Name `Asha`, consent `true`, version `v1`, `valid.jpg` | 200. `is_new` true. Token present (43 characters). One result `accepted`. Database: 1 visitor (active), 1 open consent with version `v1`, 1 active photo. | P1 | — | Not run |
| TC-REG-02 (A) | Name is trimmed | Empty database | 1) Send E3 with a padded name. | Name `   Asha   `, valid input | Stored name is `Asha`. | P1 | — | Not run |
| TC-REG-03 (A) | Empty name refused | Empty database | 1) Send E3 with an empty name. 2) Repeat with spaces only. | Name `` and `     ` | 422 `NAME_REQUIRED`, message "Please enter your name." Nothing stored. | P1 | — | Not run |
| TC-REG-04 (A) | Name length boundary | Empty database | 1) Send a 50-character name. 2) Send a 51-character name. | 50 and 51 characters of `a` | First accepted. Second 422 `NAME_TOO_LONG`, nothing stored. | P1 | — | Not run |
| TC-REG-05 (A) | Control character refused | Empty database | 1) Send a name with character code 7. | `Asha` plus code 7 | 422 `NAME_INVALID`. Nothing stored. | P1 | — | Not run |
| TC-REG-06 (A) | Script-like name kept as text | Empty database | 1) Register with the script-tag name. 2) Read E2. | `<script>alert(1)</script>` | Accepted. E2 returns the same text unchanged (as a JSON string). | P1 | — | Not run |
| TC-REG-07 (A) | Accented and other-script name | Empty database | 1) Register. 2) Read E2. | `Zoë 王 😀` | Accepted and returned identical (confirm against FR-REG-02 whether emoji are allowed). | P3 | — | Not run |
| TC-REG-08 (A) | Visitor is created even if every file is rejected | Empty database | 1) Send E3 with valid registration and only a bad file. 2) Read E2 with the returned token. | Name `Asha`, consent, `fake_image.jpg` | 200. Token returned. Result `FILE_NOT_AN_IMAGE`. Database: 1 visitor, 1 consent, 0 photos. E2 works (FR-REG-03 AC2). | P1 | — | Not run |
| TC-REG-09 (A) | Registration is all or nothing | Force the consent insert to fail (monkeypatch) | 1) Send E3 as a new visitor. | Valid input | A standard error (500 or 503). **No visitor row remains** and no photo. | P2 | — | Not run |
| TC-REG-10 (M) | Register on a phone | Built app, real phone, empty database | 1) Open the page. 2) Enter name, tick consent, choose 2 photos. 3) Press Upload. 4) Open the browser storage view and find the token. | `Asha`, two JPEGs | Per-file results show "Added". A token exists in browser storage. Form changes to the returning-visitor view. | P1 | — | Not run |

## 14.2 Duplicate Visitor Names

| ID | Feature | Preconditions | Test steps | Test input | Expected result | Priority | Actual result | Status |
|---|---|---|---|---|---|---|---|---|
| TC-DUP-01 (A) | Two visitors, same name | Empty database | 1) Register `Sam` with a photo. 2) Register another `Sam` with another photo (no token). | Two registrations named `Sam` | Two visitor rows, two different token hashes. Each token on E2 shows that visitor's own `photo_count` (1 and 1). | P1 | — | Not run |
| TC-DUP-02 (A) | My photos stay separate | TC-DUP-01 done, each with different photo counts | 1) Call E4 with each token. | Token A, Token B | Each list contains only that visitor's photos, newest first. | P1 | — | Not run |
| TC-DUP-03 (A) | Name is never an identifier | A visitor named `Sam` exists | 1) Send E3 with name `Sam`, consent, and **no token**. | Name `Sam`, no token | A **new** third visitor is created. It cannot see Sam's photos. | P1 | — | Not run |
| TC-DUP-04 (A) | Previous photos never cross visitors | Two `Sam` visitors with photos | 1) Make visitor A upload again. 2) Read E7 groups. | Token A | `previous` lists only visitor A's earlier photos, never B's (FR-PRV-04). | P1 | — | Not run |
| TC-DUP-05 (M) | Same name on two phones | Two phones, empty database | 1) Register `Sam` on phone 1. 2) Register `Sam` on phone 2. 3) Reopen both. | `Sam` on both | Each phone shows "Welcome back, Sam" and (if built) only its own photos. | P2 | — | Not run |

## 14.3 Returning Visitor Identification

| ID | Feature | Preconditions | Test steps | Test input | Expected result | Priority | Actual result | Status |
|---|---|---|---|---|---|---|---|---|
| TC-RET-01 (A) | Recognise a valid token | Registered visitor with 1 photo | 1) Call E2 with the token. | Valid token | 200 with name, `photo_count` 1, consent information. | P1 | — | Not run |
| TC-RET-02 (A) | Returning upload needs no name or consent | Registered visitor | 1) Send E3 with files and the token only. | Token, `valid.png` | 200. `is_new` false. `token` null. No new visitor row. No name or consent required. Photo count increases. | P1 | — | Not run |
| TC-RET-03 (A) | No token on a token endpoint | None | 1) Call E2 with no token header. | No token | 401 `VISITOR_NOT_RECOGNISED`, standard error. Nothing changes. | P1 | — | Not run |
| TC-RET-04 (A) | Bad tokens | None | 1) Call E2 with a random 43-character token. 2) With a 129-character token. 3) With a token containing spaces. | Three bad tokens | Each gives 401 `VISITOR_NOT_RECOGNISED`. | P1 | — | Not run |
| TC-RET-05 (A) | Bad token on upload means new visitor | None | 1) Send E3 with a bad token, **no name and no consent**. | Bad token, `valid.jpg` | 422 `NAME_REQUIRED` (treated as new). Nothing stored. | P1 | — | Not run |
| TC-RET-06 (A) | Bad token with full registration | None | 1) Send E3 with a bad token plus name and consent. | Bad token, `Asha`, consent | A new visitor is created and a new token returned (ERR-16). | P2 | — | Not run |
| TC-RET-07 (M) | Welcome back after closing the browser | Phone registered | 1) Close the browser completely. 2) Reopen the upload page. | Same phone | "Welcome back, name". No name field and no consent box (FR-ID-02). | P1 | — | Not run |
| TC-RET-08 (M) | Not you? | Returning view shown [if built] | 1) Press "Not you?". 2) Check the server with the old token. | Same phone | The new-visitor form appears. The token is gone from the browser. The server still holds the visitor and photos. | P2 | — | Not run |
| TC-RET-09 (M) | Token removed by hand | Phone registered | 1) Delete the token in browser storage. 2) Reload. | Same phone | New-visitor form shown. No error page, no error message. | P1 | — | Not run |
| TC-RET-10 (M) | Storage blocked or private window | Private window | 1) Register and upload. 2) Observe. 3) Reload. | Private window | Upload works. One warning that returning-visitor features may not work (ERR-23). After reload the visitor looks new. | P2 | — | Not run |
| TC-RET-11 (M) | Different browser or device | Phone registered | 1) Open the page in another browser. | Other browser | Treated as new (ERR-25). Recorded as a documented limitation, not a defect. | P2 | — | Not run |
| TC-RET-12 (A) | Token appears only once | Registered visitor | 1) Collect bodies of E2, returning E3 and E4. 2) Search for the token and the hash. | Responses | Neither the raw token nor any hash appears in any response after registration. | P1 | — | Not run |

## 14.4 Valid Photo Uploads

| ID | Feature | Preconditions | Test steps | Test input | Expected result | Priority | Actual result | Status |
|---|---|---|---|---|---|---|---|---|
| TC-UPL-01 (A) | Valid JPEG | New visitor | 1) Upload. 2) Inspect the folder and database. | `valid.jpg` | `accepted`. A file named 32 lowercase hex characters plus `.jpg` exists. Row `active`, content type `image/jpeg`. | P1 | — | Not run |
| TC-UPL-02 (A) | Valid PNG | New visitor | 1) Upload. 2) Inspect. | `valid.png` | `accepted`, `.png`, `image/png`. | P1 | — | Not run |
| TC-UPL-03 (A) | PNG named .jpg | New visitor | 1) Upload. | `png_as_jpg.jpg` | Accepted under its real format (stored as PNG). | P1 | — | Not run |
| TC-UPL-04 (A) | Sideways photo is corrected | New visitor | 1) Upload. 2) Read the stored image's size and a marked pixel. | `sideways.jpg` | Stored image is upright (orientation applied before metadata removed). | P2 | — | Not run |
| TC-UPL-05 (A) | Location data removed | New visitor | 1) Upload. 2) Read the stored file's metadata. | `gps_photo.jpg` | No GPS or camera metadata in the stored file (FR-STO-03). | P2 | — | Not run |
| TC-UPL-06 (A) | Resize rules | New visitor | 1) Upload both. 2) Check stored sizes. | `large_picture.jpg`, `small_picture.jpg` | Longest side of the large one is at most 1920. The small one is not enlarged. | P2 | — | Not run |
| TC-UPL-07 (A) | CMYK colour | New visitor | 1) Upload. 2) Check the stored mode. | `cmyk.jpg` | Stored image is in a normal colour mode that browsers show correctly. | P3 | — | Not run |
| TC-UPL-08 (A) | Image can be fetched | Accepted photo | 1) GET E8 for the photo. | Photo id | 200. Correct content type. Cache and `nosniff` headers as decided. Body is a valid image. | P1 | — | Not run |
| TC-UPL-09 (A) | Stored file is a new copy | New visitor | 1) Upload. 2) Compare stored bytes with the original. | `valid.jpg` | Stored bytes differ (re-encoded). The original file name does not appear anywhere in the folder. | P1 | — | Not run |
| TC-UPL-10 (M) | Real phone photos | Real phone, real photos | 1) Upload a portrait and a landscape photo, 3 MB or more. 2) View both on the display. | Real photos | Both accepted, upright and uncropped on the display. A HEIC photo (iPhone) behaviour recorded (A-12). | P1 | — | Not run |
| TC-UPL-11 (M) | Choose and drag-and-drop | Laptop | 1) Add photos with the Choose button. 2) Add photos by dragging. | Two JPEGs each way | Both ways add previews. Drag-over state is visible. | P2 | — | Not run |
| TC-UPL-12 (A) | Thumbnail [if built] | New visitor | 1) Upload. 2) GET E9. | `valid.jpg` | Thumbnail at most 320 pixels on the longest side. | P3 | — | Not run |

## 14.5 Unsupported File Formats

| ID | Feature | Preconditions | Test steps | Test input | Expected result | Priority | Actual result | Status |
|---|---|---|---|---|---|---|---|---|
| TC-FMT-01 (A) | GIF by name | Registered visitor | 1) Upload. | `animation.gif` | `FILE_TYPE_NOT_ALLOWED` ("Only JPEG and PNG photos are allowed."). No file stored. | P1 | — | Not run |
| TC-FMT-02 (A) | GIF named .jpg | Registered visitor | 1) Upload. | `gif_as_jpg.jpg` | `FILE_TYPE_NOT_ALLOWED` (real format is GIF). No file stored. | P1 | — | Not run |
| TC-FMT-03 (A) | PDF | Registered visitor | 1) Upload. | `document.pdf` | `FILE_TYPE_NOT_ALLOWED`. | P1 | — | Not run |
| TC-FMT-04 (A) | SVG | Registered visitor | 1) Upload. | `drawing.svg` | `FILE_TYPE_NOT_ALLOWED`. | P1 | — | Not run |
| TC-FMT-05 (A) | HEIC message | Registered visitor | 1) Upload. | `photo.heic` | `FILE_TYPE_NOT_ALLOWED` with the HEIC message: "This photo format is not supported. Please use JPEG or PNG." | P1 | — | Not run |
| TC-FMT-06 (A) | Early browser check matches | Helper function | 1) Run the early check on the same names and sizes. | The files above | The early check rejects the same files with the PRD messages. The server still repeats every check. | P2 | — | Not run |

## 14.6 Incorrect File Contents

| ID | Feature | Preconditions | Test steps | Test input | Expected result | Priority | Actual result | Status |
|---|---|---|---|---|---|---|---|---|
| TC-CNT-01 (A) | Text renamed .jpg | Registered visitor | 1) Upload. | `fake_image.jpg` | `FILE_NOT_AN_IMAGE` ("This file is not a valid photo."). Nothing stored. | P1 | — | Not run |
| TC-CNT-02 (A) | Empty file | Registered visitor | 1) Upload. | `empty.jpg` | `FILE_EMPTY_OR_CORRUPT`. Nothing stored. | P1 | — | Not run |
| TC-CNT-03 (A) | Truncated image | Registered visitor | 1) Upload. | `corrupt.jpg` | `FILE_EMPTY_OR_CORRUPT`. **No file left on disk**, no row. | P1 | — | Not run |
| TC-CNT-04 (A) | PDF named .jpg | Registered visitor | 1) Upload. | `fake_pdf.jpg` | `FILE_NOT_AN_IMAGE`. | P1 | — | Not run |
| TC-CNT-05 (A) | Hidden data after the image | Registered visitor | 1) Upload. 2) Search the stored file for the appended text. | `appended.jpg` | Either accepted with the appended text **absent** from the stored file, or rejected. | P1 | — | Not run |
| TC-CNT-06 (A) | Animated PNG | Registered visitor | 1) Upload. 2) Inspect. | `animated.png` | One still frame is stored, or a clear rejection (**to verify**). | P2 | — | Not run |
| TC-CNT-07 (A) | Too many pixels | Registered visitor | 1) Upload. 2) Watch memory in the test. | `huge_pixels.png` (30 MP, small file) | `IMAGE_TOO_LARGE_PIXELS`. Rejected **without being fully decoded** (TF-3). No file stored. | P1 | — | Not run |
| TC-CNT-08 (A) | Oversized fake file | Registered visitor | 1) Upload a fake image over the size limit. | `over_5mb` fake | `FILE_TOO_LARGE` (the size check comes first, TF-4). | P2 | — | Not run |

## 14.7 Oversized Files, Limits and Rate Limits

| ID | Feature | Preconditions | Test steps | Test input | Expected result | Priority | Actual result | Status |
|---|---|---|---|---|---|---|---|---|
| TC-SIZ-01 (A) | Exactly at the size limit | Registered visitor | 1) Upload. | `exactly_5mb.jpg` (5,242,880 bytes) | `accepted`. | P1 | — | Not run |
| TC-SIZ-02 (A) | One byte over | Registered visitor | 1) Upload. | `over_5mb.jpg` (5,242,881 bytes) | `FILE_TOO_LARGE` ("This photo is larger than 5 MB."), detail `max_file_bytes` 5242880. Nothing stored. | P1 | — | Not run |
| TC-SIZ-03 (A) | Whole request too large | Registered visitor | 1) Send a body larger than 27,262,976 bytes. | Oversized body | 413 `REQUEST_TOO_LARGE`. Nothing stored. Rejected before the whole body is stored (**to verify**). | P1 | — | Not run |
| TC-SIZ-04 (A) | Per-visitor photo limit | Registered visitor at the limit (use a small test limit) | 1) Upload one more photo. | `valid.jpg` | `PHOTO_LIMIT_REACHED` for that file. Count does not exceed the limit. | P2 | — | Not run |
| TC-SIZ-05 (A) | Length-less oversized body | Test server | 1) Send a streamed body with no declared length that grows past the limit. | Streamed oversized body | Stopped at the limit with 413. Memory does not grow without bound (**to verify**, BV-6). | P1 | — | Not run |
| TC-SIZ-06 (A) | Global photo cap [if built] | Cap set very low | 1) Upload beyond the cap. | Valid files | Refused with the documented message. Nothing stored beyond the cap. | P3 | — | Not run |
| TC-SIZ-07 (A) | Rate limit [if built] | Limit set very low | 1) Send more upload requests than allowed in the window. | Repeated valid requests | 429 `TOO_MANY_REQUESTS` with a `Retry-After` header. Nothing stored for refused requests. | P3 | — | Not run |
| TC-SIZ-08 (A) | Size limits come from settings | Test settings changed | 1) Change the file limit setting. 2) Check E1 and the message. | New value | E1 and messages use the new value. No number is hard-coded (NFR-MNT-01). | P2 | — | Not run |

## 14.8 Multiple File Uploads

| ID | Feature | Preconditions | Test steps | Test input | Expected result | Priority | Actual result | Status |
|---|---|---|---|---|---|---|---|---|
| TC-MUL-01 (A) | Five files at once | Registered visitor | 1) Upload five different valid files. | Five images | Five `accepted` results in the order sent. Five photos, five files. | P1 | — | Not run |
| TC-MUL-02 (A) | Six files | Registered visitor | 1) Upload six files. | Six images | 422 `TOO_MANY_FILES`. Nothing stored. | P1 | — | Not run |
| TC-MUL-03 (A) | Mixed good and bad | New visitor | 1) Upload three files. | `valid.jpg`, `fake_image.jpg`, `valid.png` | 200. Result 1 accepted, result 2 `FILE_NOT_AN_IMAGE`, result 3 accepted, in that order. Two photos stored. | P1 | — | Not run |
| TC-MUL-04 (A) | No files, and empty file picker | Registered visitor | 1) Send E3 with no files. 2) Send one empty file part with no name. | None | 422 `NO_FILES` both times. Nothing stored. | P1 | — | Not run |
| TC-MUL-05 (A) | One oversized file among valid files | Registered visitor | 1) Upload three files, one over the limit. | Two valid, `over_5mb.jpg` | Request is 200. Two accepted. One `FILE_TOO_LARGE`. | P1 | — | Not run |
| TC-MUL-06 (A) | All files rejected | Registered visitor | 1) Upload two bad files. | `fake_image.jpg`, `empty.jpg` | 200 with two error results. Nothing stored. | P1 | — | Not run |
| TC-MUL-07 (A) | Limit counting inside one request | Registered visitor close to the limit (small test limit) | 1) Upload more files than the remaining space. | Valid files | Files up to the limit accepted. The rest `PHOTO_LIMIT_REACHED`, in order. | P2 | — | Not run |
| TC-MUL-08 (M) | Multi-select on a phone | Real phone | 1) Select several photos in one go. 2) Upload. | 3 photos | All appear as previews. Results list shows each file. | P1 | — | Not run |

## 14.9 Simultaneous Uploads

| ID | Feature | Preconditions | Test steps | Test input | Expected result | Priority | Actual result | Status |
|---|---|---|---|---|---|---|---|---|
| TC-SIM-01 (A) | Ten uploads at once | Empty temporary database. Real overlap confirmed (TF-7). | 1) Ten threads each register a new visitor and upload `photo.jpg` at the same moment. | Ten visitors, same file name | Exactly 10 visitors, 10 photo rows, 10 **distinct** stored files. Each photo belongs to the right visitor. No raw database error. No file overwritten (FR-CNC-01, 02). | P1 | — | Not run |
| TC-SIM-02 (A) | Feed stays responsive | TC-SIM-01 running | 1) Request E7 repeatedly during the uploads. | E7 calls | Every request succeeds and returns valid JSON (FR-CNC-04). | P1 | — | Not run |
| TC-SIM-03 (A) | Limit race | A visitor holds 18 of 20 photos | 1) Two simultaneous uploads of 5 files each. | Two requests | The visitor never ends with more than 20 active photos (**may reveal a defect**). | P2 | — | Not run |
| TC-SIM-04 (A) | Withdrawal during upload | A visitor uploading 5 photos | 1) Start the upload. 2) Withdraw at the same moment. 3) Wait for both. | Token, 5 files | No active photo belongs to the withdrawn visitor. No file exists without an active row. (SECURITY_PRIVACY F-7.) | P1 | — | Not run |
| TC-SIM-05 (A) | Busy database handled | Lock the database briefly (test) | 1) Send an upload while the database is locked longer than the timeout. | Valid input | Waits within the timeout, or 503 `SERVER_BUSY` with `Retry-After`. Never a raw error (FR-CNC-03). | P1 | — | Not run |
| TC-SIM-06 (M) | Two phones at once | Two phones | 1) Both press Upload at the same moment. | Two sets of photos | Both succeed. Both visitors' photos appear on the display. | P2 | — | Not run |
| TC-SIM-07 (M) | Double tap on Upload | Phone | 1) Tap Upload twice quickly. | One photo | One upload only. The button shows a busy state (FR-CNC-05). | P2 | — | Not run |

## 14.10 Database Relationships and Integrity

| ID | Feature | Preconditions | Test steps | Test input | Expected result | Priority | Actual result | Status |
|---|---|---|---|---|---|---|---|---|
| TC-DB-01 (A) | Foreign keys are on | Temporary database | 1) Insert a photo whose `visitor_id` does not exist. | Missing visitor | The database refuses it. | P1 | — | Not run |
| TC-DB-02 (A) | Visitor with photos cannot be deleted | One visitor with a photo | 1) Delete the visitor row. | Visitor id | Refused (restrict rule). | P1 | — | Not run |
| TC-DB-03 (A) | Names are not unique | Empty tables | 1) Insert two visitors with the same name and different hashes. | `Sam`, `Sam` | Both saved. | P1 | — | Not run |
| TC-DB-04 (A) | Token hash is unique | One visitor | 1) Insert a second visitor with the same hash. | Same hash | Refused. | P1 | — | Not run |
| TC-DB-05 (A) | One open consent per visitor | One visitor with an open consent | 1) Insert a second open consent. | Second record | Refused. A new one is allowed after the first is withdrawn. | P1 | — | Not run |
| TC-DB-06 (A) | Status values | Temporary database | 1) Insert a visitor and a photo with an invalid status. | Status `banana` | Both refused. | P1 | — | Not run |
| TC-DB-07 (A) | Active visitor needs name and hash | Temporary database | 1) Insert an active visitor with a NULL name. 2) With a NULL hash. | NULL values | Both refused. Withdrawn visitor with NULL name and hash is accepted. A withdrawn visitor with a name is refused. | P1 | — | Not run |
| TC-DB-08 (A) | Photo numbers always increase | Several photos, the newest removed | 1) Add a new photo. | New photo | Its number is greater than every earlier number, even removed ones (cursor safety). | P1 | — | Not run |
| TC-DB-09 (A) | Foreign keys on every connection | App with several connections | 1) Run the orphan insert in a worker thread and a second connection. | Missing visitor | Refused in every connection. | P1 | — | Not run |
| TC-DB-10 (A) | No orphan after a failed upload | Force the file write to fail (monkeypatch) | 1) Upload a valid file. | `valid.jpg` | Result `SAVE_FAILED`. No photo row, no file. | P1 | — | Not run |
| TC-DB-11 (A) | Rows and files agree | After a mixed set of tests | 1) Compare active rows to files in the folder. | State after tests | Every active row has a file. Every file has a row (or is a documented leftover of a removed row). | P1 | — | Not run |
| TC-DB-12 (A) | Withdrawal timing rule | A visitor with an open consent | 1) Set `withdrawn_at` earlier than `given_at`. | Earlier time | Refused. | P2 | — | Not run |
| TC-DB-13 (A) | Eligible photos query | Active photo, removed photo, withdrawn visitor's photo | 1) Run the feed query. | Three photos | Only the active photo of an active visitor appears. | P1 | — | Not run |
| TC-DB-14 (A) | Relationship counts | One visitor, 3 photos, 1 consent | 1) Count photos and consent records through the repositories. | State | 3 photos, 1 consent, all linked to the one visitor. | P2 | — | Not run |
| TC-DB-15 (M) | Look at the real database | After a manual session | 1) Open the database in a viewer. 2) Read the three tables. | Real data | No raw token anywhere. Hashes are 64 hexadecimal characters. Names as typed (trimmed). Statuses valid. | P1 | — | Not run |

## 14.11 Consent Requirement

| ID | Feature | Preconditions | Test steps | Test input | Expected result | Priority | Actual result | Status |
|---|---|---|---|---|---|---|---|---|
| TC-CON-01 (A) | No consent | Empty database | 1) Send E3 with no consent field. 2) With `false`. | Missing, `false` | 422 `CONSENT_REQUIRED` ("Please agree to the display notice to continue."). Nothing stored. | P1 | — | Not run |
| TC-CON-02 (A) | Only exactly true counts | Empty database | 1) Send consent as `yes`, `1`, `True` with a capital, and empty. | Other values | Each refused with `CONSENT_REQUIRED`. Nothing stored. | P1 | — | Not run |
| TC-CON-03 (A) | Direct request skips the page | Server running | 1) Send a raw request with `curl` and no consent. | Raw request | Refused. Nothing stored (FR-CON-02 AC3). | P1 | — | Not run |
| TC-CON-04 (A) | Old notice version | Notice version `v1` | 1) Send consent with version `v0`. | `v0` | 409 `CONSENT_NOTICE_CHANGED` with `current_version`. Nothing stored. | P1 | — | Not run |
| TC-CON-05 (A) | Consent is recorded | New visitor | 1) Register. 2) Read the consent record. | Valid input | One record: version `v1`, a given time, no withdrawn time. It holds no name, token or photo reference. | P1 | — | Not run |
| TC-CON-06 (M) | Notice visible without extra clicks | New-visitor page | 1) Open the page. | Phone | The full notice text is visible on the page with no click or expansion (FR-CON-01 AC1). | P1 | — | Not run |
| TC-CON-07 (M) | Checkbox unticked on every load | New-visitor page | 1) Tick it. 2) Reload. | Phone | Unticked after every reload. Pressing Upload unticked shows the message and sends nothing. | P1 | — | Not run |
| TC-CON-08 (M) | Notice matches what was built | After features are final | 1) Read each point of the notice. 2) Check it against the features that exist. | Notice text | Every promise in the notice (for example deletion or withdrawal at any time) is something the app really does. | P1 | — | Not run |
| TC-CON-09 (M) | Notice changed after the page loaded | Page open, then notice version changed on the server | 1) Press Upload. | Phone | The page reloads the notice, **unticks** the box and asks for agreement again. Chosen photos are kept. | P2 | — | Not run |
| TC-CON-10 (A) | Returning visitor not asked again | Registered visitor | 1) Upload with only a token. | Token | Accepted with no consent field. | P1 | — | Not run |

## 14.12 Consent Withdrawal

| ID | Feature | Preconditions | Test steps | Test input | Expected result | Priority | Actual result | Status |
|---|---|---|---|---|---|---|---|---|
| TC-WDR-01 (A) | Confirmation required | Registered visitor with 2 photos | 1) Call E6 with no body. 2) With `confirm` false. | No confirm | 422 `CONFIRMATION_REQUIRED`. Nothing changes. | P1 | — | Not run |
| TC-WDR-02 (A) | Withdraw | Registered visitor with 2 photos | 1) Call E6 with `confirm` true. | Token, `confirm: true` | 200. | P1 | — | Not run |
| TC-WDR-03 (A) | Old token refused | After TC-WDR-02 | 1) Call E2 with the old token. | Old token | 401 `VISITOR_NOT_RECOGNISED`. | P1 | — | Not run |
| TC-WDR-04 (A) | Old token on upload means new | After withdrawal | 1) Send E3 with the old token and no consent. | Old token | Treated as a new visitor, so consent and name are required (ERR-19). | P1 | — | Not run |
| TC-WDR-05 (A) | Photos leave the feed | After withdrawal | 1) Read E7. | Feed | The visitor's photo ids are not in `photo_ids`. | P1 | — | Not run |
| TC-WDR-06 (A) | Files are deleted | After withdrawal | 1) List the photo folder. 2) GET E8 for each former photo. | Folder, ids | The files are gone. E8 returns 404 for each. | P1 | — | Not run |
| TC-WDR-07 (A) | Personal data erased | After withdrawal | 1) Read the visitor row and consent record. | Database | Visitor `withdrawn`, name NULL, hash NULL. Consent has a withdrawn time. Photos `removed`. | P1 | — | Not run |
| TC-WDR-08 (A) | Others untouched | Two visitors, one withdraws | 1) Withdraw visitor A. 2) Check B. | Two visitors | B's photos, files and token still work. | P1 | — | Not run |
| TC-WDR-09 (M) | Withdraw from the page and watch the display | Display running, phone registered with 2 photos on the loop | 1) Press the withdraw button. 2) Read the dialog. 3) Confirm. 4) Watch the display. | Phone, display | The dialog explains that all photos are removed. After confirming the page shows the new-visitor form with a success message. The display stops showing the photos within about 10 seconds plus the current showing (FR-AUT-04). Time recorded. | P1 | — | Not run |
| TC-WDR-10 (M) | Cancel keeps everything | Withdraw dialog open | 1) Cancel (or press Esc). | Phone | Nothing is removed. Focus returns to the withdraw button. | P2 | — | Not run |
| TC-WDR-11 (A) | Withdraw twice | After withdrawal | 1) Call E6 again with the old token. | Old token | 401 (token no longer valid). Nothing changes. | P2 | — | Not run |
| TC-WDR-12 (M) | Cached copy after withdrawal | A device that loaded the photo | 1) Withdraw. 2) On the device that showed it, request the image address again. | Image address | Record what happens: the server returns 404, and what a cached copy does depends on the cache header decision (SP-2). Record as information. | P2 | — | Not run |

## 14.13 Slideshow Retrieval

| ID | Feature | Preconditions | Test steps | Test input | Expected result | Priority | Actual result | Status |
|---|---|---|---|---|---|---|---|---|
| TC-SLD-01 (A) | First request | Three active photos | 1) Call E7 with no `since_id`. | None | 200. `photo_ids` has all 3. `groups` is empty. `cursor` equals the highest id. | P1 | — | Not run |
| TC-SLD-02 (A) | Only active photos | Active, removed and withdrawn photos | 1) Call E7. | State | Only active photos of active visitors are listed (FR-DSP-07). | P1 | — | Not run |
| TC-SLD-03 (A) | No personal data in the feed | A visitor named `Asha` | 1) Call E7 and search the body. | Feed | No name, token, hash or visitor number. | P1 | — | Not run |
| TC-SLD-04 (A) | Feed is never cached | Server running | 1) Read headers of E7. | Response | `no-store` (or equivalent). | P2 | — | Not run |
| TC-SLD-05 (A) | Bad cursor | Server running | 1) Call E7 with `since_id=abc`. 2) With a value above the latest. | `abc`, 99999 | `abc` gives 422 `INVALID_PARAMETER`. The high value gives a first-request-style answer with the real cursor. | P1 | — | Not run |
| TC-SLD-06 (M) | Display starts with no input | Photos exist | 1) Open the display address on a second window or device. | Display page | The slideshow starts without a click. No menus, forms or controls cover the photo (FR-DSP-01). | P1 | — | Not run |
| TC-SLD-07 (M) | Whole photo, not cropped | Portrait and landscape photos | 1) Watch both on a 16:9 screen. | `portrait.jpg`, `landscape.jpg` | Both fully visible, correct proportions. Portrait has dark bars at the sides (FR-DSP-03). | P1 | — | Not run |
| TC-SLD-08 (M) | Slide duration | Display running with the default | 1) Time five consecutive photos with a stopwatch. | Stopwatch | Each stays about 10 seconds (tolerance 1 second). Change the setting to 5 and repeat: about 5 seconds (FR-DSP-02). | P1 | — | Not run |
| TC-SLD-09 (M) | Continuous loop | Exactly 3 photos | 1) Watch for 3 full cycles. | Three photos | Order 1, 2, 3, 1, 2, 3 (FR-DSP-04). | P1 | — | Not run |
| TC-SLD-10 (M) | Missing image skipped | Delete one photo file by hand (not its row) | 1) Watch the display. | One missing file | The display continues to the next photo within 2 seconds. Nothing is shown on the screen about it (FR-DSP-06). | P1 | — | Not run |
| TC-SLD-11 (M) | No controls, no cursor | Display in full screen | 1) Move the mouse. 2) Look for scrollbars and buttons. 3) Press F. | Display | No cursor, no scrollbar, no visible control. F toggles full screen. | P2 | — | Not run |
| TC-SLD-12 (A) | Ordering rules | `slideshowQueue` | 1) Run four scenarios: first feed, new group arrives, photo removed, empty feed. | Fixed data | The next five photos match the written expectation. Duplicates are skipped. Removed photos leave the loop. | P1 | — | Not run |
| TC-SLD-13 (M) | One-hour run | Display running, several photos | 1) Leave it for an hour. 2) Look at memory and smoothness. | Display | No freeze, no slowdown, no memory growth (FV-8). Recorded. | P2 | — | Not run |
| TC-SLD-14 (M) | Single photo | Exactly one photo | 1) Watch for a minute. | One photo | It stays on screen with no flicker or repeated fade. Polling continues. | P2 | — | Not run |

## 14.14 New Photo Availability

| ID | Feature | Preconditions | Test steps | Test input | Expected result | Priority | Actual result | Status |
|---|---|---|---|---|---|---|---|---|
| TC-NEW-01 (A) | New upload appears in the feed | Feed read once (cursor known) | 1) Upload a photo. 2) Call E7 with the old cursor. | Cursor | One group whose `new` list holds the new photo. The cursor advances. | P1 | — | Not run |
| TC-NEW-02 (A) | Returning visitor's earlier photos | Visitor with 3 earlier photos | 1) Upload 1 new photo. 2) Call E7 with the old cursor. | Token | One group: `new` is the new photo. `previous` lists the visitor's earlier photos, most recent first, and only theirs (FR-PRV-01, 02, 04). | P1 | — | Not run |
| TC-NEW-03 (A) | Batch spread across two polls | 3 files uploaded as one request, polls in between (simulated) | 1) Call E7 after photo 1. 2) After photos 2 and 3. | Cursors | Second group has `previous` empty. No photo is listed twice in one sequence (FR-PRV-03). | P1 | — | Not run |
| TC-NEW-04 (M) | New photo joins the queue in time | Display running | 1) Upload a photo from the phone. 2) Time until the display's next poll picks it up (devtools network tab). | Phone, stopwatch | The feed call that contains the new photo happens within 10 seconds of the upload completing (FR-AUT-01 AC1). Time recorded. | P1 | — | Not run |
| TC-NEW-05 (M) | New photo plays next | Display running with at least 10 photos | 1) Note the photo on screen. 2) Upload. 3) Watch. | Phone | The new photo is the next one after the current photo, not at the end of the loop (FR-AUT-02). The time it first appeared on screen is recorded as information (TF-5). | P1 | — | Not run |
| TC-NEW-06 (M) | Current photo not interrupted | Photo on screen for 4 seconds | 1) Upload now. 2) Watch. | Phone | The current photo stays for its full duration. The loop does not restart (FR-AUT-03). | P1 | — | Not run |
| TC-NEW-07 (M) | No refresh needed | Display running | 1) Upload. 2) Do not touch the display page. | Phone | The photo appears without a page refresh (FR-AUT-01). | P1 | — | Not run |

## 14.15 Photo Deletion

| ID | Feature | Preconditions | Test steps | Test input | Expected result | Priority | Actual result | Status |
|---|---|---|---|---|---|---|---|---|
| TC-DEL-01 (A) | Delete own photo [if built] | Visitor with 2 photos | 1) Call E5 for one of them. | Token, photo id | 200 with `already_removed` false. Row `removed`. File deleted. | P2 | — | Not run |
| TC-DEL-02 (A) | Delete again [if built] | After TC-DEL-01 | 1) Call E5 again. | Same id | 200 with `already_removed` true. Nothing else changes. | P2 | — | Not run |
| TC-DEL-03 (A) | Someone else's photo [if built] | Two visitors | 1) Visitor B calls E5 on A's active photo. | B's token, A's photo id | 403 `NOT_PHOTO_OWNER`. Photo remains active and visible. | P1 | — | Not run |
| TC-DEL-04 (A) | No token [if built] | Active photo | 1) Call E5 with no token. | No token | 401 `VISITOR_NOT_RECOGNISED`. Photo remains. | P1 | — | Not run |
| TC-DEL-05 (A) | Unknown photo [if built] | None | 1) Call E5 with an id that does not exist. | Id 99999 | 404 `PHOTO_NOT_FOUND`. | P2 | — | Not run |
| TC-DEL-06 (A) | Deleted photo is gone everywhere [if built] | After TC-DEL-01 | 1) Read E7, E4, and E8. 2) List the folder. | State | Id missing from E7 and E4. E8 returns 404. File gone. | P2 | — | Not run |
| TC-DEL-07 (A) | Other photos unaffected [if built] | After TC-DEL-01 | 1) Check the remaining photo. | State | Remaining photo still active, served and in the feed. | P2 | — | Not run |
| TC-DEL-08 (M) | Delete and watch the display [if built] | Display running, visitor with 2 photos | 1) Delete one from "My photos" after confirming the dialog. 2) Watch the display. | Phone | The photo is not shown again after at most 10 seconds plus its current showing (FR-AUT-04). Time recorded. | P2 | — | Not run |
| TC-DEL-09 (M) | Cancel delete [if built] | Delete dialog open | 1) Cancel. | Phone | Photo remains. | P2 | — | Not run |
| TC-DEL-10 (M) | Delete is by ownership only [if built] | Two phones | 1) Phone B tries to remove phone A's photo using the page (cannot) and then by a raw request with B's token. | B's token | The page does not offer it. The raw request returns 403. | P1 | — | Not run |

## 14.16 Unauthorised Admin Requests

| ID | Feature | Preconditions | Test steps | Test input | Expected result | Priority | Actual result | Status |
|---|---|---|---|---|---|---|---|---|
| TC-ADM-01 (A) | Feature off by default | `OPERATOR_KEY` empty | 1) Call E10 with any key. | Any key | 404 `NOT_FOUND`, as if the address does not exist. | P1 | — | Not run |
| TC-ADM-02 (A) | No key and wrong key [if built] | Key set (32 characters) | 1) Call E10 with no key. 2) With a wrong key. | No key, wrong key | 401 `OPERATOR_KEY_INVALID` each time. The photo is unchanged. | P1 | — | Not run |
| TC-ADM-03 (A) | Correct key removes any photo [if built] | Key set, a visitor's photo | 1) Call E10 with the correct key. | Correct key, photo id | 200. Photo removed from the feed and its file deleted. Visitor and consent untouched. | P1 | — | Not run |
| TC-ADM-04 (A) | Visitor token is not an operator key [if built] | Key set | 1) Call E10 with a visitor token in the key header. | Visitor token | 401. | P1 | — | Not run |
| TC-ADM-05 (A) | Repeat removal [if built] | After TC-ADM-03 | 1) Call E10 again. | Same id | Safe no-op or `already_removed` true, no error. | P2 | — | Not run |
| TC-ADM-06 (A) | Key never leaks [if built] | Key set | 1) Call E10 with right and wrong keys. 2) Search captured logs and responses. | Log output | The key does not appear. Wrong attempts are logged as a **count** only. | P1 | — | Not run |
| TC-ADM-07 (A) | Short key refused [if built, SP-5] | Key shorter than 32 characters | 1) Start the app with a short key. | Short key | Operator feature refuses to start or stays off, with a clear message. | P3 | — | Not run |
| TC-ADM-08 (A) | Rate limit on E10 [if built, SP-1] | Limit set very low | 1) Send many wrong keys. | Wrong keys | 429 after the limit. | P3 | — | Not run |
| TC-ADM-09 (M) | Operator removal in practice [if built] | Display running, a visitor's photo | 1) Remove it with the operator request. 2) Watch the display. | Operator request | The photo leaves the display. A wrong key is refused. | P2 | — | Not run |
| TC-ADM-10 (M) | Admin page locked [if built] | Admin page exists | 1) Open the page. 2) Enter a wrong key and remove a photo. | Wrong key | Locked state first. Wrong key shows the "key not accepted" message and returns to the key screen after the first removal (FRONTEND_DESIGN 7.6). | P3 | — | Not run |

## 14.17 API Failures

| ID | Feature | Preconditions | Test steps | Test input | Expected result | Priority | Actual result | Status |
|---|---|---|---|---|---|---|---|---|
| TC-API-01 (A) | Standard error shape | Any failing request | 1) Trigger several errors. | Errors | Every body is `{"error": {code, message, details, request_id}}`. `X-Request-ID` equals `request_id`. | P1 | — | Not run |
| TC-API-02 (A) | Unknown address | None | 1) Request an address that does not exist. | `/api/nothing` | 404 `NOT_FOUND` in the standard shape (not the framework default). | P1 | — | Not run |
| TC-API-03 (A) | Wrong method | None | 1) Send DELETE to E1. | DELETE `/api/config` | 405 `METHOD_NOT_ALLOWED` in the standard shape. | P1 | — | Not run |
| TC-API-04 (A) | Malformed request | None | 1) Send E3 with a broken multipart body. | Broken body | 400 `MALFORMED_REQUEST`. Nothing stored. | P2 | — | Not run |
| TC-API-05 (A) | No leak in errors | Force errors | 1) Read every error body. | Errors | No token, hash, path, SQL or stack trace in any body (NFR-SEC-04). | P1 | — | Not run |
| TC-API-06 (A) | Unexpected failure | Force an exception inside a service (monkeypatch) | 1) Call an endpoint. | Forced failure | 500 `INTERNAL_ERROR` with a generic message. Details only in the log with the request id. | P1 | — | Not run |
| TC-API-07 (A) | Save failure per file | Force the storage to fail for one file | 1) Upload three files. | Three images | The failing file has `SAVE_FAILED`, the others accepted. The failing file leaves no file and no row. | P1 | — | Not run |
| TC-API-08 (A) | Database busy | Hold a write lock longer than the busy timeout | 1) Send an upload. | Valid request | 503 `SERVER_BUSY` with `Retry-After`. Nothing stored. | P2 | — | Not run |
| TC-API-09 (A) | Every documented code exists | Error catalogue | 1) Compare codes in the code with API_SPECIFICATION. | Catalogue | No undocumented code, no documented code missing. | P2 | — | Not run |
| TC-API-10 (A) | Config response | None | 1) Call E1. | None | 200. Every documented field present with the right type. Contains the consent notice and version. | P1 | — | Not run |
| TC-FE-08 (M) | Frontend reaction to a refused request | Server running | 1) Trigger a name error, a consent error and a too-many-files error from the page. | Phone | Each message is the server's wording, shown where it happened. Focus moves to the problem. Photos stay chosen. | P1 | — | Not run |
| TC-FE-09 (M) | Frontend reaction to a lost connection | Server stopped during an upload | 1) Start an upload. 2) Stop the server. | Phone | Banner "Upload did not finish. Please try again." with a Try again button. Photos kept. After restarting the server, Try again works. No partial data (ERR-13). | P1 | — | Not run |

## 14.18 Empty Slideshow

| ID | Feature | Preconditions | Test steps | Test input | Expected result | Priority | Actual result | Status |
|---|---|---|---|---|---|---|---|---|
| TC-EMP-01 (A) | Empty feed | Empty database | 1) Call E7. | None | 200 with `photo_ids` empty, `groups` empty, and a valid `cursor` (0 or the documented value). | P1 | — | Not run |
| TC-EMP-02 (M) | Friendly empty message | Empty database, display open | 1) Open the display. | Display | A friendly message inviting visitors to upload (and the join address). No blank screen and no error text (FR-DSP-05). | P1 | — | Not run |
| TC-EMP-03 (M) | First photo replaces the message | After TC-EMP-02 | 1) Upload one photo from the phone. | Phone | The message is replaced by the photo without refreshing (FR-DSP-05 AC1). | P1 | — | Not run |
| TC-EMP-04 (M) | Last photo removed | One photo on the display | 1) Remove or withdraw it. | Phone or operator | After its current showing the display returns to the empty message. | P2 | — | Not run |
| TC-EMP-05 (A) | My photos empty | Registered visitor with no photos | 1) Call E4. | Token | 200 with an empty list. | P2 | — | Not run |

## 14.19 Backend Interruption

| ID | Feature | Preconditions | Test steps | Test input | Expected result | Priority | Actual result | Status |
|---|---|---|---|---|---|---|---|---|
| TC-BKI-01 (M) | Server stops while the display plays | Display playing 3 or more photos | 1) Stop the server for 30 seconds. 2) Watch. 3) Restart the server. | Server stop and start | The display **keeps playing** the loaded photos and shows nothing about the problem. After restart it carries on **without a manual refresh** (FR-DSP-08 AC1). | P1 | — | Not run |
| TC-BKI-02 (M) | New photo after the restart | After TC-BKI-01 | 1) Upload a photo after the restart. | Phone | The display picks it up on its own. | P1 | — | Not run |
| TC-BKI-03 (M) | Display opened before the server | Server stopped | 1) Open the display page. 2) Start the server. | Display | The display shows a calm "Waiting for the connection" message, then starts on its own when the server is available (FRONTEND_DESIGN 6.6). | P2 | — | Not run |
| TC-BKI-04 (M) | Upload page with the server down | Server stopped | 1) Open the upload page. | Phone | A page-level message "We could not load the page. Please check your connection and try again." with a Try again button. | P2 | — | Not run |
| TC-BKI-05 (M) | Data survives a restart | Server with photos and a visitor | 1) Stop the server. 2) Start it. 3) Reload the phone page and the display. | Restart | Photos still play. The phone is still recognised. Files and database intact (TECH_STACK 5.3). | P1 | — | Not run |
| TC-BKI-06 (M) | Server killed during an upload | A large upload in progress | 1) Kill the server process during the upload. 2) Restart. 3) Check the folder and database. | Kill | No half-written photo file is visible or served. No active row without a file. (A leftover temporary file is acceptable only if documented.) | P2 | — | Not run |
| TC-BKI-07 (M) | Connection lost mid-upload | Phone upload in progress | 1) Switch the phone to aeroplane mode during the upload. | Phone | The page shows the "did not finish" message. No partial photo appears on the display (ERR-13). | P2 | — | Not run |
| TC-BKI-08 (A) | Stored files missing | An active row whose file was removed by hand | 1) GET E8. | Missing file | 404 `PHOTO_NOT_FOUND`, a warning in the log. The feed still lists it (the display skips it, TC-SLD-10). | P2 | — | Not run |

## 14.20 Security Cases Not Covered Above

| ID | Feature | Preconditions | Test steps | Test input | Expected result | Priority | Actual result | Status |
|---|---|---|---|---|---|---|---|---|
| TC-SEC-01 (A) | Storage refuses path tricks | Storage object with a temporary folder | 1) Try to read, write and delete with tricky names. | `../x`, absolute path, empty, null character, very long | Each refused. Nothing created or read outside the folder. | P1 | — | Not run |
| TC-SEC-02 (A) | Traversal file name in an upload | Registered visitor | 1) Upload a valid image whose file name is `../../x.jpg`. | Crafted name | Accepted or rejected, but if accepted the stored name is random. **No file appears outside the photo folder.** | P1 | — | Not run |
| TC-SEC-03 (A) | Token and hash storage | New visitor | 1) Register. 2) Read the database. | Registration | Token is 43 characters and different each time. The database holds only a 64-character hex hash. The raw token is not in the database. | P1 | — | Not run |
| TC-SEC-04 (M) | Script name is plain text everywhere | Name `<script>alert(1)</script>` registered | 1) View the upload page, the results, the welcome message and the display (if the name is shown). | Phone, display | Shown as text. No script runs. No alert. | P1 | — | Not run |
| TC-SEC-05 (A) | SQL characters | Empty database | 1) Register with the SQL-looking name. | `'; DROP TABLE visitors;--` | Accepted as text. All tables still exist. Name returned unchanged. | P1 | — | Not run |
| TC-SEC-06 (A) | Logs are clean | Captured log output | 1) Upload with a token, the script-tag name and a distinctive file name. 2) Search the captured log. | Log | The token, hash, name, file name and operator key do not appear. | P1 | — | Not run |
| TC-SEC-07 (M) | Server access log has no token | Real server | 1) Upload with a token. 2) Search the terminal output. | Terminal | No token or key. Only addresses, methods, paths and status codes. | P1 | — | Not run |
| TC-SEC-08 (M) | Token never in an address | Phone | 1) Look at every address and the network tab. | Browser | The token appears only in the request header, never in a URL or page text. | P1 | — | Not run |
| TC-SEC-09 (A) | Reaching removed or unknown photos | Active, removed, withdrawn and unknown photo ids | 1) GET E8 for each. 2) Try a non-number. | Ids, `abc` | Active: 200. Removed, withdrawn and unknown: 404. Non-number: 404 or 422 in the standard shape. | P1 | — | Not run |
| TC-SEC-10 (A) | Image response headers | Active photo | 1) Read the headers of E8. | Response | Fixed image content type from the database, `X-Content-Type-Options: nosniff`, cache header as decided in SP-2. | P1 | — | Not run |
| TC-SEC-11 (M) | Data folder not reachable | Server running | 1) Try the paths of the database and photo folder in a browser. | Addresses | All fail. No directory listing. | P1 | — | Not run |
| TC-SEC-12 (M) | CORS not enabled | Server running | 1) Request the API from a page on a different origin (or with a raw request carrying an `Origin` header). | Other origin | No permissive cross-origin headers. | P2 | — | Not run |
| TC-SEC-13 (M) | Security headers | Server running | 1) Read response headers of the pages and images. | Browser tools | `nosniff` present. A content security policy present and the built page still works [if built, SP-4]. | P2 | — | Not run |
| TC-SEC-14 (M) | Secrets not in the repository | Before submission | 1) Search the files **and Git history** for the key, tokens, `.env`. 2) Check `.env.example`. | Repository | None found. `.env.example` has placeholders only. No photos, database or logs committed (NFR-SEC-02). | P1 | — | Not run |
| TC-SEC-15 (M) | Dependencies | Before submission | 1) Run the audit tools for Python and Node packages (if available). 2) Record the output. | Audit output | Output recorded. Any known issues in pinned versions listed as known limitations. **Do not claim a clean result unless the output shows one.** | P2 | — | Not run |
| TC-SEC-16 (A) | Responses contain only documented fields | Each endpoint | 1) Compare each response with its documented fields. | Responses | No extra fields such as the hash or internal ids. | P1 | — | Not run |

## 14.21 Integration (Whole-Flow) Cases

| ID | Feature | Preconditions | Test steps | Test input | Expected result | Priority | Actual result | Status |
|---|---|---|---|---|---|---|---|---|
| TC-INT-01 (M) | Journey: new visitor to the display | Built app, display open, phone ready | 1) Register on the phone with 2 photos. 2) Watch the display. | Phone, display | Both photos appear and play. Token stored. Welcome-back view shown. | P1 | — | Not run |
| TC-INT-02 (A) | Journey: returning upload and feed | Registered visitor with photos | 1) Upload with a token. 2) Read E7. 3) GET E8. | Token | New photo in a group, previous photos listed, image served. | P1 | — | Not run |
| TC-INT-03 (M) | Journey: returning visitor | Phone with earlier photos | 1) Reopen. 2) Upload 1 more photo. 3) Watch the display. | Phone | The new photo plays next, then the visitor's earlier photos (FR-PRV-01, 02). | P1 | — | Not run |
| TC-INT-04 (A) | Upload then delete [if built] | Visitor | 1) Upload. 2) Delete. 3) Read E7, E4, E8 and the folder. | State | Gone everywhere. | P2 | — | Not run |
| TC-INT-05 (M) | Upload then delete on the display [if built] | Display | 1) Upload. 2) See it. 3) Delete. | Phone | It stops appearing. | P2 | — | Not run |
| TC-INT-06 (A) | Upload then withdraw | Visitor with photos | 1) Withdraw. 2) Check token, feed, folder, database. | State | All gone as in TC-WDR-02 to 08. | P1 | — | Not run |
| TC-INT-07 (M) | Full demo script | Fresh clone, README only | 1) Follow the README. 2) Run the suite. 3) Do the demo: register, upload, display, return, withdraw. | Clean setup | Everything works from the README alone. | P1 | — | Not run |
| TC-INT-08 (A) | Same file name, different visitors | Empty database | 1) Two visitors upload `photo.jpg`. | Two uploads | Two distinct stored files. No overwrite (FR-CNC-02). | P1 | — | Not run |

## 14.22 Frontend Manual Cases

| ID | Feature | Preconditions | Test steps | Test input | Expected result | Priority | Actual result | Status |
|---|---|---|---|---|---|---|---|---|
| TC-FE-01 (M) | No sideways scroll at phone width | Browser device mode at 360 px and a real phone | 1) Open `/`. 2) Scroll sideways. | 360 px | No sideways scrolling (NFR-USA-01). Buttons are easy to tap. | P1 | — | Not run |
| TC-FE-02 (M) | Early validation messages | New-visitor page | 1) Press Upload with an empty name. 2) With no consent. 3) With no photo. | Empty form | Each shows the PRD's message in the right place. Nothing is sent. Focus moves to the problem. | P1 | — | Not run |
| TC-FE-03 (M) | Previews and remove | New-visitor page | 1) Choose 3 photos. 2) Remove one. | 3 photos | Previews appear. Removing one updates the list. Focus moves sensibly. | P2 | — | Not run |
| TC-FE-04 (M) | Early file checks | New-visitor page | 1) Choose a GIF, an over-size file and a 6th photo. | Bad files | Bad files go to a "not added" list with reasons. | P2 | — | Not run |
| TC-FE-05 (M) | Progress and no double submit | Slow upload (large files or throttled network) | 1) Press Upload. 2) Press it again. | 5 photos | A progress indicator shows. The button is busy. Only one request is sent. | P2 | — | Not run |
| TC-FE-06 (M) | Mixed result display | New-visitor page | 1) Upload a good and a bad file. | `valid.jpg`, `fake_image.jpg` | Per-file list with icon and words for each. Summary banner matches (not colour alone). | P1 | — | Not run |
| TC-FE-07 (M) | All accepted banner | New-visitor page | 1) Upload 2 good files. | 2 photos | Success banner. Previews cleared. | P2 | — | Not run |
| TC-FE-10 (M) | Keyboard only | Laptop | 1) Complete an upload using only the keyboard. | Keyboard | Every control reachable and visible focus. Upload completes. | P2 | — | Not run |
| TC-FE-11 (M) | Laptop layout | Laptop at 1280 px | 1) Open both pages. | 1280 px | Layout centred and readable. | P2 | — | Not run |
| TC-FE-12 (M) | Reduced motion | System setting on | 1) Run the display. | Setting | No fade between slides (if the fade is built). | P3 | — | Not run |
| TC-FE-13 (M) | Zoom | Browser zoom 200 percent | 1) Use the upload page. | 200 percent | No lost content or sideways scroll. | P3 | — | Not run |
| TC-FE-14 (M) | Display on the real screen | Real TV or monitor at 1920 by 1080 | 1) Look from the viewing distance. | Screen | Text and photos readable. Edges not cut off (overscan). Adjust and record. | P2 | — | Not run |

---

# 15. Results Section (Complete During Implementation)

**Rules for completing this section.** Fill it in **as you run tests**, not afterwards from memory. Only record what you observed. Use the status words below. Never change an expected result to make a case pass.

| Status | Meaning |
|---|---|
| **Not run** | Default. Not yet executed. |
| **Pass** | Executed, and the observed result matched the expected result |
| **Fail** | Executed, and the observed result did not match. A defect row is required. |
| **Blocked** | Could not run because something else is broken. Say what. |
| **Skipped** | Not run on purpose (for example an **[if built]** feature that was not built). Give the reason. |

## 15.1 Run Record (One Per Test Session)

| Field | Run 1 | Run 2 | Run 3 |
|---|---|---|---|
| Date and time | | | |
| Tester | | | |
| Git commit | | | |
| Operating system | | | |
| Python version | | | |
| Node version | | | |
| Pinned library versions (see `requirements.txt` and `package.json`) | | | |
| Browser and version | | | |
| Phone model and browser | | | |
| Screen and resolution | | | |
| Settings changed from defaults | | | |
| Automated command run and the final summary line copied here | | | |
| Evidence saved at (path or screenshot names) | | | |

## 15.2 Decisions That Change Expected Results

| Decision | Value chosen | Date | Rows affected |
|---|---|---|---|
| Rate limit and global cap built? (SP-1) | | | TC-SIZ-06, 07, TC-ADM-08 |
| Photo cache header value (SP-2) | | | TC-UPL-08, TC-SEC-10, TC-WDR-12 |
| Operator endpoint E10 built? (SP-3) | | | TC-ADM-xx |
| Content security policy built? (SP-4) | | | TC-SEC-13 |
| Minimum operator key length enforced? (SP-5) | | | TC-ADM-07 |
| Single-photo deletion (E5) and My photos built? | | | TC-DEL-xx |
| Withdrawal built? And the final consent notice wording | | | TC-WDR-xx, TC-CON-08 |
| Thumbnails built? (A-15) | | | TC-UPL-12 |
| Concurrency method used (TF-7) | | | TC-SIM-01 to 04 |
| Admin page built? | | | TC-ADM-10 |

## 15.3 Summary by Area (Fill In After Each Run)

| Area | Cases | Pass | Fail | Blocked | Skipped | Not run |
|---|---|---|---|---|---|---|
| Registration (REG) | 10 | | | | | |
| Duplicate names (DUP) | 5 | | | | | |
| Returning visitor (RET) | 12 | | | | | |
| Valid uploads (UPL) | 12 | | | | | |
| Unsupported formats (FMT) | 6 | | | | | |
| Incorrect contents (CNT) | 8 | | | | | |
| Size, limits, rate (SIZ) | 8 | | | | | |
| Multiple files (MUL) | 8 | | | | | |
| Simultaneous (SIM) | 7 | | | | | |
| Database (DB) | 15 | | | | | |
| Consent (CON) | 10 | | | | | |
| Withdrawal (WDR) | 12 | | | | | |
| Slideshow (SLD) | 14 | | | | | |
| New photos (NEW) | 7 | | | | | |
| Deletion (DEL) | 10 | | | | | |
| Admin (ADM) | 10 | | | | | |
| API failures (API and FE-08, 09) | 10 + 2 | | | | | |
| Empty slideshow (EMP) | 5 | | | | | |
| Backend interruption (BKI) | 8 | | | | | |
| Security (SEC) | 16 | | | | | |
| Integration (INT) | 8 | | | | | |
| Frontend (FE) | 12 | | | | | |
| **Total** | **215** | | | | | |
| Of which **P1** | 145 (P2: 60, P3: 10) | | | | | |
| Of which automated (A) | 142 (P1: 107) | | | | | |
| Of which manual (M) | 73 (P1: 38) | | | | | |

## 15.4 Defect Log

| Defect ID | Found in test | Date | Description (what happened) | Severity (High, Medium, Low) | Cause (when known) | Fix and commit | Retest result | Status (Open, Fixed, Known limitation) |
|---|---|---|---|---|---|---|---|---|
| DEF-001 | | | | | | | | |
| DEF-002 | | | | | | | | |
| DEF-003 | | | | | | | | |
| DEF-004 | | | | | | | | |
| DEF-005 | | | | | | | | |

Also record every **AI mistake** found through testing in `AI_LOG.md` (the assignment needs at least two, with how each was noticed and fixed).

## 15.5 Acceptance Sign-Off (PRD Section 7)

Complete after the related cases are `Pass` (or have a recorded limitation). Sign only what you have verified.

| Feature | All related P1 cases Pass? | Known limitations | Signed (name, date) |
|---|---|---|---|
| New visitor registration | | | |
| Returning visitor identification | | | |
| Public display consent | | | |
| Photo upload | | | |
| File validation and limits | | | |
| Photo storage | | | |
| Slideshow display | | | |
| Automatic updates (timings recorded) | | | |
| Previous photo retrieval | | | |
| Multiple simultaneous uploads | | | |
| Consent withdrawal (Should) | | | |
| Photo deletion (Should) | | | |
| **MVP release gate:** all Must Have pass, documents exist, demo runs from the README on a clean setup | | | |

## 15.6 Requirement Traceability (Must Have)

Each requirement must have at least one passing case before submission.

| Requirement | Cases |
|---|---|
| FR-REG-01 to 04 | TC-REG-01 to 10, TC-DUP-01 to 05 |
| FR-ID-01, 02, 03, 03a, 05 | TC-RET-01 to 12, TC-SEC-03, TC-SEC-08 |
| FR-CON-01 to 03 | TC-CON-01 to 10 |
| FR-UPL-01 to 04 | TC-UPL-01 to 11, TC-MUL-01 to 08 |
| FR-VAL-01 to 05 | TC-FMT-01 to 06, TC-CNT-01 to 08, TC-SIZ-01 to 05 |
| FR-STO-01, 02, 04, 05 | TC-UPL-01, 08, 09, TC-DB-10, 11, TC-SEC-01, 02, 09, 10, TC-API-07 |
| FR-DSP-01 to 07 | TC-SLD-01 to 12, TC-EMP-02, 03 |
| FR-AUT-01 to 03 | TC-NEW-01, 04 to 07 |
| FR-PRV-01 to 04 | TC-NEW-02, 03, TC-DUP-04, TC-INT-03 |
| FR-CNC-01 to 04 | TC-SIM-01, 02, 05, TC-INT-08 |
| NFR-SEC-01 to 04 | TC-FMT, TC-CNT, TC-SEC-01 to 16, TC-API-05 |
| NFR-PRV-02 | TC-DB-15, TC-SEC-03 |
| NFR-REL-01, 02 | TC-DB-10, 11, TC-BKI-05, 06 |
| NFR-USA-01, 03 | TC-FE-01, 02, TC-API-01 |
| NFR-COMP-02 | TC-FE-14, TC-INT-07 |
| NFR-TST-01 | This document and Section 15 |
| NFR-DOC-01, 02 | TC-INT-07 and the IMPLEMENTATION_PLAN checklist |
| Should Have (if built): FR-WDR-01 to 04, FR-DEL-01 to 05, FR-ID-04, FR-DSP-08, FR-AUT-04, FR-CNC-05, FR-VAL-06, 07, FR-STO-03, FR-UPL-05 | TC-WDR-xx, TC-DEL-xx, TC-RET-08, TC-BKI-01 to 04, TC-NEW (removal timing), TC-SIM-07, TC-SIZ-04, TC-FMT-06, TC-UPL-04, 05, TC-FE-05 |

## 15.7 Known Limitations Accepted (Record, Do Not Hide)

| Limitation | Source | Recorded where |
|---|---|---|
| Different browser or device is a new visitor | PRD ERR-25 | README |
| Two tabs registering at once may create two visitors | PRD ERR-24 | README |
| Withdrawal cannot recall what people have seen, or cached copies | SECURITY_PRIVACY 16.2 | README and notice |
| HEIC behaviour on real phones | ASSUMPTIONS A-12 | README |
| In-process test client may not give real concurrency | TF-7 | Section 15.1 |
| Tests did not cover: hosting, HTTPS, large load, formal accessibility, penetration testing | Section 2.2 | README |
| (Add more here as found) | | |

## 15.8 Exit Criteria

| # | Criterion | Met? |
|---|---|---|
| 1 | The whole automated suite passes from a fresh clone, and the summary line is saved | |
| 2 | Every **P1** case is `Pass`, or `Skipped` with a reason, or `Fail` with a recorded **known limitation** that you accept | |
| 3 | The 10-simultaneous-upload test passes and the concurrency method is recorded | |
| 4 | No open defect marked High | |
| 5 | Manual results are recorded with dates and real observations | |
| 6 | The acceptance sign-off table is complete | |
| 7 | The consent notice matches the features built (TC-CON-08) | |
| 8 | The security checks TC-SEC-14 (secrets) and TC-SEC-06 (logs) pass | |
| 9 | The demo script ran end to end from the README on a clean setup | |

---

# 16. Schedule (Fits IMPLEMENTATION_PLAN)

| When | Testing work | Time (estimate) |
|---|---|---|
| Day 1 | Image processor tests (D1-05). First test of E1 (D1-03). | Inside those tasks |
| Day 2 | Repository, storage, security, visitor service and upload tests written with each task (D2-01 to D2-05) | Inside those tasks |
| Day 3 | Manual upload page checks and the first phone test (D3-03, D3-04) | Inside those tasks |
| Day 4 | Slideshow service tests, queue check, error tests, security checklist, withdrawal tests (D4-01 to D4-07) | Inside those tasks |
| Day 5 (D5-01) | Complete the contract tests and concurrency tests, run the **whole** suite from a fresh clone | 0.9 h |
| Day 5 (D5-02) | Manual **P1** set on real devices, record results | 0.6 h |
| Day 5 | Fix failures from the buffer (D5-B), then re-run the suite | Buffer |

**Reality check.** The manual P1 set has 38 cases (counted by script). That is more than the 0.6 hours in D5-02 if each is run separately. Run them as grouped sessions (one phone session and one display session cover many cases) and expect about 1.0 hour, taking the difference from the Day 5 buffer. If the buffer is already spent, cut P1 cases in the order the schedule allows and leave them `Not run`. P2 and P3 manual cases fit only if earlier days finish early. This plan does not hide that: any manual case not run stays `Not run`.

---

# 17. Decisions and Open Items

| ID | Decision or question | Default in this plan |
|---|---|---|
| TP-1 | Use the **P1 minimum set** for submission (all automated, manual P1 only)? | Yes |
| TP-2 | Use the Node built-in test runner or hand-checks for the frontend helpers, not a new dependency? | Yes (verify the runner exists) |
| TP-3 | Automated browser testing is FUTURE, not in the MVP | Yes |
| TP-4 | Where to store evidence | `docs/test-results/` plus Section 15 |
| TP-5 | Confirm what "within 10 seconds" means (TF-5) | Enters the playback queue within 10 seconds |
| TP-6 | Which file error code a PDF gets (TF-4) | `FILE_TYPE_NOT_ALLOWED` because of its extension |
| TP-7 | Confirm whether emoji and other scripts are allowed in names (FR-REG-02) | Allowed, test marked P3 |

---

# Glossary

| Term | Meaning |
|---|---|
| **Test case** | One check with a known input and an expected result |
| **Automated test** | A test run by a command with no person needed |
| **Manual test** | A test a person performs and records |
| **Unit test** | A test of one small part alone |
| **API (contract) test** | A test that calls the server's endpoints and checks the documented answers |
| **Integration test** | A test of several parts working together |
| **Regression test** | Re-running tests after a change to see whether something broke |
| **Fixture** | Shared test set-up, such as a registered visitor |
| **Builder** | A small function that creates test data, such as an image |
| **Mock** | A fake part used instead of the real one |
| **Monkeypatch** | A temporary change made during a test to force a failure |
| **Test client** | A tool that calls the web application in-process, without a network |
| **Boundary test** | A test at the exact limit (for example exactly 5 MiB and one byte more) |
| **Soak test** | Running something for a long time to find slow problems |
| **Defect** | A difference between expected and actual behaviour |
| **Known limitation** | A defect or gap that is accepted and written down |

---

# Document Control

| Version | Date | Change |
|---|---|---|
| 1.0 | 17 Nov 2026 | Initial test plan. **No test has been run.** All Actual result cells are blank and all Status cells read Not run. |

**Approvals needed before testing:** TP-1 to TP-7 (Section 17), and the decisions in Section 15.2 once they are made.
