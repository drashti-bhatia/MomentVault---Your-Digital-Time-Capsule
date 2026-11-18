# CODING_STANDARDS.md

# Visitor Photo Slideshow (MomentVault): Coding Standards

| Item | Detail |
|---|---|
| Product | MomentVault, implementing Assignment 1: Visitor Photo Slideshow |
| Document version | 1.0 (draft for approval) |
| Date | 17 November 2026 |
| Based on | ARCHITECTURE.md v1.1, DATABASE_DESIGN.md, API_SPECIFICATION.md, BACKEND_DESIGN.md, FRONTEND_DESIGN.md, SECURITY_PRIVACY.md, IMPLEMENTATION_PLAN.md, TESTING_PLAN.md |
| Technology | Python, FastAPI, SQLAlchemy 2.0, Pydantic, Pillow, pytest, React, JavaScript, Vite, plain CSS, SQLite |
| Audience | The developer, Claude, Gemini, and anyone reviewing the code |
| Not included | The application itself. The examples are short, made-up illustrations of style. |

**How to read this document.** Every rule says what to do, and most say why. Rules are tagged **MUST** (a review failure if broken), **SHOULD** (the default, with a reason needed to break it), **MAY** (a free choice), **MUST NOT** and **SHOULD NOT** (the same strength, for things to avoid). Examples marked **Bad** show the mistake and **Good** shows the fix. IDs such as `FR-ID-05`, `AD-04`, `E3` and `API-06` point to the approved documents.

**What was checked, and what was not.** Every Python example and every plain JavaScript example in this document was run through a syntax check (Python's parser and Node's `--check`). That proves they are well formed. It does **not** prove they work in the real application, because FastAPI, SQLAlchemy and httpx were not available when this was written, and no application code exists yet. The React (JSX) examples were **not** machine-checked and were read by eye only. Treat every example as a style illustration. Names such as `DomainError(...)` follow the approved designs but must be confirmed against the real files when they are written.

---

## Read This First: Findings and Decisions That Need Your Approval

These are places where the approved documents are silent or where one rule had to be chosen. None changes an approved decision.

| # | Finding | What this document does | Decision |
|---|---|---|---|
| **CF-1** | **No formatter or linter is named in TECH_STACK.** Without one, style rules are only a request. | Rules are written so they can be followed **by hand**. A formatter is recommended as an **optional** extra (Ruff for Python, Prettier and ESLint for JavaScript) in Section 20. Nothing in this document requires installing one. | CS-1 |
| **CF-2** | **The Python version is not fixed in the approved documents.** Examples use `X \| None` and `list[str]` (Python 3.10 or newer). | Confirm the version on Day 1 and write it into `requirements.txt` notes and the README. | CS-2 |
| **CF-3** | **JSON fields are `snake_case` (API_SPECIFICATION 2.2) but JavaScript variables are conventionally `camelCase`.** Converting every response would add a mapping layer nobody asked for. | **Do not convert.** Objects that come from the server keep their `snake_case` field names inside the frontend (`visitor.photo_count`). Everything the developer names alone is `camelCase`. Section 11 explains the one exception rule. | CS-3 |
| **CF-4** | **Line length.** PEP 8 says 79. Most formatters use 88. | **88** characters for code. Comments and docstrings wrap at **79**. This is allowed by PEP 8 when a team agrees. | CS-4 |
| **CF-5** | **No Git message convention is approved.** IMPLEMENTATION_PLAN says only "one task, one commit, say what and why". | A light version of Conventional Commits is proposed (Section 18). | CS-5 |
| **CF-6** | **A document cannot make an AI follow rules.** Claude and Gemini only follow what is in front of them. | A short paste-in prompt block (Section 0.2) and a review checklist (Section 19) make the document usable as shared rules. The developer still reads every change. | CS-6 |
| **CF-7** | **Some rules here repeat rules in BACKEND_DESIGN and FRONTEND_DESIGN.** Two copies can drift apart. | Where a rule already exists there, this document **points to it** instead of re-writing it (for example layer rules, Section 3.3 of BACKEND_DESIGN). If the two disagree, the **design document wins**, and this document is corrected. | CS-7 |

---

## Coverage of the Request

| # | Requested topic | Section |
|---|---|---|
| 1 | Naming conventions | 1 |
| 2 | File naming | 2 |
| 3 | Python coding standards | 3 |
| 4 | Python type hints | 4 |
| 5 | Function and class design | 5 |
| 6 | Exception handling | 6 |
| 7 | Logging conventions | 7 |
| 8 | React component naming | 8 |
| 9 | React hooks usage | 9 |
| 10 | State management conventions | 10 |
| 11 | JavaScript naming | 11 |
| 12 | CSS organisation | 12 |
| 13 | API response consistency | 13 |
| 14 | Database access conventions | 14 |
| 15 | Comments and documentation | 15 |
| 16 | Code duplication prevention | 16 |
| 17 | Security coding practices | 17 |
| 18 | Git commit message conventions | 18 |
| Extra | Shared AI rules, one-page summary, tools, review checklist, decisions | 0, 19, 20, 21 |

---

# 0. How Humans and AI Assistants Use This Document

## 0.1 The Ten Rules to Remember

If you remember nothing else, remember these. Each is explained later.

| # | Rule | Section |
|---|---|---|
| 1 | **Layers only call downwards**: routes → services → repositories. Routes never touch the database. Services never import FastAPI. | 5.4 |
| 2 | **Services commit. Repositories never commit.** | 14.3 |
| 3 | **Services raise domain errors with a catalogue code.** They never build HTTP responses. | 6.2 |
| 4 | **Errors are one shape**: `{"error": {code, message, details, request_id}}`. Success is the bare resource. | 13 |
| 5 | **Never log or return** a token, a token hash, a name, an original file name, a file path or a stack trace. | 7.3, 17 |
| 6 | **Only `apiClient` talks to the server.** Only hooks call `apiClient`. Components draw. | 9, 10 |
| 7 | **Render visitor text as text.** Never as HTML. | 17.4 |
| 8 | **No raw SQL strings with values pasted in.** Use `select()` with bound values. | 14.2 |
| 9 | **Names say what a thing is**, spelled out, in the case of its language (Section 1). | 1 |
| 10 | **Small and explainable.** If you cannot explain it in a live demo in one sentence, simplify it. | 5.1 |

## 0.2 Shared Prompt Block for Claude and Gemini

Paste this at the start of every coding session with either assistant. It is deliberately short so that it is not ignored.

```text
You are helping a beginner build MomentVault (Visitor Photo Slideshow).
Follow CODING_STANDARDS.md and the approved design documents.
Rules:
1. Do not invent requirements, endpoints, error codes or files. If something
   is missing, say so and ask.
2. Write the smallest, simplest code that does the task. No extra layers,
   libraries, patterns, or "future-proofing".
3. Respect the layers: routes -> services -> repositories. Services commit,
   repositories never do. Services never import FastAPI.
4. Errors use the catalogue codes in API_SPECIFICATION 4.3 and one JSON shape.
5. Never log or return tokens, token hashes, visitor names, original file
   names, file paths or stack traces.
6. Use SQLAlchemy select() with bound values. No SQL built by joining strings.
7. React: components draw, hooks connect, only apiClient calls the server.
   Render visitor text as text.
8. Explain each new file in plain words after writing it, and list how to
   test it. Do not claim code works unless it was actually run.
9. Change only the files for this task. Do not rewrite unrelated code.
10. If the standards and the design documents disagree, stop and say so.
```

## 0.3 Order of Authority

When two sources disagree, the higher one wins. The assistant must say so instead of choosing silently.

| Order | Source |
|---|---|
| 1 | PRD.md (requirements) |
| 2 | API_SPECIFICATION.md and DATABASE_DESIGN.md (contracts) |
| 3 | SECURITY_PRIVACY.md |
| 4 | ARCHITECTURE.md, BACKEND_DESIGN.md, FRONTEND_DESIGN.md |
| 5 | **This document** |
| 6 | An assistant's habits or preferences (**never** a reason to break rows 1 to 5) |

---

# 1. Naming Conventions

**Principle.** A name says **what a thing is or does**, in whole words. A reader should not need to open the definition to guess it.

## 1.1 One Table for the Whole Project

| Thing | Python | JavaScript and React | Database | JSON (API) | CSS |
|---|---|---|---|---|---|
| Variables and parameters | `snake_case` | `camelCase` | n/a | n/a | n/a |
| Functions | `snake_case` verb phrase | `camelCase` verb phrase | n/a | n/a | n/a |
| Classes | `PascalCase` | n/a | n/a | n/a | n/a |
| React components | n/a | `PascalCase` | n/a | n/a | n/a |
| Constants | `UPPER_SNAKE_CASE` | `UPPER_SNAKE_CASE` | n/a | n/a | n/a |
| Booleans | `is_`, `has_`, `can_` | `isX`, `hasX`, `canX` | `is_` or `has_` column if needed | `is_new` | n/a |
| Files | `snake_case.py` | See Section 2 | n/a | n/a | `lowercase.css` |
| Tables | n/a | n/a | plural `snake_case` | n/a | n/a |
| Columns | n/a | n/a | singular `snake_case` | `snake_case` | n/a |
| Error codes | `UPPER_SNAKE_CASE` | `UPPER_SNAKE_CASE` | n/a | `UPPER_SNAKE_CASE` | n/a |
| CSS classes, tokens | n/a | n/a | n/a | n/a | `kebab-case` |

## 1.2 Rules for Every Name

