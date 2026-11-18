# PROJECT_BRIEF.md

# MomentVault: Your Digital Time Capsule

| Item | Detail |
|---|---|
| Project type | MCA Semester 1 academic assignment (Assignment 1: Visitor Photo Slideshow), built to portfolio standard |
| Version | MVP brief v1.0 |
| Date | 13 November 2026 |
| Planned effort | About 20 focused hours over 5 days |
| Status | Draft for approval before architecture and coding begin |

**How to read this document.** Every feature is labelled one of three ways:

- **[REQUIRED]** is stated in the original assignment.
- **[DESIGN DECISION]** is something the assignment says the design must address, so a choice has to be made and recorded.
- **[PROPOSED]** is an optional enhancement that I suggest. It is not in the assignment and is only built if time allows.

Nothing in this document should be treated as a requirement unless it carries the first or second label.

---

## 1. Executive Summary

MomentVault is a web application for events, offices, labs or exhibitions where people visit and share photos. A visitor opens the upload page, enters their name, agrees to a consent notice and uploads one or more photos. As soon as the upload completes, the photo appears in a slideshow on a separate TV or computer display. If the same person visits again, the slideshow also plays their earlier photos.

The application is built with Python (FastAPI), React (Vite) and SQLite (SQLAlchemy). It is scoped so that one beginner developer, using AI coding assistants, can build, test, document and demonstrate it in about 20 hours.

The brand name "MomentVault" and the tagline "Your Digital Time Capsule" describe how the app is presented: moments are stored safely and can come back later. The assignment defines the actual behaviour, and this brief follows the assignment.

## 2. Problem Statement

At shared places such as events or reception areas, visitors often take photos but there is no simple way to show them together on a shared screen. Common problems are:

- Sharing photos usually needs accounts, apps or cables.
- A shared display needs someone to load photos manually.
- Returning visitors are not recognised, so their earlier contributions are lost.
- Photos of people are shown in public, which raises privacy and consent questions.
- Uploads from many people at once can fail or corrupt data if they are not handled carefully.

The assignment asks for an application that solves these problems and also shows good engineering judgement. The assignment's grading covers design decisions, how the AI was directed and how its mistakes were caught and fixed, not only whether the app works.

## 3. Proposed Solution

MomentVault has two main screens connected by one backend:

1. **Upload page [REQUIRED].** This runs on the visitor's phone or computer. It asks for the visitor's name and photos.
2. **Display page [REQUIRED].** This runs full screen on a smart TV or computer. It plays a slideshow, and each photo is shown for a set duration.
3. **Backend API [REQUIRED, implied].** This receives uploads, validates them, stores them, identifies returning visitors and tells the display page which photos to show.

**How it works, in plain language**

1. A visitor opens the upload page and enters their name.
2. The visitor reads and accepts the consent notice [DESIGN DECISION].
3. The visitor selects photos and uploads them.
4. The server checks each file (type, size, whether it is a real image), stores it and records it in the database.
5. The display page picks up the new photo and shows it for the configured duration [REQUIRED].
6. When a returning visitor uploads, the display also plays that visitor's previous photos [REQUIRED].

## 4. Target Users and User Roles

| Role | Who they are | What they do | Status |
|---|---|---|---|
| **Visitor** | A person at the event or venue | Enters name, gives consent, uploads photos | [REQUIRED] |
| **Display screen** | A TV or computer showing the slideshow, with no login and no interaction | Plays photos | [REQUIRED] |
| **Returning visitor** | A visitor who has uploaded before | Same as visitor, and their older photos also play | [REQUIRED] |
| **Operator / organiser** | The person running the screen | Removes an unwanted photo | [PROPOSED] |

The assignment does not mention an administrator. If an Operator role is added, it should stay minimal (see Section 7).

## 5. Project Objectives

**Product objectives**

1. Deliver the behaviour described in the assignment: name and photo upload, instant slideshow, returning-visitor playback.
2. Make the three design areas explicit and defensible: returning-visitor identification, privacy and consent, and file validation with upload limits and concurrent uploads.

**Learning and academic objectives**

