# TECH_STACK.md

# Visitor Photo Slideshow (MomentVault): Technology Stack Evaluation

| Item | Detail |
|---|---|
| Product | MomentVault, implementing Assignment 1: Visitor Photo Slideshow |
| Document version | 1.0 (draft for approval) |
| Date | 14 November 2026 |
| Based on | Original assignment, PROJECT_BRIEF.md, PRD.md, ASSUMPTIONS.md |
| Purpose | Evaluate the proposed stack, check it against a 20-hour MVP, and record the choice with reasons (feeds ARCHITECTURE.md) |

---

## How to Read This Document

- **Selected stack** (Parts 2 to 5) is what the MVP will use. Part 8 holds **future migration options**, which are kept separate on purpose and are **not** part of the MVP.
- **Verified facts** come from sources checked on **14 November 2026** (listed in Part 11). Hosting prices, free allowances and limits change often, so **re-check the source page on the day you decide**. Where sources disagreed, this document says so.
- **Estimates** (hours, sizes) are my judgement, not measurements. They are labelled "estimate".
- This document does **not** claim that any tool or hosting plan is free. It records what the sources say and what must still be checked. The selected tools are open-source projects, but I have not re-verified their licence texts in this session.
- No implementation code is included.

---

# 1. Executive Summary

**Verdict: the proposed stack is appropriate for a 20-hour MVP, with four conditions.**

| # | Condition | Why |
|---|---|---|
| 1 | Keep the build to the **Must Have** requirements first | React is the biggest learning load for a beginner. The Should Have items (withdrawal, deletion, EXIF removal, thumbnails) probably do not all fit in 20 hours (see Part 6). |
| 2 | Use **plain synchronous FastAPI endpoints and synchronous SQLAlchemy**, with SQLite in WAL mode and short transactions | Keeps concurrency understandable and sufficient for about 10 simultaneous uploads. |
| 3 | Treat **local disk as the only photo storage** and keep the database and photos together in **one data folder** | Hosting platforms that erase local files on restart would destroy both. Verified for Render's free service (Part 5.3). |
| 4 | Demo **locally** first; deploy only if a host with persistent storage is confirmed | Matches assumption A-30 (OPEN). |

**Three things the proposed list leaves out.** FastAPI needs a server to run it (**Uvicorn**), a package for reading uploaded files and forms (**python-multipart**), and **httpx** for its test client. They are required supporting packages, not extra features (Part 3.2). Pydantic arrives automatically with FastAPI.

**Biggest risks:** (1) React timers and state in the display page, (2) SQLite behaviour under simultaneous uploads, (3) losing photos on a host with temporary storage, (4) total time. Mitigations are in Part 6.

---

# 2. Selected Stack at a Glance

| Layer | Technology | Role | Essential or optional |
|---|---|---|---|
| Backend language | Python | All server code, tests | Essential |
| Backend framework | FastAPI | API routes, input validation, file upload handling | Essential |
| Data validation | Pydantic (comes with FastAPI) | Describes and checks request and response data | Essential |
| ORM | SQLAlchemy | Python classes mapped to database tables | Essential |
| Database | SQLite | One-file database | Essential |
| Image processing | Pillow | Verify, rotate, resize, re-encode, thumbnails | Essential |
| Frontend library | React | Upload page and display page | Essential |
| Frontend language | JavaScript | Frontend code | Essential |
| Frontend tooling | Vite | Dev server and production build | Essential |
| Styling | CSS (plain) | Layout and appearance | Essential |
| Frontend packages | npm | Installs frontend tools | Essential |
| Backend testing | Pytest + FastAPI TestClient | Automated API tests | Essential |
| Frontend testing | React testing tools | UI tests | **Optional, not in MVP** (Part 4.16) |
| Version control | Git, GitHub | History and public repository | Essential |
| Python isolation | Virtual environment (venv) | Keeps project packages separate | Essential |

## Principles used for choosing

1. Prefer the simplest tool that meets the PRD (the engineering rules ask for simple, maintainable, beginner-friendly choices).
2. Avoid extra libraries. Every dependency must be justified.
3. The developer must be able to explain each part in a live demo.
4. Do not silently change anything approved in PRD.md or ASSUMPTIONS.md.

---

# 3. Dependency Overview

## 3.1 Selected technologies

Evaluated in detail in Part 4.

## 3.2 Required supporting packages (not in the proposed list)

| Package | Purpose | Where used | Beginner difficulty | Limitation | Alternatives | Essential? |
|---|---|---|---|---|---|---|
| **Uvicorn** | The program that actually runs the FastAPI app and listens for requests | Starting the backend | Low | Use **one worker process** with SQLite (Part 5.1) | Hypercorn, Gunicorn with Uvicorn workers (not needed) | Essential |
| **python-multipart** | Lets FastAPI read forms and file uploads | Upload route | Low (install and forget) | Easy to forget, and uploads then fail with an unclear error | None needed | Essential |
| **httpx** | HTTP client library that FastAPI's TestClient is built on | Tests | Low | Test errors appear if missing | Requests (older route, not recommended) | Essential for automated tests |
| **pydantic-settings** or **python-dotenv** | Read settings (limits, paths, secrets) from a `.env` file | Configuration | Low | One more dependency | Plain environment variables with no package | **Optional** |

