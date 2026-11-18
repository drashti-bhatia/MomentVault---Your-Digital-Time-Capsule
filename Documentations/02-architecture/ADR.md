# ADR.md

# Visitor Photo Slideshow (MomentVault): Architecture Decision Records

| Item | Detail |
|---|---|
| Product | MomentVault, implementing Assignment 1: Visitor Photo Slideshow |
| Document version | 1.0 (draft for review) |
| Date | 17 November 2026 |
| Based on | Assignment_1.pdf, PROJECT_BRIEF.md, PRD.md, ASSUMPTIONS.md, TECH_STACK.md, ARCHITECTURE.md v1.1, DATABASE_DESIGN.md, API_SPECIFICATION.md, BACKEND_DESIGN.md, FRONTEND_DESIGN.md, SECURITY_PRIVACY.md, IMPLEMENTATION_PLAN.md |
| Scope | Ten major decisions made while planning the **initial local MVP** |
| Not included | Source code. Small design choices that already have an `AD-nn` number in ARCHITECTURE.md are summarised in Section 4, not repeated. |

**What is an ADR?** An Architecture Decision Record is a short note that answers three questions about one important choice: *what did we choose, why, and what would make us change our mind?* A new developer, or a marker in a live demo, can read it and understand a choice without guessing.

**What was checked, and what was not.** The context, options and trade-offs below come from the project's own documents. No code exists yet, and FastAPI, SQLAlchemy and httpx behaviours are **not verified** here (they are listed as "to verify" in BACKEND_DESIGN). Where a consequence depends on such a behaviour, the ADR says so.

---

## 1. Read This First

### 1.1 How "Status" Is Used

| Status | Meaning |
|---|---|
| **Accepted** | You, the developer, stated this as a fixed starting point (in the assignment, the brief, or a request), so it is not open for debate in this document. |
| **Proposed** | Written down in a document marked "draft for approval" with **no recorded confirmation** from you. It is the current plan, but it can still change. |
| **Accepted in part** | The core idea is accepted, but a named detail is still Proposed. Both parts are listed, so you can see exactly what still needs your confirmation. |
| **Superseded** / **Deprecated** | Not used yet. They appear when a later ADR replaces this one. |

> **Why not mark everything "Accepted"?** Every design document so far says "draft for approval", and the conversation holds no record of you approving individual decisions. Marking them all final would claim a confirmation that is not on record. If you confirm a decision, change its status and add the date (Section 5).

### 1.2 Summary of the Ten Decisions

| ADR | Decision | Status | Still needs your confirmation |
|---|---|---|---|
| ADR-001 | FastAPI as the backend framework | **Accepted** | Synchronous endpoints (T-02) |
| ADR-002 | React and Vite for the frontend | **Accepted** | No routing library, no state library (T-05) |
| ADR-003 | SQLite as the initial database | **Accepted** | WAL mode, busy wait, one worker (T-03) |
| ADR-004 | SQLAlchemy as the ORM | **Accepted** | Synchronous sessions, services commit (T-02, AD-04) |
| ADR-005 | Local filesystem for initial image storage | **Proposed** | The whole decision (A-18, T-04) |
| ADR-006 | A unique visitor return token, not name-only identification | **Accepted in part** | Accepted: never identify by name alone. **Proposed:** the token mechanism, local-storage-plus-header (AD-07), same-browser limit (A-03) |
| ADR-007 | Polling instead of WebSockets | **Proposed** | The whole decision (A-25) |
| ADR-008 | Layered backend architecture | **Accepted in part** | Accepted: simple layers, no forced MVC. **Proposed:** thin repository modules (AD-03) |
| ADR-009 | Separate visitor and TV display interfaces | **Proposed** | The whole decision, including the optional operator page |
| ADR-010 | Consent-based public photo display | **Accepted in part** | Accepted: consent is required. **Proposed:** the tick-box mechanism, notice wording, withdrawal |

### 1.3 Why Some Are Accepted

| Evidence | Decisions |
|---|---|
| The technology list was given by you at the start (Python, FastAPI, React, JavaScript, Vite, SQLite, SQLAlchemy, Pillow, pytest). TECH_STACK calls it "the approved stack". | ADR-001 to ADR-004 |
| You asked for a simple layered backend and no forced MVC. | ADR-008 (the layering) |
| The assignment requires privacy and consent to be addressed (F-07), and you stated that names must not identify visitors uniquely. | ADR-010 (consent is required), ADR-006 (not by name) |

### 1.4 Findings

| # | Finding | Handling |
|---|---|---|
| **ADR-F1** | **Accepted technology can still hide Proposed details.** "Use FastAPI" is accepted, but "use synchronous endpoints" is a separate choice that only TECH_STACK T-02 proposes. | The "Status" line of each ADR names the Proposed detail. They are not hidden inside an Accepted heading. |
| **ADR-F2** | **No ADR covers the whole of the "Proposed" list in TECH_STACK Part 10 and ARCHITECTURE Part 17** (24 smaller decisions). Making 24 more ADRs would bury the ten that matter. | Section 4 lists the main ones and how to promote them to an ADR. |
| **ADR-F3** | **ADR-006 depends on an open confirmation.** ASSUMPTIONS A-03 says same-browser recognition needs your confirmation ("Required"), because an assessor may expect recognition across devices. | The ADR records the limit plainly and names the fix (a recovery code, E-01) as a trigger. |
| **ADR-F4** | **"10 seconds" is interpreted, not defined, in the PRD.** The ADR-007 reasoning uses the ARCHITECTURE reading: a new photo **enters the playback queue** within about the poll interval, but is **on screen** only after the current slide ends. | Recorded in ADR-007. PRD wording can be clarified (ARCHITECTURE finding 1). |
| **ADR-F5** | **ADR-010 and the withdrawal feature are linked.** The consent notice promises that photos can be removed. IMPLEMENTATION_PLAN P-2 handles this by building withdrawal, or else removing the promise from the notice. | Recorded in ADR-010. |