3. Be able to explain every part of the submission in a live demo.
4. Show regular, meaningful Git commits.
5. Record every assumption in `ASSUMPTIONS.md` and key AI interactions, including at least two AI mistakes and their fixes, in `AI_LOG.md`.
6. Practise professional habits: documentation, automated tests, secret handling and clear architecture.

## 6. Core Features

### 6.1 Required features

| ID | Feature | Source |
|---|---|---|
| F1 | Visitor is asked for their name and photos | [REQUIRED] |
| F2 | Photo appears in the slideshow as soon as its upload completes | [REQUIRED] |
| F3 | Each photo plays for a specific, set duration on a TV or computer display | [REQUIRED] |
| F4 | Returning visitors' previous photos are also played | [REQUIRED] |

### 6.2 Design areas the assignment says must be addressed

| ID | Area | What must be decided and documented |
|---|---|---|
| D1 | Returning-visitor identification | How the system decides that a visitor has been here before. A name alone is not reliable, because two people can share a name and one person can type it differently. The chosen method and its weaknesses must be documented. |
| D2 | Privacy and consent | What the visitor is told, what they agree to, how long photos are kept and what control they have, because photos of people are displayed publicly. |
| D3 | File validation | Which file types are accepted, how the server confirms a file is really an image and what happens to invalid files. |
| D4 | Upload size limits | Maximum size per file and per request, and the message shown when a limit is exceeded. |
| D5 | Simultaneous uploads | What happens when several people upload at once, including safe file naming and safe database writes. |

### 6.3 Proposed approach for the design areas (for your approval)

These are recommendations, not requirements. Final choices go in `ARCHITECTURE.md` and `ASSUMPTIONS.md`.

- **D1.** Combine the name with a random visitor token that the browser stores. This allows a returning visitor to be recognised on the same device. Limitation: a different device or cleared browser data means the visitor looks new.
- **D2.** A consent checkbox with a short plain-language notice, required before upload. The notice states that photos will be shown publicly on a screen. Visitors who do not tick the box cannot upload.
- **D3.** Accept JPEG and PNG only. Check the file extension, the content type and the actual image content using Pillow. Reject anything that is not a real image. Store files under server-generated names, never under the visitor's file name.
- **D4.** Set a per-file and a per-request limit. Reject larger files with a clear error message.
- **D5.** Give each upload a unique generated file name. Let SQLite handle writes through SQLAlchemy sessions. Cover this with a test that sends several uploads at the same time.

### 6.4 Proposed enhancements [PROPOSED]

Only build these after all required features and tests are complete.

- Operator can remove a photo from the display.
- Visitor can delete their own photos.
- Slideshow shows the visitor's name as a caption.
- Image resizing for smoother display.

## 7. Scope and Out-of-Scope Features

### In scope for the MVP

- Upload page with name field, consent checkbox and photo selection.
- Display page with a full-screen slideshow.
- Backend API with validation, storage and retrieval.
- SQLite database for visitors and photos.
- Returning-visitor playback.
- Documented privacy and consent approach.
- Automated tests for the main features.
- Required repository documents (Section 9).

### Out of scope for the MVP

These are not requested by the assignment and are excluded to protect the 20-hour limit:

- User accounts, passwords and login systems.
- Video uploads.
- Social features such as likes, comments and sharing.
- Face recognition or any automated identification of people.
- Cloud storage services and paid infrastructure.
- Native mobile apps.
- Advanced admin dashboards and analytics.
- Automated content moderation using AI.
- Multi-event or multi-tenant support.

## 8. Technology Overview

| Technology | Role in the project | Why it suits a beginner |
|---|---|---|
| **Python** | Backend language | Readable, and widely supported by AI assistants |
| **FastAPI** | Web API framework | Simple routes, automatic input validation and built-in interactive API docs |
| **SQLAlchemy** | Database access layer | Lets the app work with Python classes instead of raw SQL |
| **SQLite** | Database | A single file, with no server to install, so setup is simple |
| **Pillow** | Image checking and processing | Confirms a file is a real image |
| **React** | Frontend UI | Organises the upload and display pages as components |
| **Vite** | Frontend build tool | Fast setup and a simple development server |
| **Pytest** | Automated testing | Simple test syntax and good FastAPI support |
| **Git / GitHub** | Version control and submission | Required for the commit history and public repository |
| **Claude and Gemini** | AI coding assistants | Allowed by the assignment, and every use must be explainable |