Notes:
- CORS support is built into FastAPI, so it needs no extra package.
- The testing tools' need for httpx comes from search results about FastAPI's TestClient. **Confirm the exact install list in FastAPI's current documentation at setup time**, and pin all versions in `requirements.txt`.

## 3.3 Tools you need installed

| Tool | Requirement | Source |
|---|---|---|
| Python | A version supported by your FastAPI, SQLAlchemy and Pillow versions (check at setup) | To verify at setup |
| Node.js | Vite's current guide says **Node 20.19+ or 22.12+** | Vite docs, 14 Nov 2026 |
| npm | Comes with Node.js | n/a |
| Git | Any current version | n/a |
| A GitHub account | The assignment requires a public repository | Assignment |

---

# 4. Technology Evaluations

Rating scale for **beginner difficulty**: Low, Medium or High. Learning times are **estimates** for someone with basic Python knowledge.

## 4.1 Python

| Question | Answer |
|---|---|
| **Purpose** | Language for the whole backend and the tests. |
| **Why suitable** | Readable, widely taught, and every other backend tool here (FastAPI, SQLAlchemy, Pillow, Pytest) is a Python library. Good AI-assistant support. |
| **Where used** | API, validation, storage, image processing, database, tests. |
| **Beginner difficulty** | Low to Medium. Type hints (used heavily by FastAPI) are the main new idea. |
| **Advantages** | One language across all backend work. Large community. |
| **Limitations** | Slower than compiled languages (irrelevant at this scale). CPU-heavy image work can queue under load. Needs a virtual environment. |
| **Alternatives** | Node.js with Express (one language for both sides), Java with Spring (heavier). |
| **Essential or optional** | Essential. |

## 4.2 FastAPI

| Question | Answer |
|---|---|
| **Purpose** | Web framework that turns Python functions into API routes. |
| **Why suitable** | Built-in input validation through Pydantic, support for file uploads, an automatic interactive API page (useful for manual testing while learning), and a ready TestClient. |
| **Where used** | All backend routes: registration, upload, photo list, photo serving, deletion, withdrawal. |
| **Beginner difficulty** | Medium. New ideas: route decorators, dependency injection, and the difference between ordinary and `async` functions. |
| **Advantages** | Less code for validation, clear error responses, built-in CORS support, good documentation. |
| **Limitations** | Needs Uvicorn and python-multipart. It does not by itself enforce the PRD's per-file size limit, so that check must be written and tested (FR-VAL-03). Many online examples use `async` style that is harder to combine with Pillow and sync SQLAlchemy. |
| **Alternatives** | Flask (simpler, more tutorials, less built in), Django (everything included, heavier and more concepts). |
| **Essential or optional** | Essential. |

**Decision:** use ordinary (`def`) endpoints, not `async def`. Pillow and sync SQLAlchemy block while working, and FastAPI runs ordinary endpoints in a worker thread pool, so blocking work does not freeze the server. This is simpler for a beginner.

## 4.3 Pydantic

| Question | Answer |
|---|---|
| **Purpose** | Describes the shape of data (for example, a name of 1 to 50 characters) and checks it. |
| **Why suitable** | Installed automatically with FastAPI. Matches PRD FR-REG-02 and the configurable limits in ASSUMPTIONS.md. |
| **Where used** | Request and response models, and (optionally) settings. |
| **Beginner difficulty** | Low. |
| **Advantages** | Clear, automatic error messages. Shows up in the API documentation page. |
| **Limitations** | It checks fields, not file contents. Pillow does that. Older tutorials use an earlier major version with different syntax, so copy only current examples. |
| **Alternatives** | Manual `if` checks, dataclasses, marshmallow. |
| **Essential or optional** | Essential (arrives with FastAPI). |

## 4.4 SQLAlchemy

| Question | Answer |
|---|---|
| **Purpose** | Maps Python classes (Visitor, Photo, Consent record) to database tables, so you write Python instead of raw SQL. |
| **Why suitable** | Queries are built safely (no hand-assembled SQL, so SQL injection risk is lower). The same code can later move to PostgreSQL (Part 8). |
| **Where used** | Models, database sessions, queries for visitors, photos and consent. |
| **Beginner difficulty** | **Medium to High.** It is likely the hardest backend tool: sessions, relationships, and the difference between old-style and current-style code in tutorials. |
| **Advantages** | Safe queries, portable, well documented, works well with FastAPI. |
| **Limitations** | Many tutorials use an older style, causing confusion. A session must not be shared between requests (important for concurrency). |
| **Alternatives** | Python's built-in `sqlite3` with plain SQL (fewer concepts, more manual code), SQLModel (adds another layer over SQLAlchemy and Pydantic), Peewee. |
| **Essential or optional** | Essential (it is in the proposed stack). |

