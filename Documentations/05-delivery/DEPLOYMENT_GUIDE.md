# DEPLOYMENT_GUIDE.md

# Visitor Photo Slideshow (MomentVault): Deployment Guide

| Item | Detail |
|---|---|
| Product | MomentVault, implementing Assignment 1: Visitor Photo Slideshow |
| Document version | 1.0 (draft for review) |
| Date | 18 November 2026 |
| Based on | TECH_STACK.md, ARCHITECTURE.md v1.1 (Part 14), BACKEND_DESIGN.md (Parts 5, 6, 20), FRONTEND_DESIGN.md (Part 11), SECURITY_PRIVACY.md (Part 25), IMPLEMENTATION_PLAN.md (D5-05), ASSUMPTIONS.md (A-18, A-30) |
| Audience | A beginner who has just cloned the repository, and the developer who wants to deploy it |
| Not included | Application code. No hosting account was used. **No command in this guide has been run against the real project yet** (the project code does not exist at the time of writing). |

---

## Read This First

### What This Guide Is, and Is Not

This guide tells you how to **run** the application from a fresh clone, how to **build** it for the demo, and how to **think about** putting it on the internet. It does **not** claim that any deployment works. Every step has a result you must record yourself (Section 21).

### How to Read the Labels

| Label | Meaning |
|---|---|
| **Placeholder** such as `<REPO_URL>` | A value you must fill in. It is **not** a real value. Never type the angle brackets. |
| **Approved** | Taken from the project's design documents |
| **Convention** | A normal choice for Python or Vite projects that the design documents do not fix. Confirm it against the real files. |
| **To verify** | Not checked yet. Check it the first time you run the step, and write down what happened. |
| **Fact, dated** | A statement about a hosting service, read on an official page on **18 November 2026**. Prices and limits change. Re-check before you rely on them. |

### The Short Version (Recommendation)

| Goal | What to do | Section |
|---|---|---|
| **Assignment demo (recommended)** | Run it on **your own computer**. Phones and the TV join the **same Wi-Fi**. Use the **built** frontend served by FastAPI, on **one address**. | 10, 12, 13 |
| **A public link (optional)** | Only on a host that gives a **persistent disk** (not a free temporary one), after a **restart test**. Cost and limits must be checked on the host's own page. | 14 |
| **Never** | Put the live demo on a free host with a temporary filesystem and expect photos to stay. | Key Concepts |

This matches ARCHITECTURE 14.1 and TECH_STACK 5.3: a deployed link is **optional** for the assignment, and the local demo is the **selected** option (assumption A-30, still marked OPEN).

### Findings (Things You Must Know Before You Start)

| # | Finding | How this guide handles it |
|---|---|---|
| **DG-F1** | **No application code exists yet.** The folder structure, settings and start-up behaviour come from BACKEND_DESIGN and FRONTEND_DESIGN. Names such as the application object `app` in `backend/app/main.py` follow the design but are not confirmed. | Every command is marked as written for the **approved structure**. The first time you run it, record the result in Section 21. IMPLEMENTATION_PLAN D5-05 already requires a run from a clean folder. |
| **DG-F2** | **Two settings defaults point at different places.** `DATA_DIR` defaults to `./data` (the design draws `data/` at the **repository root**), but `FRONTEND_DIST_DIR` defaults to `../frontend/dist` (which only works when you run from `backend/`). Run from `backend/`, `./data` would create `backend/data/`, not the root folder. | This guide runs the server **from `backend/`** and sets `DATA_DIR=../data` in the `.env` file, so both end up where the design says. **Confirm** against `core/settings.py` once written. |
| **DG-F3** | **Where the `.env` file is read from is not stated.** The design says `.env.example` sits at the repository root. A settings reader normally looks in the folder you run the program from. | This guide copies it to `backend/.env` (the folder you run from). If your edits seem ignored, see Troubleshooting T-09. **To verify.** |
| **DG-F4** | **There is no API address or CORS setting in the approved design.** The frontend uses relative addresses only (FRONTEND_DESIGN 11.7) and the settings list has **no** allowed-origins setting (BACKEND_DESIGN 20.2). | **Separate frontend hosting is not supported** without changing the design. This guide documents **one** supported way to deploy: FastAPI serves the built frontend (Option A). Section 16 explains what would have to change otherwise. |
| **DG-F5** | **The Python version is not fixed in the approved documents** (CODING_STANDARDS CS-2). | Section 2 gives the rule: use the version recorded in the README and pinned in `requirements.txt`. Record it on Day 1. |
| **DG-F6** | **Hosting facts change.** TECH_STACK's table was read on 14 November. This guide re-read the official pages on 18 November (Section 14.1). | Those facts are dated and linked. Prices were **not** verified, except where stated. |
| **DG-F7** | **A free host that "works" in a quick test can lose everything later.** The loss happens on a restart, a redeploy or a spin-down, not at upload time. | Section 14.4 is a restart test you must pass before trusting any host. |
| **DG-F8** | **A managed database alone does not save the photos.** The photos are files. If the database survives but the files vanish, the app lists photos whose files are gone. The start-up clean-up in the design handles the opposite problem (files without rows). | Explained in Key Concepts. |
| **DG-F9** | **Plain HTTP on shared Wi-Fi exposes the visitor token** (SECURITY_PRIVACY F-6, decision SP-6). | Demo on a **private network or phone hotspot** you control. Section 19. |
| **DG-F10** | **Building the frontend needs Node.js, running the backend needs Python.** Some hosts offer only one in their build environment. | Section 14.5 gives two ways to handle this and says what to check. |

---

## Key Concepts: Where Your Data Lives

This app keeps **two kinds of data**: a **database file** (SQLite) and **photo files**. Where they are stored decides whether they survive. These four terms are used by every hosting service, and they are the heart of this guide.

### The Four Storage Types

| | **Temporary filesystem** | **Persistent disk** | **Managed database** | **Object storage** |
|---|---|---|---|---|
| **Plain meaning** | The folder the app runs in. It is **thrown away** when the service restarts, redeploys or goes to sleep. | A **separate drive** attached to your service. What is written to its mount path stays. | A database **run for you** on its own server. You connect over the network. | A **bucket of files** reached over the internet by a key, not a folder on your server. |
| **Also called** | Ephemeral filesystem | Disk, volume | Managed PostgreSQL or MySQL | S3-compatible storage, bucket |
| **Survives a restart?** | **No** | Yes, **but only the mount path** | Yes (it is a separate service) | Yes |
| **Survives a redeploy?** | **No** | Yes, under the mount path | Yes | Yes |
| **Holds a SQLite file safely?** | **No** (the file is erased) | **Yes**, if it is a real local disk (not a network drive) | **Not applicable.** It replaces SQLite. | Not suitable for a live SQLite file |
| **Holds the photo folder?** | **No** | **Yes** | **No** (it stores rows, not folders of image files, in this design) | Yes, but the **storage code must change** |
| **Does this app work with it today?** | Only for throwaway tests | **Yes. This is what the design assumes.** | **No.** Needs code changes (below) | **No.** Needs code changes (below) |
| **Typical cost** | Included, often free | Usually **paid**, or small on some plans | Often paid, sometimes a limited free option | Pay for storage and transfer |
| **Main risk** | **Silent data loss** | Only one service instance can use it. Redeploys can briefly stop the app. | Different database engine, so tests and settings must be redone | Needs an account, secret keys and cost checks |

### Why the Differences Matter for MomentVault

1. **SQLite is a file.** If the file lives on a temporary filesystem, **every visitor, consent record and photo row disappears** on restart. The app has no migration or backup tool.
2. **Photos are files too.** They live in `data/photos/` and `data/thumbs/` beside the database (ARCHITECTURE 14, assumption A-18). The same rule applies to them.
3. **The two must survive together.** The design puts both under one `data/` folder so that a single persistent location covers both (TECH_STACK 5.2). If only one survives, the other points at nothing.

```text
Temporary filesystem:   restart  ->  database gone AND photos gone      (app starts empty)
Persistent disk:        restart  ->  database kept AND photos kept      (what we want)
Managed DB + temporary: restart  ->  database kept, photos GONE         (broken images!)
Persistent disk + one writer:       one service instance only           (matches SQLite)
```

### What Switching Would Cost

