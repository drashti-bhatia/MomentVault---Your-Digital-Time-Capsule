# ASSUMPTIONS.md

# Visitor Photo Slideshow (MomentVault): Assumptions Register

| Item | Detail |
|---|---|
| Product | MomentVault, implementing Assignment 1: Visitor Photo Slideshow |
| Document version | 1.0 (draft for approval) |
| Date | 14 November 2026 |
| Based on | Original assignment, PROJECT_BRIEF.md v1.0, PRD.md v1.0 |
| Purpose | The assignment says: "You are free to assume any gaps in the requirements, but you must document every assumption in ASSUMPTIONS.md." This file is that record. |

---

## How to Read This Document

Three kinds of statements are kept strictly apart.

| Kind | Label | Meaning |
|---|---|---|
| **Fact from the assignment** | **F-xx** | Something the assignment text actually says. Listed in Part 1. |
| **Assumption by the team** | **A-xx** | A gap in the assignment that we filled with our own decision. Listed in Part 3. **None of these is an assignment requirement.** |
| **Recommended future enhancement** | **E-xx** | A possible improvement that is not in the assignment and not in the MVP. Listed in Part 5. |

**Status values used in Part 3**

| Status | Meaning |
|---|---|
| **ASSUMED** | A reasonable decision was made. Work can proceed. |
| **ASSUMED + CONFIRM** | The decision is workable, but its impact is high or it interprets the assignment wording, so it should be confirmed. |
| **OPEN** | The decision cannot responsibly be made by the developer alone. A **provisional default** is used so work can continue, but the question remains and is listed in Part 4. |

**Confirmation needed values:** *No* (proceed), *Recommended* (please check, low risk if wrong), *Required* (please confirm before the related work starts).

**ID note.** PRD.md Section 1.6 listed eight early assumptions (AS-01 to AS-08). This document is the complete register and replaces that list. Part 6 maps the old IDs to the new ones.

---

# Part 1: Facts Stated in the Assignment

These are the only things in this document that are original requirements.

| ID | Fact (assignment wording, summarised) |
|---|---|
| F-01 | Each visitor is asked for their name and photos. |
| F-02 | As soon as an upload completes, the photo plays in a slideshow for a specific duration. |
| F-03 | The slideshow plays on a smart TV or computer display. |
| F-04 | If the person is visiting again, the slideshow also plays their previous photos. |
| F-05 | The team is free to assume any gaps, but must document every assumption in ASSUMPTIONS.md. |
| F-06 | The design must address how a returning visitor is identified. |
| F-07 | The design must address privacy and consent, because photos of people are displayed publicly. |
| F-08 | The design must address file validation, upload size limits and what happens when several people upload at once. |
| F-09 | The repository must contain README.md, ASSUMPTIONS.md, AI_LOG.md, ARCHITECTURE.md, tests (automated, or a documented manual test plan with results), `.env.example` (placeholders only) and a regular, meaningful commit history. |
| F-10 | AI tools may be used, but the student must be able to explain every part of the submission. |
| F-11 | AI_LOG.md needs 5 to 10 key interactions, including at least 2 cases where the AI was wrong. |
| F-12 | Submission is one public GitHub repository, accessible without login. A short live demo may be required. |
| F-13 | A deployed link in the README is optional ("if available"). |

**What the assignment does NOT say** (each of these is handled as an assumption in Part 3): the value of the "specific duration", how a returning visitor must be recognised, whether recognition must work across devices, display order, size limits, allowed image formats, number of photos per request, consent mechanism, consent withdrawal, photo deletion, data retention, admin or operator functions, TV behaviour, behaviour without internet, refresh timing, and deployment.

---

# Part 2: Index of Assumptions

| ID | Topic | Status | Confirmation needed |
|---|---|---|---|
| A-01 | Visitor identification method | ASSUMED | No |
| A-02 | Duplicate visitor names | ASSUMED | No |
| A-03 | Returning visitors across browsers and devices | ASSUMED + CONFIRM | **Required** |
| A-04 | Shared devices and name changes | ASSUMED | No |
| A-05 | Meaning of "visiting again" and "previous photos" | ASSUMED | No |
| A-06 | When a visitor is registered | ASSUMED | No |
| A-07 | Consent mechanism | ASSUMED + CONFIRM | Recommended |
| A-08 | Consent notice content, minors and people in photos | ASSUMED + CONFIRM | Recommended |
| A-09 | Consent withdrawal | ASSUMED | No |
| A-10 | Photo deletion | ASSUMED | No |
| A-11 | Data retention | **OPEN** | **Required** before any public deployment |
| A-12 | Allowed image formats | ASSUMED | No (verify on real phones) |
| A-13 | Maximum file size | ASSUMED + CONFIRM | Recommended |
| A-14 | Photos per request and per visitor | ASSUMED | No |
| A-15 | Image checking and processing | ASSUMED + CONFIRM | Recommended |
| A-16 | Invalid files | ASSUMED | No |
| A-17 | Duplicate images | ASSUMED | No |
| A-18 | Image storage | ASSUMED | No |
| A-19 | Simultaneous uploads | ASSUMED | No |
| A-20 | Photo display duration | **OPEN** | **Required** |
| A-21 | Slideshow ordering | ASSUMED | No |
| A-22 | New and previous photos shown together | ASSUMED + CONFIRM | Recommended |
| A-23 | Limits on the "play next" queue | ASSUMED + CONFIRM | Recommended |
| A-24 | More than one display | ASSUMED | No |
| A-25 | Slideshow refresh interval | ASSUMED | No |
| A-26 | TV display behaviour | **OPEN** | **Required** (demo setup) |
| A-27 | Internet disconnection | ASSUMED | No |
| A-28 | Public access to the pages | ASSUMED | No |
| A-29 | Admin and operator permissions | ASSUMED + CONFIRM | Recommended |
| A-30 | Deployment and network setup | **OPEN** | Required only if a deployed link is wanted |
| A-31 | Language, time zone and timestamps | ASSUMED | No |
| A-32 | Testing approach | ASSUMED | No |