**Decisions to carry into ARCHITECTURE.md:**
- Use **synchronous** SQLAlchemy with **one session per request**.
- Create tables when the app starts. **Do not add Alembic** (a migration tool) in the MVP.
- SQLite does not enforce foreign key links unless that is turned on for each connection, so this must be configured and tested.

## 4.5 SQLite

| Question | Answer |
|---|---|
| **Purpose** | Stores visitors, consent records and photo information in a single file. |
| **Why suitable** | No server to install or run. Ships with Python. Tests can use a temporary database. Enough for a small event. |
| **Where used** | All persistent data except the image files themselves. |
| **Beginner difficulty** | Low. |
| **Advantages** | Zero setup, easy to copy and inspect, easy to reset for demos. |
| **Limitations** | **Only one writer at a time**, even in WAL mode. WAL mode does not work over a network filesystem and needs all users of the database on the same computer. The file must live on storage that survives restarts. See Part 5.1. |
| **Alternatives** | PostgreSQL or MySQL (need a database server). |
| **Essential or optional** | Essential. |

## 4.6 Pillow

| Question | Answer |
|---|---|
| **Purpose** | Opens images, proves they are real, applies rotation, removes hidden metadata, scales, re-saves and makes thumbnails. |
| **Why suitable** | It is the standard Python image library and is needed for PRD FR-VAL-02, FR-VAL-05 and FR-STO-03 (the checks and processing described in ASSUMPTIONS A-12 and A-15). |
| **Where used** | The upload route, after the size check and before the database write. |
| **Beginner difficulty** | Medium. Pitfalls: a quick "verify" does not fully decode the image, the default pixel safety limit must be configured to match the PRD, and rotation information must be applied **before** metadata is removed or photos appear sideways. |
| **Advantages** | One library covers validation, cleaning and resizing. |
| **Limitations** | Does not read HEIC photos without an extra package (assumption A-12 rejects HEIC). Large images use memory and CPU. |
| **Alternatives** | ImageMagick/Wand (needs a system program), OpenCV (heavy), sharp (Node.js). |
| **Essential or optional** | Essential. |

## 4.7 React

| Question | Answer |
|---|---|
| **Purpose** | Builds the user interface from small reusable pieces (components) that update when data changes. |
| **Why suitable** | Matches the required stack, is popular in portfolios and has a huge community. Both pages (upload and display) have changing state, which React handles well. |
| **Where used** | Upload page (form, consent, results, "My photos") and display page (slideshow, queue, polling). |
| **Beginner difficulty** | **Medium to High.** Components, props, state and hooks are all new. The slideshow timer is the hardest part. |
| **Advantages** | AI assistants know it well. Clear structure. Many examples. |
| **Limitations** | More concepts than plain HTML and JavaScript. Timers and polling need care: stale values inside timers and cleanup of timers are common bugs. In development mode, React can run some code twice on purpose, which can confuse beginners. |
| **Alternatives** | Plain HTML plus JavaScript served by FastAPI (least to learn, honestly enough for two pages), Vue, Svelte, server templates with HTMX. |
| **Essential or optional** | Essential (selected). |

**Honest note:** for two pages, plain JavaScript would be quicker to build. React is kept because it is in the approved stack and is a stronger portfolio item. The main cost is learning time, accounted for in Part 6.

## 4.8 JavaScript

| Question | Answer |
|---|---|
| **Purpose** | The language of the frontend. |
| **Why suitable** | Runs in every browser. TypeScript is not requested, and leaving it out avoids another learning layer. |
| **Where used** | All React code and the small storage and request helpers. |
| **Beginner difficulty** | Medium. Asynchronous code (promises and `fetch`) is the main hurdle. |
| **Advantages** | No compile step beyond Vite. |
| **Limitations** | No type checking, so mistakes appear when the page runs. |
| **Alternatives** | TypeScript (safer, but more to learn). |
| **Essential or optional** | Essential. |

**Decision:** use the browser's built-in `fetch` for requests. Do not add Axios or similar.

## 4.9 Vite

| Question | Answer |
|---|---|
| **Purpose** | Creates the frontend project, runs a fast development server and builds the production files. |
| **Why suitable** | Official React template, minimal setup. Its development server can forward `/api` requests to the backend, which solves cross-origin problems during development (Part 5.4). |
| **Where used** | Development and the production build. |
| **Beginner difficulty** | Low. |
| **Advantages** | Fast startup, small configuration. Supports separate entry pages if needed. |
| **Limitations** | The development server is for development only. Needs a recent Node.js (20.19+ or 22.12+ per its current guide). |
| **Alternatives** | Next.js (a server framework, unnecessary here), Parcel, older bundlers. |
| **Essential or optional** | Essential. |