| Switch | What would have to change | Already planned? |
|---|---|---|
| SQLite to a managed database (PostgreSQL) | Database connection setting, SQLite-only start-up settings (WAL, busy wait), data types and locking, a database driver, and **every test rerun** | A future option, TECH_STACK M-1. Not in the MVP. |
| Local photo folder to object storage | The storage module, the way photos are served, new secret keys, a new dependency | A future option, TECH_STACK M-2. Not in the MVP. |

> **Rule for this guide:** do **not** assume a free hosting service keeps files or SQLite data. A service is "persistent" only if its **own documentation says so** and **your restart test passes** (Section 14.4).

---

## Table of Contents (Your 20 Topics)

| # | Topic | Section |
|---|---|---|
| 1 | System prerequisites | 1 |
| 2 | Required software versions | 2 |
| 3 | Repository setup | 3 |
| 4 | Python virtual environment setup | 4 |
| 5 | Backend dependency installation | 5 |
| 6 | Frontend dependency installation | 6 |
| 7 | Environment variable configuration | 7 |
| 8 | Database initialisation | 8 |
| 9 | Image storage directory setup | 9 |
| 10 | Running FastAPI locally | 10 |
| 11 | Running React locally | 11 |
| 12 | Accessing the application | 12 |
| 13 | Production frontend build | 13 |
| 14 | Backend deployment (including hosting options) | 14 |
| 15 | Frontend deployment | 15 |
| 16 | CORS configuration | 16 |
| 17 | Persistent database configuration | 17 |
| 18 | Persistent image storage | 18 |
| 19 | HTTPS configuration | 19 |
| 20 | Deployment troubleshooting | 20 |
| Extra | Results log, decisions, sources | 21, 22, 23 |

---

# Part A: Run It Locally from a Fresh Clone

Follow Sections 1 to 13 **in order**. Each step has a **Check** line. If the check fails, stop and use Section 20.

## The Project Layout You Should See

This is the approved structure (BACKEND_DESIGN 5, FRONTEND_DESIGN 9). Folder names after cloning use your repository's name, shown here as `<REPO_FOLDER>`.

```text
<REPO_FOLDER>/
  .env.example               placeholders only
  .gitignore                 data/, .env, venv, caches
  README.md
  backend/
    requirements.txt         pinned Python packages
    pytest.ini
    app/
      main.py                creates the FastAPI app
      ...                    api, schemas, models, services, repositories, db, core, storage
    tests/
  frontend/
    package.json
    vite.config.js
    index.html
    src/
  data/                      created at start-up. NOT in Git.
    momentvault.db           the database
    photos/                  display images
    thumbs/                  thumbnails
```

---

## 1. System Prerequisites

| Need | Why | How to check |
|---|---|---|
| A computer with **Windows 10 or 11, macOS, or Linux** | Runs the server | n/a |
| **Python** (Section 2) | The backend | `python --version` |
| **Node.js and npm** (Section 2) | Builds and runs the frontend | `node --version` and `npm --version` |
| **Git** | Clones the repository | `git --version` |
| A **terminal**: PowerShell or Command Prompt on Windows, Terminal on macOS or Linux | Runs the commands | n/a |
| An **internet connection** for the install steps | Downloads packages | n/a |
| **About 1 GB of free disk space** (estimate, not measured) | Packages, the frontend dependencies, and photos | n/a |
| For the phone and TV demo: **one Wi-Fi network** that all devices share | Phones cannot use `localhost` (Section 12) | n/a |
| A **modern browser** (Chrome or Edge, PRD NFR-COMP-02) | Opens the pages | n/a |

> **On Windows,** if the command `python` is not found, try `py --version`. If `py` works, use `py` in place of `python` everywhere below. This is a common Windows difference, not an error in the project.

**Check:** the three commands `python --version`, `node --version` and `git --version` each print a version number.

---

## 2. Required Software Versions

| Software | Required version | Source |
|---|---|---|
| **Node.js** | **20.19 or newer, or 22.12 or newer** | Vite's own guide: "Vite requires Node.js version 20.19+, 22.12+" (**fact, dated 18 Nov 2026**, see Sources) |
| **npm** | Comes with Node.js | n/a |
| **Python** | A version supported by **all** pinned packages. **Write the exact version you used into `README.md` and keep it the same everywhere.** The design examples use **3.10 or newer** syntax. | TECH_STACK 3.3 ("check at setup"), CODING_STANDARDS CS-2. **Not fixed in the approved documents (DG-F5).** |
| **Git** | Any current version | TECH_STACK 3.3 |
| **Package versions** (FastAPI, Uvicorn, python-multipart, SQLAlchemy 2.x, Pillow, Pydantic, pydantic-settings, pytest, httpx; React, react-dom, Vite) | **The exact versions pinned in `backend/requirements.txt` and `frontend/package.json`.** This guide does not state them because they are chosen on Day 1. | IMPLEMENTATION_PLAN D1-01, rule 4 |

Do **not** edit pinned versions "to make it work" without writing the reason in `AI_LOG.md`.

**Check:** `node --version` shows `v20.19` or higher, or `v22.12` or higher. If lower, install a newer Node.js before continuing (Vite will fail otherwise).

---

## 3. Repository Setup

```bash
# Placeholder: replace <REPO_URL> with the public GitHub address of the project.
git clone <REPO_URL>
cd <REPO_FOLDER>
```

```bash
# Check you are in the right place (macOS / Linux)
ls
```

```powershell
# Check you are in the right place (Windows PowerShell)
dir
```

**Check:** you can see `backend`, `frontend`, `.env.example` and `README.md`. (If `.env.example` seems missing, it is a hidden file because its name starts with a dot. Use `ls -a` on macOS or Linux, or `dir -Force` in PowerShell.)

> **Never commit** the `data/` folder, a `.env` file, or the `venv` folder (NFR-SEC-02). They are already listed in `.gitignore`. Run `git status` before every commit.

---

## 4. Python Virtual Environment Setup

A **virtual environment** is a private folder of Python packages for this project only. It keeps this project's libraries from clashing with other projects (TECH_STACK 4.17).

Create it **inside `backend/`**. (`venv` is already in `.gitignore`.)

**macOS or Linux**

```bash
cd backend
python3 -m venv venv
source venv/bin/activate
```

**Windows (PowerShell)**

```powershell
cd backend
python -m venv venv
venv\Scripts\Activate.ps1
```

> If PowerShell says that running scripts is disabled, run this once in the **same window**, then activate again. It changes the rule only for this window:
> `Set-ExecutionPolicy -Scope Process -ExecutionPolicy RemoteSigned`

**Windows (Command Prompt)**

```bat
cd backend
python -m venv venv
venv\Scripts\activate.bat
```

**Check:** your prompt now starts with `(venv)`. Then run `python --version` again and confirm it is the version you intended.

> Every time you open a new terminal for the backend, you must **activate the virtual environment again** (the `source ...` or `Activate.ps1` line). Without it, you will see "module not found" errors.

---

## 5. Backend Dependency Installation

With the virtual environment active and while inside `backend/`:

```bash
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
```

(Use `python -m pip`, not just `pip`, so that the packages go into **this** Python.)

**Check:**

```bash
python -m pip list
```

The list shows the pinned packages. The ones this project needs are: FastAPI, Uvicorn, python-multipart, SQLAlchemy, Pillow, Pydantic and pydantic-settings (the settings reader, BACKEND_DESIGN BD-04), plus pytest and httpx for the tests (TECH_STACK 3.2). **Python-multipart is easy to forget**: without it, photo uploads fail with an unclear error.

> If a package from that list is missing from `requirements.txt` after a fresh clone, that is a **project bug**. Record it in `AI_LOG.md` and add it with its pinned version. Do not guess a version.

**Run the backend tests (recommended, from `backend/`):**

```bash
python -m pytest
```

Record the real result in Section 21. **Do not write "tests pass" until you see them pass.** (`pytest.ini` is in `backend/`, so running from there is what the design expects.)

---

## 6. Frontend Dependency Installation

Open a **second terminal** (the virtual environment is **not** needed for this one).

```bash
cd <REPO_FOLDER>/frontend
npm ci
```

`npm ci` installs **exactly** what `package-lock.json` lists, and is the right command for a fresh clone. It needs `package-lock.json` to be committed. **If it reports that there is no lock file**, use `npm install` instead, and note in `AI_LOG.md` that the lock file was missing (it should be committed so everyone gets the same versions).