---

# Part 3: Assumptions

Every entry uses the same eight fields: topic, assumption, reason, alternatives considered, impact on implementation, limitations or risks, and whether confirmation is needed. A ninth line, "Assignment says", shows what is fact and what is gap.

---

## Group A: Visitor Identity

### A-01: Visitor identification method
- **Status:** ASSUMED
- **Assignment says:** The design must address how a returning visitor is identified (F-06). It does not say how.
- **Assumption:** When a visitor registers, the server creates a long random value (a *visitor token*, at least 128 bits, generated with a secure random source). The browser stores it and sends it with every later request. The server saves only a one-way scrambled form of the token (a *hash*), not the token itself, and finds the visitor by comparing hashes. The visitor's **name is never used to identify anyone**.
- **Reason:** It needs no accounts, no email, no phone number and no extra personal data. It works on any phone. A beginner can explain it clearly in a demo. Saving only the hash means a leaked database does not hand out usable tokens.
- **Alternatives considered:**
  - Identify by name only. Rejected: names repeat, people misspell them, and anyone typing "Priya" would see or remove Priya's photos.
  - Name plus email or phone. Rejected: collects personal data, needs verification, goes beyond the assignment.
  - Browser cookie instead of browser storage. Possible. A secure cookie is harder for malicious scripts to read, but the frontend and backend run at different addresses during development, which makes cookies harder to set up. The final choice is recorded in ARCHITECTURE.md. Both satisfy the PRD.
  - Device fingerprinting. Rejected: invasive and unreliable.
  - Accounts and passwords. Rejected: out of scope (PRD OOS-03).
- **Impact on implementation:** A token generator on the server, a hashed token column on the visitor record, a request header (or cookie) carrying the token, a small storage helper in the frontend that tolerates storage being unavailable, and tests for valid, missing and invalid tokens.
- **Limitations or risks:** Anyone who can use the same browser profile acts as that visitor. A malicious script on the page could steal the token, so visitor-supplied text must always be displayed safely (PRD NFR-SEC-03). If the token is lost, the identity is lost (see A-03).
- **Confirmation needed:** No.

### A-02: Duplicate visitor names
- **Status:** ASSUMED
- **Assignment says:** Nothing about duplicate names.
- **Assumption:** Duplicate names are allowed. Two visitors called "Sam" are two separate visitors. The system never shows "name already taken", never merges by name and never looks anyone up by name.
- **Reason:** Blocking duplicates would wrongly refuse real people and would reveal which names are already used. Merging by name would mix up strangers' photos.
- **Alternatives considered:** Reject duplicates (unfair, leaks information). Add a number such as "Sam (2)" (confusing, still not a real identifier). Merge by name (unsafe).
- **Impact on implementation:** No uniqueness rule on the name column. A test with two visitors called "Sam" proving their photos stay separate.
- **Limitations or risks:** If names are shown as captions (a Could Have), two identical names look the same on screen. Captions are also unverified: someone can type another person's name.
- **Confirmation needed:** No.

### A-03: Returning visitors across browsers and devices
- **Status:** ASSUMED + CONFIRM
- **Assignment says:** "If the person is visiting again" (F-04). It does not say the person must be recognised on a different phone or after clearing browser data.
- **Assumption:** A returning visitor is recognised **only on the same browser that holds the token**. A different browser, a different device, a private window or cleared browser data means the person is treated as a new visitor. There is no recovery and no merging in the MVP.
- **Reason:** Without accounts or personal data, there is no safe way to know that two devices belong to the same person. This is the simplest honest solution.
- **Alternatives considered:**
  - A one-time recovery code shown after registration that the visitor can type on another device (future enhancement E-01).
  - Email or phone login. Rejected: out of scope.
  - Name plus a chosen PIN. Possible, but PINs are guessable and it adds fields the assignment does not mention.
  - Fingerprinting. Rejected.
- **Impact on implementation:** Matches PRD ERR-25 and OOS-05. The README must state this limit. The demo must use the same browser for the "returning visitor" scenario.
- **Limitations or risks:** **High.** (1) An assessor may expect recognition across devices. (2) A visitor who loses the token **cannot delete or withdraw their earlier photos**, and those photos stay on the display. See A-10, A-11 and A-29 for the related risk and a suggested mitigation.
- **Confirmation needed:** **Required.** Please confirm that same-browser recognition satisfies "visiting again". If it does not, a recovery-code feature (E-01) must be designed before coding.

### A-04: Shared devices and name changes
- **Status:** ASSUMED
- **Assignment says:** Nothing.
- **Assumption:** One browser holds one visitor identity. The name is fixed at registration and cannot be edited in the MVP. A "Not you?" link (Should Have, FR-ID-04) clears the token on that browser only, and first warns the person that they will no longer be able to manage their earlier photos from this browser.
- **Reason:** Keeps identity simple. The warning prevents accidental loss of control.
- **Alternatives considered:** Allow name editing. Allow several profiles per browser. Silent switch with no warning.
- **Impact on implementation:** A confirmation message before clearing the token. Nothing is deleted from the server.
- **Limitations or risks:** Photos of an abandoned identity stay on the display until someone removes them (see A-29).
- **Confirmation needed:** No.