## 4.10 CSS

| Question | Answer |
|---|---|
| **Purpose** | Styles both pages. |
| **Why suitable** | No dependency, and the needs are modest: a responsive phone layout and a full-screen, dark display with photos fitted to the screen. |
| **Where used** | Upload page (phone width) and display page (1920 x 1080). |
| **Beginner difficulty** | Low to Medium (flexible layouts and fitting images without cropping). |
| **Advantages** | Nothing to install or learn beyond CSS itself. |
| **Limitations** | More manual work than a UI kit. |
| **Alternatives** | Tailwind CSS or Bootstrap (extra setup and dependencies). Not selected. |
| **Essential or optional** | Essential (plain CSS). Frameworks are optional and not selected. |

## 4.11 npm

| Question | Answer |
|---|---|
| **Purpose** | Installs and runs the frontend tools and libraries. |
| **Why suitable** | Comes with Node.js and is the default for the Vite React template. |
| **Where used** | Frontend setup and the `dev` and `build` commands. |
| **Beginner difficulty** | Low. |
| **Advantages** | Standard and well documented. |
| **Limitations** | Installs many small packages, so keep dependencies few and commit the lock file. |
| **Alternatives** | pnpm, Yarn. |
| **Essential or optional** | Essential. |

## 4.12 Pytest

| Question | Answer |
|---|---|
| **Purpose** | Runs the automated backend tests. |
| **Why suitable** | Plain `assert` statements and fixtures that create a temporary database and a temporary photo folder. Satisfies "automated tests" in the assignment. |
| **Where used** | Tests for validation, limits, tokens, consent, deletion, withdrawal and concurrency. |
| **Beginner difficulty** | Low to Medium. Fixtures are the new idea. |
| **Advantages** | Short tests, clear failure messages. |
| **Limitations** | Concurrency tests need threads and take care to be reliable. |
| **Alternatives** | `unittest` (built in, more verbose). |
| **Essential or optional** | Essential. |

## 4.13 FastAPI TestClient

| Question | Answer |
|---|---|
| **Purpose** | Calls your API from a test without starting a real server. |
| **Why suitable** | Supports file uploads and headers, so you can test PRD requirements end to end through the API. |
| **Where used** | Backend tests. |
| **Beginner difficulty** | Low to Medium. |
| **Advantages** | Fast, no ports, and simple to read. |
| **Limitations** | Runs inside the same process, so it does not prove behaviour with several real server processes or real network delays. It depends on httpx (Part 3.2). |
| **Alternatives** | httpx's async client, tests against a running server, browser-based end-to-end tools (not selected). |
| **Essential or optional** | Essential for the automated tests. The assignment would also accept a documented manual test plan. |

## 4.14 Git

| Question | Answer |
|---|---|
| **Purpose** | Records the project history. |
| **Why suitable** | The assignment requires regular, meaningful commits, and a single final commit loses marks. |
| **Where used** | The whole project. |
| **Beginner difficulty** | Medium (commits and basic branches are enough). |
| **Advantages** | Safe experimentation and visible progress. |
| **Limitations** | Easy to commit unwanted files. Use `.gitignore` for the database file, the photo folder, `node_modules`, virtual environments and `.env`. |
| **Alternatives** | None practical for this assignment. |
| **Essential or optional** | Essential. |

## 4.15 GitHub

| Question | Answer |
|---|---|
| **Purpose** | Hosts the repository for submission. |
| **Why suitable** | The assignment asks for a **public** repository accessible without login. |
| **Where used** | Remote copy, submission link, and optionally a place to keep notes. |
| **Beginner difficulty** | Low. |
| **Advantages** | Visible history and portfolio value. |
| **Limitations** | Public means everything pushed is visible, including mistakes. Never push secrets. Remove them from history if pushed by accident, and change the secret. |
| **Alternatives** | GitLab or Bitbucket (the assignment names GitHub). |
| **Essential or optional** | Essential. |

## 4.16 React testing tools (optional)

The usual tools are Vitest with React Testing Library. I have not verified their current setup steps in this session.

| Question | Answer |
|---|---|
| **Purpose** | Automated tests for React components and logic. |
| **Why suitable** | Could protect the slideshow queue logic from regressions. |
| **Where used** | Possibly one small test file for the queue logic. |
| **Beginner difficulty** | Medium. A new tool plus new concepts on top of React. |
| **Advantages** | Catches queue and ordering bugs automatically. |
| **Limitations** | Setup and learning time, extra dependencies, and the display's visual behaviour still needs manual testing. |
| **Alternatives** | A documented **manual test plan with recorded results** (explicitly allowed by the assignment). |
| **Essential or optional** | **Optional. Not included in the MVP.** |