**Check:** a `node_modules/` folder now exists inside `frontend/`, and the command ends without red error lines. The approved frontend has only these runtime dependencies: `react`, `react-dom`, plus the build tools `vite` and its React plugin. Nothing else (FRONTEND_DESIGN 9.2).

> `node_modules/` is large and is **not** committed. Do not delete `package-lock.json`.

---

## 7. Environment Variable Configuration

All settings are read **once at start-up** from **environment variables**, optionally filled from a `.env` file that is **not** in Git (BACKEND_DESIGN 20.1). Every setting has a safe default **except** the operator key, which defaults to **off**.

**Step 1. Copy the example file** (from the repository root, then it goes next to where you run the server, see DG-F2 and DG-F3):

```bash
# macOS / Linux, from <REPO_FOLDER>
cp .env.example backend/.env
```

```powershell
# Windows PowerShell, from <REPO_FOLDER>
Copy-Item .env.example backend\.env
```

**Step 2. Open `backend/.env` in a text editor.** Each line has the form `NAME=value` (no spaces around `=`).

### The Approved Settings (the only ones that exist)

These 16 settings come from BACKEND_DESIGN 20.2. **There are no others.** If you find another name in the code or from an AI assistant, it is invented. Check `.env.example`, which is the authority once the code exists.

| Setting | Default | Meaning | Change for the demo? |
|---|---|---|---|
| `DATA_DIR` | `./data` | Folder for the database and photos | **Yes. Set to `../data`** when you run from `backend/` (DG-F2) |
| `SLIDE_SECONDS` | `10` | Seconds each photo is shown | No |
| `POLL_SECONDS` | `5` | How often the display asks for changes | No |
| `MAX_FILE_MB` | `5` | Largest file, in MiB | No |
| `MAX_FILES_PER_SUBMISSION` | `5` | Files per request | No |
| `MAX_MEGAPIXELS` | `25` | Largest image size | No |
| `NAME_MAX_LENGTH` | `50` | Longest visitor name | No |
| `MAX_PHOTOS_PER_VISITOR` | `20` | Active photos per visitor | No |
| `REPLAY_PREVIOUS_MAX` | `20` | Most earlier photos replayed | No |
| `MAX_IMAGE_SIDE_PX` | `1920` | Longest side after resizing | No |
| `THUMBNAIL_SIDE_PX` | `320` | Longest side of a thumbnail | No |
| `MAX_CONCURRENT_IMAGE_JOBS` | `4` | Images processed at once | No |
| `DB_BUSY_TIMEOUT_MS` | `5000` | How long a write waits if the database is busy | No |
| `FRONTEND_DIST_DIR` | `../frontend/dist` | Where the built React files are | No (when running from `backend/`) |
| `LOG_LEVEL` | `INFO` | How much is logged | Optional |
| `OPERATOR_KEY` | empty (**feature off**) | **A secret.** Turns on the optional operator removal endpoint (E10) | Only if you build and need E10 |

**A minimal `backend/.env` for a local demo** (only the one value that must differ from the defaults; the rest use their defaults):

```text
DATA_DIR=../data
```

### The Operator Key (Only If You Use It)

- The key is **optional**. Leave `OPERATOR_KEY` **empty** to keep the feature off. E10 then answers "not found".
- If you set one, SECURITY_PRIVACY decision SP-5 recommends **at least 32 characters**. Make a random one:

```bash
python -c "import secrets; print(secrets.token_urlsafe(32))"
```

(This prints a 43-character random string. It was checked to run and print 43 characters. Copy it into `.env` as `OPERATOR_KEY=<the printed string>`.)

- **Never** put the real key in `.env.example`, in Git, in a screenshot, in `AI_LOG.md`, or in a chat with an AI assistant.
- Treat the `.env` file as private. A key leaked to a public repository must be changed at once.

### Rules for Settings

| Rule | Reason |
|---|---|
| Settings are read **once at start-up**. **Restart the server after any change.** | BACKEND_DESIGN 20.1 |
| A wrong value (a negative or zero number, a `MAX_FILES_PER_SUBMISSION` larger than `MAX_PHOTOS_PER_VISITOR`, a folder that cannot be written) makes the server **refuse to start** and name the setting. | BACKEND_DESIGN 20.3 |
| The frontend needs **no** environment variables. All its addresses are relative. | FRONTEND_DESIGN 11.7 |
| The consent notice text and its version are **not** settings. They live in one place in the code. | BACKEND_DESIGN 20.4 (BD-05, awaiting your confirmation) |

**Check:** `backend/.env` exists, has `DATA_DIR=../data`, and is **not** shown by `git status` (it is ignored).

---

## 8. Database Initialisation

**There is no separate "set up the database" command.** The database is created **automatically** the first time the server starts (ARCHITECTURE 14.6, BACKEND_DESIGN 6.3). At start-up the app:

1. Reads and validates the settings.
2. Creates the data folders if they are missing.
3. Opens `momentvault.db`, switches on WAL mode, the busy wait and **foreign keys**.
4. Creates the missing tables (`visitors`, `consent_records`, `photos`) and sets the schema version.
5. Checks that foreign keys are really on, and **stops with a clear message** if not.

**Check it worked** (after the first start in Section 10):

```bash
# From backend/, with the virtual environment active.
python -c "import sqlite3; c = sqlite3.connect('../data/momentvault.db'); print(sorted(r[0] for r in c.execute(\"select name from sqlite_master where type='table'\")))"
```

Expected: a list that includes `consent_records`, `photos` and `visitors` (plus any internal tables). **To verify**: record what you actually see.

> On Windows PowerShell the quoting above can break. If it does, open `../data/momentvault.db` with a SQLite viewer (the TESTING_PLAN lists one) and look for the three tables.

**Important facts about this database:**

| Fact | Consequence |
|---|---|
| There is **no migration tool** in the MVP (TECH_STACK Part 7, ARCHITECTURE 11) | If a table definition changes, the simplest fix for a demo is to **recreate** the database (below). Do not pretend a migration exists. |
| `momentvault.db` has **extra files** (`-wal` and `-shm`) while the app runs | Part of WAL mode. Copy all of them, or stop the app first (Section 17). |
| The database is **not** in Git | A fresh clone starts with an **empty** database. That is expected. |

**Reset between demos (this erases everything):**

1. Stop the server (press `Ctrl+C` in its terminal).
2. Delete the **contents** of the `data/` folder (or the whole folder).
3. Start the server again. It recreates everything.

A scripted reset is a future item (ARCHITECTURE E-05).

---

## 9. Image Storage Directory Setup

**You normally do nothing.** At start-up the app creates `data/`, `data/photos/` and `data/thumbs/` if missing, resolves their full paths, and stops with a clear message if it cannot (BACKEND_DESIGN 6.3 step 3).

If you want to create them by hand (or the app reports it cannot), from `<REPO_FOLDER>`:

```bash
# macOS / Linux
mkdir -p data/photos data/thumbs
```

```powershell
# Windows PowerShell
New-Item -ItemType Directory -Force data\photos, data\thumbs
```

| Folder | Holds | Who writes |
|---|---|---|
| `data/photos/` | The re-encoded display images, with **random 32-character names** | The app only |
| `data/thumbs/` | Thumbnails | The app only |
| `data/momentvault.db` | The database | The app only |

| Rule | Reason |
|---|---|
| **Never put the folder inside `frontend/`** or serve it as plain static files. Photos reach browsers **only** through the API by photo id. | A removed photo must not stay reachable (ASSUMPTIONS A-18) |
| **Never commit `data/`.** | Privacy. It would publish visitors' photos. |
| The folder must be **writable** by the user who runs the server. | Otherwise start-up stops with a message naming the problem |
| **Never edit or add files by hand** while the app is running. | The database and the folder must agree |
| Space: an estimate (not measured) is that a stored photo is a few hundred kilobytes up to about 1 MB, so 0.5 GB might hold roughly 500 to 1,000 photos. **Measure real phone photos** before relying on this. | TECH_STACK 5.2 |

**Check:** after the first start, `data/photos` and `data/thumbs` exist.

---

## 10. Running FastAPI Locally

Use the terminal where the virtual environment is **active**, and run from **`backend/`** (DG-F2).

### Development mode (restarts when you change code)