---

## 2. Template Used for Every ADR

Every record has these eleven parts, in this order: **ADR ID, Decision title, Status, Context, Problem, Options considered, Decision, Reasons, Positive consequences, Negative consequences, Future reconsideration conditions.**

---

## 3. The Decisions

---

### ADR-001: FastAPI as the Backend Framework

**ADR ID:** ADR-001

**Decision title:** Use FastAPI (with Uvicorn) as the backend web framework.

**Status:** **Accepted.** The framework is part of your stated stack. **Proposed detail:** synchronous (ordinary, non-async) endpoints (TECH_STACK T-02, ARCHITECTURE AD-05).

**Context:** The backend must receive photo uploads, check them, store them, and answer questions from two kinds of browser page. It is built by one MCA beginner in about 20 hours, with AI assistants, and every part must be explainable in a live demo. Python is the language you know best.

**Problem:** Which Python web framework gives a clear, small, well-documented API with automatic input checking, without a long learning curve?

**Options considered:**

| Option | For | Against | Verdict |
|---|---|---|---|
| **FastAPI** | Python type hints become automatic request checking (with Pydantic). Generates interactive API documentation. Has a test client. Many beginner tutorials. | Does not limit upload size by itself (our own guard is needed). Older tutorials use older syntax. | **Chosen** |
| Flask | Very small and simple | No built-in request validation or API documentation. More hand-written checking code. | Not chosen |
| Django (with its REST add-on) | Complete, with an admin area and its own database layer | Large, with many concepts to learn. Its own ORM would replace SQLAlchemy (ADR-004). Too heavy for two pages and about 10 endpoints. | Not chosen |
| A non-Python framework (for example Node.js) | One language across the project | You would learn a new backend language in a 20-hour project. | Not chosen |

**Decision:** Build the backend with FastAPI, served by one Uvicorn process. Use ordinary functions (not `async def`) for endpoints that call Pillow and SQLAlchemy, because those libraries block.

**Reasons:**
- Automatic checking of requests and a visible, generated description of the API help a beginner see what the server accepts.
- It is already in your stated stack. Changing it would also change the testing and schema tools.
- `TestClient` allows real endpoint tests without starting a server (TESTING_PLAN).

**Positive consequences:**
- Less checking code to write by hand.
- The API contract (API_SPECIFICATION) maps directly onto Pydantic schemas.
- Automatic documentation is useful in a demo.

**Negative consequences:**
- **Upload size is not capped by the framework**, so a request size guard is custom code that must be written and tested (ARCHITECTURE AD-19, to verify on the pinned versions).
- Mixing synchronous and asynchronous code is a known beginner trap. The rule is therefore simple: synchronous everywhere in the MVP.
- Several FastAPI behaviours used by the designs are **not verified yet** (listed as "to verify" in BACKEND_DESIGN 24.3).

**Future reconsideration conditions:** Revisit the synchronous-endpoint detail if many displays or visitors cause slow responses and measurement shows that blocking calls are the cause. Revisit the framework only if Day 1 checks show an unsolvable problem with the upload guard or test tooling on the pinned versions.

---

### ADR-002: React and Vite for the Frontend

**ADR ID:** ADR-002

**Decision title:** Build the frontend with React, JavaScript and Vite, with plain CSS.

**Status:** **Accepted.** React, JavaScript and Vite are part of your stated stack. **Proposed detail:** no routing library, no state library, no UI kit (TECH_STACK T-05, ARCHITECTURE AD-17).

**Context:** The product has an upload page for phones and a full-screen slideshow for a TV (ADR-009). The slideshow needs timers, polling, a waiting line of new photos, and clean-up. The developer must be able to explain each component and hook.

**Problem:** Which frontend approach lets a beginner build both pages, including the timer-heavy slideshow, in the available time, and also gives a good portfolio result?

**Options considered:**

| Option | For | Against | Verdict |
|---|---|---|---|
| **React with Vite** | Component and hook structure fits the pages. Fast development server. Strong portfolio value. | More concepts than plain JavaScript. Timers and polling cause well-known mistakes (stale values, missing clean-up). | **Chosen** |
| Plain HTML and JavaScript | Fewer concepts. TECH_STACK's honest note says it would be **quicker to build** for two pages. | Harder to keep a growing page tidy. Weaker portfolio signal. | Not chosen (see reasons) |
| Vue or Svelte | Gentle learning curve | Not in your stated stack. You would learn an extra framework. | Not chosen |
| A full-stack React framework such as Next.js | Server rendering, routing | Adds a Node server beside FastAPI and many concepts the product does not need | Not chosen |
| Server-rendered pages from FastAPI templates | One program | The slideshow still needs a lot of browser JavaScript. Mixes page rendering into the backend. | Not chosen |