| Rule | Level | Reason |
|---|---|---|
| Use whole words. `photo_count`, not `pc` or `ph_cnt`. | MUST | Beginner readability |
| Well-known short forms are allowed: `id`, `url`, `db`, `ms`, `mb`. | MAY | Universally understood |
| Do not use one-letter names except `i`, `j` in a tiny loop, `e` for a caught error or event, and `_` for "unused". | MUST | Searchability |
| Do not use `data`, `info`, `temp`, `obj`, `thing`, `stuff`, `manager`, `helper`, `util` as a whole name. | SHOULD NOT | They say nothing. Use `upload_result`, not `data`. |
| A **boolean** reads as a yes or no question. `is_active`, never `active_flag` or `status_ok`. | MUST | `if is_active:` reads like English |
| A **function** starts with a verb. A **value** is a noun. | MUST | `count_active_photos(...)` and `photo_count` |
| Names carry the **unit** when a number could be confused: `max_file_bytes`, `slide_seconds`, `poll_interval_ms`. | MUST | API_SPECIFICATION 2.2: sizes are bytes, durations are seconds |
| Use the **project's words**: visitor, photo, consent, withdraw, remove, operator, display, slideshow, feed, cursor. Do not rename them (not `user`, `image_owner`, `picture`, `revoke`). | MUST | Same word in the PRD, the code, the tests and the demo |
| Spell as in the documents: **British spelling in visitor-facing text** (`unauthorised`, `recognised`). Code identifiers follow the library or the document (`VISITOR_NOT_RECOGNISED`). | SHOULD | Consistency with API_SPECIFICATION |

**Bad and good**

```python
# Bad: unclear, abbreviated, and the flag name hides its meaning
def proc(d, flg):
    tmp = d["n"]
    return tmp if flg else None

# Good: says what it is and what it does
def get_visitor_name(visitor: dict[str, str], is_name_visible: bool) -> str | None:
    return visitor["name"] if is_name_visible else None
```

## 1.3 Project Vocabulary (Use Exactly These Words)

| Word | Meaning | Do not call it |
|---|---|---|
| visitor | A person who uploads photos. **Not unique by name** (FR-REG). | user, customer, account |
| token | The secret the browser keeps to prove "this is me" | session, cookie, login |
| token hash | What the database stores in place of the token | password, key |
| operator | The person running the event (E10) | admin (the page may say "Operator"; the route file is `operator_routes`) |
| photo | One uploaded image | picture, upload (an *upload* is the act) |
| withdraw (consent) | The visitor removes all their photos and their consent | revoke, unsubscribe |
| remove | Mark a photo `removed` and delete its file | delete (when the database row stays) |
| feed | The E7 response the display polls | stream, playlist |
| cursor | The highest photo id the display has seen (`since_id`) | offset, page |

---

# 2. File Naming

| Kind | Rule | Example |
|---|---|---|
| Python files and folders | `snake_case`, singular for one concept | `photo_service.py`, `error_handlers.py` |
| Python layer suffix | The file name ends with its layer, so it is clear in a search result | `photo_routes.py`, `photo_service.py`, `photo_repository.py`; models and schemas share the plain name in their own folder (`models/photo.py`, `schemas/photo.py`) |
| Python tests | `test_<area>.py`, mirrors the layers (BACKEND_DESIGN 5) | `test_photo_service.py` |
| Test functions | `test_<behaviour in words>` | `test_rejects_text_file_renamed_jpg` |
| React components | `PascalCase.jsx`, file name **equals** the component name | `PhotoPicker.jsx` |
| React hooks | `useThing.js` (`camelCase`, begins with `use`), one hook per file | `useSlideshow.js` |
| Other JavaScript | `camelCase.js` | `apiClient.js`, `slideshowQueue.js`, `photoChecks.js` |
| CSS | `lowercase.css`, named after the page or the group | `tokens.css`, `upload.css`, `display.css` |
| Markdown documents | `UPPER_SNAKE_CASE.md` for project documents (as already used) | `CODING_STANDARDS.md` |
| Folders in `frontend/src` | `lowercase`, plural for groups | `components`, `hooks`, `pages`, `lib` |
| Test image files, if any are committed | Describe the case, lowercase with hyphens, tiny size | `not-an-image.txt` |

| Rule | Level | Reason |
|---|---|---|
| **One main thing per file.** A component per `.jsx`. A hook per `.js`. A layer per `.py`. | MUST | Easy to find and explain |
| **Do not create files outside the folder structures** in BACKEND_DESIGN 5 and FRONTEND_DESIGN 9 without saying why. | MUST | Stops an assistant inventing new folders |
| **No spaces, capitals (except components and project documents) or dates** in file names. | MUST | Works on every operating system and in URLs |
| **Never name a file `utils.py`, `helpers.js`, `misc.js` or `common.js`** for a mix of things. (`schemas/common.py` is allowed: it holds the shared error shape and time format, a stated purpose.) | SHOULD NOT | A junk drawer grows without limit |
| **Never commit** `.env`, the database, photos, logs or `node_modules` (NFR-SEC-02). | MUST | Security and privacy |

---

# 3. Python Coding Standards

## 3.1 The Base Rule

Follow **PEP 8**, with the choices in this section. When a rule here and PEP 8 differ, this document is the project's chosen variation. Do not add other style rules from memory.

## 3.2 Layout

| Topic | Rule | Level |
|---|---|---|
| Indentation | 4 spaces. Never tabs. | MUST |
| Line length | **88** for code. **79** for comments and docstrings (CS-4). | SHOULD |
| Quotes | Double quotes `"` for text. | SHOULD |
| Blank lines | Two between top-level functions and classes. One between methods. | MUST |
| Trailing commas | Add one when a list, call or dictionary is split over several lines. | SHOULD |
| Operators and commas | Spaces as in PEP 8: `a = b + 1`, `f(x, y)`. No spaces inside brackets. | MUST |
| String building | **f-strings** for text. Never `+` in a loop, never `%`. | SHOULD |
| File ending | One newline at the end. No trailing spaces. | MUST |

## 3.3 Imports

Three groups, one blank line between, each in alphabetical order: **standard library**, **third-party**, **this project**. Use **absolute** imports from `app` (`from app.core.errors import DomainError`). Import a **module** when the module is a layer, so the layer shows at the call.

```python
# Good
import logging
from pathlib import Path

from fastapi import APIRouter, Depends
from sqlalchemy import select
from sqlalchemy.orm import Session

from app.models.photo import Photo
from app.repositories import photo_repository
```

```python
# Bad: wildcard hides where a name comes from; relative import climbs folders;
# one import line mixes groups
from app.models import *
from ..repositories.photo_repository import count_active_for_visitor, add
import os, sys
```

| Rule | Level |
|---|---|
| **No** `from x import *`. | MUST |
| **No** unused imports. **No** imports inside functions, except to break a real cycle (and then say why in a comment). | MUST |
| Layers are called through their **module**: `photo_repository.add(...)`, not `from ... import add`. Several repositories each have an `add`. | SHOULD |
| Standard names are imported by name: `from pathlib import Path`. | MAY |

## 3.4 Python Basics to Prefer

| Prefer | Instead of | Reason |
|---|---|---|
| `pathlib.Path` for paths | String joining | Safer, clearer, and the base for the safe-path rule (17.5) |
| `with` blocks for files and sessions | Manual open and close | Always closes, even after an error |
| List or dictionary comprehensions for **simple** transforms | A loop that only builds a list | Shorter. **Not** when it needs more than one condition or nesting: use a loop. |
| `is None` and `is not None` | `== None` | PEP 8 |
| `if items:` for "not empty" | `if len(items) > 0:` | Idiomatic |
| Early return for a failure | A deep pyramid of `if` blocks | Flat code is easier to read |
| Constants in `settings` or a named constant | A number in the middle of code ("magic number") | `5 * 1024 * 1024` hides that it is the file limit |
| `datetime` objects in UTC inside code | Hand-built time strings | The API converts at the edge (API_SPECIFICATION 2.2) |

```python
# Bad: magic number, deep nesting, loop only to build a list
def check_size(size):
    if size is not None:
        if size > 0:
            if size <= 5242880:
                return True
            else:
                return False
    return False

# Good: named limit from settings, flat, early returns
def is_size_allowed(size_bytes: int, max_file_bytes: int) -> bool:
    if size_bytes <= 0:
        return False
    return size_bytes <= max_file_bytes
```

## 3.5 What Not To Do

| Do not | Why |
|---|---|
| Use mutable defaults: `def f(items=[])`. Use `None` and create the list inside. | Classic bug: the list is shared between calls |
| Use global variables that change (`current_visitor = ...`). | Hides state. FastAPI dependencies exist for this (BACKEND_DESIGN 13). |
| Use `eval`, `exec`, `pickle` on outside data, `os.system`, or `subprocess` with outside text. | Security (17.6) |
| Use `print()` for anything except a throwaway during debugging, and **delete it before committing**. | Use `logging` (Section 7) |
| Leave commented-out code in a commit. | Git remembers old code |
| Write `except:` or `except Exception: pass`. | Hides bugs (Section 6) |
| Use `assert` to check outside input. | Assertions can be switched off. Raise a domain error. |

---

# 4. Python Type Hints

**Principle.** Type hints are **documentation the computer can check**. They also let FastAPI and Pydantic read what a function expects. The aim is clarity, not perfection.

## 4.1 Rules