**Justification test, to apply at the end of Day 3:** add React tests only if (a) all Must Have backend tests pass, (b) the display queue logic is written as a small standalone function, and (c) at least about 2 hours remain. Otherwise use the manual test plan. This is the stack decision the instruction allowed "only if justified by the available time".

## 4.17 Virtual environment (venv)

| Question | Answer |
|---|---|
| **Purpose** | Gives the project its own private set of Python packages. |
| **Why suitable** | Python includes it, so nothing extra is installed. |
| **Where used** | Backend development and test runs. |
| **Beginner difficulty** | Low to Medium. The activate command differs between operating systems, and forgetting to activate it is the most common mistake. |
| **Advantages** | Prevents version clashes, and makes `requirements.txt` accurate. |
| **Limitations** | Must be activated in each new terminal. Never commit the environment folder. |
| **Alternatives** | conda, Poetry, uv. Not needed. |
| **Essential or optional** | Essential. |

---

# 5. Key Technical Concerns

## 5.1 SQLite Write Concurrency

**Verified facts (SQLite documentation, 14 Nov 2026)**
- In WAL mode, readers do not block writers and a writer does not block readers.
- There is still **only one writer at a time**.
- WAL **does not work over a network filesystem**, and all processes using the database must be on the same computer.
- SQLite automatically merges the log back into the main file ("checkpoint") once the log reaches a size threshold, with no work needed from the application.

**What it means for 10 simultaneous uploads (PRD FR-CNC-01)**
Ten uploads do not need ten simultaneous writers. The database work per upload is a few small inserts. The slow parts are receiving the file and processing it with Pillow, and **neither needs the database lock**. So the design rule is: **finish all image work first, then open a short database write.**

**Design rules (to be fixed in ARCHITECTURE.md)**

| # | Rule | Reason |
|---|---|---|
| 1 | Turn on WAL mode once at startup | The display's reads should not wait for uploads' writes. |
| 2 | Set a busy wait (timeout) on connections | A second writer waits briefly instead of failing immediately. |
| 3 | One SQLAlchemy session per request, never shared | Shared sessions cause mixed-up data under concurrency. |
| 4 | Keep write transactions short, and do image processing outside them | Shortens lock time. |
| 5 | Use ordinary `def` endpoints | FastAPI runs them in separate threads. |
| 6 | Run **one** Uvicorn worker process | Multiple processes make SQLite locking harder to reason about. Not needed at this scale. |
| 7 | Keep the database file and the `-wal` and `-shm` files on **local** storage | WAL needs a local filesystem. |
| 8 | If the database is still busy after waiting, return a friendly "try again" message and leave no partial data | PRD FR-CNC-03 and ERR-15. |
| 9 | Make sure the database foreign keys are enabled | SQLite does not enforce them by default. |

**What this does and does not prove**
- It is sufficient for a small event (assumption A-08). A passing 10-upload test on a development machine shows the design is sound. It does **not** prove production behaviour.
- Under contention, expect uploads to become **slower**, not to fail. Measure the time in the test and record it.
- Backing up SQLite in WAL mode needs care. Copying only the main file while the app runs can miss recent data. For the MVP, stop the app before copying, and copy all three files.

**When to migrate:** see Part 8, item M-1.

## 5.2 Image Storage Persistence

**Design (matches assumption A-18)**
- Photos are stored as files in a folder on the server's local disk.
- The database stores only the file name and details.
- **Keep the database and the photo folder together under one data folder** (for example, `data/`), so one persistent location covers both and they cannot drift apart.
- Both the data folder and any `.env` file are excluded from Git.
- Delete means removing both the file and the record (PRD FR-DEL-03, FR-STO-04).

**Space estimate (estimate, not measured):** after scaling to 1920 pixels (assumption A-15), a stored photo is probably a few hundred kilobytes up to about 1 MB. At that size, 0.5 GB would hold on the order of 500 to 1,000 photos. **Measure real phone photos during testing** before relying on this.

**Persistence is a hosting question.** On a laptop the folder simply stays. On a host with temporary storage, both the photos and the SQLite file disappear. Details are in Part 5.3.

**Backups:** none in the MVP. A manual copy of the data folder is the simple backup, and it should be done with the app stopped.

## 5.3 Deployment Constraints

**Requirements this app places on any host**

| Need | Why |
|---|---|
| A long-running Python process | The FastAPI app runs continuously. |
| **Persistent local storage** for the database and photos | SQLite and local photos (A-18). |
| A single application instance | One writer, and a disk that can only attach to one instance. |
| HTTPS | Recommended by PRD NFR-PRV-03. |
| Phones can reach the address | Visitors use their own devices. |

**What the sources say (verified 14 Nov 2026; re-check before use)**