**Decision:** Use React with Vite. Use React's built-in tools only (`useState`, `useReducer`, `useEffect`, `useRef`, one context). In development Vite forwards `/api` to the backend. For the demo, FastAPI serves the built files.

**Reasons:**
- It is part of the stated stack and suits a portfolio project.
- The component and hook structure separates "draws" from "connects", which is easy to explain.
- Vite starts quickly and has a built-in proxy, so the browser sees one address.

**Positive consequences:**
- Reusable parts (button, banner, dialog) keep the design consistent.
- Pure logic (slideshow ordering, file checks) can sit outside React and be tested with a plain runner.
- Few dependencies: no router, no state library, no UI kit.

**Negative consequences:**
- **Learning load.** TECH_STACK names slideshow timer bugs as risk R1 (high likelihood, high impact). The mitigation is to build a plain version first and add polling second.
- A build step is needed for the demo, and the backend must serve the built files.
- React in development mode may run some code twice on purpose, which can confuse a beginner.

**Future reconsideration conditions:** If the display page cannot be made reliable in its budgeted time on Day 3 or Day 4, consider a plain JavaScript display page while keeping React for the upload page. Add a router or state library only if a third or fourth page makes the current page switch hard to follow.

---

### ADR-003: SQLite as the Initial Database

**ADR ID:** ADR-003

**Decision title:** Use SQLite as the database for the initial MVP.

**Status:** **Accepted.** SQLite is part of your stated stack. **Proposed details:** WAL mode with a busy wait, short write transactions, and one Uvicorn worker (TECH_STACK T-03, ARCHITECTURE AD-21).

**Context:** The system stores visitors, consent records and photo information. The expected load is a small event: a handful of visitors uploading at once and one or two displays. There is no database server, no budget, and no operations experience.

**Problem:** Which database is simplest to run and explain, and still strong enough for several simultaneous uploads?

**Options considered:**

| Option | For | Against | Verdict |
|---|---|---|---|
| **SQLite** | Nothing to install. One file. Easy to copy, inspect and reset for demos. Tests use a temporary file. | **One writer at a time**, even in WAL mode. Needs storage that survives restarts. WAL does not work over a network drive. Most constraints cannot be altered in place. | **Chosen** |
| PostgreSQL | Many simultaneous writers, richer features | A server to install, run and secure. More work than the 20-hour budget allows. | Not chosen (future option M-1) |
| MySQL or MariaDB | Similar to PostgreSQL | Same cost as PostgreSQL, with nothing extra for this project | Not chosen |
| JSON files | No database tool | No safe simultaneous writes, no constraints, hard to query | Not chosen |
| In-memory data only | Simplest | All data lost on restart | Not chosen |

**Decision:** Use SQLite in WAL mode with a busy wait, foreign keys on for every connection, short write transactions, and one Uvicorn worker. Image processing happens **before** the short write.

**Reasons:**
- It matches the size of the problem and the time available.
- Photo processing is the slow part. The database write is quick, so one writer at a time is enough for ten simultaneous uploads (to be proved by Day 1 spike S2 and the concurrency tests).
- SQLAlchemy (ADR-004) keeps a later move to another database possible.

**Positive consequences:**
- Zero setup and an easy demo reset.
- Simple backups (copy the data folder with the app stopped).
- Database rules can be enforced with `CHECK` and unique constraints (DATABASE_DESIGN).

**Negative consequences:**
- **Only one writer at a time**: very heavy simultaneous writing would produce "busy" waits, handled as a `SERVER_BUSY` answer.
- **Cannot scale to several server processes** without changing the database (ARCHITECTURE AD-21 depends on this).
- **Changing a `CHECK` rule means creating a new table**, copying the data and renaming it, so rules are kept short.
- On a hosting service with temporary storage, the database disappears on restart (TECH_STACK 5.3).

**Future reconsideration conditions:** Move to PostgreSQL (TECH_STACK M-1) if any of these become true: many people write at the same time and "busy" answers appear in normal use, more than one server instance is needed, or the chosen host has no persistent local disk.

---

### ADR-004: SQLAlchemy as the ORM

**ADR ID:** ADR-004

**Decision title:** Use SQLAlchemy 2.0 (typed models and `select()` queries) to talk to the database.

**Status:** **Accepted.** SQLAlchemy is part of your stated stack. **Proposed details:** synchronous sessions, and "services commit, repositories never commit" (TECH_STACK T-02, ARCHITECTURE AD-04).

**Context:** The code needs to read and write three tables safely. User-supplied values (names, ids) reach the queries, so injection must be prevented. Layers (ADR-008) need a clear place for queries and for transactions.

**Problem:** How should the application talk to SQLite so that queries are safe, readable to a beginner, and the database can be changed later?

**Options considered:**