| Rule | Level | Reason |
|---|---|---|
| **Every function** has hints for **all parameters and the return value**, including `-> None`. | MUST | A reader sees the contract without reading the body |
| Use the built-in forms: `list[str]`, `dict[str, int]`, `tuple[int, int]`, and `X \| None`. Not `List`, `Dict`, `Optional` from `typing`. | SHOULD | Newer and shorter (CF-2) |
| Say `\| None` **explicitly** wherever `None` can come back. A function that may find nothing returns `Photo \| None`. | MUST | Forces the caller to handle "not found" |
| Model attributes use SQLAlchemy 2.0 **`Mapped[...]`** typed columns (BACKEND_DESIGN 9). | MUST | The approved style |
| Input and output shapes are **Pydantic schemas**, not loose dictionaries. | MUST | Validation and automatic documentation. A dictionary has no checked shape. |
| Do not use `Any` unless you write a comment saying why. | SHOULD NOT | `Any` switches checking off |
| Do not write hints that are wrong "to make it pass". Wrong hints are worse than none. | MUST | They mislead the assistant and the reader |
| Local variables **do not** need hints when the type is obvious. | MAY | Avoids noise |
| Aliases are fine for a repeated, meaningful type, such as `PhotoId = int`. | MAY | Do not add one for a single use |

```python
# Bad: nothing says what comes in, what goes out, or that None is possible
def find_photo(photo_id, session):
    return session.get(Photo, photo_id)

# Good: contract visible; the caller must handle "not found"
def get_by_id(session: Session, photo_id: int) -> Photo | None:
    return session.get(Photo, photo_id)
```

## 4.2 Hints and the Layers

| Layer | What the hints show |
|---|---|
| Routes | Pydantic schema parameters and `response_model` (the shape promised in API_SPECIFICATION) |
| Services | Take plain values and a `Session`. **Return Pydantic objects** (BACKEND_DESIGN 3.6), never live model objects. |
| Repositories | Take a `Session`. Return a **model** (or `None`, a list, or a number). |
| Image processor | Takes `bytes`. Returns a small result object. No database types. |

## 4.3 Checking Hints

A type checker (such as `mypy`) is **optional** (Section 20). The hints are **required even if no checker is ever run**, because they are for the reader and for FastAPI. Do not spend project hours fighting a checker.

---

# 5. Function and Class Design

## 5.1 The Explainability Test

> **If you cannot explain a function in one sentence in a live demo, it is too big or badly named.**

| Rule | Level | Reason |
|---|---|---|
| A function does **one job**. Its name is that job. | MUST | Testable and explainable |
| Aim for **under about 30 lines**. A longer function needs a reason, or it is split. | SHOULD | BACKEND_DESIGN 11.3: "Keep functions short and named after use cases" |
| At most **about four parameters**. More than that means a group of values wants to be a schema or a small class. | SHOULD | Easy to call correctly |
| Nesting **no deeper than three levels**. Use early returns. | SHOULD | Flat reads better |
| A function either **does something** (changes data, writes a file) or **answers something** (returns a value), **not both**, unless the name says so. | SHOULD | `get_...` must not change data |
| Do not change the arguments you receive (for example append to a passed list) without saying so in the name or docstring. | MUST | Surprising changes cause bugs |
| **Pure logic is separate from input and output.** Validation rules, ordering and limits are plain functions that take values and return values. | SHOULD | They can be tested with no database and no files |

## 5.2 Functions, Not Classes, by Default

The approved design uses **functions in modules** for services and repositories (BACKEND_DESIGN 12.1). A beginner needs very few classes.

| Use a class for | Do not use a class for |
|---|---|
| A **model** (table) | A "service" with no data of its own |
| A **Pydantic schema** | A "manager", "handler" or "processor" that only groups functions |
| The **domain error** type | Wrapping a single function |
| **Settings** | An inheritance tree. **No inheritance** except the framework bases (model base, schema base, `Exception`). |
| A small **result object** returned by a function, when a tuple would be unclear | A design pattern for its own sake (factory, singleton, strategy) |

```python
# Bad: a class that holds nothing, only to group functions (extra thing to explain)
class PhotoServiceManager:
    def __init__(self):
        pass

    def upload(self, session, files):
        ...

# Good: module-level functions; the module name is the group
# app/services/photo_service.py
def upload_photos(session: Session, visitor_id: int, files: list[UploadedFile]) -> UploadOutcome:
    ...
```

## 5.3 No Premature Abstraction

| Do not | Instead |
|---|---|
| Write an "interface" or "base repository" before two real uses need it. | Write the concrete function. |
| Add a parameter "in case it is needed later". | Add it when it is needed. |
| Add a setting for something that never changes. | A named constant. |
| Optimise (caching, batching, clever SQL) before measuring. | Write the clear version. ARCHITECTURE sets the limits that matter (the processing gate, the 5 MiB limit). |
| Add a library for one small job. | Use the standard library or Pillow. New dependencies need approval (TECH_STACK). |

**The rule of three.** Write it plainly the first time. Allow a second copy. **At the third**, extract it (Section 16).

## 5.4 Layer Rules (Pointer, Not a Copy)

The authoritative table is **BACKEND_DESIGN 3.3 "Who May Call Whom"**. In short:

| Layer | May call | Must not |
|---|---|---|
| Routes (`api/`) | Services, schemas, dependencies | Repositories, storage, Pillow |
| Services | Repositories, storage, image processor, other services (visitor → consent only), core helpers | FastAPI, routes |
| Repositories | Models, the session | Services, storage, **commit** |
| Storage | File system, settings | Database |
| Image processor | Pillow | Database, disk |

A **route function** contains only: read the request, call **one** service function, return the result (BACKEND_DESIGN 7.2). If a route has an `if` about a business rule, that rule belongs in a service.

```python
# Bad: the route queries the database, builds a rule, and returns a model object
@router.get("/api/visitors/me")
def get_me(token: str = Header(...), session: Session = Depends(get_session)):
    visitor = session.query(Visitor).filter(Visitor.token_hash == hash_token(token)).first()
    if visitor is None or visitor.status != "active":
        raise HTTPException(status_code=401)
    return visitor

# Good: the route only connects; the rule lives in the service
@router.get("/api/visitors/me", response_model=VisitorProfile)
def get_current_visitor(visitor: VisitorContext = Depends(require_visitor)) -> VisitorProfile:
    return visitor_service.get_profile(visitor)
```

---

# 6. Exception Handling

**Principle.** An exception is for something that **should not happen in the normal flow**. Expected outcomes (a bad file, an already-removed photo) are **results**, not crashes. The full design is BACKEND_DESIGN 18.

## 6.1 Rules

| Rule | Level | Reason |
|---|---|---|
| **Never** write a bare `except:`. **Never** write `except Exception: pass`. | MUST | Hides bugs and breaks cleanup |
| Catch the **narrowest** exception you can handle, around the **smallest** block of code. | MUST | A wide `try` catches the wrong thing |
| Catch an exception **only if you can do something useful**: clean up, translate, or record. Otherwise let it travel up. | MUST | BACKEND_DESIGN 18.5 |
| **Do not use exceptions for normal outcomes.** "Photo already removed" is a normal `200` (FR-DEL-05). One bad file in a batch is a **result entry**, not a raised error. | MUST | E3 always returns `200` with per-file results |
| **Clean up before re-raising**, not after. Delete the half-written file, then raise. | MUST | No orphan files (FR-STO-04) |
| When you translate one exception into another, **keep the cause**: `raise DomainError(...) from error`. | SHOULD | The log shows the real cause |
| Never put a token, hash, file path, SQL or stack trace into an **error message returned to the visitor**. | MUST | FR-ID-05, NFR-SEC-04 |
| An error message for a visitor comes **from the catalogue**, never from the exception's text. | MUST | Library messages leak details |
| Unexpected errors are **logged once, with the request id and stack trace**, at the catch-all handler. Do not log the same error at every layer. | SHOULD | One event, one log line |

## 6.2 Who Raises, Who Catches, Who Answers

| Layer | Job |
|---|---|
| Services | **Raise** a domain error with a **catalogue code** (`NAME_TOO_LONG`) |
| Routes | Do **not** catch domain errors. They let them travel. |
| Error handlers (`core/error_handlers.py`) | **Turn** domain errors into the standard JSON and the right status |
| Catch-all handler | Turns any other exception into `INTERNAL_ERROR` with a generic message |
| Storage and image processor | Raise or return a **specific** failure. They never decide the HTTP status. |

```python
# Bad: swallows everything, returns a vague fact, and leaks internal text
try:
    result = process_image(file_bytes)
except Exception as e:
    return {"error": str(e)}

# Good: catches only what the processor documents, uses a catalogue code,
# keeps the cause for the log, never sends library text to the visitor
try:
    result = image_processor.process(file_bytes)
except image_processor.NotAnImageError as error:
    raise DomainError("FILE_NOT_AN_IMAGE") from error
```

> The exact exception names (`NotAnImageError`) and the `DomainError` signature are **illustrative**. Define the real ones once in `core/errors.py` and `image_processor.py`, then use them everywhere.

## 6.3 Cleanup That Must Not Be Skipped

Use `try` / `finally` or a `with` block when something must be released: the session (the dependency does it), the processing gate (a slot taken must be released), a temporary file.

```python
# Good: the gate slot is always released, even if processing fails
def process_with_gate(file_bytes: bytes) -> ProcessedImage:
    with processing_gate:
        return image_processor.process(file_bytes)
```

---

# 7. Logging Conventions

The design is **BACKEND_DESIGN 19**. This section is the coding rule set that makes it real.

## 7.1 Rules

| Rule | Level | Reason |
|---|---|---|
| Use Python's **`logging`** module. **No `print`.** | MUST | Levels, request ids, one place to switch off |
| One logger per file: `logger = logging.getLogger(__name__)` at the top. | MUST | The file shows in the output |
| Log an **event name plus `key=value` facts**, in lowercase: `logger.info("photo_accepted photo_id=%s ms=%s", photo_id, elapsed_ms)`. | MUST | Searchable, no sentences to parse |
| Use `%s` style arguments in the log call, not an f-string. | SHOULD | The text is only built if the line is actually written |
| The **request id** is added by the logging filter. **Do not add it by hand.** | MUST | BACKEND_DESIGN 19.2 |
| Log **one line per event**. No banners, no decoration, no blank lines. | SHOULD | Readable in a terminal |
| Log **once**. Do not log and then raise the same problem at every layer. | SHOULD | Avoids duplicates |
| **Never** build a log line by dumping a whole request, response, header set, or object. Pick the fields. | MUST | Tokens live in headers (FR-ID-05) |