| Option | Does it keep photos and SQLite? | Evidence and notes |
|---|---|---|
| **Your own computer (local network demo)** | Yes, while the folder remains | Selected for the MVP (A-30 provisional default). Phones must use the computer's network address, not "localhost". |
| **Render, free web service** | **No** | Render's documentation says free web services have an ephemeral filesystem: uploaded images and local SQLite databases are lost on every redeploy, restart or spin-down. Free web services spin down after 15 minutes without traffic. They also do not support persistent disks. |
| **Render, paid web service with a persistent disk** | Yes, under the disk's mount path only | Render says only data under the mount path is kept. A disk-backed service cannot scale to multiple instances and does not get zero-downtime deploys. I did not verify paid prices in this session. |
| **PythonAnywhere** | Unclear | Its help page describes ASGI (the type FastAPI uses) as **experimental or beta**, with no static file mappings, a very limited management interface and pricing not finalised. Test before committing. I did not verify the free plan's other limits. |
| **Fly.io** | Possible with a volume | Volumes are billed. Sources disagree on whether any free allowance exists: one Fly.io page lists the earlier free allowance under discontinued plans, and a November 2026 article says new accounts get only a short trial. **Check the official pricing page.** |
| **Railway** | Possible with a volume | Railway's documentation says volumes on Free and Trial plans are 0.5 GB. Secondary sources describe the free plan as a time-limited trial with small credits. Details vary between sources. **Check the official pricing page.** |
| **Serverless platforms** | Not verified | Typically do not keep local files between requests. Treat as unsuitable unless a specific platform is verified. |

**Recommendation**
1. **Build and demonstrate locally.** This is the safe default and meets the assignment (a deployed link is optional).
2. Decide on deployment **only after the MVP works**.
3. If deploying, choose a host that offers a **persistent disk or volume**, confirm its current cost and limits on its official page, and run a restart test: upload a photo, restart the service, confirm the photo and the visitor are still there.
4. Do **not** put the live demo on a host with temporary storage. You would lose all photos and visitors at the next restart.

## 5.4 Frontend and Backend Separation

**Two programs, one product.** React (served by Vite during development) and FastAPI are separate programs that talk through an HTTP API.

| Aspect | During development | For the demo or deployment |
|---|---|---|
| Frontend | Vite dev server (a different address and port) | Production build (static files) |
| Backend | Uvicorn on its own port | Uvicorn |
| How they connect | Vite forwards `/api` requests to the backend (a "proxy"), so the browser sees one address | See options below |

**Production options**

| | **Option A: FastAPI serves the built frontend (recommended)** | **Option B: separate frontend hosting and API hosting** |
|---|---|---|
| Addresses | One address for everything | Two addresses |
| Cross-origin setup (CORS) | Not needed | Required and must be restricted (PRD NFR-SEC-05) |
| Phone access on a local network | One address to share | Two programs to expose |
| Persistence | One place to configure | Backend host still needs persistent storage |
| Complexity | Backend must serve static files and handle the `/display` page address | Two deployments to manage |
| Best for | The demo and a single-host deployment | Later growth |

**Recommendation:** Option A. In development, use the Vite proxy. This also removes cross-origin problems in development and keeps the choice between a browser cookie and browser storage for the visitor token open (assumption A-01), because the browser sees a single address.

**Notes for ARCHITECTURE.md**
- Put all API routes under a common prefix such as `/api`, separate from the pages.
- The two pages (upload and display) can be done without a routing library: either by reading the page address or by using two separate entry pages in Vite. **Recommend not adding React Router**, to avoid a dependency. This is to be decided in ARCHITECTURE.md.
- If the backend serves the display page address, it must return the same frontend entry file for that address.
- Phones cannot reach `localhost`. For the demo, run the backend so that it listens on the local network, open the computer's network address on the phone, and keep both on the same Wi-Fi. For this reason, the demo should use the **built frontend served by FastAPI**, which needs only one address. Vite's dev server also needs to be told to listen on the network if used.
- Both processes must allow only the intended addresses (CORS) if Option B is ever used.

---

# 6. Is the Stack Appropriate for 20 Hours?

## 6.1 Time budget (estimate)

| Area | Hours (estimate) | Notes |
|---|---|---|
| Environment, repository, project skeleton | 1.5 | Python venv, Node, Vite, Git, `.gitignore`, `.env.example` |
| Backend core: models, registration, token, consent, upload, validation, storage | 6 | Biggest block. Pillow and SQLAlchemy have the steepest learning curve. |
| Backend: photo list, serving, display updates | 1.5 | |
| Frontend: upload page | 3 | Form, consent, results |
| Frontend: display page | 3 | Slideshow, queue, polling, empty and error states |
| Automated tests (backend) and manual test plan | 2.5 | Including the 10-upload test |
| Documentation: README, AI_LOG, ARCHITECTURE | 2 | |
| Buffer | 0.5 | |
| **Total** | **20** | **Must Have scope only** |