| Option | For | Against | Verdict |
|---|---|---|---|
| **SQLAlchemy 2.0 ORM** | Values are bound, never pasted into SQL. Typed models. Works with several databases. Widely used. | Another concept to learn. Many old tutorials use the earlier 1.x style. | **Chosen** |
| Plain `sqlite3` with hand-written SQL | No extra library. Shows the SQL directly. | Easy to build SQL from strings by mistake. Ties code to SQLite. More repeated code. | Not chosen |
| SQLModel | Combines models and schemas in one class | Younger project. Blurs the model and schema roles BACKEND_DESIGN keeps apart. | Not chosen |
| Another ORM (Peewee, Tortoise) | Simpler in places | Less common with FastAPI. Not in your stated stack. | Not chosen |
| SQLAlchemy in asynchronous mode | Fits `async def` endpoints | Harder to learn and debug. Pillow is blocking anyway (ADR-001). | Not chosen |

**Decision:** Use SQLAlchemy 2.0 with typed `Mapped` columns and the `select()` style. Queries live only in repository modules. Services own the commit. Sessions are created per request by a FastAPI dependency.

**Reasons:**
- Bound values prevent SQL injection by default (SECURITY_PRIVACY).
- Models describe the three tables in one readable place.
- It keeps open the move to PostgreSQL (ADR-003).

**Positive consequences:**
- No hand-built SQL strings in application code.
- Clear transaction rules (one unit of work, one commit).
- Tests can use a temporary database with the same models.

**Negative consequences:**
- **Learning cost.** Beginners may confuse a *model* (table) with a *schema* (request or response shape). BACKEND_DESIGN Section 4 explains this.
- **The exact option names are not verified** (for example, keeping objects readable after commit, BV-2). The sandbox did not have SQLAlchemy. These are Day 1 checks.
- The ORM can hide the real SQL. DATABASE_DESIGN keeps the intended SQL beside each query so the meaning is visible.

**Future reconsideration conditions:** Reconsider if the ORM is the main cause of repeated Day 2 delays (for example, the session or transaction behaviour cannot be made to match the design after the Day 1 checks). Reconsider the synchronous style only together with ADR-001.

---

### ADR-005: Local Filesystem for the Initial Image Storage

**ADR ID:** ADR-005

**Decision title:** Store photo files in a folder on the server's disk, and store only file details in the database.

**Status:** **Proposed.** Written in ASSUMPTIONS A-18 and TECH_STACK T-04, both marked as drafts. It is not part of your stated stack.

**Context:** Each accepted photo is re-encoded by the server (rotated, scaled to at most 1920 pixels, metadata removed) and kept for display. The app has no cloud account, no secrets and no paid services (PRD OOS-02, OOS-11). The first target is a local-network demo.

**Problem:** Where should the image files live, and how are they delivered to the browser without exposing the folder?

**Options considered:**

| Option | For | Against | Verdict |
|---|---|---|---|
| **Files in a local folder, details in the database** | Simplest. No account or keys. Easy to inspect and delete. Database stays small. | Disk can fill. No backups. Lost if the host has temporary storage. Not usable with several server instances. | **Proposed** |
| Image bytes inside the database (as blobs) | One file holds everything | Database file grows fast and slows backups. Harder to inspect or serve. | Not chosen |
| Cloud object storage (an S3-compatible service) | Scales, durable | Needs an account, secret keys and cost checks. Extra code. Beyond the assignment. | Not chosen (future option M-2) |
| Serve the folder directly as static files | Least code | Risks directory listing and access to files that should be removed. Names would be guessable or exposed. | Not chosen |

**Decision (proposed):** Keep display images and thumbnails under one `data/` folder next to the database, outside Git. File names are random, made by the server (32 hex characters). Photos are delivered **only** through an API route by photo id, and the file name is read from the database, never from the request.

**Reasons:**
- It needs nothing beyond what the machine already has.
- Putting the database and photos in **one** data folder keeps them from drifting apart (TECH_STACK 5.2).
- Delivery through a route means a removed photo can be refused instead of leaking from a folder.

**Positive consequences:**
- No keys or accounts to protect (NFR-SEC-02 stays simple).
- Withdrawal and deletion can remove the file and the record together (FR-STO-04).
- The storage code sits in one module (`photo_storage`), so it can be replaced.

**Negative consequences:**
- **No backups.** A manual copy of the data folder (with the app stopped) is the only backup.
- **A host with temporary storage erases photos on restart.** A deployed link needs a persistent disk and a restart test (A-30 is still OPEN).
- The disk can fill. The 20-photo limit per visitor does not cap the total, so flooding is a known gap (SECURITY_PRIVACY F-1).
- Files and database rows can disagree after a crash. The design uses a fixed order (write the file first, then the row; on removal change the row first, then delete the file) and an optional start-up clean-up.

**Future reconsideration conditions:** Move to object storage (TECH_STACK M-2) if the app needs several server instances or a host without a persistent disk. Revisit sooner if real phone photos are much larger than the estimate (a few hundred kilobytes to about 1 MB after scaling, an estimate to be measured).

---

### ADR-006: Unique Visitor Return Token, Not Name-Only Identification

**ADR ID:** ADR-006

**Decision title:** Recognise a returning visitor by a secret random token kept in their browser, never by name.

**Status:** **Accepted in part.**
- **Accepted:** a visitor is never identified by name alone (you stated that names must not be assumed unique).
- **Proposed:** the exact mechanism. A 256-bit random token, stored in browser local storage, sent in an `X-Visitor-Token` header, only its SHA-256 hash stored on the server (ASSUMPTIONS A-01, ARCHITECTURE AD-07). The ARCHITECTURE finding 3 asks you to confirm the storage choice.
- **Awaiting confirmation (required):** that "same browser only" satisfies "visiting again" (ASSUMPTIONS A-03).