```bash
cd <REPO_FOLDER>/backend
uvicorn app.main:app --reload --port 8000
```

### Demo mode (phones and the TV can connect)

```bash
cd <REPO_FOLDER>/backend
uvicorn app.main:app --host 0.0.0.0 --port 8000
```

| Part | Meaning |
|---|---|
| `uvicorn` | The program that runs the app and listens for requests |
| `app.main:app` | "In the folder `app`, in the file `main.py`, use the object named `app`". **Convention, to verify (DG-F1).** |
| `--reload` | Restart on code changes. **Development only. Never in the demo** (BACKEND_DESIGN 6.5). |
| `--host 0.0.0.0` | Listen on **all network addresses**, so other devices can connect. Without it, only the computer itself can. |
| `--port 8000` | The port number. **Convention** (the address example in FRONTEND_DESIGN 6.8 uses it). Any free port works. |

> **One worker only.** Do **not** add `--workers`. The image-processing gate, SQLite's single writer and the app's in-memory state all assume **one process** (ARCHITECTURE 14.4, AD-21).

**What you should see:** log lines in the terminal, including a "started" line with a summary of the settings (never secrets). The terminal **is** the log in the demo (BACKEND_DESIGN 19). Keep it open. Stop the server with `Ctrl+C`.

**Check:** open <http://localhost:8000/api/config> in a browser. It should show the app's public settings as JSON (this is endpoint E1, API_SPECIFICATION). **To verify.**

> `0.0.0.0` makes the app reachable by **everyone on the network**. Use it only on a network you trust (Section 19).

---

## 11. Running React Locally

Use the **second** terminal. The backend (Section 10, **development mode**) must already be running on port 8000, because Vite **forwards** `/api` requests to it (FRONTEND_DESIGN 11.7).

```bash
cd <REPO_FOLDER>/frontend
npm run dev
```

(`npm run dev` is the standard script that Vite projects define. **Convention, to verify** in `frontend/package.json`, "scripts".)

Vite prints a local address, by default **<http://localhost:5173>** (FRONTEND_DESIGN 11.7).

| Open | You should see |
|---|---|
| <http://localhost:5173/> | The visitor upload page |
| <http://localhost:5173/display> | The TV display page |

**Check:** the upload page loads and it can read the settings from the backend (for example, the consent notice appears). If the page loads but shows a "cannot reach the server" message, the backend is not running or is on a different port. See T-05 in Section 20.

> **For a phone, do not use the Vite server.** Use the **built** version served by FastAPI (Section 13 and 12). It needs only one address (FRONTEND_DESIGN 11.7).

---

## 12. Accessing the Application

### On the Same Computer

| Address | Page | When it works |
|---|---|---|
| `http://localhost:5173/` | Upload (visitor) page | Development (Section 11) |
| `http://localhost:5173/display` | TV display page | Development |
| `http://localhost:8000/` | Upload page | After the production build (Section 13) |
| `http://localhost:8000/display` | TV display page | After the production build |
| `http://localhost:8000/api/config` | Public settings (JSON) | Whenever the backend runs |
| `http://localhost:8000/docs` | Interactive API documentation | Local demo only. Left **on** locally. Switch **off** for any public deployment (BACKEND_DESIGN 6.6, a later decision). |

### From a Phone or the TV on the Same Wi-Fi

A phone **cannot** use `localhost`, because that means "this very device". It must use the **computer's address on the network**.

**Step 1. Find the computer's network address.**

```bash
# macOS
ipconfig getifaddr en0        # Wi-Fi is usually en0. If empty, check System Settings > Network.
# Linux
hostname -I
```

```powershell
# Windows PowerShell
ipconfig
# Look for "IPv4 Address" under your Wi-Fi adapter, for example 192.168.x.x
```

**Step 2. Start the backend in demo mode** (`--host 0.0.0.0`, Section 10).

**Step 3. Open on the phone or TV:**

| Device | Address (placeholder values) |
|---|---|
| Visitor's phone | `http://<COMPUTER_IP>:8000/` |
| TV or monitor | `http://<COMPUTER_IP>:8000/display` |

`<COMPUTER_IP>` is the address from Step 1, for example one that looks like `192.168.x.x`. **Do not copy that example. Use yours.**

**Step 4. Network checklist** (ARCHITECTURE 14.7):

| Check | Why |
|---|---|
| Phones, the TV and the computer are on the **same Wi-Fi** | Different networks cannot see each other |
| Some guest or public Wi-Fi networks block devices from reaching each other ("client isolation") | If the phone cannot connect, try a **phone hotspot** instead |
| The computer's **firewall** allows the chosen port (8000) | The most common cause of "cannot connect" |
| The server was started with `--host 0.0.0.0` | Otherwise only the computer itself can connect |
| The display opens in **full screen** by one manual step, and sleep is off | Browsers do not allow automatic full screen (A-26) |
| **Rehearse with the real phone and the real TV** | Not assumed to work (A-12, A-26) |
| Use a **private network or hotspot you control** | Plain HTTP exposes the token on shared Wi-Fi (SECURITY_PRIVACY F-6, SP-6) |

---

## 13. Production Frontend Build

The demo serves **built** files, not the Vite development server. Build once, and again after any frontend change.

```bash
cd <REPO_FOLDER>/frontend
npm run build
```

(`npm run build` is the standard Vite script. **Convention, to verify** in `package.json`.)