## 7.2 Levels

| Level | Use for | Example event |
|---|---|---|
| `DEBUG` | Detail for the developer. Off by default. | `gate_wait ms=40` |
| `INFO` | Normal, meaningful events | `photo_accepted`, `visitor_registered`, `upload_summary` |
| `WARNING` | Something went wrong but the app handled it | `file_rejected code=FILE_NOT_AN_IMAGE size=1204`, `wrong_operator_key` (count only) |
| `ERROR` | A bug or failure that needs attention (with the stack trace) | `unhandled_exception` |
| `CRITICAL` | Not used in this MVP | |

## 7.3 What Never Appears in a Log

Matches BACKEND_DESIGN 19.4. **Never log:** visitor tokens, token hashes, the operator key, visitor names, original file names, file contents, request bodies, full file-system paths. Log the **internal visitor id**, the **photo id**, the **error code**, and sizes and times.

```python
# Bad: leaks the name, the original file name and the whole headers
logger.info(f"Upload by {name}: {file.filename} headers={request.headers}")

# Good: only ids, a code and a size
logger.warning("file_rejected code=%s size=%s", error_code, size_bytes)
logger.info("photo_accepted photo_id=%s ms=%s", photo_id, elapsed_ms)
```

---

# 8. React Component Naming

The component tree and folders are fixed in **FRONTEND_DESIGN 8 and 9**. These are the naming rules that fit them.

| Rule | Level | Example |
|---|---|---|
| A component is a **function** with a **`PascalCase`** name. | MUST | `PhotoPreviews` |
| The file name **equals** the component name. One main component per file. | MUST | `PhotoPreviews.jsx` |
| Name it by **what it shows or does**, a noun phrase. | MUST | `ConsentNotice`, `UploadResults`, `OperatorKeyForm` |
| **Pages** end in `Page`. Only pages connect hooks. | MUST | `UploadPage`, `DisplayPage` |
| Reusable parts live in `components/ui/`. They have **no knowledge of photos or visitors**. | SHOULD | `Button`, `Banner` |
| **Callback properties** start with `on`. The function that handles it starts with `handle`. | MUST | `onRemove` (property), `handleRemoveClick` (handler) |
| **Boolean properties** read as questions: `isBusy`, `hasError`, `canUpload`. | MUST | `<Button isBusy />` |
| Do **not** put `Component`, `Container`, `Wrapper` or `Manager` in a name. | SHOULD NOT | They add nothing |
| Do **not** name a component after its **look** (`BlueBox`, `BigText`). Name it by **role**. | SHOULD NOT | Designs change |
| Keep a component **under about 150 lines** (FRONTEND_DESIGN 8.3). | SHOULD | Split it if it grows |
| **A component draws or connects, not both.** Small components get everything as properties. | MUST | FRONTEND_DESIGN 8.3 |

## 8.1 Component Shape

Use **function components**, **named exports in the file's own name**, **destructured props** with a clear list, and **no default values hidden deep inside**. Order inside a file: imports, the component, small local helpers below it.

```jsx
// Bad: connects and draws, calls the server directly, vague names
export default function Box({ d }) {
  const [x, setX] = useState([]);
  useEffect(() => { fetch("/api/photos/mine").then(r => r.json()).then(setX); }, []);
  return <div>{x.map(p => <img src={p.url} />)}</div>;
}

// Good: draws only. Data and events arrive as properties.
export function PhotoGrid({ photos, onRemovePhoto }) {
  return (
    <ul className="photo-grid">
      {photos.map((photo) => (
        <li key={photo.id}>
          <img src={photo.image_url} alt="Your uploaded photo" />
          <button type="button" onClick={() => onRemovePhoto(photo.id)}>
            Remove
          </button>
        </li>
      ))}
    </ul>
  );
}
```

| Rule | Level |
|---|---|
| A list item has a **stable `key`** from the data (`photo.id`). **Never** the array index for a list that can change. | MUST |
| Every `<img>` has an `alt`. Decorative images use `alt=""`. | MUST (FRONTEND_DESIGN 12) |
| A `<button>` that is not a form submit has `type="button"`. | MUST |
| Use real elements (`button`, `label`, `ul`) before `div` and `span`. | MUST |
| **No class components.** | MUST |

> **Note on named versus default export.** The example uses a **named** export so a search finds the exact name. Pick one style and keep it for the whole project. (CS-8, Section 21.)

---

# 9. React Hooks Usage

**Principle.** Hooks hold the **connection logic** (talking to the server, timers, who-am-I). Components stay simple. The hook list is fixed in FRONTEND_DESIGN 9 and 8.2.

## 9.1 Rules of Hooks (React's Own, Not Optional)

| Rule | Why |
|---|---|
| Call hooks **only at the top level** of a component or another hook. Never inside `if`, a loop, or a nested function. | React depends on the order of calls |
| Call hooks **only from** function components or custom hooks. | Same |

## 9.2 Project Rules

| Rule | Level | Reason |
|---|---|---|
| A **custom hook's name starts with `use`**, in `camelCase`, one per file. | MUST | Names tell React and the reader it is a hook |
| **Only hooks call `apiClient`.** Components never call the server. | MUST | One place for server rules (FRONTEND_DESIGN 8.3, 11) |
| A hook **returns an object** with named fields (not a long array) once it returns more than two things. | SHOULD | `const { photos, removePhoto } = useMyPhotos()` reads clearly |
| `useEffect` is **only** for starting and stopping things in the outside world: loading data, timers, listeners. **Not** for working out a value from other values. | MUST | The most common beginner bug |
| Every `useEffect` that starts a **timer, listener or request** **cleans up** in its returned function. | MUST | Otherwise: duplicate timers, memory growth on a TV running all day (FV-8) |
| The **dependency list** is complete and correct. Do not add or remove names to silence a warning without understanding why. | MUST | A wrong list causes stale data or loops |
| `useRef` holds things that must **not** trigger a redraw: timer handles, the **cursor**, the latest value inside a timer. | SHOULD | FRONTEND_DESIGN 10.2 |
| Do not use `useMemo`, `useCallback` or `React.memo` **unless a measured problem needs them**. | SHOULD NOT | Premature optimisation. The app is small. |
| Do not read or write browser storage inside components. Use `tokenStorage`. | MUST | It must never break the page (ERR-23) |

```jsx
// Bad: effect used to compute a value, no cleanup on a timer
function Counter({ photos }) {
  const [count, setCount] = useState(0);
  useEffect(() => { setCount(photos.length); }, [photos]);
  useEffect(() => { setInterval(poll, 5000); }, []);
}

// Good: value is just calculated; the timer is stopped on exit
function usePolling(callback, intervalMs) {
  const callbackRef = useRef(callback);
  callbackRef.current = callback;

  useEffect(() => {
    const timerId = setInterval(() => callbackRef.current(), intervalMs);
    return () => clearInterval(timerId);
  }, [intervalMs]);
}
// In a component: const count = photos.length;  (no state, no effect)
```

## 9.3 Loading Data in a Hook

| Rule | Level |
|---|---|
| A hook that loads data exposes **three** things: the data, a loading flag, and an error. Use the **same names** in every hook: `isLoading`, `error`, plus the data. | SHOULD |
| Ignore an answer that arrives **after the component is gone or the request is outdated** (a flag set in the cleanup). | MUST |
| Never start two identical requests because of a double click: the hook guards it (`isUploading`, FR-CNC-05). | MUST |
| The hook converts a failure into what the component needs. The **component** never reads `error.status`. It reads **`error.code`** (Section 13.2). | SHOULD |

---

# 10. State Management Conventions

**Decision FE-01 (approved): React's built-in tools only.** No Redux, Zustand or other store. (FRONTEND_DESIGN 10.)

## 10.1 Where State Lives