**Context:** The assignment requires that the design address how a returning visitor is identified (F-06) and that the slideshow then plays that person's earlier photos. It does not say how. There are no accounts, no email and no phone numbers (PRD OOS-03), and extra personal data is to be avoided.

**Problem:** How can the system recognise "the same visitor" without collecting more personal data, and without letting one person see or remove another's photos?

**Options considered:**

| Option | For | Against | Verdict |
|---|---|---|---|
| **Random token in the browser, hash on the server** | No extra personal data. Works on any phone. Easy to explain. A leaked database does not leak usable tokens. | Recognises only the same browser. A lost token cannot be recovered. | **Chosen (details proposed)** |
| Name only | Simplest | Names repeat and are misspelled. Anyone typing "Priya" would see or remove her photos. | **Rejected** |
| Name plus email or phone | Works across devices | Collects personal data, needs verification, goes beyond the assignment | Rejected |
| Name plus a chosen PIN | Cross-device | PINs are guessable, adds fields the assignment does not mention | Rejected |
| Accounts and passwords | Standard | Out of scope for the MVP (OOS-03) | Rejected |
| Device fingerprinting | No user step | Invasive and unreliable | Rejected |

**Sub-decision (proposed): where the token is kept.**

| | **Local storage plus request header (proposed)** | Secure cookie |
|---|---|---|
| Easy to explain and test | Yes. Tests set a header. | Needs cookie handling in tests |
| Protection from a malicious script | **Weaker**: a script on the page could read it | Stronger if hidden from scripts |
| Cross-site request tricks | Not exposed (the browser does not add the header itself) | Needs extra protection |
| Plain HTTP on a local network | Works | `Secure` cookies need HTTPS |

**Decision (proposed):** Duplicate names are allowed and never merged. The server creates a 256-bit token at registration, the browser keeps it, and the server stores only its hash. Visitors can press "Not you?" to clear it. A visitor who loses the token is treated as new.

**Reasons:**
- It collects **no** personal data beyond the name the visitor gives.
- The token cannot be guessed (256 bits from a secure random source), unlike a name or a sequential number.
- Local storage plus a header fits plain HTTP on a local network and is simple to test.

**Positive consequences:**
- Two visitors called "Sam" stay separate (tested by a duplicate-name case).
- A database leak does not hand out working tokens.
- No cookies, so no cross-site request protection is needed for the visitor routes.

**Negative consequences:**
- **Recognition is same-browser only.** A different browser, device, private window or cleared data means a new visitor. Photos of the old identity cannot be removed by the visitor and stay on the display until the operator removes them (only if the optional operator removal, E10, is built; A-03, A-10, A-29).
- **Weaker script protection.** A script on the page could read the token (SECURITY_PRIVACY F-4). Mitigations: visitor text is always shown as text, no third-party scripts, and a content security policy header is recommended.
- On plain HTTP the token travels unencrypted on the local network (SECURITY_PRIVACY F-6). It is accepted for a trusted local demo only.
- Anyone using the same browser profile acts as that visitor.

**Future reconsideration conditions:**
- If an assessor or user expects recognition on another device: add a **recovery code** (ASSUMPTIONS E-01) before coding that part.
- If the app moves to a public host: switch to a protected cookie with HTTPS (ARCHITECTURE 9.3a).
- If token theft by a script is ever demonstrated, or a content security policy cannot be applied.

---

### ADR-007: Polling Instead of WebSockets

**ADR ID:** ADR-007

**Decision title:** The display page asks the server for changes every 5 seconds, rather than holding a live connection open.

**Status:** **Proposed.** ASSUMPTIONS A-25 is marked ASSUMED, and TECH_STACK lists WebSockets as "not included". There is no recorded confirmation.

**Context:** The assignment says a photo should play "as soon as an upload completes" (F-02), with no timing given. The PRD treats "within 10 seconds" as the target (FR-AUT-01, P-08). One or two displays are expected (A-24). The server keeps no state about any display.

**Problem:** How does the display learn that a new photo, a removal or a withdrawal has happened, using the least complex method that meets the target?

**Options considered:**

| Option | For | Against | Verdict |
|---|---|---|---|
| **Polling every 5 seconds** | Simplest to build, test and explain. Works on any network and any TV browser. Server stays stateless. | Constant small requests. Changes appear up to one interval later. | **Proposed** |
| WebSockets | Instant updates | Server must hold open connections and manage them. Harder to test. Reconnect logic needed. Some networks and TV browsers behave badly. | Not chosen (future option M-3) |
| Server-Sent Events | One-way push, simpler than WebSockets | Still a long-lived connection to manage. Not needed for this size. | Not chosen |
| Long polling | Closer to instant | More complex than polling, with less benefit | Not chosen |
| Polling every 2 seconds | Faster | More load for small gain | Not chosen |
| Polling every 30 seconds | Very light | Too slow for the target | Not chosen |

**Decision (proposed):** The display calls the slideshow feed (E7) every 5 seconds (a setting), passing a **cursor** (the highest photo id it has seen). The server answers from the database each time and remembers nothing about the display. The waiting line and the loop live in the browser's memory.