**Where the output goes:** `frontend/dist/` (Vite's usual output folder). The backend finds it through `FRONTEND_DIST_DIR`, whose default `../frontend/dist` is correct when you run from `backend/`.

**Then start the backend in demo mode** (Section 10) and open `http://localhost:8000/` and `http://localhost:8000/display`.

| What the backend does with the built files | Source |
|---|---|
| Returns the built page for **`/` and `/display` only** | ARCHITECTURE 2.2 |
| Serves the built assets (scripts, styles) | BACKEND_DESIGN 6.4 |
| Answers any other unknown address with the standard **404 not found** | ARCHITECTURE 2.2 |
| Registers the `/api` routes **first** so they are never hidden by the page rule | BACKEND_DESIGN 6.4 |
| If `frontend/dist` is missing, **warns but still starts**, because the API works without the frontend | BACKEND_DESIGN 20.3 |

**Check:** `frontend/dist/index.html` exists, and `http://localhost:8000/` shows the upload page (not "not found"). Reload `http://localhost:8000/display` in the browser. It must still show the display page, not a 404.

> `frontend/dist/` is a **build result**. Do not edit it. Whether it is in `.gitignore` is not stated in the approved documents (BACKEND_DESIGN lists only `data/`, `.env`, `venv` and caches). **Confirm**, and prefer building on each machine.

---

# Part B: Putting It on a Host (Optional)

> **Before you read on.** A deployed link is **optional** for the assignment. The approved plan is to demonstrate **locally** and decide on deployment only **after the MVP works** (TECH_STACK 5.3, ASSUMPTIONS A-30 is still **OPEN**). Nothing in Part B has been tried. Every host name, address and value below is either a **fact with a date and a source**, or a **placeholder**.

## 14. Backend Deployment

### 14.1 Hosting Options for a Beginner

**What the app needs from any host** (TECH_STACK 5.3):

| Need | Why |
|---|---|
| A **long-running Python process** | The FastAPI app runs all the time |
| **Persistent local storage** | SQLite and the photo folder are files (Key Concepts) |
| **One application instance** | One database writer, and a disk can attach to only one instance |
| **HTTPS** | Recommended for any deployed version (PRD NFR-PRV-03) |
| Phones can reach the address | Visitors use their own devices |

**Options compared.** "Facts" were read on official pages on **18 November 2026**. "Beginner-friendliness" is **my judgement**, not a measurement. **Prices were not verified** unless stated. Check each host's own pricing page before you decide.

| Option | Keeps SQLite and photos? | Facts, dated (see Sources) | Beginner-friendliness (judgement) | Verdict for this project |
|---|---|---|---|---|
| **Your own computer (local network demo)** | **Yes**, while the `data/` folder stays | Selected for the MVP (ARCHITECTURE 14.1, A-30 provisional default). Phones use the computer's network address, not `localhost`. | **Easiest.** No account, no cost, no hosting concepts. | **Recommended.** Needs the computer awake and a network you trust. |
| **Render, free web service** | **No** | "Like all Render services, Free web services have an ephemeral filesystem." Local file changes, "uploaded images, local SQLite databases, etc." are lost on redeploy, restart or spin-down. A free service **spins down after 15 minutes without inbound traffic**, and waking takes about a minute. "Paid services can preserve local filesystem changes by attaching a persistent disk, but Free web services cannot." | Easy to start. **Dangerous for this app.** | **Do not use for real data.** Fine only for a throwaway test of the build. |
| **Render, paid web service with a persistent disk** | **Yes, under the disk's mount path only** | Disks attach to **paid** web services. "Only filesystem changes under this path are preserved across deploys and restarts." A disk serves **one instance** and cannot be used if you scale to several. **Adding a disk disables zero-downtime deploys**, so each deploy briefly stops the service. Build commands cannot reach the disk. Snapshots are taken once every 24 hours and kept for seven days, and a restore reverts the **whole** disk. A disk can be enlarged but not shrunk. **Price not verified.** | Moderate. Web dashboard, no server administration. | **A reasonable choice if you must have a public link.** Pass the restart test (14.4) first. |
| **Railway, service with a volume** | **Yes, on the volume's mount path** | Volume size limits: **Free and Trial 0.5 GB, Hobby 5 GB, Pro 50 GB.** One volume per service. **Replicas cannot be used with volumes.** Redeploying a service with a volume causes **brief downtime**. Backups (manual and automatic) are supported for volumes. **Plan prices and any free allowance were not verified**, and one plan may be a time-limited trial. Check Railway's pricing page. | Moderate. | **Possible.** Check plan and cost. Pass the restart test. |
| **Fly.io, app with a volume** | **Possible with a volume** | Pricing page: "Every new user gets a $1 free trial, no credit card required, and invoices under $5 are waived." **Volumes cost $0.15 per GB per month and "bill on provisioned capacity, not usage."** Volume behaviour (one machine, backups) was **not verified** in this session. | Harder: uses a command-line tool. | **Possible**, but not the easiest first deployment. |
| **PythonAnywhere** | **Not verified** | Its help page titles ASGI deployment **"(beta)"** and calls it "an experimental feature". It says: "There is no support for static file mappings", "a very limited web UI", and that command syntax and the API are **not guaranteed** to stay the same. Long-term pricing is not settled. | Unclear | **Not recommended** for a first deployment: experimental support for FastAPI. (Our app serves its own built pages, so the static-file limit may not matter. **To verify.**) |
| **A rented Linux server (a "VPS")** | **Yes**, it is a normal computer with a disk | Not evaluated in this session. | **Hardest**: you must manage the system, a process manager, a firewall and an HTTPS proxy. | Not recommended in a 20-hour MVP. |
| **Serverless platforms** | **Not verified**, and typically not | They typically do not keep local files between requests (TECH_STACK 5.3). | n/a | **Treat as unsuitable** unless a specific platform is verified. |
| **A tunnel from your own computer to the internet** | Yes (it is your computer) | SECURITY_PRIVACY 25: **do not expose the demo through a tunnel without HTTPS and the stated controls.** A tunnel makes the app public to everyone. | Easy | **Avoid.** Real visitors' photos would be public over plain HTTP. |

> **Do not assume a free plan keeps files.** The question to ask any host is: *"Where is my disk, does it survive a restart and a redeploy, and what does it cost?"* The answer must be in **their documentation**, and **your restart test must confirm it**.

### 14.2 Before Any Public Deployment (Gate)

The design lists these as **required or recommended before the app is public**. A public link without them is **not safe to use with real people's photos.**

| # | Item | Source | Done? |
|---|---|---|---|
| 1 | **Retention decision**: how long photos are kept (assumption A-11 is OPEN and **required** before any public deployment) | ASSUMPTIONS A-11 | |
| 2 | **Consent notice wording reviewed.** It is **not legal advice** and has had no legal review. | ASSUMPTIONS A-08, PRD OOS-15 | |
| 3 | **HTTPS** is in place (Section 19) | SECURITY_PRIVACY 25 | |
| 4 | **Upload flooding** is limited, or the limitation is accepted in writing (a per-visitor limit does not cap the total) | SECURITY_PRIVACY F-1, SP-1 | |
| 5 | **Operator key** set to a **random 32+ character** value, kept as a secret in the host's settings, **or** left empty with E10 off | SECURITY_PRIVACY SP-5 | |
| 6 | **API documentation pages (`/docs`) switched off** | BACKEND_DESIGN 6.6 | |
| 7 | **Restart test passed** (14.4) | TECH_STACK 5.3 | |
| 8 | **README states the honest limits**: plain-HTTP demo on a private network, no automatic expiry, no legal review, no age check, consent is self-declared | SECURITY_PRIVACY | |
| 9 | Token storage reconsidered for HTTPS (a protected cookie option) | SECURITY_PRIVACY F-4, ADR-006 | |

### 14.3 Generic Deployment Outline (Placeholders Only)

This is a **template**. Host names and menu names differ. **Do not copy a value that is not yours.**

**Step 1. Choose a host with a persistent disk or volume** and read its documentation on disks and on the service type you need. Write down: the **mount path**, the **size**, the **cost**.

**Step 2. Create a web service from your GitHub repository**, using a branch you have tested locally.

**Step 3. Set the build step.** One way (the same commands as locally, with host-neutral options):

```bash
python -m pip install -r backend/requirements.txt
npm ci --prefix frontend
npm run build --prefix frontend
```

The build must produce `frontend/dist/`. **Check that the build environment has both Python and Node.js** (see 14.5).

**Step 4. Set the start command:**

```bash
cd backend && uvicorn app.main:app --host 0.0.0.0 --port <HOST_PORT>
```

`<HOST_PORT>` is **the port your host tells you to listen on.** Some hosts hand you the number as a variable. **Use exactly the form your host documents.** The application has no `PORT` setting of its own. Keep **one worker** and **no `--reload`**.

**Step 5. Set the environment variables in the host's settings page (not in Git).** Use **only** the 16 approved names (Section 7). The ones you will normally set:

| Setting | Value on the host |
|---|---|
| `DATA_DIR` | `<PERSISTENT_MOUNT_PATH>/data`. **It must be inside the persistent mount path**, or it will be erased (Key Concepts). |
| `FRONTEND_DIST_DIR` | `../frontend/dist` (correct because the start command runs from `backend/`) |
| `OPERATOR_KEY` | `<A_RANDOM_SECRET_OF_32_OR_MORE_CHARACTERS>`, entered as a **secret** in the host's settings, or left empty |
| `LOG_LEVEL` | `INFO` |
| Everything else | Leave at the default |

Also set the **Python and Node.js versions** the host asks for to the ones you recorded (Section 2): `<PYTHON_VERSION>` and `<NODE_VERSION>`. **Do not assume the host's default is the same.**

**Step 6. Attach the disk** at `<PERSISTENT_MOUNT_PATH>`, **one instance only**.

**Step 7. Deploy and open the host's address:** `<DEPLOYED_URL>` (a placeholder. Use the address your host gives you. **Do not invent one.**) Then open `<DEPLOYED_URL>/api/config`.

**Step 8. Run the restart test (14.4).** Do not announce the link to anyone before it passes.

> **Build commands cannot write to the disk on some hosts** (for example, Render says build and pre-deploy commands cannot access it). That is fine: the app creates its folders and database **when it starts**, not at build time.

### 14.4 The Restart Test (Do Not Skip)

This is the test that finds a host that **silently loses data**. TECH_STACK 5.3 requires it. Use a **test** visitor and a **test** picture (not a real person).

| # | Action | Expected result | Actual result | Pass/Fail |
|---|---|---|---|---|
| R-1 | Open `<DEPLOYED_URL>/`, register a test visitor, upload one test photo | The photo appears on `<DEPLOYED_URL>/display` | | Not run |
| R-2 | Note the visitor's name and that the photo shows under "My photos" | | | Not run |
| R-3 | **Restart** the service from the host's dashboard | Service comes back | | Not run |
| R-4 | Reload the same browser on `<DEPLOYED_URL>/` | The **same visitor is recognised** and the photo is **still there** | | Not run |
| R-5 | **Redeploy** (trigger a new deploy, for example by a harmless commit) | Service comes back | | Not run |
| R-6 | Repeat R-4 | Visitor and photo **still there** | | Not run |
| R-7 | On a **free** plan with idle spin-down, wait past the idle time, then reload | Visitor and photo **still there**. The first answer may be slow. | | Not run |
| R-8 | Check `<PERSISTENT_MOUNT_PATH>/data` holds the database and photo folders | Present | | Not run |

**If R-4, R-6 or R-7 fails, the host does not keep your data. Stop and do not use it.**

### 14.5 Python and Node.js on the Same Host (DG-F10)

The app needs Python at **run** time and Node.js at **build** time (to produce `frontend/dist/`). Check the host's documentation for **both**. Two ways to handle it:

| Way | How | Trade-off |
|---|---|---|
| **A. Build on the host** | The build step runs `npm ci` and `npm run build` (14.3). Needs Node.js available in the Python build environment. | No build output in Git. **Some hosts' Python environments have no Node.js.** To verify on your host. |
| **B. Build on your computer and commit `frontend/dist/`** | Run `npm run build` locally, commit the output, and let the host only run Python | Works on a Python-only host. But **a built output in Git can go out of date** and clutters history. It also needs `dist` **removed from `.gitignore`** if it is ignored. |
| (Docker) | One image containing both | Future (TECH_STACK M-5). Not in the MVP. |

Decision DG-4 (Section 22).

---

## 15. Frontend Deployment

**Supported way (Option A, approved): the frontend is deployed *inside* the backend.** FastAPI serves the built React files and the `/api` routes from **one address** (ARCHITECTURE 14, TECH_STACK 5.4). So "deploying the frontend" means:

1. Build it (Section 13).
2. Make sure the built `frontend/dist/` is **present where the backend runs**, and `FRONTEND_DIST_DIR` points to it.
3. Deploy the backend (Section 14).

| Why one address | Reason |
|---|---|
| No cross-origin settings | Section 16 |
| One address to give to phones and the TV | `<DEPLOYED_URL>/` and `<DEPLOYED_URL>/display` |
| The token and the pages share one origin | Browser storage belongs to **one origin** |

**Not supported by the current design: hosting the frontend on a separate static host** (Option B, "two addresses"). The frontend has **no setting for the API address** and its image links are **relative** (`/api/photos/{id}/image`). See Section 16.

**Check:** on the deployed address, `<DEPLOYED_URL>/` shows the upload page and `<DEPLOYED_URL>/display` (also after a browser reload) shows the display page.

---

## 16. CORS Configuration

### What CORS Is

A browser normally stops a page from one address (an "origin": protocol, host and port) from reading answers from a **different** origin. **CORS** (Cross-Origin Resource Sharing) is the way a server says "this other origin is allowed".

### What This Project Needs: Nothing

| Situation | CORS needed? |
|---|---|
| **Development**: Vite forwards `/api` to the backend, so the browser sees **one** address | **No** (TECH_STACK 5.4) |
| **Local demo and Option A deployment**: FastAPI serves pages and API from **one** address | **No** |
| **Separate frontend host (Option B)** | **Yes**, and more (below). **Not supported.** |

**There is no CORS or "allowed origins" setting in the approved settings list** (BACKEND_DESIGN 20.2). ARCHITECTURE 14.5 says allowed origins are "none", to be used "only if cross-origin use is ever enabled".

> **Do not "fix" a browser error by allowing all origins.** An AI assistant may suggest `allow_origins=["*"]`. That would break the restriction in PRD NFR-SEC-05 and SECURITY_PRIVACY. If you see a CORS error in the browser console, it means the page is being opened from the wrong address. See T-21.

### If You Ever Need Option B (Not Part of the MVP)

All of these would have to change, so it is a **design change with its own ADR**, not a setting (TECH_STACK M-11):

| # | Change | Why |
|---|---|---|
| 1 | A new **allowed-origins setting**, restricted to the real frontend address | The only safe way to allow one origin |
| 2 | The frontend needs an **API base address** | Today every address is relative |
| 3 | **Photo links** (`/api/photos/{id}/image`) must point at the API host | Relative links would point at the frontend host and break (AD-16) |
| 4 | The server must allow the custom headers `X-Visitor-Token` and `X-Operator-Key` | Browsers send an extra "preflight" request for custom headers (general browser behaviour, **to verify** for this project) |
| 5 | **HTTPS on both** hosts | Mixed content is blocked |
| 6 | Re-run the **security and API tests** | Several approved controls assume one origin |

---

## 17. Persistent Database Configuration

**Approved design: SQLite in WAL mode on a disk that stays** (ARCHITECTURE 14.4, TECH_STACK 5.1).

| Setting or rule | Value | Why |
|---|---|---|
| Location | `DATA_DIR`, file `momentvault.db` | One folder for database and photos (A-18) |
| Local demo | `DATA_DIR=../data` when running from `backend/` (DG-F2) | The folder simply stays on your computer |
| Deployed | `DATA_DIR=<PERSISTENT_MOUNT_PATH>/data` | **Only the mount path survives** |
| Mode | **WAL**, busy wait `DB_BUSY_TIMEOUT_MS` (default 5000), foreign keys on for every connection | Set by the app at start-up. Do not change it by hand. |
| Processes | **One** Uvicorn worker | SQLite has **one writer** |
| Instances | **One** | A disk attaches to one instance |
| Storage type | A **real local disk** | **WAL does not work over a network drive** (TECH_STACK 4.5). Do not point `DATA_DIR` at a shared network folder. |

### Backing Up

There is **no automatic backup** in the app (TECH_STACK 5.2).

| Way | How |
|---|---|
| **Manual copy** | **Stop the app**, then copy the **whole `data/` folder**. In WAL mode, extra `-wal` and `-shm` files exist, and a copy taken **while running** may miss recent changes (ARCHITECTURE 14.8). |
| **Host snapshots** | If the host offers disk snapshots, read **what** they cover and **how restore works**. For example, Render's snapshot restore **reverts the whole disk**, so changes after the snapshot are lost. |

### If You Wanted a Managed Database Instead

It is **not** a configuration change. It would mean moving from SQLite to PostgreSQL (TECH_STACK M-1): a new connection setting and driver, removing the SQLite-only start-up steps, re-checking the `CHECK` rules and types, and **re-running every test**. **Do not attempt it inside the 20-hour MVP.**

---

## 18. Persistent Image Storage

**Approved design: photo files in `data/photos/` and `data/thumbs/`, beside the database, on the same persistent disk** (A-18, TECH_STACK 5.2).

| Rule | Detail |
|---|---|
| **Same disk as the database** | `DATA_DIR` holds both. If one is on a disk and the other is not, they **drift apart**. |
| **Only the mount path survives** | A photo folder outside it is erased on restart. |
| **Never serve the folder directly** | Photos go out through the API by photo id, so a removed photo can be refused. |
| **Random server-made names** | The visitor's file name is never used in a path. |
| **Size** | Estimate only (Section 9). **Measure** with real phone photos. A **full disk** is a real risk: the per-visitor limit of 20 does **not** cap the total (SECURITY_PRIVACY F-1). |
| **Withdrawal and deletion** | Remove the file **and** the record together. Files are deleted **after** the database commit. |

### Object Storage (Not Part of the MVP)

Moving photos to object storage (TECH_STACK M-2) would mean: a new account and **secret keys** (stored in the host's secret settings, never in Git), a new dependency, a **replacement for the storage module**, and a different way of delivering photos. The database would store object keys instead of file names. **It is the right answer when you need more than one server instance or a host with no disk.** It is not needed for a single-instance demo on a persistent disk.

### The Broken-Images Case (DG-F8)

If the **database survives but the photo files do not** (for example a managed database with a temporary filesystem), the app would list photos whose files are gone. The start-up clean-up described in the design (ARCHITECTURE 14.6, step 4, a Should Have) removes files that have **no row**, which is the **opposite** problem. **Keep database and photos on the same persistent disk.**

---

## 19. HTTPS Configuration

### Why It Matters

Over plain **HTTP**, anyone on the same network may be able to read **everything**, including the **visitor token** sent in the `X-Visitor-Token` header and the photos. With **HTTPS** the traffic is encrypted between the browser and the server (SECURITY_PRIVACY 25.1). PRD NFR-PRV-03 asks for HTTPS in **deployed** versions.

### Local Demo: HTTP Only, On a Private Network

| Fact | Handling |
|---|---|
| **HTTPS is not available on the local network** in this design (ARCHITECTURE 14.7) | The app uses no feature that needs HTTPS, **to be confirmed on real phones** |
| Plain HTTP exposes the token on shared Wi-Fi (F-6) | **Demo only on a private network or a phone hotspot you control** (SP-6). Say so in the README. |
| **Do not** use a tunnel to publish the local demo | Section 14.1 |

### Deployed: Use the Host's HTTPS

| Case | What to do |
|---|---|
| **A managed host** (Render, Railway, Fly.io and similar) | Many hosts give an `https://` address and manage the certificate for you. **Confirm in your host's documentation that it does, for your plan and for a custom domain**, rather than assuming. This guide did not verify it. |
| **Your own Linux server** | Put a **reverse proxy** (for example Caddy or nginx) in front of Uvicorn. It holds the certificate and passes requests to `http://127.0.0.1:<APP_PORT>`. You need a **domain name** (`<YOUR_DOMAIN>`) pointing at the server. Follow the proxy's own documentation. **No configuration is given here because it was not tested.** |

### After HTTPS Is On

| Item | Detail |
|---|---|
| **No mixed content** | All the app's addresses are relative, so they follow the page's `https://`. Check the browser console for blocked "mixed content". |
| **Security headers** | `nosniff` and a content security policy as in SECURITY_PRIVACY. Add the **strict transport security** header **only after** HTTPS has been verified (SECURITY_PRIVACY security-headers table: it forces HTTPS, so it is hard to undo). |
| **Token storage** | Reconsider a protected cookie, with protection against cross-site requests (SECURITY_PRIVACY F-4, ADR-006). Not part of the MVP. |
| **Visitors lose recognition when the address changes** | Browser storage belongs to one origin. Moving from `http://<COMPUTER_IP>:8000` to `https://<DEPLOYED_URL>` makes everyone a **new visitor**. Warn people before switching. |
| **Operator key** | The key travels in a header. **Do not use E10 on a shared network without HTTPS** (SECURITY_PRIVACY). |

**Check:** `<DEPLOYED_URL>` begins with `https:`, the browser shows no certificate warning, and the restart test (14.4) still passes.

---

## 20. Deployment Troubleshooting

Find the symptom, check the likely cause, then apply the fix. **These are likely causes, not proven ones.** Record anything you learn in `AI_LOG.md`.

| ID | Symptom | Likely cause | What to do |
|---|---|---|---|
| **T-01** | `python` is "not found" (Windows) | The launcher is named `py` | Use `py --version`, and `py -m venv venv` |
| **T-02** | PowerShell says running scripts is disabled when activating | Windows script policy | `Set-ExecutionPolicy -Scope Process -ExecutionPolicy RemoteSigned` in the **same window**, then activate again. Or use Command Prompt (`activate.bat`). |
| **T-03** | `ModuleNotFoundError: No module named 'app'` when starting Uvicorn | You are not in `backend/` | `cd <REPO_FOLDER>/backend` and run again |
| **T-04** | `No module named 'fastapi'` (or another package) | The virtual environment is **not active**, or packages were installed into another Python | Activate `venv` (Section 4). Install with `python -m pip install -r requirements.txt`. |
| **T-05** | The upload page loads but shows "cannot reach the server", or the Vite terminal shows proxy errors | The backend is not running, or is on a different port than the one Vite forwards to | Start the backend (Section 10) on port 8000. Check `vite.config.js` forwards `/api` to that port. |
| **T-06** | "Address already in use" | Another program (or an old server) holds port 8000 | Stop the old server, or use a different `--port` (and change Vite's forwarding to match) |
| **T-07** | The phone cannot open `http://<COMPUTER_IP>:8000/` | Server started without `--host 0.0.0.0`; **firewall**; different Wi-Fi; guest-network device isolation; wrong IP | Restart in demo mode. Allow port 8000 in the firewall. Put both on the same network, or use a phone hotspot. Re-read the IP (Section 12). |
| **T-08** | The server **refuses to start** and names a setting | Settings validation (a number not positive, `MAX_FILES_PER_SUBMISSION` above `MAX_PHOTOS_PER_VISITOR`, a folder that cannot be written, foreign keys not on) | Fix the named setting in `backend/.env`. See Section 7. |
| **T-09** | Changes to `.env` seem ignored, or a `data/` folder appears in the **wrong place** (for example `backend/data`) | `.env` is read from the folder you start in (DG-F3); settings are read **once** at start-up; `DATA_DIR` is relative to where you run | Confirm `backend/.env` exists and contains `DATA_DIR=../data`. **Restart** the server. Delete the stray folder. |
| **T-10** | `/display` shows "not found" or a blank page, especially after a reload | The frontend was **not built**, `FRONTEND_DIST_DIR` is wrong, or the server is not returning the page for `/display` | Run `npm run build` (Section 13). Check `frontend/dist/index.html` exists. Look for the "dist missing" warning in the server log. |
| **T-11** | `npm` errors about the Node version | Node.js is older than 20.19 / 22.12 | Install a newer Node.js (Section 2) |
| **T-12** | `npm ci` fails ("no package-lock.json" or "out of sync") | The lock file is missing or does not match `package.json` | Use `npm install` once, then **commit the lock file** (DG-5) |
| **T-13** | Photo uploads fail with an unclear error | `python-multipart` is missing from the environment | Check `python -m pip list`. Install the pinned version from `requirements.txt`. |
| **T-14** | "database is locked", or the app answers it is busy (`SERVER_BUSY`) | More than one worker or instance, the database on a **network drive**, or many writes at once | Run **one worker**, one instance, and a **local** disk (Section 17). Do not use `--workers`. |
| **T-15** | After a restart or redeploy, **all visitors and photos are gone** | A **temporary filesystem** (Key Concepts), or `DATA_DIR` is outside the mount path | Attach a persistent disk, set `DATA_DIR` **inside** its mount path, redo the restart test. **Lost data cannot be recovered.** |
| **T-16** | Photos are listed but the **images are broken** | Database kept but the photo files were lost, or the files and database are on different disks | Keep both under one persistent `DATA_DIR` (DG-F8). Remove the orphan rows or reset the demo. |
| **T-17** | The first page load is **very slow** on a free host | Free services may **spin down** when idle and take about a minute to wake (Render fact, dated) | Expected on that plan. It also means a free service is a poor choice for a live demo. |
| **T-18** | The host's **build fails** (Node or Python "not found") | The build environment does not have both runtimes (DG-F10) | Use way B in 14.5, or choose a host that has both |
| **T-19** | "Permission denied" writing to `data/` | The folder is not writable by the running user | Make the folder writable for that user. On a host, set the disk's mount path and permissions as its documentation says. |
| **T-20** | Browser warns about HTTPS or blocks "mixed content" | A certificate problem, or an `http://` address typed into an `https://` page | Check the certificate and open the `https://` address. The app's own links are relative. |
| **T-21** | A CORS error in the browser console | The page is opened from a different address than the API (for example from a file or a different port), or Option B is attempted | Open the app from the **one** address (Section 12). **Do not** allow all origins (Section 16). |
| **T-22** | `pytest` finds nothing, or import errors in tests | Not running from `backend/` (where `pytest.ini` is), or the environment is not active | `cd backend`, activate the environment, run `python -m pytest` |
| **T-23** | `.env.example` is "missing" | It is a hidden file (name starts with a dot) | `ls -a` (macOS or Linux), `dir -Force` (PowerShell) |
| **T-24** | A returning visitor is treated as **new** | A different browser, a private window, cleared data, or a **different address** (`localhost:8000` vs `<COMPUTER_IP>:8000` are different origins with separate browser storage) | Use the **same browser and the same address**. This is the approved limit (ASSUMPTIONS A-03). |
| **T-25** | The TV page works but the screen sleeps or leaves full screen | Browsers do not allow automatic full screen, and TV sleep settings (A-26) | Enter full screen manually and turn off sleep. Rehearse on the real TV. |

### A Safe Debugging Order

1. Is the **virtual environment active**, and are you in `backend/`?
2. Does `http://localhost:8000/api/config` answer?
3. Read the **server terminal**. It is the log (BACKEND_DESIGN 19).
4. Does the browser's **developer tools** console or Network tab show the failing request and its status?
5. Change **one** thing at a time, and write down what you tried in `AI_LOG.md`.
6. **Never** paste the operator key, a visitor token or a real photo into an AI chat.

---

## 21. Deployment Results Log

**Nothing below has been run.** Fill in each row yourself the first time you do the step, from a **clean clone** (IMPLEMENTATION_PLAN D5-05 requires this). Write what you **actually saw**. A failure that is written down is more useful than a claimed success.

**Environment used for this run:**

| Item | Value |
|---|---|
| Date | |
| Operating system | |
| Python version | |
| Node.js version | |
| Git commit tested | |
| Run by | |

### Local Run (Part A)

| ID | Step | Section | Expected | Actual result | Pass/Fail |
|---|---|---|---|---|---|
| L-01 | Clone the repository | 3 | `backend`, `frontend`, `.env.example` present | | Not run |
| L-02 | Create and activate the virtual environment | 4 | Prompt shows `(venv)` | | Not run |
| L-03 | Install backend packages | 5 | No errors. Package list matches the pinned versions | | Not run |
| L-04 | Run backend tests | 5 | Tests pass (**record the real numbers**) | | Not run |
| L-05 | Install frontend packages | 6 | `node_modules/` exists, no errors | | Not run |
| L-06 | Copy `.env.example` to `backend/.env`, set `DATA_DIR=../data` | 7 | File exists, not shown by `git status` | | Not run |
| L-07 | Start the backend (development mode) | 10 | Server starts, "started" log line | | Not run |
| L-08 | Database created | 8 | `data/momentvault.db` has the three tables | | Not run |
| L-09 | Photo folders created | 9 | `data/photos` and `data/thumbs` exist | | Not run |
| L-10 | Open `/api/config` | 10 | JSON settings | | Not run |
| L-11 | Start Vite and open `localhost:5173` | 11 | Upload page loads and reads the settings | | Not run |
| L-12 | Production build | 13 | `frontend/dist/index.html` exists | | Not run |
| L-13 | Start in demo mode, open `localhost:8000/` and `/display`, reload `/display` | 12, 13 | Both pages load, reload keeps the page | | Not run |
| L-14 | Open the app from a **real phone** via `<COMPUTER_IP>` | 12 | Upload page loads | | Not run |
| L-15 | Upload a test photo and see it on the display | 12 | Appears on `/display` | | Not run |
| L-16 | Stop the server, start again, check the data stayed | 8, 17 | Visitor and photo still there | | Not run |
| L-17 | Reset: stop, empty `data/`, start | 8 | Empty database recreated | | Not run |

### Hosted Deployment (Part B, Only If You Do It)

| Item | Value |
|---|---|
| Host name | `<HOST_NAME>` |
| Plan and monthly cost (from the host's own page, with date read) | `<PLAN_AND_COST>` |
| Persistent disk or volume: mount path and size | `<PERSISTENT_MOUNT_PATH>`, `<SIZE>` |
| Deployed address | `<DEPLOYED_URL>` |
| Date deployed | |

| ID | Step | Section | Actual result | Pass/Fail |
|---|---|---|---|---|
| H-01 | Build succeeds on the host | 14.3 | | Not run |
| H-02 | App starts, `/api/config` answers | 14.3 | | Not run |
| H-03 | Restart test R-1 to R-8 | 14.4 | | Not run |
| H-04 | HTTPS works, no warnings | 19 | | Not run |
| H-05 | Public-deployment gate items 1 to 9 | 14.2 | | Not run |

---

## 22. Decisions Needing Your Approval

| ID | Decision | Alternative | Recommendation |
|---|---|---|---|
| **DG-1** | Run the server **from `backend/`**, with `backend/.env` and `DATA_DIR=../data` (DG-F2, DG-F3) | Change the settings defaults so everything is relative to the repository root | Keep this guide's way. Update it if `settings.py` is written differently. |
| **DG-2** | **Option A only**: FastAPI serves the built frontend. Separate frontend hosting is **not supported**. | Support Option B (needs a design change, an ADR and several code changes, Section 16) | Option A only |
| **DG-3** | **Local demo is the primary path.** A hosted link is optional and only after the MVP works. A-30 stays OPEN. | Deploy early | Local first (TECH_STACK 5.3) |
| **DG-4** | Build the frontend **on the host** (way A), or commit `dist` (way B) | Docker (future) | Way A if the host has Node.js. Otherwise way B, noted in the README. |
| **DG-5** | **Commit `package-lock.json`** and use `npm ci` | `npm install` only | Commit it. It gives everyone the same versions. |
| **DG-6** | Use **port 8000** and the start command `app.main:app` (both **conventions**) | Another port or module path | Keep, and change this guide if the code differs |
| **DG-7** | **Switch off `/docs`** for any public deployment | Leave on | Switch off. It also needs a setting or a code change, **which is not in the approved settings list.** Decide how. |
| **DG-8** | The **public-deployment gate** (14.2) must be met before sharing a public link | Share early | Meet the gate |

---

## 23. Sources, Glossary and Document Control

### Sources (Hosting and Tool Facts, Read 18 November 2026)

Prices and limits change. Re-read the page before you decide.

- [Render: Deploy for Free](https://render.com/docs/free), free web services have an ephemeral filesystem, spin down after 15 minutes of no traffic, and cannot have persistent disks.
- [Render: Persistent Disks](https://render.com/docs/disks), paid services only, only the mount path is preserved, one instance, no zero-downtime deploys with a disk, daily snapshots kept 7 days.
- [Railway: Volumes reference](https://docs.railway.com/volumes/reference), size limits by plan, one volume per service, no replicas with volumes, brief downtime on redeploy.
- [Fly.io pricing](https://fly.io/pricing.md), $1 trial for new users, volumes $0.15 per GB per month.
- [PythonAnywhere: Deploying ASGI sites (beta)](https://help.pythonanywhere.com/pages/FastAPI), experimental, no static file mappings, limited web interface.
- [Vite: Getting Started](https://vite.dev/guide/), Node.js 20.19+ or 22.12+.
- Project documents: TECH_STACK.md (5.1 to 5.4, 3.2, 3.3), ARCHITECTURE.md (2.2, 14), BACKEND_DESIGN.md (5, 6, 20), FRONTEND_DESIGN.md (11.7), SECURITY_PRIVACY.md (25), ASSUMPTIONS.md (A-11, A-18, A-30), IMPLEMENTATION_PLAN.md (D5-05).

### What Was and Was Not Checked

| Checked | Not checked |
|---|---|
| The hosting statements above, against the official pages, on 18 November 2026 | **Prices** (except Fly.io's two quoted figures), and Railway plan details beyond volume sizes |
| Node.js requirement for Vite, against Vite's guide | **Any command against the real project.** No project code existed. |
| The token-generation command ran and printed 43 characters | Whether `app.main:app`, `npm run dev`, `npm run build` and the `.env` location match the final code (DG-F1) |
| Setting names against BACKEND_DESIGN 20.2 | Whether any host provides HTTPS or Node.js for your plan |

### Glossary

| Term | Meaning |
|---|---|
| **Virtual environment (venv)** | A private folder of Python packages for one project |
| **Origin** | Protocol, host and port together, for example `http://localhost:8000` |
| **CORS** | Browser rules that decide whether a page may read answers from another origin |
| **Ephemeral (temporary) filesystem** | A disk that is erased on restart, redeploy or sleep |
| **Persistent disk (volume)** | A disk attached to a service that keeps data under its mount path |
| **Mount path** | The folder where a persistent disk appears. Only data under it survives. |
| **Managed database** | A database run for you by a provider, reached over the network |
| **Object storage** | Files kept in a remote bucket and reached by key |
| **WAL mode** | A SQLite setting that lets readers continue while one writer writes |
| **Reverse proxy** | A program in front of your app that handles HTTPS and passes requests on |
| **Spin-down** | A free host stopping an idle service to save resources |
| **Placeholder** | A made-up name in angle brackets (`<REPO_URL>`) that you must replace |

### Document Control

| Item | Detail |
|---|---|
| Document | DEPLOYMENT_GUIDE.md |
| Version | 1.0 (draft for review) |
| Date | 18 November 2026 |
| Status | Draft. **No step has been run.** Section 21 is blank on purpose. |
| Next review | After Day 1 (versions and file locations are fixed) and after D5-05 (clean-clone run), when conventions in DG-F1 are confirmed or corrected |