**Reality check.** This budget leaves almost no room for **Should Have** items (withdrawal, deletion, EXIF removal and thumbnails, "My photos", "Not you?"). I estimate those add about 4 to 6 more hours. Planning documents (brief, PRD, assumptions, this file) also take time. If the 20 hours are strict, **plan to finish Must Have first, then add Should Have in this order**: deletion, withdrawal, EXIF removal, "My photos". Re-plan after Day 1 using your real speed.

## 6.2 Learning load

| Tool | Learning difficulty | Time pressure |
|---|---|---|
| React (hooks and timers) | Medium to High | **Highest** |
| SQLAlchemy | Medium to High | High |
| FastAPI | Medium | Medium |
| Pillow | Medium | Medium |
| Pytest, Vite, Git, npm, venv, CSS | Low to Medium | Low |

## 6.3 Risk register

| # | Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|---|
| R1 | Slideshow timer and state bugs in React | High | High | Keep the queue logic in one small function. Build a plain version first, add polling second. |
| R2 | Time overrun | High | High | Must Have first. Cut list below. |
| R3 | Photos lost on a host with temporary storage | Medium | High | Demo locally. Restart test before any deployment. |
| R4 | SQLite "database is busy" under load | Medium | Medium | Part 5.1 rules and the 10-upload test. |
| R5 | Sideways or huge photos from phones | Medium | Medium | Apply rotation before stripping metadata. Test with real phone photos. |
| R6 | Outdated tutorial code (older SQLAlchemy, Pydantic, React styles) | High | Medium | Use official docs. Pin versions. Ask the AI for current-style examples. |
| R7 | Not able to explain AI-written code in the demo | Medium | High | Read and comment every file. Keep code small. Log AI mistakes in AI_LOG.md. |
| R8 | Phones cannot reach the demo | Medium | High | Rehearse on the demo network. Use the built frontend and the computer's network address. |

**Cut list if time runs short (drop in this order):** optional React tests, Could Have items, thumbnails, EXIF removal, "My photos", "Not you?", withdrawal, deletion. Never cut the consent step, validation, size limits, tests or documentation.

## 6.4 Verdict

**Appropriate, with the conditions in Part 1.** The stack is mainstream, well documented and supported by AI assistants. Its main weakness is learning load, not capability. React and SQLAlchemy are heavier than the problem strictly needs, but both are in the approved stack and add portfolio value, so they are kept.

---

# 7. Deliberately Not Included

To follow "do not introduce unnecessary technologies", these are **not** in the MVP.

| Not included | Reason |
|---|---|
| React Router | Two pages can be handled without it. |
| Axios | Browser `fetch` is enough. |
| Redux or other state libraries | React's own state is enough. |
| TypeScript | Not requested. Adds learning load. |
| Tailwind, Bootstrap or other UI kits | Plain CSS is enough. |
| Alembic (migrations) | Tables are created at start. No schema changes after release. |
| Docker | Not needed for a local demo. Optional later. |
| Celery, Redis or task queues | Processing happens inside the request. |
| WebSockets | Polling is enough (A-25). |
| Authentication libraries (JWT, OAuth) | The visitor token is a simple random value (A-01). |
| Cloud storage or CDN | Local disk is enough (A-18). |
| Rate-limiting library | Could Have (FR-VAL-08), not in MVP. |
| Browser-testing tools such as Playwright | Manual test plan instead. |

---

# 8. Future Migration Options (not part of the MVP)

These are listed so limitations have a visible path forward. **None is selected.**

| ID | Migration | Consider when | What would change | Not in MVP because |
|---|---|---|---|---|
| M-1 | SQLite to PostgreSQL | Many simultaneous writers, more than one server instance, or a host without suitable local storage | Database connection setting, data types and locking behaviour. A database server is needed. SQLAlchemy keeps most code unchanged, but tests must be rerun. | Adds a server to install and run. |
| M-2 | Local disk to object storage (an S3-compatible service) | Several server instances, or a host without persistent disks | Storage code and serving of photos. The database would store object keys. | Needs a provider account, keys (secrets) and cost checks. |
| M-3 | Polling to Server-Sent Events or WebSockets | Instant updates or many displays | Display update path, and the server holds open connections. | More complex, and polling meets the 10-second target. |
| M-4 | Alembic migrations | After a first real release, when the database structure must change without losing data | Adds migration scripts. | Not needed before release. |
| M-5 | Docker | Repeatable deployment | Adds a Dockerfile and a container workflow. | Extra concepts for a beginner. |
| M-6 | Continuous integration with GitHub Actions | A team or frequent changes | Runs tests on every push. | Optional extra, not an assignment item. |
| M-7 | TypeScript | A larger frontend | Rename files and add types. | Learning load. |
| M-8 | React Router | More pages | Adds routing. | Two pages do not need it. |
| M-9 | Frontend tests with Vitest and React Testing Library | If not done earlier | See Part 4.16. | Time. |
| M-10 | Rate limiting and moderation tools | Public deployment | New middleware and an approval workflow. | Beyond the assignment. |
| M-11 | Separate frontend and API hosting (Option B) | Growth or a static host for the frontend | CORS, two deployments. | Single-origin is simpler. |
| M-12 | HEIC support | Many iPhone photos rejected | Extra image package. | Extra dependency (A-12). |