**Reasons:**
- The simplest method that meets the target, in a project measured in hours.
- A stateless server survives restarts without losing anything.
- It keeps the display working on any browser or network.

**Positive consequences:**
- New, removed and withdrawn photos all use the same path.
- If the server restarts, the display keeps playing what it has and recovers on the next poll, without a manual refresh (FR-DSP-08).
- Easy to test with ordinary requests.

**Negative consequences:**
- **"Within 10 seconds" does not mean "on screen within 10 seconds."** With the no-interrupt rule (FR-AUT-03) and a 10-second slide, a new photo **enters the queue** within about the poll interval but appears on screen only after the current slide, which can be roughly 15 seconds or more in the worst case. This reading is in ARCHITECTURE finding 1 and is **not yet confirmed** in the PRD.
- A removed or withdrawn photo can stay on screen until its current showing ends.
- Each open display makes a request every 5 seconds, even when nothing changes.
- Several displays are not synchronised (A-24).
- Reload of the display page loses the waiting line (accepted in A-21).

**Future reconsideration conditions:** Move to Server-Sent Events or WebSockets (M-3) if updates must be instant, if many displays are expected, or if the poll traffic becomes a measured problem. Revisit the 5-second value if the PRD target is changed to mean "on screen" within a fixed time.

---

### ADR-008: Layered Backend Architecture

**ADR ID:** ADR-008

**Decision title:** Organise the backend as simple layers: routes, services, repositories, with models, schemas, storage and an image processor.

**Status:** **Accepted in part.**
- **Accepted:** a simple layered backend, and MVC not forced onto it (you asked for this).
- **Proposed:** repositories as **thin modules of functions** with a review trigger after Day 2 (ARCHITECTURE AD-03), and services deciding commits (AD-04).

**Context:** The backend must validate uploads, apply consent and ownership rules, run queries, write files and return a standard error format. The developer must explain each part, and tests should be possible without a web server.

**Problem:** How should the code be divided so that web handling, business rules, queries and file work do not end up in one function, without adding more structure than a beginner can follow?

**Options considered:**

| Option | For | Against | Verdict |
|---|---|---|---|
| **Simple layers (routes, services, repositories)** | One job per layer. Rules testable without the web. Easy to explain in a diagram. | More files. A little indirection. Needs discipline, because no tool enforces it. | **Chosen** |
| Everything in the route functions | Fewest files | One function mixes request, rules, queries and files. Hard to test and explain. | Not chosen |
| MVC | Familiar name | The server returns no views, so "view" has no meaning. It would create empty folders. | Not chosen |
| Hexagonal or clean architecture | Very flexible | Too much ceremony for about ten endpoints | Not chosen |
| Full repository pattern with interfaces | Easy to swap the database | Extra classes and interfaces the project does not need | Not chosen |
| No repository layer (queries in services) | Fewer files | Queries scattered across services | Not chosen |

**Decision:** Dependencies point **down only**. Routes call services. Services call repositories, storage and the image processor. Repositories run queries and never commit. Services never import FastAPI and raise domain errors, which error handlers turn into the standard JSON. Services and repositories are **modules of functions**, not classes. The full rules are BACKEND_DESIGN 3.3.

**Reasons:**
- A route that only reads the request, calls one service and returns the result is easy to explain.
- Business rules ("no photo without consent", "check ownership") live in one place, away from HTTP.
- Tests can call a service or repository directly.

**Positive consequences:**
- Replacing the database touches repositories. Replacing file storage touches one module. Changing the update method touches one route and one hook (ARCHITECTURE 16).
- Clear places for transaction and error rules.
- The structure gives a clean demo story: request in, one layer at a time, response out.

**Negative consequences:**
- About **30 source files** for a small product (BACKEND_DESIGN 5.1).
- **Nothing checks the layer rules automatically.** A reviewer, or the checklist in CODING_STANDARDS, has to do it. AI assistants may put queries in routes unless reminded.
- For very small features, three files may feel like too much.

**Future reconsideration conditions:** At the **review point after Day 2**, merge a layer file into its neighbour if it contains only a few lines (the likeliest are consent into visitor for services and repositories, which break no approved decision). Introduce full repository classes or interfaces only if a second database implementation is actually needed.

---

### ADR-009: Separate Visitor and TV Display Interfaces

**ADR ID:** ADR-009

**Decision title:** Provide separate pages for visitors (`/`) and the TV display (`/display`), served by one application.

**Status:** **Proposed.** Written in ARCHITECTURE 2.2 and FRONTEND_DESIGN 1.2 (drafts). The optional operator page (`/admin`) has its own scope question and is **not** decided here.

**Context:** Two very different users use the system. A visitor on a phone fills in a form, ticks consent and chooses photos. A TV shows photos full screen, with no buttons, from a distance, possibly for hours. The assignment speaks of "a smart TV or computer display" as the place photos play (F-03).

**Problem:** Should the upload form and the slideshow be one page with modes, or separate pages, and how should they be served?

**Options considered:**