| Kind of state | Tool | Notes |
|---|---|---|
| A simple value (a field's text, dialog open) | `useState`, **in the component that uses it** | Keep state as close to its use as possible |
| A state with several steps (the upload: `idle`, `uploading`, `done`, `failed`) | `useReducer` in a hook | The reducer lists **every allowed move**. "What can happen next" is visible. |
| Starting and stopping outside things | `useEffect` | Section 9 |
| A value that must not redraw | `useRef` | Timer handles, the cursor |
| Settings loaded once from E1 | **One context** | The only context. Do not add another without approval. |
| The visitor token | `tokenStorage` | The **only** thing that survives a reload (FR-ID-01) |

## 10.2 Rules

| Rule | Level | Reason |
|---|---|---|
| **Never store a value you can calculate.** Do not keep `photoCount` in state when `photos.length` exists. | MUST | Two copies drift apart |
| **Never copy props into state** "to edit". Use the prop, or lift the state. | SHOULD | Stale copies |
| **State is never changed directly.** Make a new array or object: `[...photos, newPhoto]`, `photos.filter(...)`. | MUST | React will not redraw otherwise |
| Use **named status values**, not several booleans. One `status` of `"idle" \| "uploading" \| "done" \| "failed"` beats `isUploading`, `isDone`, `isFailed` (which allow impossible combinations). | SHOULD | Impossible states cannot exist |
| Status strings are **constants in one place**, not typed by hand in many files. | SHOULD | A typo then fails loudly |
| **Pure logic lives outside React**, in plain functions (`slideshowQueue.js`, `photoChecks.js`), so it can be tested with no browser. | MUST | TESTING_PLAN uses the built-in Node test runner for these |
| **Do not pass props through more than two levels.** If you do, rethink the split. | SHOULD | Avoids "prop drilling" |
| **No global mutable variables** in modules (`let currentToken`). | MUST | Hidden state |

```javascript
// Bad: impossible combinations allowed, direct mutation
const [isUploading, setIsUploading] = useState(false);
const [isDone, setIsDone] = useState(false);
photos.push(newPhoto);

// Good: one status, new array
const UPLOAD_STATUS = { IDLE: "idle", UPLOADING: "uploading", DONE: "done", FAILED: "failed" };
const nextPhotos = [...photos, newPhoto];
```

---

# 11. JavaScript Naming and Style

Plain **modern JavaScript (ES modules)**. No TypeScript in this MVP (TECH_STACK).

## 11.1 Naming

| Thing | Rule | Example |
|---|---|---|
| Variables, functions, parameters | `camelCase` | `photoCount`, `loadMyPhotos` |
| Functions | A **verb phrase** | `readToken`, `buildUploadForm`, `isFileTypeAllowed` |
| Booleans | `is`, `has`, `can`, `should` prefix | `isUploading`, `hasConsent` |
| Constants that never change | `UPPER_SNAKE_CASE` | `MAX_FILES_PER_REQUEST` |
| Event handlers | `handle` + what happened | `handleSubmit`, `handleRemoveClick` |
| Time and size values | Carry the **unit** | `pollIntervalMs`, `maxFileBytes`, `slideSeconds` |
| Error codes | The server's `UPPER_SNAKE_CASE` string, **copied exactly** | `"VISITOR_NOT_RECOGNISED"` |
| Arrays | Plural noun | `photos`, `results` |
| One item from an array | The singular | `photos.map((photo) => ...)` |

## 11.2 Names for Data That Comes From the Server (CS-3)

JSON fields are `snake_case` (API_SPECIFICATION 2.2). The frontend **does not convert them**. The rule is simple:

| Where the object was made | Field style | Example |
|---|---|---|
| **By the server** (any parsed JSON response) | `snake_case`, **unchanged** | `visitor.photo_count`, `feed.photo_ids`, `result.error.code` |
| **By our own frontend code** (state objects, the error object `apiClient` builds, props we define) | `camelCase` | `error.requestId`, `uploadState.selectedFiles` |
| **Our own local variables** that hold a value copied out of a server object | `camelCase` | `const photoCount = visitor.photo_count;` |

Reason: one conversion layer would be one more thing to explain, test and get wrong. If your editor warns about `snake_case` properties, switch that single check off for properties (Section 20).

```javascript
// Bad: inconsistent, half converted, and an unclear name
const v = await getVisitor();
const wrongCount = v.photoCount;    // undefined: the server sends photo_count
const IsNew = v.is_new;

// Good: server fields read as sent; our own names are camelCase
const visitor = await apiClient.getCurrentVisitor();
const photoCount = visitor.photo_count;
const isNew = visitor.is_new;
```

## 11.3 Style

| Rule | Level | Reason |
|---|---|---|
| `const` by default. `let` only when the variable is reassigned. **Never `var`.** | MUST | Prevents accidental change |
| `===` and `!==`. Never `==`. | MUST | No surprising type changes |
| Semicolons at the end of statements. Double quotes for text. 2-space indentation. | SHOULD | One style everywhere (CS-9) |
| **Template strings** for text with values: `` `Photo ${id}` ``. | SHOULD | Clearer than `+` |
| `async` / `await` with `try` / `catch` around **only** the call that can fail. Not `.then` chains. | SHOULD | Reads top to bottom |
| **Never leave an unhandled promise.** Every `await` or `.then` has a place where a failure is handled. | MUST | Silent failures on a TV are hard to see |
| **ES modules** (`import` / `export`). No `require`. No `window.something = ...`. | MUST | Normal for Vite |
| Arrow functions for short callbacks. A normal `function` for named, top-level functions. | MAY | Named functions show clearly in error traces |
| **No `console.log` in committed code**, except `console.error` for a real failure that the page cannot show. | SHOULD | Noise, and a risk of printing a token (FR-ID-05) |
| **No magic numbers or strings.** The limits come from **E1** (`/api/config`), not from constants typed in the frontend. | MUST | FRONTEND_DESIGN 4.3. The server owns the numbers. |
| Do not write clever one-liners. Two clear lines beat one clever line. | SHOULD | Live-demo explainability |

```javascript
// Bad: unhandled failure, == comparison, a limit typed into the frontend, var
var tooMany = files.length == 5;
fetch("/api/photos", { method: "POST", body: form });

// Good: limit from the settings, strict comparison, failure handled in one place
const isTooMany = files.length > config.max_files_per_request;
try {
  const outcome = await apiClient.uploadPhotos(form);
  handleOutcome(outcome);
} catch (error) {
  handleUploadFailure(error);
}
```

> **Check:** the field name `max_files_per_request` above is illustrative. Use the exact field names in API_SPECIFICATION E1.

---

# 12. CSS Organisation

The design system (tokens, spacing, components) is in **FRONTEND_DESIGN 2 and 3**. This section says how to write the CSS so it stays tidy. **Plain CSS only**: no Tailwind, no Sass, no CSS-in-JS (FRONTEND_DESIGN 9.2).

## 12.1 Files

| File | Holds |
|---|---|
| `tokens.css` | **Every design token**: colours, spacing, radius, motion, type scale. Nothing else. |
| `base.css` | Reset, page defaults, the focus ring, the reduced-motion rule |
| `ui.css` | The eight reusable parts (button, text field, checkbox, banner, dialog, header, progress, icons) |
| `upload.css`, `display.css`, `admin.css` | One page each |

Import order in `main.jsx`: `tokens.css`, `base.css`, `ui.css`, then the page files. Later files may use tokens from earlier ones, never the reverse.

## 12.2 Rules

| Rule | Level | Reason |
|---|---|---|
| **Use tokens, not raw values.** `color: var(--color-text);` not `color: #0F172A;`. `padding: var(--space-4);` not `padding: 16px;`. | MUST | One change updates the whole design. The contrast ratios were calculated for these tokens. |
| **Do not add a new colour, size or shadow** without adding a token and a reason. | MUST | Keeps the design consistent |
| Class names are **`kebab-case`** and describe the **role**, not the look: `photo-grid`, `banner`, `banner-error`, `is-busy`. Not `blue-box`, `mt-16`. | MUST | Designs change, roles do not |
| **Prefix a part's classes with its name**: `photo-grid`, `photo-grid-item`, `photo-grid-remove`. | SHOULD | Avoids clashes without extra tools |
| **State** classes start with `is-` or `has-` (`is-selected`, `has-error`). | SHOULD | Same words as the JavaScript booleans |
| **Style by class.** No styling by element id. **No `!important`.** | MUST | Both make later changes hard |
| **Keep selectors short**: one class, or one class plus one state or element. No chains deeper than two. | SHOULD | Easy to override and to read |
| **Mobile first.** Write the phone layout first, then add `@media (min-width: ...)` for `tablet` (600 px), `desktop` (1024 px), `wide` (1440 px). | MUST | FRONTEND_DESIGN 2.8 |
| **Do not remove the focus outline** (`outline: none`) unless something equally visible replaces it. | MUST | Keyboard users (FRONTEND_DESIGN 12) |
| Respect **reduced motion** by using the motion tokens, which become `0 ms` (FRONTEND_DESIGN 2.7). | MUST | Accessibility |
| **No inline `style` attributes** in JSX, except a value that is truly calculated at run time (for example a progress bar width). | SHOULD | Keeps look in the CSS |
| **No third-party fonts or images by URL.** Fonts are local (NFR-PRV-02). | MUST | The page needs no internet |
| Order inside a rule: layout (display, position, size), spacing, type, colour, other. | MAY | Easier to scan |
| Comment only to explain a **why** (a workaround, a TV quirk), not what a property does. | SHOULD | |

```css
/* Bad: raw values, look-based name, id selector, !important, desktop first */
#box1 { color: #0F172A; padding: 16px; margin-top: 16px !important; }
.blue-button { background: #1D4ED8; }
@media (max-width: 600px) { .grid { display: block; } }

/* Good: tokens, role-based names, one-level selectors, mobile first */
.photo-grid {
  display: grid;
  gap: var(--space-4);
  grid-template-columns: 1fr;
}

.photo-grid-item {
  border: 1px solid var(--color-border);
  border-radius: var(--radius-md);
  background: var(--color-surface);
}

@media (min-width: 600px) {
  .photo-grid { grid-template-columns: repeat(2, 1fr); }
}
```

> The TV page uses the **display theme tokens** (FRONTEND_DESIGN 2.2). Its colours are set once on the display page's root, so `display.css` still reads `var(--color-...)` with the same names.

---

# 13. API Response Consistency

The contract is **API_SPECIFICATION**. This section is the rule set that keeps backend and frontend code consistent with it. **The specification wins** if there is any difference.

## 13.1 Backend Rules

| Rule | Level | Reason |
|---|---|---|
| **Success is the bare resource.** No `{"data": ...}` and no `{"success": true}`. The status code says success or failure (API-04). | MUST | One simple rule |
| **Every error is** `{"error": {"code", "message", "details", "request_id"}}`, produced **only** by the error handlers. A route never builds an error body by hand. | MUST | The frontend has one error path |
| **Error codes come from the catalogue** (API_SPECIFICATION 4.3). A new code needs the catalogue and the tests updated **first**. | MUST | Codes never change meaning |
| The **message** comes from the catalogue. Numbers in messages (5 MB, 50, 5, 20) come from **settings**. | MUST | A changed limit changes the message |
| **JSON field names are `snake_case`.** Times are ISO 8601 UTC with `Z`. Sizes are bytes, durations are seconds. Missing is `null`, an empty list is `[]`. A documented field is **always present**. | MUST | API_SPECIFICATION 2.2 |
| **Every response has a `response_model`** (a Pydantic schema). Returning a model object or a dictionary straight out is not allowed. | MUST | The schema decides **exactly** which fields leave. This is how a token hash never leaks. |
| **E3 always returns `200`** with a `results` list, in the order the files were sent. A bad **file** is a result entry, a bad **request** is an error response. | MUST | API_SPECIFICATION E3 |
| **Status codes follow the catalogue.** Do not use `400` for a case that has its own code. | MUST | Consistency |
| **Image links are relative** (`/api/photos/121/image`). | MUST | No server address is configured (AD-16) |
| **No trailing slashes** and no extra routes "for convenience". | MUST | API_SPECIFICATION 2.1 |

```python
# Bad: invented shape, invented code, a model object returned directly
@router.delete("/api/photos/{photo_id}")
def delete_photo(photo_id: int, session=Depends(get_session)):
    photo = session.get(Photo, photo_id)
    if not photo:
        return {"success": False, "msg": "no such photo"}
    return photo

# Good: the service raises a catalogue code, the handler shapes the error,
# the route returns the declared schema
@router.delete("/api/photos/{photo_id}", response_model=DeletePhotoResult)
def delete_photo(
    photo_id: int,
    visitor: VisitorContext = Depends(require_visitor),
    session: Session = Depends(get_session),
) -> DeletePhotoResult:
    return photo_service.delete_own_photo(session, visitor, photo_id)
```

## 13.2 Frontend Rules

| Rule | Level | Reason |
|---|---|---|
| `apiClient` returns **parsed JSON** on success. On failure it throws **one kind of error object**: `code`, `message`, `details`, `requestId`, `status`. | MUST | FRONTEND_DESIGN 11.2 |
| A network failure or an unreadable body becomes the **client-side code `NETWORK_ERROR`** in the same object. | MUST | Same path for every failure |
| **Decide behaviour by `code`. Show `message`.** Never decide by the message text or by `status` alone. | MUST | Messages may be reworded. Codes never change meaning. |
| An **unknown code**: show the message, allow a retry, change nothing else. | MUST | FRONTEND_DESIGN 11.4 |
| **Ignore response fields you do not know.** | MUST | The server may add fields |
| Do not write server wording in the frontend for server errors. Show **the server's message**. (The early-check messages in FRONTEND_DESIGN 5.2 are the stated exception.) | SHOULD | One source of wording |
| **Never set the `Content-Type` header yourself** on the upload. The browser adds the boundary. | MUST | The most common upload mistake (API_SPECIFICATION 5.5) |

```javascript
// Bad: decides by message text, assumes a shape
if (error.response.data.msg.includes("token")) { logout(); }

// Good: decides by code, shows the server's message
switch (error.code) {
  case "VISITOR_NOT_RECOGNISED":
    clearTokenAndShowNewVisitorForm();
    break;
  default:
    showError(error.message);
}
```

## 13.3 Adding or Changing an Endpoint (Order of Work)

1. Change **API_SPECIFICATION** first (or confirm it already says this).
2. Add or update the **schema**.
3. Add the **service** function and its **test**.
4. Add the **route** (it only connects).
5. Add the **`apiClient`** function and use it from a **hook**.
6. Update **TESTING_PLAN** if the behaviour is new.

An assistant must **not** add an endpoint that is not in the specification (Section 0.2, rule 1).

---

# 14. Database Access Conventions

The design is **DATABASE_DESIGN** and **BACKEND_DESIGN 8, 9 and 12**. SQLAlchemy 2.0 style.

## 14.1 Naming (From DATABASE_DESIGN 5.1)

| Thing | Rule | Example |
|---|---|---|
| Tables | Plural `snake_case` | `visitors`, `consent_records`, `photos` |
| Columns | Singular `snake_case` | `token_hash`, `created_at` |
| Model classes | Singular `PascalCase` | `Visitor`, `ConsentRecord`, `Photo` |
| Constraints and indexes | **Written out by name** with a prefix | `ck_photos_status`, `uq_consent_one_open_per_visitor`, `ix_photos_status_id` |
| Status values | Lowercase words from a short list | `active`, `removed`, `withdrawn` |
| Repository functions | Start with the **verb that says what the query does** | `get_by_token_hash`, `list_active_ids`, `count_active_for_visitor`, `add`, `mark_removed` |

## 14.2 Query Rules

| Rule | Level | Reason |
|---|---|---|
| Write queries with **`select(...)`** and run them through the session. | MUST | SQLAlchemy 2.0 style (BACKEND_DESIGN 9.3) |
| **Never build SQL by joining or formatting strings with outside values.** Values are always **bound** by SQLAlchemy. | MUST | SQL injection (SECURITY_PRIVACY) |
| If raw SQL is ever unavoidable (for example the connection settings), it is a **fixed text with no outside value** and a comment explains why. | MUST | |
| **Queries live only in repository files.** Services and routes never import `select`. | MUST | One place for every query |
| A repository function does **one query's job** and has a name that says it. | MUST | Easy to find and test |
| **Select only what the use case needs**, and filter in the database, not in Python. (Count with a count query, not by loading rows.) | SHOULD | Simple and fast enough |
| **Always say which statuses you want.** A query for photos names `status == "active"` explicitly. | MUST | A removed photo must never leak (FR-PRV, FR-DSP-07) |
| **List queries have a defined order** (`order_by` photo id). | MUST | The feed and the cursor depend on it |
| **No lazy surprises.** Do not touch a related object after the session has closed. Services return Pydantic objects. | MUST | BACKEND_DESIGN 8.4 |
| Do not use `session.query(...)` (the old style). | SHOULD NOT | One style only |

```python
# Bad: string-built SQL with a value pasted in; query in a service; no order
def find_photos(session, visitor_id):
    return session.execute(
        f"SELECT * FROM photos WHERE visitor_id = {visitor_id}"
    ).fetchall()

# Good: bound value, explicit status, explicit order, in a repository
def list_active_for_visitor(session: Session, visitor_id: int) -> list[Photo]:
    statement = (
        select(Photo)
        .where(Photo.visitor_id == visitor_id, Photo.status == "active")
        .order_by(Photo.id)
    )
    return list(session.scalars(statement))
```

## 14.3 Sessions and Transactions (BACKEND_DESIGN 8.3 and 8.4)

| Rule | Level | Reason |
|---|---|---|
| **Services commit. Repositories never commit.** A repository may `flush` to learn a new id. | MUST | AD-04 |
| **One unit of work, one commit.** The units are listed in BACKEND_DESIGN 8.4 (register, one photo, delete, withdraw, operator removal). | MUST | All-or-nothing changes |
| **Keep write transactions short.** Do the image work and file writing **before** opening the write. | MUST | SQLite allows one writer |
| **The session comes from the dependency.** Never create a session by hand in a route or service. Never keep a session in a global or share it between threads. | MUST | It is closed and rolled back for you |
| **Files and database in the safe order.** Add: file first, row last, delete the file if the row fails. Remove: row first, file after the commit. | MUST | AD-13 and AD-14 |
| **Foreign keys must be on for every connection.** This is set once in `db/database.py`. Do not turn them off for convenience. | MUST | DATABASE_DESIGN and BACKEND_DESIGN 8.2 |

## 14.4 Changing the Schema

| Rule | Level |
|---|---|
| The schema is changed **in DATABASE_DESIGN first**, then in the model, then in `init_db`. | MUST |
| SQLite cannot alter most constraints in place. A change to a `CHECK` is a **new table, copy, rename**. Plan it, or avoid it. | MUST |
| No migration tool is added in the MVP. For the demo, the database can be recreated. **Say so**, do not pretend a migration exists. | SHOULD |
| Never edit the **real** database by hand during a demo and then forget. Use a copy. | SHOULD |

---

# 15. Comments and Documentation

## 15.1 Principle

**Code says what. Comments say why.** If a comment only repeats the code, delete it or rename something so the comment is not needed.

## 15.2 Comments

| Rule | Level | Example |
|---|---|---|
| Comment **why**: a decision, a trap, a rule from a document, a workaround. | MUST | `# Row first, file after commit (AD-14): a crash must not leave a visible photo without a file.` |
| **Link to the requirement** when a line exists because of one: `FR-DEL-05`, `AD-13`, `ERR-16`. | SHOULD | Shows the code is not invented |
| Do **not** comment obvious lines. | MUST NOT | `# add one to count` |
| Mark unfinished or doubtful work `TODO(name): ...` with a reason, and **list it in the commit or the README**. No bare `TODO`. | SHOULD | |
| **Delete commented-out code.** | MUST | Git keeps history |
| Mark anything that depends on an **unverified framework behaviour** with the existing label `to verify` and the item number (for example `BV-2`). | SHOULD | Matches BACKEND_DESIGN 24.3 |
| Comments are in **plain English**, short, and wrap at 79. | SHOULD | |

## 15.3 Docstrings (Python)

Short and only where they help. A one-line docstring in plain words is enough for most public functions in `services/`, `repositories/` and `core/`. Longer text only for a rule that is **not obvious** from the name and hints.

```python
# Bad: repeats the signature, wastes lines
def count_active_for_visitor(session: Session, visitor_id: int) -> int:
    """
    This function counts active photos for a visitor.

    Args:
        session: the session
        visitor_id: the visitor id

    Returns:
        the count
    """

# Good: one line, says what the name cannot
def count_active_for_visitor(session: Session, visitor_id: int) -> int:
    """Count a visitor's photos that are still shown (status active)."""
```

| Rule | Level |
|---|---|
| Docstring style: **triple double quotes**, an imperative first line ending with a full stop (`"""Return ..."""`). | SHOULD |
| **No docstring needed** for tests (the name says it), for one-line private helpers, or for routes (the response model and the generated API page document them). | MAY |
| In React, a comment above a component or hook: **one line saying its job**, matching FRONTEND_DESIGN 8.3 ("one clear job in one sentence"). | SHOULD |

## 15.4 Project Documents

| Rule | Level |
|---|---|
| **Update the document when the code changes the behaviour.** README, API_SPECIFICATION, DATABASE_DESIGN, TESTING_PLAN. | MUST |
| **README.md** says how to install, run, test and demo, in the order a stranger would do it. Commands are tested by actually running them before they are written. | MUST |
| **AI_LOG.md** records the assistant's mistakes **when they happen**: what it got wrong, how you noticed, how you fixed it (IMPLEMENTATION_PLAN rule 6). | MUST |
| **.env.example** lists every setting with a **placeholder**, never a real secret (NFR-SEC-02). | MUST |
| Never write "this works" in a document until it has been **run**. Write what was checked and what was not. | MUST |

---

# 16. Code Duplication Prevention

**Principle.** Duplicated code is changed in one place and forgotten in the other. But **a wrong abstraction is worse than a copy**. Aim for the middle.

## 16.1 The Rule of Three

| Count | Action |
|---|---|
| First time | Write it plainly |
| Second time | A copy is acceptable. Note it (for example in the commit body) so that you remember. |
| Third time | **Extract it** into one named function, constant or component |

**Exception, extract at once:** anything that is a **rule, limit, message, or code** from a document. These must exist **once**.

## 16.2 Where Shared Things Live

| Duplicated thing | One home |
|---|---|
| A limit (5 MiB, 5 files, 20 photos, 50 characters, 25 MP) | **Settings** (`core/settings.py`), exposed to the frontend by **E1** |
| An error code, its status and its message | The **error catalogue** (`core/errors.py`) |
| The consent notice text and its version | `core/consent_notice.py` |
| Token creation, hashing, key comparison | `core/security.py` |
| The standard error shape and time format | `schemas/common.py` |
| Safe file writing, reading and deleting | `storage/photo_storage.py` |
| A repeated query | A **repository function** |
| Building a test image or request | `tests/helpers/` and **fixtures** in `conftest.py` |
| A repeated piece of screen | A **component** (a `ui/` part if it knows nothing about photos) |
| A repeated piece of connection logic | A **hook** |
| A repeated colour, size, gap | A **token** in `tokens.css` |
| The server address and headers | **`apiClient`** only |
| A repeated pure rule on the display or in checks | `slideshowQueue.js`, `photoChecks.js`, `format.js` |

## 16.3 Bad and Good

```python
# Bad: the same limit typed in three files; if the limit changes, one is forgotten
if len(files) > 5: ...                     # photo_routes.py
if count + len(files) > 20: ...            # photo_service.py
MAX_BYTES = 5 * 1024 * 1024                # image_processor.py

# Good: one source
if len(files) > settings.max_files_per_request:
    raise DomainError("TOO_MANY_FILES")
```

## 16.4 When Not to Merge

| Do not merge when | Reason |
|---|---|
| The two pieces look alike **today** but belong to different features and will change for different reasons. | They will drift apart, and the shared version gets flags and special cases. |
| Merging needs a **boolean parameter** that switches behaviour (`do_thing(x, is_special=True)`). | That is two functions. |
| The shared function would need **more explaining** than the two copies. | Explainability wins. |

## 16.5 Spotting Duplication from an Assistant

AI assistants **repeat** code freely because it is easy for them. In review (Section 19), search for: the same number or message in two files, two functions with the same body, a second `fetch` call outside `apiClient`, a second place that builds an error response, a second place that hashes a token.

---

# 17. Security Coding Practices

The full design and the threat table are in **SECURITY_PRIVACY**. This section is the **coding checklist** that comes from it. It makes no claim that the application is fully secure.

## 17.1 The Rules

| # | Rule | Level | Reason |
|---|---|---|---|
| 1 | **Never trust the browser.** Every check in the frontend is repeated on the server. | MUST | Frontend checks are conveniences only |
| 2 | **Validate every input at the edge**: name (1 to 50 characters, no control characters), file count, size, type, **real image content**, pixel count, ids. | MUST | FR-VAL |
| 3 | **Decide image type by its content**, never by the file name or the declared type. | MUST | Disguised files (see the upload threats in SECURITY_PRIVACY) |
| 4 | **Re-encode every image** with Pillow and store the new file. Never store the original bytes. | MUST | Removes hidden data and metadata (FR-STO-03) |
| 5 | **Random server-made file names** (32 hex characters). The visitor's file name is **never** used in a path. | MUST | No path tricks, no overwrite (FR-CNC-02) |
| 6 | **Check ownership in the service** for every delete and own-photo read. | MUST | FR-DEL-02 |
| 7 | **Compare secrets in constant time**: `hmac.compare_digest` for the operator key. | MUST | No timing leak |
| 8 | **Make tokens with the `secrets` module** (256-bit, URL-safe). **Store only the SHA-256 hash.** Never use `random` for secrets. | MUST | Approved design |
| 9 | **Return generic errors.** No path, SQL, stack trace, token or hash in a response (NFR-SEC-04). | MUST | |
| 10 | **No secrets in the code, the repository, the frontend build or the logs.** Settings come from the environment. `.env` is ignored by Git. | MUST | NFR-SEC-02 |
| 11 | **Public image endpoints answer only for active photos of active visitors**, and give `404` otherwise. | MUST | A removed photo must not be reachable |
| 12 | **Set the approved headers** (`nosniff`, the cache decision, the content-security policy if built). Do not weaken them to make something work. | MUST | SECURITY_PRIVACY decisions |
| 13 | **Do not add a library, endpoint or permission** that is not approved. Say why and ask. | MUST | Smaller surface |
| 14 | **No `eval`, `exec`, `pickle`, `os.system` or shell commands** built from input. | MUST | |
| 15 | **Keep dependencies pinned**, and run the dependency check before submission. Record the output honestly. | SHOULD | TESTING_PLAN security tests |

## 17.2 Tokens and Keys

| Rule | Level |
|---|---|
| The token travels **only** in the `X-Visitor-Token` header. **Never** in a URL, a log, an error, a page, or browser address history. | MUST |
| The operator key travels **only** in `X-Operator-Key`. It is **never** in local storage or a URL. The route answers `404` when no key is configured. | MUST |
| A **wrong** key or token is logged as a **count only**. The value is never written. | MUST |

## 17.3 Uploads (Order Matters)

The approved order is in BACKEND_DESIGN 14 and 15. Do not reorder or skip a step. In short: size check before decoding, content check before saving, re-encode, save under a random name, **then** commit the row. A failed step leaves **no file and no row**.

## 17.4 Output: Showing Visitor Text Safely

| Rule | Level |
|---|---|
| In React, show visitor text as normal **text** in JSX (`{visitor.name}`). **Never** use `dangerouslySetInnerHTML`. | MUST |
| **Never** build HTML by joining strings with visitor text. | MUST |
| **Never** use `innerHTML`, `document.write` or `eval`. | MUST |
| Never set a link or image source from visitor text. Image links come from the server's `image_url` field. | MUST |

```jsx
// Bad: a name like <img src=x onerror=...> would run
<div dangerouslySetInnerHTML={{ __html: `Welcome back, ${visitor.name}` }} />

// Good: the name is only ever text
<p>Welcome back, {visitor.name}</p>
```

## 17.5 Paths

When code touches the file system, build the path from the **configured folder plus a server-made name**, using `pathlib`, and confirm that the final path is still **inside** that folder before reading or deleting. Never join a path from request input.

```python
# Bad: a visitor-supplied name decides where the file goes
target = f"data/photos/{upload.filename}"

# Good: server-made name inside the configured folder
stored_name = f"{secrets.token_hex(16)}.jpg"   # 32 hex characters
target = photos_dir / stored_name
```

## 17.6 Privacy in Code

| Rule | Level |
|---|---|
| **Collect nothing extra.** No new personal field, no analytics, no third-party script, font or image (NFR-PRV-02). | MUST |
| **Withdrawal really erases**: name and token hash set to `NULL`, photos marked removed, consent closed, files deleted after the commit. Do not "soft keep" the name anywhere (a log, a cache, a test file). | MUST |
| **Never log** names or original file names. | MUST |
| Do not use real people's photos or names in tests, examples or the repository. Use generated images. | MUST |

## 17.7 Honest Wording

Do not write comments, documents or messages that say the app is "secure", "safe" or "compliant". Say **what was done and what was tested**. The honest list of limits is SECURITY_PRIVACY.

---

# 18. Git Commit Message Conventions

**Approved already:** one task, one commit (at least), a message that says **what changed and why** (IMPLEMENTATION_PLAN). The shape below is a proposal (CS-5).

## 18.1 Format

```text
<type>(<area>): <short summary in the imperative, no full stop>

<optional body: why this change, and what was checked>
```

| Part | Rule |
|---|---|
| **Summary** | Imperative mood ("add", "fix"), lowercase after the type, **about 50 characters, at most 72**, no full stop |
| **Body** | Optional. Wrap at 72. Say **why**, not a list of every line changed. Say **how it was checked** and **what was not**. |
| **Blank line** | Between summary and body |
| **One purpose per commit.** A commit that does two things is two commits. | |

## 18.2 Types and Areas

| Type | Use for |
|---|---|
| `feat` | A new behaviour a visitor or the operator can see |
| `fix` | A bug fix |
| `test` | Adding or changing tests only |
| `docs` | README, ASSUMPTIONS, AI_LOG, and other documents |
| `refactor` | Changing structure with **no change in behaviour** |
| `style` | Formatting only, no change in meaning |
| `chore` | Set-up, dependencies, configuration |

| Area (the layer or the page) | Examples |
|---|---|
| `api`, `service`, `repo`, `model`, `schema`, `core`, `storage`, `db` | `feat(service): add photo upload limits` |
| `ui`, `upload-page`, `display`, `admin`, `hooks`, `css` | `feat(display): poll the feed every 5 seconds` |
| `tests`, `readme`, `plan` | `docs(readme): add run and test steps` |

## 18.3 Examples

```text
# Bad
update stuff
fixed bug
WIP
Claude made changes to everything

# Good
feat(service): reject files that are not real images
fix(repo): list only active photos in the slideshow feed
test(api): cover oversized upload and the 413 response
docs(ai-log): record wrong cascade rule suggested by the assistant
chore(backend): pin dependency versions
```

A good body:

```text
fix(service): clear name and token hash on consent withdrawal

The visitor row kept the name after withdrawal, which breaks FR-WDR-03.
Name and token_hash are now set to NULL in the same transaction that
marks the photos removed.

Checked: test_withdrawal_clears_visitor passes locally (output recorded
in TESTING_PLAN 15). Not checked: behaviour with two simultaneous
withdrawals.
```

## 18.4 Rules

| Rule | Level | Reason |
|---|---|---|
| **Commit small and often**: after each working step, not at the end of the day. | MUST | Reviewers and the assignment look at the history |
| **Never commit** secrets, `.env`, the database, photos, logs, `node_modules`, or the virtual environment. Check `git status` before every commit. | MUST | NFR-SEC-02 |
| **Do not commit code that does not start** or breaks the tests that passed before. | SHOULD | Every commit is a safe point |
| **A message must be true.** Do not write "tested" unless tests were run. Write "not tested" when that is the case. | MUST | The same honesty rule as the documents |
| **AI help is recorded in `AI_LOG.md`**, not hidden. If an assistant wrote most of a change, the body may say so. | SHOULD | The assignment requires a log |
| **Do not rewrite shared history** (`push --force`) on the submission branch. | MUST | Keeps the record intact |
| Use a **short-lived branch** per task if you like (`feat/upload-limits`), merged when it works. For one developer, committing straight to `main` is acceptable. | MAY | Simple is fine |
| Commit messages made by an assistant must follow the same format. **Re-read and edit them.** | MUST | You must be able to explain every commit |

---

# 19. Review Checklist for Every Change (Human or AI)

Use this before each commit. It is also a prompt for asking Claude or Gemini to **review** a change.

| # | Question | Pass? |
|---|---|---|
| 1 | Does the change match **one task** and only the files that task needs? | |
| 2 | Does it match the **approved documents** (no invented endpoint, field, code or file)? | |
| 3 | Can I explain **each new function and file** in one sentence? | |
| 4 | Are names complete, in the right case, and in the project's vocabulary (Section 1)? | |
| 5 | Layers respected: no database in routes, no FastAPI in services, no commit in repositories? | |
| 6 | Every Python function has type hints and returns a schema (service) or model (repository)? | |
| 7 | No bare `except`, no hidden failure, cleanup before re-raise, catalogue codes only? | |
| 8 | Logging: `logging`, `key=value`, **no** token, hash, name, file name, path? | |
| 9 | Every limit and message from settings or the catalogue, **not typed in again**? | |
| 10 | Queries use `select()` with bound values, explicit status, explicit order? | |
| 11 | Frontend: component draws or connects, only hooks call `apiClient`, effects clean up? | |
| 12 | Visitor text shown as **text**. No `dangerouslySetInnerHTML`. | |
| 13 | CSS uses tokens, role-based class names, mobile first, no `!important`? | |
| 14 | No leftover `print`, `console.log`, commented-out code, or stray `TODO`? | |
| 15 | A test exists for the new behaviour (TESTING_PLAN), and it was **actually run**? | |
| 16 | No secret, database, photo or log in `git status`? | |
| 17 | Documents updated if behaviour changed? `AI_LOG.md` updated if the assistant was wrong? | |
| 18 | The commit message follows Section 18 and is **true**? | |

---

# 20. Tools (Optional, Not Required)

CF-1: the approved TECH_STACK names none of these. They are **suggestions** that make the rules automatic. **Do not spend more than about 20 minutes** setting any of them up. If a tool is installed, record it in `requirements.txt` or `package.json` and the README, and **run it before committing**.

| Tool | Purpose | Notes |
|---|---|---|
| **Ruff** (Python) | Checks PEP 8 style, unused imports, import order, and formats code | One tool for the job of several. Set line length to 88. |
| **mypy** (Python) | Checks type hints | Optional. Never fight it for hours (Section 4.3). |
| **Prettier** (JavaScript, CSS) | Formats code | Semicolons on, double quotes, 2 spaces |
| **ESLint** (JavaScript, React) | Finds mistakes, enforces the Rules of Hooks | Use the **React hooks rules** (they catch wrong dependency lists). Turn the `camelcase` check off for object properties (Section 11.2). |
| **EditorConfig** | Same indentation and line endings in every editor | A tiny file, no installation |
| **Pre-commit hook** | Runs the formatter before each commit | **Future.** Not needed for the MVP. |

> No tool was installed or run in writing this document. The statements about what each does are general knowledge, not results.

---

# 21. Decisions, Open Items and Limits

## 21.1 Decisions Needing Approval

| ID | Decision | Alternative | Recommendation |
|---|---|---|---|
| **CS-1** | Formatter and linter are **optional** (Section 20), not required | Require Ruff, Prettier and ESLint from Day 1 | Optional. Setup time is better spent on features. Revisit if style drifts. |
| **CS-2** | Python version fixed on **Day 1** and written in the README. Examples use 3.10+ syntax. | Use `typing.List` and `Optional` for older versions | Use the new syntax |
| **CS-3** | Server objects keep `snake_case` fields in the frontend. No conversion layer. | Convert to `camelCase` in `apiClient` | No conversion. Less to explain and test. |
| **CS-4** | 88 characters for code, 79 for comments and docstrings | 79 everywhere | 88 / 79 |
| **CS-5** | Light Conventional Commits (`type(area): summary`) | Free-form messages | Use the format. It costs seconds and reads well on GitHub. |
| **CS-6** | Paste the Section 0.2 block into every Claude or Gemini session | Rely on the assistant reading the whole document | Use the block |
| **CS-7** | Where a design document already has a rule, this document points to it and the **design document wins** | Copy all rules here | Point, do not copy |
| **CS-8** | **Named exports** for components, hooks and library functions (`export function PhotoGrid`). Entry files such as `App.jsx` may use a default export. | Default exports everywhere | Named, so a search finds the exact name |
| **CS-9** | JavaScript style: semicolons, double quotes, 2-space indent | Prettier defaults or no semicolons | As written. Matches the Python quote style. |

## 21.2 What This Document Does Not Do

| Not covered | Why |
|---|---|
| **No application code.** All code here is illustration. | The request |
| **No proof that the examples run in the app.** Only syntax was checked, and only for Python and plain JavaScript. | No application, and no FastAPI, SQLAlchemy or httpx in the writing environment |
| **No claim of fully secure code.** Following the rules reduces common mistakes. It does not make the app secure. | SECURITY_PRIVACY |
| **No accessibility or testing rules in full.** These are in FRONTEND_DESIGN 12 and TESTING_PLAN. | Avoid copies that drift (CS-7) |
| **No performance rules** beyond "do not optimise early". | MVP scope |

## 21.3 Likely Review Questions (Practise Your Answers)

| Question | Short answer |
|---|---|
| Why are there almost no classes in the services? | A module of functions is simpler to read and test (BACKEND_DESIGN 12.1). Classes are used where data has a shape: models, schemas, errors, settings. |
| Why do you not convert `snake_case` to `camelCase`? | The server's names pass through unchanged. One less layer to explain, and no mapping bugs (CS-3). |
| Why do services commit and repositories not? | One place for transaction behaviour. A use case either fully succeeds or fully rolls back (AD-04). |
| Why no Redux? | Nothing is shared between pages. Each page's state is small and local (FE-01). |
| How do you know the AI's code is correct? | I read it, I run the tests, I check it against this document and the design documents, and I record the AI's mistakes in `AI_LOG.md`. |

---

# Glossary

| Term | Meaning |
|---|---|
| **PEP 8** | Python's official style guide |
| **Type hint** | A note on a function saying what kinds of value go in and come out |
| **Domain error** | An error the application raises on purpose, with a code from the catalogue |
| **Bound value** | A value that SQLAlchemy passes to the database separately from the SQL text, so it cannot change the query |
| **Component** | A React function that returns a piece of screen |
| **Hook** | A React function whose name starts with `use`, used to hold behaviour |
| **Prop (property)** | A value or function handed to a component |
| **Token (CSS)** | A named design value such as `--space-4` |
| **Magic number** | A number typed into code with no name explaining it |
| **Rule of three** | Copy twice, extract on the third use |
| **Conventional Commits** | A common way to write commit messages as `type(area): summary` |

---

# Document Control

| Item | Detail |
|---|---|
| Document | CODING_STANDARDS.md |
| Version | 1.0 (draft for approval) |
| Date | 17 November 2026 |
| Status | Draft. Awaiting approval of CS-1 to CS-9. |
| Next review | After Day 2 (backend done) and after Day 4 (frontend done), to correct any rule the real code shows to be impractical |
| Changes | Any change to a rule that is also in a design document is made in **both** places (CS-7) |