---

# 9. Consistency with Approved Documents

| # | Finding | Status |
|---|---|---|
| 1 | The proposed list omits Uvicorn, python-multipart and httpx, which are required. | **Flagged.** They are listed as essential supporting packages (Part 3.2). This is the only change to the stack as proposed. |
| 2 | Pydantic is listed as separate, but it is installed with FastAPI. | Consistent. |
| 3 | ASSUMPTIONS A-19 says SQLite should use WAL mode and a busy wait. | Consistent (Part 5.1). |
| 4 | ASSUMPTIONS A-25 chooses polling. | Consistent. No extra technology is needed. |
| 5 | ASSUMPTIONS A-18 chooses local-folder storage. | Consistent. This document adds the "one data folder" rule. |
| 6 | ASSUMPTIONS A-30 (deployment) is OPEN. | This document supplies verified facts. The recommendation stays local. The question remains open. |
| 7 | ASSUMPTIONS A-01 left the choice between browser storage and a cookie to ARCHITECTURE.md. | The dev proxy and Option A (single address) make a cookie practical. **Input for the decision, not a decision.** |
| 8 | PRD FR-VAL-03 (size limit) cannot rely on FastAPI alone. | Noted. The size check must be written and tested. |
| 9 | PRD NFR-COMP-02 (Chrome or Edge at 1920 x 1080) and assumption A-26 (TV browser OPEN). | Unaffected. The stack produces a standard web page. |
| 10 | PRD FR-STO-03 and ASSUMPTIONS A-15 (rotation, scaling, thumbnails) rely on Pillow. | Consistent. Confirms Pillow is essential. |

No conflicts require changing PRD.md or ASSUMPTIONS.md because of this document.

---

# 10. Decisions Summary

| # | Decision | Status |
|---|---|---|
| T-01 | Use the proposed stack, plus Uvicorn, python-multipart and httpx | Proposed for approval |
| T-02 | Synchronous endpoints and synchronous SQLAlchemy | Proposed |
| T-03 | SQLite in WAL mode, with a busy wait, short transactions and one Uvicorn worker | Proposed |
| T-04 | Database file and photos in one local data folder, excluded from Git | Proposed |
| T-05 | Plain CSS, built-in `fetch`, no routing library, no state library | Proposed |
| T-06 | Vite dev proxy in development, FastAPI serving the built frontend for the demo (Option A) | Proposed |
| T-07 | Local demo first. Deploy only to a host with a persistent disk, after a restart test | Proposed (A-30 stays OPEN) |
| T-08 | Pytest and TestClient for automated backend tests. Manual plan for display and phone tests. React tests optional | Proposed |
| T-09 | No tool in Part 7 is added to the MVP | Proposed |

---

# 11. Sources Checked (14 November 2026)

| Topic | Source |
|---|---|
| Render free services: ephemeral filesystem, spin-down, no persistent disks | https://render.com/docs/free |
| Render persistent disks: mount path, single instance, no zero-downtime deploys | https://render.com/docs/disks |
| PythonAnywhere ASGI (beta) limitations | https://scriptagc.wasmer.app/https_help_pythonanywhere_com/pages/ASGIAPI (mirror of PythonAnywhere help page) |
| Fly.io pricing and free allowance (sources disagree) | https://fly.io/docs/about/discontinued-plans/ and https://snapdeploy.dev/blog/leaving-flyio-after-free-tier |
| Railway volumes | https://docs.railway.com/volumes/reference |
| Railway free plan and trial (secondary) | https://www.usagepricing.com/blueprint/railway |
| SQLite write-ahead logging | https://www.sqlite.org/wal.html and the SQLite documentation sources found in search |
| Vite requirements | https://vite.dev/guide/ |
| FastAPI TestClient and httpx | Search results only. **Confirm in FastAPI's own documentation at setup.** |

**Not verified in this session:** licence texts, current library versions, paid hosting prices, PythonAnywhere's free plan limits, Pillow's behaviour on specific files, and the exact setup steps for React testing tools. Check these when you reach each step.

---

# Document Control

| Version | Date | Change |
|---|---|---|
| 1.0 | 14 Nov 2026 | First draft |

**Next steps:** (1) Approve the decisions in Part 10. (2) Write ARCHITECTURE.md, with the data model, folder structure, API specification and at least one diagram. (3) Answer the open questions in ASSUMPTIONS.md Part 7.