| Option | For | Against | Verdict |
|---|---|---|---|
| **Separate pages at `/` and `/display`, one app** | Each page has one job, its own layout and its own theme (light for phones, dark and large for the TV). A TV never shows a form. A phone never loads slideshow code it does not need to run. | Two entry addresses to route on the server. Some shared code. | **Proposed** |
| One page that switches between upload and display modes | One address | Mixed concerns. Risk of showing controls on the TV. Harder to keep the TV page clean. | Not chosen |
| Two separate projects (two repositories or hosts) | Strong separation | Needs cross-origin setup (CORS), two deployments, two addresses for phones. Far more than needed. | Not chosen (future option M-11) |
| A native or TV app | Best TV experience | Out of scope and not in your stack | Not chosen |

**Decision (proposed):** One React application with a tiny page switch in `App` (no routing library). `/` is the visitor upload page. `/display` is the TV page. FastAPI serves the same built entry file for these two addresses only, and answers "not found" for other unknown addresses. All data calls go through `/api`.

**Reasons:**
- The two audiences need different layouts, sizes and behaviour (FRONTEND_DESIGN 1.2).
- The display must be free of controls and reachable by a plain address for the TV browser.
- One origin means no CORS set-up and one address to give to phones (ARCHITECTURE AD-16).

**Positive consequences:**
- The display can be tested and tuned on its own, including its manual steps (A-26).
- Visitor code never loads on the TV and the reverse.
- Simple to explain: "this address is for phones, that address is for the screen".

**Negative consequences:**
- **The display page is public.** Anyone who knows the address can open it. It shows only photos of consenting visitors, and no names, tokens or visitor numbers (SECURITY_PRIVACY).
- The server must be taught the two page addresses (AD-17), which is a small extra rule to test.
- The TV browser may need one manual step to enter full screen (A-26 is OPEN, and PRD FR-DSP-01 says "no interaction", which conflicts and is already flagged).
- Some code (the API client, the design tokens) is shared between pages and must stay neutral.

**Future reconsideration conditions:** Reconsider if more pages or roles appear (add a router, ARCHITECTURE M-8), if the display must be hosted separately from the API (Option B, M-11, with CORS), or if the real TV cannot run the display page and a different approach is needed (the Day 1 check S4).

---

### ADR-010: Consent-Based Public Photo Display

**ADR ID:** ADR-010

**Decision title:** A photo is stored and shown publicly only after the uploader gives explicit, recorded consent, which the server enforces.

**Status:** **Accepted in part.**
- **Accepted:** consent is required, because the assignment says photos of people are displayed publicly and privacy and consent must be addressed (F-07), and consent and withdrawal are in the scope you listed.
- **Proposed:** the mechanism and wording (ASSUMPTIONS A-07, A-08, "ASSUMED + CONFIRM"): an unticked checkbox, a versioned notice, once per registration, and the **withdrawal** feature (A-09). Your confirmation of the once-per-registration rule and the final notice wording is recommended.

**Context:** Photos of people appear on a screen that others can see. The assignment leaves the mechanism to the team. There is no lawyer review available (the notice is not legal advice, PRD OOS-15), no age verification (OOS-13), and no way to ask the other people in a photo.

**Problem:** How can the system show visitors' photos publicly in an honest, testable and respectful way, with a real way to stop?

**Options considered:**

| Option | For | Against | Verdict |
|---|---|---|---|
| **Unticked checkbox plus plain notice, once per registration, enforced by the server, recorded with notice version and time** | Explicit, simple, testable, cannot be bypassed from the browser | Self-declared by the uploader. Covers later uploads too. | **Proposed** |
| Tick on every upload | Stronger | Annoying for several photos | Not chosen |
| Pressing "Upload" counts as consent | Fewest steps | Not explicit | Rejected |
| Pre-ticked box | Fewest clicks | Not valid, explicit consent | Rejected |
| A separate terms page | Complete text | Visitors skip it | Not chosen |
| A moderator approves each photo before display | Highest control | Needs staffed moderation and an approval workflow. Out of scope (M-10). | Not chosen |
| No consent step | No work | Does not meet the assignment | Rejected |

**Decision (proposed mechanism):** A new visitor must tick an unticked checkbox beside a plain-English notice (version `v1`). The server refuses any upload without consent and records who consented, when, and which notice version. Returning visitors see a reminder and a withdrawal link. **Withdrawal** is one all-or-nothing change: the name and token hash are cleared, the photos are marked removed, the consent record is closed, and the files are deleted after the commit. Photos are shown only for active visitors with active consent.

**Reasons:**
- Explicit consent at the point of upload is the honest way to meet the assignment.
- Checking on the server means a modified browser cannot skip it.
- The notice **promises** that photos can be removed. A promise with no feature behind it would be dishonest, so withdrawal must be built or the promise removed (IMPLEMENTATION_PLAN P-2).

**Positive consequences:**
- The system can show **who agreed to which notice, and when**, without keeping personal data beyond the visitor id, time and version.
- Withdrawal is simple to explain and to test (files really gone, name really cleared).
- The display only ever receives photos that passed this rule.

**Negative consequences:**
- **Consent is self-declared** by the uploader. It is not consent from other people in the photo, and the system cannot check it (SECURITY_PRIVACY F-10).
- **The notice has not had legal review.** A child could still tick the box. This design makes **no legal compliance claim**.
- **Withdrawal only works from the browser that holds the token** (ADR-006), and it cannot undo what people have already seen.
- A one-year cache on photo images could let a browser keep showing a withdrawn photo, so SECURITY_PRIVACY proposes a short, private cache (F-2, SP-2). This is not yet confirmed.
- No retention period is decided (A-11 is OPEN and **required before any public deployment**).