### A-05: Meaning of "visiting again" and "previous photos"
- **Status:** ASSUMED
- **Assignment says:** The slideshow "also plays their previous photos" (F-04).
- **Assumption:** "Visiting again" means any later visit from a recognised browser, whether minutes or days later, while the photos are still kept. "Previous photos" means that visitor's **active** photos (not deleted, not withdrawn) uploaded **before the current submission**. This includes earlier submissions made in the same visit.
- **Reason:** It is the simplest definition and avoids inventing a notion of a "visit" or "session".
- **Alternatives considered:** Only photos from earlier days. Only the last N photos. Only photos from a previous event.
- **Impact on implementation:** Previous photos are found by visitor ID and status. No session logic.
- **Limitations or risks:** A visitor who uploads three times in ten minutes triggers three replays (see A-23).
- **Confirmation needed:** No.

---

## Group B: Registration, Consent, Withdrawal, Deletion and Retention

### A-06: When a visitor is registered
- **Status:** ASSUMED
- **Assignment says:** Each visitor is asked for their name and photos (F-01).
- **Assumption:** There is no separate sign-up step. The upload form asks for name, consent and photos together. The visitor record is created at the first submission whose name and consent are valid, even if every photo in it is rejected.
- **Reason:** One form is the simplest experience and matches the assignment wording.
- **Alternatives considered:** A two-step flow (name first, photos second). Creating the visitor only after the first valid photo.
- **Impact on implementation:** The first response returns the token. A visitor without any photos can exist, which is harmless.
- **Limitations or risks:** Repeated bad submissions could create many empty visitor records (see rate limiting, E-04).
- **Confirmation needed:** No.

### A-07: Consent mechanism
- **Status:** ASSUMED + CONFIRM
- **Assignment says:** Privacy and consent must be addressed because photos are displayed publicly (F-07). The mechanism is not specified.
- **Assumption:** A new visitor must tick a **checkbox that is unticked by default**, next to a plain-language notice, before submitting. The server also refuses any submission without consent. The system records who consented, when, and which notice version. Consent given at registration covers that visitor's later uploads until withdrawn. Returning visitors do not tick again but see a reminder and a withdraw link.
- **Reason:** Explicit, simple, testable, and enforced where it cannot be bypassed.
- **Alternatives considered:** Tick on every upload (stronger, but annoying). Treat pressing the upload button as consent (not explicit). Pre-ticked box (not valid consent). A separate terms page (visitors skip it).
- **Impact on implementation:** A consent record table and a consent field in the upload request. A test that a request without consent is rejected.
- **Limitations or risks:** Consent is self-declared by the uploader, not by other people in the photo. Consent at registration covers later photos that the visitor may take of other people.
- **Confirmation needed:** Recommended. Decide whether once-per-registration is enough, or whether each upload should need a tick.

### A-08: Consent notice content, minors and people in photos
- **Status:** ASSUMED + CONFIRM
- **Assignment says:** Nothing about the wording.
- **Assumption:** The notice (version `v1`) is written by the team in plain English and states the points listed in PRD FR-CON-01: photos are shown publicly on a screen; the visitor owns the photos or has permission, and people clearly visible have agreed; photos stay until deleted or consent is withdrawn; deletion and withdrawal are possible at any time; the visitor is an adult or has a guardian's permission; names are shown if that optional feature is built. There is **no age check**. Photos already shown on the screen cannot be taken back from people who saw them.
- **Reason:** Covers the main privacy risks honestly with a notice an ordinary visitor will read.
- **Alternatives considered:** A long legal policy (not read). Legal review (not available for an MVP). Age verification (out of scope, PRD OOS-13).
- **Impact on implementation:** The notice text lives in one place with a version label. A change to the text means a new version.
- **Limitations or risks:** The wording has not been reviewed by a lawyer and is not legal advice (PRD OOS-15). A child could still tick the box.
- **Confirmation needed:** Recommended. The final wording should be approved before the notice is built. A draft can be prepared on request.

### A-09: Consent withdrawal
- **Status:** ASSUMED (recommended enhancement, Should Have)
- **Assignment says:** Nothing. Withdrawal is **not** an assignment requirement.
- **Assumption:** A recognised visitor can withdraw after a confirmation step. Withdrawal then: deletes all their photo files; removes their photos from the display; clears their stored name; marks the consent record "withdrawn" with a time (keeping only visitor ID, time and notice version, with no personal data); invalidates the token; and clears the browser token. Later uploads need fresh consent and count as a new visitor. Withdrawal cannot be undone.
- **Reason:** Visitors should be able to stop being shown publicly. A clean, final removal is simpler to build and to explain than partial hiding.
- **Alternatives considered:** Hide photos but keep them (still holds the data). Keep the name. Allow reactivation. Handle withdrawals by emailing an organiser (no infrastructure).
- **Impact on implementation:** A removal routine shared with deletion, a display update path, and tests proving the files are gone from the storage folder.
- **Limitations or risks:** Works only from the browser holding the token (A-03). It cannot recall what has already been seen on the screen.
- **Confirmation needed:** No.