Detailed reasons for these choices, the data model and diagrams belong in `ARCHITECTURE.md`.

## 9. Expected Deliverables

| Deliverable | Content | Required by assignment |
|---|---|---|
| Public GitHub repository | Accessible without login | Yes |
| `README.md` | What the app does, tech stack, setup and run instructions, deployed link if available | Yes |
| `ASSUMPTIONS.md` | Every gap filled and why that option was chosen | Yes |
| `AI_LOG.md` | 5 to 10 key prompts or interactions, including at least 2 cases where the AI was wrong, how it was noticed and how it was fixed | Yes |
| `ARCHITECTURE.md` | Stack choice with reasons, data model and at least one diagram | Yes |
| Tests | Automated tests, or a documented manual test plan with results | Yes |
| `.env.example` | Placeholder values only, with no real secrets | Yes |
| Commit history | Regular, meaningful commits showing how the project evolved | Yes |
| Working application | Upload page, display page and backend | Yes |
| Live demo readiness | Ability to explain every part | Possible |
| `PROJECT_BRIEF.md` | This document | Project planning, not listed by the assignment |

## 10. Success Criteria

The MVP is successful when all of these can be shown:

**Functional**
1. A visitor can enter a name, consent and upload a photo.
2. The photo appears on the display page soon after the upload finishes, and plays for the configured duration.
3. A returning visitor's earlier photos also play in the slideshow.
4. Invalid files (wrong type, fake image, too large) are rejected with a clear message.
5. Several simultaneous uploads complete without lost photos or crashes.

**Quality**
6. Automated tests exist for upload validation, size limits, returning-visitor identification and concurrent uploads, and they pass.
7. No secrets are committed, and `.env.example` contains only placeholders.

**Documentation and process**
8. All required documents exist and are accurate.
9. The Git history shows regular, meaningful commits.
10. The developer can explain every major component in a live demo.

## 11. Project Constraints

| Constraint | Detail |
|---|---|
| Time | About 20 focused hours over 5 days |
| Experience | Beginner developer, so implementations must be simple and understandable |
| Technology | Python, FastAPI, React, Vite, SQLite and SQLAlchemy |
| Dependencies | Keep to a minimum, and justify each one |
| Budget | Free tools only |
| Team | One developer assisted by AI tools |
| Explainability | Every part of the code must be explainable by the developer |
| Security | No secrets in the repository, and uploads must be treated as untrusted input |
| Scope | The assignment is the source of truth, and no invented requirements |

## 12. Known Limitations

These are accepted trade-offs of the MVP and should be stated honestly in the README:

1. **Returning-visitor identification is not perfect.** A token stored in the browser is lost if the visitor changes device or clears browser data.
2. **Names are not unique.** Two people can enter the same name, so the name alone cannot identify a visitor.
3. **SQLite is not designed for heavy concurrent writing.** It is suitable for a small event but not for large-scale use.
4. **Local file storage.** Photos live on the server's disk, so they are lost if the server is reset, and the setup is not suited to large scale.
5. **No automatic content moderation.** Inappropriate photos could be displayed unless an operator removes them, which is why the operator feature is worth considering as an enhancement.
6. **Consent is self-declared.** The app cannot verify that the visitor has the consent of other people appearing in their photo.
7. **Photos are shown publicly by design.** The privacy approach reduces risk but cannot remove it.
8. **Deployment may be limited.** A deployed link is optional, and free hosting may not suit file storage or the display page.
9. **Not tested at scale.** Performance has been checked only for small groups.

---

## Open Questions for Approval

Please confirm these before architecture work begins. Each answer will be recorded in `ASSUMPTIONS.md`.

1. Is the browser-token approach acceptable for identifying returning visitors?
2. How long should each photo play, as a default? Should this be configurable?
3. Should photos be kept permanently or deleted after a set period?
4. Should the Operator and delete features be attempted if time remains?

## Document Control

| Version | Date | Change |
|---|---|---|
| 1.0 | 13 Nov 2026 | Initial draft based on Assignment 1 |