**Future reconsideration conditions:**
- Before any public deployment: decide the retention period (A-11), get the notice wording reviewed, and consider tick-on-every-upload or moderation (M-10).
- If any complaint shows that the self-declared consent is not enough.
- If the notice text changes: bump its version (the server then asks visitors to agree again).

---

## 4. Other Decisions Not Given a Full ADR Here

These are recorded with an ID in ARCHITECTURE.md (17) or TECH_STACK.md (10). Each is also still **Proposed** unless noted. Write a full ADR for one if it becomes controversial, or if you must defend it in a demo.

| Source ID | Decision | Probably the next ADRs |
|---|---|---|
| AD-06 | One combined endpoint for registration and upload | Candidate |
| AD-08, AD-13, AD-14 | One commit per photo; files before row on upload, row before files on removal | Candidate (consistency rules) |
| AD-09, AD-10, AD-11 | Stateless feed with a cursor, the queue in the browser, photo id as the cursor | Covered partly by ADR-007 |
| AD-16 | One origin: FastAPI serves the built frontend | Covered partly by ADR-009 |
| AD-19, AD-23, AD-24 | Request size guard, image-processing gate (default 4), pixel limit checked from the header | Candidate (upload safety) |
| AD-20 | One error shape with stable codes | Candidate |
| AD-22 | Removed photos keep a row without personal data | Candidate |
| T-08 | pytest and TestClient for automated tests, manual plan for display and phones | See TESTING_PLAN |
| A-30 | Deployment is local first. **Still OPEN.** | Needs your decision before an ADR |

---

## 5. Using and Changing This Document

### 5.1 How to Change a Status

| Situation | What to do |
|---|---|
| You confirm a Proposed decision | Change its status to **Accepted**, add `Confirmed by: <you>, <date>` under it, and leave the rest unchanged. |
| You change your mind | **Do not edit the old ADR.** Add a new ADR with the new decision. Change the old one to **Superseded by ADR-0nn**. The old reasons stay as history. |
| A trigger in "Future reconsideration conditions" happens | Write a new ADR (or update the status) and say which trigger fired. |
| A design document and an ADR disagree | Say so and stop. Do not pick silently. The ADR records the decision. The design document records the detail. |

### 5.2 Rules for Writing a New ADR

1. Number it in order (ADR-011 next). Never reuse a number.
2. Use all eleven parts, in the order in Section 2.
3. Say what is **Accepted** and what is only **Proposed**, and why.
4. List at least two real options, including the one rejected.
5. Write at least one **negative** consequence. A decision with none has not been thought through.
6. Make the reconsideration conditions **specific and observable** ("busy errors appear in normal use"), not vague ("if needed").
7. Do not claim something works if it was not run. Write "to verify" and say where it is checked.

### 5.3 Likely Review Questions (Practise Your Answers)

| Question | Short answer |
|---|---|
| Why not just identify visitors by name? | Names repeat and can be typed by anyone. One "Priya" would see another's photos. A random token proves "this browser registered this visitor" without extra personal data (ADR-006). |
| What happens if I switch phones? | You are treated as a new visitor (same browser only). A recovery code is the future fix (ADR-006). |
| Why SQLite when it allows only one writer? | The slow work is image processing, which happens before a short write. Ten simultaneous uploads are a test target (ADR-003). |
| Why polling and not WebSockets? | Simplest method that meets the target, with no state on the server. The cost is up to one interval of delay (ADR-007). |
| Why does the TV have its own page? | A TV needs no controls and a different layout. A phone needs a form. Mixing them risks showing a form on the TV (ADR-009). |
| Is the consent legally valid? | This is not legal advice and makes no compliance claim. It is explicit, recorded and enforced by the server, but self-declared (ADR-010). |

---

# Glossary

| Term | Meaning |
|---|---|
| **ADR** | Architecture Decision Record: a short note on one major choice |
| **ORM** | Object-Relational Mapper: code that lets you work with database rows as objects |
| **WAL mode** | A SQLite setting that lets readers keep working while one writer writes |
| **Polling** | The display asking the server "anything new?" at regular intervals |
| **Cursor** | The highest photo id the display has already seen |
| **Token** | A secret random value the browser keeps to show "this is me" |
| **Token hash** | A one-way scrambled form of the token, which is all the server stores |
| **Consent record** | A row saying who agreed to which notice, and when |
| **Withdrawal** | The visitor asking for all their photos and their name to be removed |
| **CORS** | Browser rules for requests between different web addresses |
| **Thin repository** | A small module of query functions, one per table, with no commit and no business rules |

---

# Document Control

| Item | Detail |
|---|---|
| Document | ADR.md |
| Version | 1.0 (draft for review) |
| Date | 17 November 2026 |
| Status of this document | Draft. The Accepted decisions are those you stated yourself. All others are Proposed until you confirm them. |
| Open items that affect an ADR | A-03 (same-browser recognition, ADR-006), A-11 (retention, ADR-010), A-26 (TV browser, ADR-009), A-30 (deployment, ADR-005), PRD FR-AUT-01 wording (ADR-007) |
| Next review | After Day 2 (backend structure review point), and when any open item above is settled |