### A-10: Photo deletion
- **Status:** ASSUMED (recommended enhancement, Should Have)
- **Assignment says:** Nothing. Deletion is **not** an assignment requirement.
- **Assumption:** Only the owner (the request carrying the owner's token) can delete a photo. Deleting removes the stored file permanently and marks the record as removed. The display stops showing it within the refresh time. Repeated deletion gives a friendly message.
- **Reason:** Gives visitors control over what is shown publicly.
- **Alternatives considered:** A "trash" with undo. Delete by secret link. Letting anyone delete. Operator-only deletion.
- **Impact on implementation:** An ownership check, a confirmation step, removal of the file, and a display update.
- **Limitations or risks:** If the visitor loses the token, nobody but the developer can remove the photo (A-03, A-29).
- **Confirmation needed:** No.

### A-11: Data retention
- **Status:** **OPEN**
- **Assignment says:** Nothing about how long photos are kept.
- **Provisional default (so work can continue):** Photos and names are kept until the visitor deletes them or withdraws consent. There is no automatic expiry. The consent notice says so.
- **What needs to be decided:** For a real public deployment, (1) how long photos should be kept, (2) who is responsible for the data, and (3) whether everything should be erased automatically after an event ends (for example after 24 hours or 30 days).
- **Alternatives considered:** Automatic expiry after N days. A manual "clear all data" step at event end (E-05). Keeping data only while the display is running.
- **Impact on implementation:** None for the MVP. An expiry rule would later need a scheduled cleanup job and a notice change.
- **Limitations or risks:** Without expiry, photos of people stay indefinitely on disk. This project makes no claim of legal compliance.
- **Confirmation needed:** **Required** before any public deployment. It does not block building the MVP.

---

## Group C: Upload Rules and Storage

### A-12: Allowed image formats
- **Status:** ASSUMED
- **Assignment says:** File validation must be addressed (F-08). Formats are not specified.
- **Assumption:** Only **JPEG** and **PNG** are accepted. The server judges the file by its real content, not its name. A file with PNG content named `.jpg` is accepted by its real format and stored with the matching extension (this settles PRD FR-VAL-02 AC4). GIF, WebP, HEIC, HEIF, BMP, TIFF, SVG, PDF and everything else are rejected with a clear message.
- **Reason:** Both formats are handled natively by Pillow and are common. SVG files can contain scripts. GIF can be animated. HEIC needs an extra library, which conflicts with the rule to avoid unnecessary dependencies.
- **Alternatives considered:** Add WebP (easy with Pillow, but rarer from phones). Add HEIC through an extra package. Accept every format Pillow can read (larger attack surface).
- **Impact on implementation:** The photo picker lists the allowed types. The server checks real content. Tests use the sample files in PRD Appendix B.
- **Limitations or risks:** Some iPhone photos are HEIC. Many phones and browsers convert a photo to JPEG when it is picked through a web page, but this depends on the device and browser and **has not been verified**. It must be tested on real phones before the demo (PRD ERR-12).
- **Confirmation needed:** No.

### A-13: Maximum file size
- **Status:** ASSUMED + CONFIRM
- **Assignment says:** Upload size limits must be addressed (F-08). No value is given.
- **Assumption:** 5 MB per photo (PRD parameter P-02). The server enforces it while the data arrives and discards oversized data immediately.
- **Reason:** Limits memory, disk and display bandwidth, and protects against abuse.
- **Alternatives considered:** 2 MB (too small). 10 MB (friendlier for modern phones). Shrinking photos in the browser before upload (more frontend work, E-06).
- **Impact on implementation:** A configuration value, a server check, an early warning in the browser, and tests at exactly 5 MB and just above it.
- **Limitations or risks:** Modern phone cameras can produce photos of several MB, so some real photos may be rejected. This should be checked with real phone photos. If many fail, raise the limit to 10 MB (a configuration change plus a PRD update).
- **Confirmation needed:** Recommended.

### A-14: Photos per request and per visitor
- **Status:** ASSUMED
- **Assignment says:** "Photos" are plural (F-01). No limits are given.
- **Assumption:** Up to **5 photos per submission** (P-03), so a request can be at most about 25 MB. A visitor may hold at most **20 active photos** (P-07, Should Have). The server enforces both. Extra files are rejected.
- **Reason:** Bounds the work per request and stops one person filling the slideshow.
- **Alternatives considered:** One photo per request (simple, but more taps). 10 photos. No limit.
- **Impact on implementation:** A count check before processing and a per-visitor count check.
- **Limitations or risks:** No limit per network address exists, so a determined person could register repeatedly (E-04).
- **Confirmation needed:** No.

### A-15: Image checking and processing
- **Status:** ASSUMED + CONFIRM
- **Assignment says:** File validation must be addressed (F-08). Processing is not mentioned.
- **Assumption:** The server opens each image with Pillow and rejects it if it is not a real, fully readable JPEG or PNG, or if it is larger than 25 megapixels (P-05). It then makes a clean stored copy that: shows the photo upright by applying its rotation information; removes hidden metadata such as GPS location (EXIF data); is scaled down so the longest side is at most 1920 pixels (never enlarged); and keeps the format. It also makes a small thumbnail (about 320 pixels) for the "My photos" view.
- **Reason:** Re-saving proves the file is a real image and removes location data. Applying the rotation **before** removing metadata prevents sideways photos, a common bug. Scaling saves disk and speeds up the TV. Thumbnails avoid loading up to 20 large photos on a phone.
- **Alternatives considered:** Keep originals untouched (keeps GPS data, bigger files). Resize only when serving (more work per request). No thumbnails (slow "My photos").
- **Impact on implementation:** A processing function in the upload path. PRD FR-STO-03 only mentions re-encoding and metadata, so scaling and thumbnails are **additions to the PRD** (see Part 6).
- **Limitations or risks:** Re-saving a JPEG loses a little quality. 1920 pixels is slightly soft on a 4K TV.
- **Confirmation needed:** Recommended.

### A-16: Invalid files
- **Status:** ASSUMED
- **Assignment says:** File validation must be addressed (F-08).
- **Assumption:** Each file in a submission is checked separately. Invalid files are rejected and the valid ones are still stored. The visitor sees a result for each file with a plain reason. Rejected files leave nothing behind. Messages never show file paths or technical details.
- **Reason:** One bad file should not cost the visitor all their photos.
- **Alternatives considered:** All-or-nothing (simpler but frustrating).
- **Impact on implementation:** The response lists a result per file. Tests cover mixed submissions.
- **Limitations or risks:** The response format is slightly more complex.
- **Confirmation needed:** No.

### A-17: Duplicate images
- **Status:** ASSUMED
- **Assignment says:** Nothing.
- **Assumption:** The same image uploaded twice is stored twice. There is no duplicate detection.
- **Reason:** Detection adds code and decisions the assignment does not need.
- **Alternatives considered:** Compare a fingerprint of the image content and skip repeats.
- **Impact on implementation:** None.
- **Limitations or risks:** A visitor can fill their 20-photo allowance with copies.
- **Confirmation needed:** No.

### A-18: Image storage
- **Status:** ASSUMED
- **Assignment says:** Nothing about storage.
- **Assumption:** Photo files are kept in a dedicated folder on the server, outside the frontend code and **excluded from Git**. Each gets a random unique name. The database stores the file name and the details (owner, time, status). Photos reach the browser only through the application by photo ID, never by browsing the folder. A file is written first and the record saved after; if either step fails, both are undone.
- **Reason:** Simple, cheap, and easy to explain. Generated names avoid overwriting and path tricks.
- **Alternatives considered:** Store images inside the database (bloats the database file). Cloud storage (cost, secret keys, extra complexity). Serve the folder directly as static files (risks directory listing and unintended access).
- **Impact on implementation:** A storage folder setting, a `.gitignore` entry, a photo-serving route and a cleanup step on failure.
- **Limitations or risks:** The disk can fill up. There are no backups. Some hosting services erase local files on restart (A-30).
- **Confirmation needed:** No.

### A-19: Simultaneous uploads
- **Status:** ASSUMED
- **Assignment says:** The design must address what happens when several people upload at once (F-08).
- **Assumption:** The target is **at least 10 simultaneous uploads** without loss. Every request is independent: it has its own unique file names and its own database session. SQLite is set up to wait briefly when busy and to let readers (the display) continue while a write is in progress. Image processing happens before the short database write. If the database is still busy, the visitor sees "please try again" and nothing is half-saved.
- **Reason:** Meets the assignment with the chosen stack and a beginner-friendly design.
- **Alternatives considered:** A server such as PostgreSQL (heavier setup). A background worker queue (more moving parts). One global lock (safe but slow).
- **Impact on implementation:** Database settings, a unique-name rule and an automated test that sends 10 uploads at once, including 10 files with the same name.
- **Limitations or risks:** SQLite allows only one writer at a time. That is fine for short writes at event scale but not for large crowds. Passing the test on one development machine does not prove behaviour in production.
- **Confirmation needed:** No.

---

## Group D: Slideshow and Display

### A-20: Photo display duration
- **Status:** **OPEN**
- **Assignment says:** The photo plays "for a specific duration" (F-02). The number is not given, and the phrase has two possible meanings.
- **Provisional default (so work can continue):** Reading **A**: each photo is shown for **10 seconds** (P-01), set in one configuration value.
- **What needs to be decided:**
  - **Reading A:** each photo stays on screen N seconds each time it plays.
  - **Reading B:** after upload, a photo stays in the slideshow for a fixed total period (for example 30 minutes) and then leaves.
  - Which value is acceptable (we suggest 5 to 15 seconds for reading A).
- **Reason for the default:** Reading A is the usual meaning of a slideshow. Reading B would also conflict with replaying previous photos, because old photos would already have expired.
- **Alternatives considered:** Reading B. Both together.
- **Impact on implementation:** Reading B would need an expiry time on every photo and a cleanup rule.
- **Limitations or risks:** A wrong reading could be marked down in assessment.
- **Confirmation needed:** **Required.** Please confirm the meaning with the instructor.

### A-21: Slideshow ordering
- **Status:** ASSUMED
- **Assignment says:** Nothing about order.
- **Assumption:** The display uses two lists. (1) A **"play next" queue**, first in first out, for new uploads and returning-visitor sequences. (2) A **main loop** of all active photos in upload order, oldest first, repeating. When the queue has entries, the next slide comes from the queue; otherwise from the loop. New photos are also added to the end of the main loop. The queue lives in the display page's memory; if the display page is reloaded, the queue is lost and the loop restarts from the beginning.
- **Reason:** Delivers "as soon as an upload completes" and a predictable, testable loop.
- **Alternatives considered:** Random order (fair but hard to test). Newest first. A playlist built and kept by the server (survives reloads, but more code).
- **Impact on implementation:** Queue logic in the display page, tested with fixed data. **PRD FR-AUT-02 needs a small clarification:** "plays next" means next in the queue. If other sequences are already waiting, the new photo waits behind them (see Part 6).
- **Limitations or risks:** A reload loses the queue. A long queue delays new photos (A-23).
- **Confirmation needed:** No.

### A-22: New and previous photos shown together
- **Status:** ASSUMED + CONFIRM
- **Assignment says:** The slideshow "also plays" a returning visitor's previous photos (F-04).
- **Assumption:** "Also" means **together**. When a returning visitor's upload completes, the queue gets their new photos first, followed by their earlier photos, **most recent first**. After that, all of them continue in the main loop.
- **Reason:** It makes the returning-visitor feature visible and fits the wording.
- **Alternatives considered:** Previous photos only in the normal loop (they would play eventually, so the feature would be invisible and hard to demo). Previous photos before the new ones. Previous photos only on request.
- **Impact on implementation:** Matches PRD FR-PRV-01 to FR-PRV-03.
- **Limitations or risks:** A sequence of up to 25 photos takes about 4 minutes of screen time at 10 seconds each.
- **Confirmation needed:** Recommended, because this interprets the assignment wording.

### A-23: Limits on the "play next" queue
- **Status:** ASSUMED + CONFIRM
- **Assignment says:** Nothing.
- **Assumption:** In the MVP there is no cap beyond the 20 photos per visitor, but: a photo already waiting in the queue is never added a second time; if the same visitor uploads again while their sequence is still queued, only the new photos are added, at the end of the queue; and the number of earlier photos replayed is held in a configuration value (proposed parameter P-10, default "all up to the visitor's limit").
- **Reason:** Keeps the MVP simple but leaves a safe place to tighten it.
- **Alternatives considered:** Replay at most the 5 most recent photos. Allow a replay only once every 10 minutes per visitor.
- **Impact on implementation:** Duplicate check when adding to the queue. One extra setting.
- **Limitations or risks:** At a busy event the queue can become minutes long, so "plays next" no longer feels like "as soon as the upload completes". A cap should be chosen after trying the display with several visitors.
- **Confirmation needed:** Recommended.

### A-24: More than one display
- **Status:** ASSUMED
- **Assignment says:** "A smart TV or computer display", singular (F-03).
- **Assumption:** Several display pages may be open at once (for example a TV and a laptop). Each is independent, has its own queue and loop, and is **not synchronised** with the others.
- **Reason:** Keeps the server stateless about displays.
- **Alternatives considered:** A server-controlled shared playlist.
- **Impact on implementation:** None extra.
- **Limitations or risks:** Two screens will not show the same photo at the same moment.
- **Confirmation needed:** No.

### A-25: Slideshow refresh interval
- **Status:** ASSUMED
- **Assignment says:** The photo plays "as soon as an upload completes" (F-02). No timing is given.
- **Assumption:** The display page asks the server for changes **every 5 seconds** (called *polling*). Together with processing time this keeps new, deleted and withdrawn photos within the 10-second target (PRD P-08). The interval is a configuration value.
- **Reason:** Polling is the simplest method to build, test and explain, and it works through any network.
- **Alternatives considered:** WebSockets or Server-Sent Events (instant, but harder for a beginner and for some hosts). Long polling. A 2-second interval (faster, more load). A 30-second interval (too slow).
- **Impact on implementation:** A timer in the display page and a lightweight route returning the current photo list or recent changes. The final method is recorded in ARCHITECTURE.md.
- **Limitations or risks:** Delay of up to about 5 seconds. Browsers slow timers in hidden tabs, so the display must be the visible tab.
- **Confirmation needed:** No.

### A-26: TV display behaviour
- **Status:** **OPEN**
- **Assignment says:** The slideshow plays on "a smart TV or computer display" (F-03). The device is not specified.
- **Provisional default (so work can continue):** The display is a **web page opened in a modern browser**, designed first for Chrome or Edge on a computer connected to a TV or monitor at **1920 x 1080 (16:9)**. Photos are scaled to fit without cropping, on a dark background, with large readable text. The page needs no interaction while running. **Browsers do not allow a page to enter full screen by itself**, so someone must do one setup step (for example press F11, or use a kiosk mode) when starting the display. The venue is expected to turn off TV and computer sleep.
- **What needs to be decided:** Which device will be used in the demo, and whether it is the TV's own built-in browser. Built-in TV browsers are often old and may not run the modern React build, so the actual TV must be **tested**. If it fails, the fallback is a laptop connected by HDMI.
- **Alternatives considered:** A native TV app (out of scope). Casting from a phone.
- **Impact on implementation:** README setup steps for the display. A 16:9 layout. A real-device test.
- **Limitations or risks:** The PRD says the display "needs no interaction". That is true while running, but starting full screen needs the one setup step above (see Part 6).
- **Confirmation needed:** **Required.** Please tell us what screen will be used for the demo.

### A-27: Internet disconnection
- **Status:** ASSUMED
- **Assignment says:** Nothing.
- **Assumption:** Three cases:
  1. **Display loses its connection to the server:** it keeps playing the photos already loaded in the browser, retries in the background and updates by itself when the connection returns. No error text appears on the TV.
  2. **Display page is reloaded or started while the server is unreachable:** it shows a simple "Reconnecting" message, because there is no offline mode.
  3. **Visitor's phone loses connection during an upload:** the upload fails, the server keeps no partial file or record, and the visitor sees "Upload did not finish, please try again". There is no automatic retry or offline queue.
  If everything runs on a local network, no internet is needed at all.
- **Reason:** Covers the realistic failures with simple behaviour.
- **Alternatives considered:** A browser offline cache so the display survives reloads (E-07). Automatic upload retry.
- **Impact on implementation:** Error handling and retry timer in the display page. Tests that stop and restart the server.
- **Limitations or risks:** Photos not yet loaded in the browser cannot be shown while offline. A long outage repeats the same few photos.
- **Confirmation needed:** No.

---

## Group E: Access, Administration and Deployment

### A-28: Public access to the pages
- **Status:** ASSUMED
- **Assignment says:** Visitors give their name and photos. The display shows them. No login is mentioned.
- **Assumption:** Both the upload page and the display page are open to **anyone who has the address**. There is no login, no event code and no secret link. The venue shares the upload address (for example on a poster).
- **Reason:** A TV cannot log in, and visitors should be able to upload with only a name.
- **Alternatives considered:** A secret display address. Allowing only the venue Wi-Fi. A short event code before uploading.
- **Impact on implementation:** None extra.
- **Limitations or risks:** Anyone with the address can watch the display and upload, so unwanted content is possible. Mitigations are the limits, the consent step and operator removal (A-29).
- **Confirmation needed:** No.

### A-29: Admin and operator permissions
- **Status:** ASSUMED + CONFIRM
- **Assignment says:** No administrator or operator is mentioned.
- **Assumption:** There are **no admin accounts and no dashboard** in the MVP. A photo can be removed only by its owner. For emergencies the developer can remove a photo by hand (a short manual procedure will be written in the README). An optional **operator removal** feature (Could Have, FR-OPR) would use a secret key from environment settings, with no login screen. Only a placeholder goes in `.env.example`.
- **Reason:** Stays within the assignment and the 20-hour limit.
- **Alternatives considered:** A full admin dashboard (out of scope, PRD OOS-08). A moderation queue that approves each photo before display (good for privacy but adds a whole workflow).
- **Impact on implementation:** None for the MVP, apart from the README procedure.
- **Limitations or risks:** Inappropriate content stays on screen until removed. Photos whose owner lost the token (A-03, A-04) can be removed only by the developer. Operator removal is **Could Have** in the PRD, but it is the only practical way to fix both problems.
- **Confirmation needed:** Recommended. Consider promoting FR-OPR-01 to Should Have.

### A-30: Deployment and network setup
- **Status:** **OPEN**
- **Assignment says:** The README includes a deployed link "if available" (F-13). Deployment is optional.
- **Provisional default (so work can continue):** Develop and demonstrate **locally**. The backend and frontend run on one computer. Visitors' phones join the same Wi-Fi and open the computer's **local network address** (phones cannot use "localhost"). The display opens on the same computer or a second screen. The server must allow requests from the frontend's address. A public deployment is an optional extra after the MVP.
- **What needs to be decided:** (1) Whether a deployed link is wanted. (2) Which host. Many free hosts erase local files on restart, which would delete the photos and the SQLite file, so this must be checked first. (3) HTTPS and how the frontend is served in production.
- **Alternatives considered:** Deploy from the start. Use a tunnelling service for demos.
- **Impact on implementation:** README run instructions for the local network. Allowed addresses in the settings file. Nothing else for the MVP.
- **Limitations or risks:** A local-only demo may be judged less impressive than a live link. A hurried deployment could lose data or expose secrets.
- **Confirmation needed:** Required only if a deployed link is wanted.

---

## Group F: Other

### A-31: Language, time zone and timestamps
- **Status:** ASSUMED
- **Assignment says:** Nothing.
- **Assumption:** The interface is in English only. Times are stored in UTC. No times are shown to visitors.
- **Reason:** Avoids time zone bugs and extra work.
- **Alternatives considered:** Several languages (E-08). Local time storage.
- **Impact on implementation:** Timestamps created in UTC.
- **Limitations or risks:** Visitors who do not read English may not understand the consent notice.
- **Confirmation needed:** No.

### A-32: Testing approach
- **Status:** ASSUMED
- **Assignment says:** "Automated tests, or a documented manual test plan with results" (F-09).
- **Assumption:** The backend gets **automated Pytest tests** for validation, limits, token handling, consent, deletion, withdrawal and simultaneous uploads. Display behaviour, phone behaviour and the TV test are covered by a **manual test plan with recorded results**. Frontend automated tests are not required.
- **Reason:** Automated tests where they are cheap and valuable, manual checks where a real screen is needed.
- **Alternatives considered:** Frontend test tools such as Vitest (extra dependencies and time). Manual testing only.
- **Impact on implementation:** Sample files from PRD Appendix B. A results table in the repository.
- **Limitations or risks:** Display timing is checked by eye.
- **Confirmation needed:** No.

---

# Part 4: Open Items Summary

These are the items that cannot be settled responsibly by the developer alone.

| ID | Question to decide | Provisional default in use | Who can decide | Blocks |
|---|---|---|---|---|
| A-11 | How long should photos be kept in a real deployment, and should they expire? | Kept until deleted or withdrawn | You, with the instructor or venue owner | Public deployment only |
| A-20 | Does "specific duration" mean seconds per photo, or total time in the slideshow? What value? | 10 seconds per photo | Instructor | Display logic |
| A-26 | What screen will run the display in the demo? Does its browser work? | Chrome or Edge on a computer at 1920 x 1080 | You | Demo preparation |
| A-30 | Is a deployed link wanted, and where? | Local network demo | You | Optional deployment |

---

# Part 5: Recommended Future Enhancements

These are **not** part of the assignment and **not** part of the MVP. They are listed so the limitations above have a visible path forward.

| ID | Enhancement | Why it would help | Related |
|---|---|---|---|
| E-01 | Recovery code to restore a visitor on another device | Fixes cross-device recognition and lost-token problems | A-03, A-10 |
| E-02 | Moderation queue that approves photos before display | Prevents unwanted content on a public screen | A-28, A-29 |
| E-03 | Operator dashboard with login | Easier removal and event control | A-29 |
| E-04 | Rate limiting per network address | Stops spam registration and uploads | A-06, A-14 |
| E-05 | Automatic expiry or a "clear all data" action at event end | Supports a retention policy | A-11 |
| E-06 | Shrink photos in the browser before upload | Fewer rejections for large phone photos | A-13 |
| E-07 | Browser offline cache for the display | Display survives reloads without a connection | A-27 |
| E-08 | Several interface languages | Wider audience | A-31 |
| E-09 | A QR code for the upload address, shown on the display | Easier for visitors to join | A-28 |
| E-10 | Secret display address | Keeps the screen from being watched by outsiders | A-28 |
| E-11 | HEIC support | Accepts more iPhone photos | A-12 |
| E-12 | Server-held playlist | Queue survives display reloads and screens stay in sync | A-21, A-24 |

---

# Part 6: Link to the PRD

## 6.1 Old assumption IDs

| PRD ID | Now covered by |
|---|---|
| AS-01 (display loops all photos) | A-21 |
| AS-02 (new photo plays next, then previous photos) | A-21, A-22 |
| AS-03 (token in browser, never name) | A-01, A-02 |
| AS-04 (new device means new visitor) | A-03 |
| AS-05 (display page is public) | A-28 |
| AS-06 (photos kept until removed) | A-11 |
| AS-07 (no age verification) | A-08 |
| AS-08 (small-scale use) | A-19, A-30 |

## 6.2 Updates the PRD needs

Before coding, these changes to PRD.md should be approved. Nothing has been changed silently.

| # | PRD item | Finding | Suggested change |
|---|---|---|---|
| 1 | FR-AUT-02 ("plays next") | "Next" is not literal when other sequences are already queued. | Clarify: the new photo plays next **in the queue** (A-21). |
| 2 | FR-VAL-02 AC4 | Left open between "reject" and "accept by real format". | Settled: accept by real format (A-12). |
| 3 | ERR-12 (HEIC) | Handling was left to this document. | Settled: reject with a clear message; verify on real phones (A-12). |
| 4 | FR-STO-03 | Mentions re-encoding and metadata only. | Add upright rotation, 1920-pixel scaling and thumbnails (A-15). |
| 5 | FR-WDR-03 | Does not say what happens to the visitor's name. | Add: the stored name is cleared on withdrawal (A-09). |
| 6 | FR-DSP-01 ("no interaction") | Browsers require one manual step to enter full screen. | Add a note: one setup step at start (A-26). |
| 7 | FR-PRV-01 | Does not limit how many earlier photos replay. | Add the setting P-10 (A-23). |
| 8 | Parameter P-02 (5 MB) | May reject many real phone photos. | Keep for now; re-check with real photos (A-13). |
| 9 | FR-OPR-01 (Could Have) | It is the only practical fix for orphaned photos. | Consider promoting it to Should Have (A-29). |

---

# Part 7: Confirmation Checklist

Please answer these before architecture work starts.

1. **A-03:** Is recognising a returning visitor on the **same browser** acceptable for "visiting again"?
2. **A-20:** Does "specific duration" mean seconds per photo (our default, 10 seconds)?
3. **A-26:** What screen will run the display in the demo, and can we test its browser?
4. **A-07:** Is consent once per registration enough, or should each upload need a tick?
5. **A-22:** Should a returning visitor's new and previous photos play together (our default)?
6. **A-13:** Keep 5 MB per photo, or raise it to 10 MB after testing real phone photos?
7. **A-29:** Should operator removal move up to Should Have?
8. **A-30:** Do you want a deployed link, or is a local demo enough?
9. **A-11:** Is "kept until deleted" acceptable for the MVP?

---

# Appendix: Terms Used

| Term | Meaning |
|---|---|
| **Token** | A long random value that acts like a ticket the browser shows to prove who it is. |
| **Hash** | A one-way scrambled form of a value. It can be compared but not turned back into the original. |
| **EXIF data** | Hidden information inside a photo, such as camera details and GPS location. |
| **Polling** | The display asking the server "anything new?" at regular intervals. |
| **Queue (first in, first out)** | A waiting line where the earliest entry is served first. |
| **Provisional default** | A temporary choice that lets work continue until an OPEN question is answered. |

---

# Document Control

| Version | Date | Change |
|---|---|---|
| 1.0 | 14 Nov 2026 | First draft, based on the assignment, PROJECT_BRIEF.md v1.0 and PRD.md v1.0 |

**Next steps:** (1) Answer the checklist in Part 7. (2) Approve the PRD updates in Part 6.2. (3) Move the final decisions into the version of this file that goes in the repository. (4) Write ARCHITECTURE.md, the data model and the API specification.
