# AI_LOG.md

# Visitor Photo Slideshow (MomentVault): AI Interaction Log

| Item | Detail |
|---|---|
| Product | MomentVault, implementing Assignment 1: Visitor Photo Slideshow |
| Document | AI_LOG.md, **blank initial template** |
| Template version | 1.0 |
| Template date | 18 November 2026 |
| Developer | `<YOUR NAME>` |
| Log status | **EMPTY. No interactions have been recorded yet.** |
| Assignment requirement | "5 to 10 key prompts or interactions, including at least 2 cases where the AI was wrong, how you noticed, and how you fixed it." (Assignment_1.pdf, repository requirements) |
| AI tools allowed in this project | Claude and Gemini |

> **This file is a template.** It contains **no** logged interactions, **no** recorded mistakes and **no** test results. Every entry below is empty. Only the **developer** may fill it in, from what **actually happened**. A log written from imagination, or filled in afterwards to look good, defeats its purpose and breaks the assignment's own rules.

---

## 1. Purpose and Honesty Rules

### 1.1 What This Log Is For

This log shows **how you directed the AI, how you checked its work, and how you caught and fixed its mistakes.** The assignment says you are graded "not only on whether it works, but on the decisions you make, how you direct the AI, and how you catch and fix its mistakes." The log is the evidence.

### 1.2 The Rules (Read Before Writing Anything)

| # | Rule | Why |
|---|---|---|
| **R1** | **Record only real interactions.** If it did not happen, it does not go in. | Invented entries are dishonest and are easy to spot in a live demo. |
| **R2** | **Do not fabricate mistakes.** Never invent or exaggerate an AI error to reach the "at least 2" requirement. See Section 7 if you are short. | The requirement is for *real* cases. |
| **R3** | **Paste the original prompt exactly as you sent it.** Do not improve it, shorten it or correct typos afterwards. | A cleaned-up prompt hides how you actually directed the AI. |
| **R4** | **Keep AI words and your words apart.** What the AI said or suggested goes in the *AI* rows. Your judgement and choices go in the *Developer* rows. Never write an AI suggestion as if it were your own decision. | Responsible use means you can show *you* decided. |
| **R5** | **Never write "tested", "works" or "verified" without evidence.** Evidence means a command and its real output, a test ID and its real result, a screenshot reference, or a commit. See Section 2.3. | An AI-generated feature is untested until *you* run it. |
| **R6** | **A blank "Mistake" field is allowed.** Write "None identified". Do not invent a problem. A response with no problem is a valid entry. | Honest logs are not all failures. |
| **R7** | **Protect secrets and people.** Never paste a visitor token, the operator key, a real `.env`, a password, a real person's name or photo. Replace with `[REDACTED]`. | NFR-SEC-02 and privacy rules in SECURITY_PRIVACY.md |
| **R8** | **Log in time order.** Entries are numbered in the order they happened. Do not insert or backdate. | The assignment asks for how the project *evolved*. |
| **R9** | **Date and time are real.** Use the time you sent the prompt (India Standard Time). It should fit your commit history. | The log and the commits must agree. |
| **R10** | **Write the review and learning in your own words.** Do not ask an AI to write the "Developer review" or "Developer learning" rows. | Those rows are the proof of human oversight. |
| **R11** | **Log key interactions, not every message.** Choose 5 to 10 that show important decisions, checking, or corrections. | The assignment asks for 5 to 10. |
| **R12** | **If something was not checked, say so.** "Not verified" is an honest and acceptable answer. | Never claim more than you did. |

### 1.3 How to Fill This Template (Short Version)

1. **Fill the numbered slots in Section 6, top to bottom, as things happen.**
2. After each AI session that mattered, write the entry **the same day**, while you remember.
3. Put each entry's ID in the matching **phase section** in Section 5 (the ten required sections) and in the index in Section 3.
4. At the end, **delete unused empty slots**, keep the IDs consecutive, and complete Sections 8 to 10.

---

## 2. Definitions and Evidence Rules

### 2.1 AI Suggestion vs Developer Decision

| Term | Meaning | Where it goes |
|---|---|---|
| **AI suggestion** | Anything the AI proposed, wrote, claimed or explained | Rows 5 and 6 (and the quoted part of row 8) |
| **Developer decision** | What **you** chose to do about it: accept, change, reject, or ask again | Rows 7 and 9 |
| **Developer evidence** | What **you** ran or compared to check it | Row 10 |

**Decision words to use in row 7:** `Accepted as is` / `Accepted with changes` / `Rejected` / `Asked again`. Say **why**.

### 2.2 What Counts as a Mistake or Limitation (Row 8)

Use these labels so the log reads consistently. **Use one only if it truly applies.**

| Label | Meaning |
|---|---|
| `WRONG-FACT` | A statement that was false (a feature, a number, a behaviour) |
| `OUTDATED` | Code or advice for an older library version that did not work with the pinned version |
| `INVENTED` | An endpoint, setting, field, function or requirement that does not exist or was never approved |
| `DOC-CONFLICT` | Output that disagreed with an approved document |
| `BUG` | Code that ran but gave a wrong result |
| `SECURITY` | A weakness or an unsafe suggestion |
| `OVER-ENGINEERED` | More layers, libraries or code than the task needed |
| `UNVERIFIED-CLAIM` | The AI said something worked or was tested, with no proof |
| `LIMITATION` | Not a mistake: something the AI could not or did not do (for example it had no way to run the code) |
| `OTHER` | Anything else (explain) |

For every **mistake** (not a plain limitation), row 8 must say **three things**: **what was wrong**, **how you noticed**, and **what evidence you have** (an error message, a failing test ID, a document line, a screenshot reference).

### 2.3 What Counts as Verification (Row 10)

| Verification type | Acceptable evidence |
|---|---|
| **Ran the code** | The exact command and the real output (or its file name) |
| **Automated test** | The TESTING_PLAN case ID, the command, and the real result (including a failure) |
| **Manual test** | The TESTING_PLAN manual case ID, the device used, and what you saw |
| **Compared with a document** | The document name and section you compared with |
| **Peer or second-tool check** | Which tool or person, and what they found |
| **Not verified** | Say exactly that, and why |

| Verification status (write one in row 10) | Use when |
|---|---|
| **Verified, evidence: ...** | You ran or checked it and have evidence |
| **Partly verified: ...** | Only some of it was checked (say which) |
| **Not verified** | Nothing was run or compared |
| **Not applicable (explanation only)** | The AI only explained a concept and produced nothing to run |

> **Never** write "Verified" with no evidence beside it.

---

## 3. Log Index (Chronological)

Fill one row per entry **as you write it**. Leave unused rows blank, then delete them at the end.

| Interaction ID | Date | Tool | Phase (Section 5) | Short title | Mistake or limitation? (None / Mistake / Limitation) | Verification status |
|---|---|---|---|---|---|---|
| AI-001 | | | | | | |
| AI-002 | | | | | | |
| AI-003 | | | | | | |
| AI-004 | | | | | | |
| AI-005 | | | | | | |
| AI-006 | | | | | | |
| AI-007 | | | | | | |
| AI-008 | | | | | | |
| AI-009 | | | | | | |
| AI-010 | | | | | | |

### 3.1 Summary Counts (Fill In at the End)

| Measure | Required | Your count |
|---|---|---|
| Key interactions logged | 5 to 10 | |
| Interactions with Claude | n/a | |
| Interactions with Gemini | n/a | |
| Entries where the AI was **wrong** (a real mistake, not a plain limitation) | **at least 2** | |
| Of those, entries that have all of: what was wrong, how I noticed, how I fixed it, and evidence | all of them | |
| Entries with "Not verified" | n/a (honest) | |

---

## 4. How to Write One Entry (Guide to the Twelve Fields)

| # | Field | Write | Whose words |
|---|---|---|---|
| 1 | **Interaction ID** | `AI-001`, `AI-002`, ... in order. Never reuse or skip. | Developer |
| 2 | **Date** | Date and time you sent the prompt, India Standard Time, for example `18 Nov 2026, 14:35 IST`. | Developer |
| 3 | **AI tool used** | **Claude** or **Gemini**. Add the model name **only if the tool showed it** to you. Do not guess. | Developer |
| 4 | **Task objective** | One or two sentences: what you wanted to achieve and which part of the project it belongs to. | Developer |
| 5 | **Original prompt** | The exact text you sent. Long prompts: paste the whole thing, or the key part and say "(shortened: full text in `<file>`)". Say whether the shared prompt block (CODING_STANDARDS 0.2) was included (yes/no). Redact secrets. | Developer (what you sent) |
| 6 | **AI response summary** | A short, neutral summary of what the AI **actually returned**: files, steps, claims. Include any claim like "this works" or "tested", and mark it as **the AI's claim**. | AI output, summarised by you |
| 7 | **Developer review** | What **you** did to read and judge it: what you read, what you compared it with, what you did not understand and how you found out. End with your **decision**: Accepted as is / Accepted with changes / Rejected / Asked again, and why. | Developer |
| 8 | **Mistake or limitation identified** | Tick one: None identified / Mistake / Limitation. If a mistake: **what was wrong**, **how I noticed**, **evidence**, and the label from Section 2.2. | Developer |
| 9 | **Correction made** | What **you** changed or asked, and the file or commit. "Not applicable" if there was no mistake. | Developer |
| 10 | **Verification method** | How you checked the **final** result (Section 2.3), with evidence, and the verification status. | Developer |
| 11 | **Final outcome** | What ended up in the project (files, commit), or that it was dropped. Say honestly if it still has known problems. | Developer |
| 12 | **Developer learning** | What **you** learned that you can now explain (a concept, a trap, a better way to prompt). In your own words. | Developer |

**Extra lines above each entry table** (not part of the twelve fields): **Phase** (one of the ten in Section 5), **Related documents** (for example `API_SPECIFICATION 5.5`), **Related files or commit** (commit hash or file path, if any).

---

## 5. Log by Phase

Every entry belongs to **one** of the ten phases below. Write the entry **once**, in Section 6 (time order), and list its ID here. Entries are never copied between sections.

> The "Questions to ask yourself" are prompts for **your own critical review**. They are **not** examples of mistakes that happened.

### 5.1 Requirement analysis

| Item | Detail |
|---|---|
| Typical documents or files | PRD.md, ASSUMPTIONS.md, PROJECT_BRIEF.md |
| Interaction IDs in this phase | |
| Phase summary (written by me at the end: what the AI helped with, what I decided myself, what I checked) | |

**Questions to ask yourself while reviewing AI output in this phase:**

- Did the AI add a requirement the assignment or PRD does not contain?
- Did I compare its summary with the assignment text (Assignment_1.pdf) word by word?
- Is every assumption it made written in ASSUMPTIONS.md, with a reason?

### 5.2 Architecture

| Item | Detail |
|---|---|
| Typical documents or files | ARCHITECTURE.md, TECH_STACK.md, ADR.md |
| Interaction IDs in this phase | |
| Phase summary (written by me at the end: what the AI helped with, what I decided myself, what I checked) | |

**Questions to ask yourself while reviewing AI output in this phase:**

- Did I understand each choice well enough to defend it in a live demo?
- Did I ask for alternatives, and did I weigh them myself?
- Did the AI add technology or layers that are not needed?

### 5.3 Database design

| Item | Detail |
|---|---|
| Typical documents or files | DATABASE_DESIGN.md, models, constraints |
| Interaction IDs in this phase | |
| Phase summary (written by me at the end: what the AI helped with, what I decided myself, what I checked) | |

**Questions to ask yourself while reviewing AI output in this phase:**

- Do the tables and rules match the PRD (for example: names are not unique)?
- Did I check the constraints and relationships by running them, not only by reading?
- Did the AI use a database feature that SQLite does not support?

### 5.4 Backend implementation

| Item | Detail |
|---|---|
| Typical documents or files | backend/app/**, backend/tests/** |
| Interaction IDs in this phase | |
| Phase summary (written by me at the end: what the AI helped with, what I decided myself, what I checked) | |

**Questions to ask yourself while reviewing AI output in this phase:**

- Does the code respect the layer rules (routes, services, repositories)?
- Did the AI use outdated library syntax or invent a function, setting or endpoint?
- Did I run the code and its tests myself, and do I have the output?

### 5.5 Frontend implementation

| Item | Detail |
|---|---|
| Typical documents or files | frontend/src/** |
| Interaction IDs in this phase | |
| Phase summary (written by me at the end: what the AI helped with, what I decided myself, what I checked) | |

**Questions to ask yourself while reviewing AI output in this phase:**

- Do components only draw, and do only hooks call the server?
- Did the AI invent an API field or endpoint that the specification does not have?
- Did I open the page in a browser (and on a phone, if relevant) and look at the result?

### 5.6 Debugging

| Item | Detail |
|---|---|
| Typical documents or files | Any file; error messages; logs |
| Interaction IDs in this phase | |
| Phase summary (written by me at the end: what the AI helped with, what I decided myself, what I checked) | |

**Questions to ask yourself while reviewing AI output in this phase:**

- Did I read the actual error message before asking the AI?
- Did I understand the cause, or did I only paste a fix that made the error disappear?
- Did I re-run the failing step to prove the fix worked?

### 5.7 Security review

| Item | Detail |
|---|---|
| Typical documents or files | SECURITY_PRIVACY.md, upload and token code |
| Interaction IDs in this phase | |
| Phase summary (written by me at the end: what the AI helped with, what I decided myself, what I checked) | |

**Questions to ask yourself while reviewing AI output in this phase:**

- Did I check the AI's security advice against SECURITY_PRIVACY.md?
- Did any prompt, log or screenshot contain a token, key or real personal data?
- Did I test the abuse case (bad file, wrong token, missing consent) rather than assume it is handled?

### 5.8 Testing

| Item | Detail |
|---|---|
| Typical documents or files | TESTING_PLAN.md, tests, manual test results |
| Interaction IDs in this phase | |
| Phase summary (written by me at the end: what the AI helped with, what I decided myself, what I checked) | |

**Questions to ask yourself while reviewing AI output in this phase:**

- Did the AI write tests that could actually fail, or tests that only repeat the code?
- Did I run them and keep the output? Which TESTING_PLAN case IDs does this cover?
- Did I record real results only, including failures?

### 5.9 Documentation

| Item | Detail |
|---|---|
| Typical documents or files | README.md, ASSUMPTIONS.md, ARCHITECTURE.md, DEPLOYMENT_GUIDE.md |
| Interaction IDs in this phase | |
| Phase summary (written by me at the end: what the AI helped with, what I decided myself, what I checked) | |

**Questions to ask yourself while reviewing AI output in this phase:**

- Did I run every command in the document from a clean copy of the repository?
- Does any sentence claim something that was not tested?
- Does the document say what is not built?

### 5.10 Final review

| Item | Detail |
|---|---|
| Typical documents or files | Whole repository, AI_LOG.md, commit history |
| Interaction IDs in this phase | |
| Phase summary (written by me at the end: what the AI helped with, what I decided myself, what I checked) | |

**Questions to ask yourself while reviewing AI output in this phase:**

- Can I explain every file and function without help?
- Do dates in this log match the commit history?
- Did I remove unused template text and every placeholder?

---

## 6. Interaction Log (Chronological)

**Slots are numbered in time order.** Use them from the top. **Delete any slot you do not use** and keep the used IDs consecutive. **Everything below is empty on purpose.**


### AI-001: `<short title>`

**Phase:** `<one of the ten in Section 5>`  |  **Related documents:** `<section references>`  |  **Related files or commit:** `<path or hash>`  |  **Slot status:** Not used

| # | Field | Whose words | Entry |
|---|---|---|---|
| 1 | Interaction ID | Developer | AI-001 |
| 2 | Date | Developer | |
| 3 | AI tool used | Developer | ☐ Claude  ☐ Gemini  (model shown by the tool, if any: ) |
| 4 | Task objective | Developer | |
| 5 | Original prompt (verbatim) | Developer (what I sent) | |
| 6 | AI response summary | AI output, summarised by me | |
| 7 | Developer review (and decision) | Developer | |
| 8 | Mistake or limitation identified | Developer | ☐ None identified  ☐ Mistake  ☐ Limitation<br>What was wrong:<br>How I noticed:<br>Evidence:<br>Label (Section 2.2): |
| 9 | Correction made | Developer | |
| 10 | Verification method | Developer | Status: |
| 11 | Final outcome | Developer | |
| 12 | Developer learning | Developer | |

---

### AI-002: `<short title>`

**Phase:** `<one of the ten in Section 5>`  |  **Related documents:** `<section references>`  |  **Related files or commit:** `<path or hash>`  |  **Slot status:** Not used

| # | Field | Whose words | Entry |
|---|---|---|---|
| 1 | Interaction ID | Developer | AI-002 |
| 2 | Date | Developer | |
| 3 | AI tool used | Developer | ☐ Claude  ☐ Gemini  (model shown by the tool, if any: ) |
| 4 | Task objective | Developer | |
| 5 | Original prompt (verbatim) | Developer (what I sent) | |
| 6 | AI response summary | AI output, summarised by me | |
| 7 | Developer review (and decision) | Developer | |
| 8 | Mistake or limitation identified | Developer | ☐ None identified  ☐ Mistake  ☐ Limitation<br>What was wrong:<br>How I noticed:<br>Evidence:<br>Label (Section 2.2): |
| 9 | Correction made | Developer | |
| 10 | Verification method | Developer | Status: |
| 11 | Final outcome | Developer | |
| 12 | Developer learning | Developer | |

---

### AI-003: `<short title>`

**Phase:** `<one of the ten in Section 5>`  |  **Related documents:** `<section references>`  |  **Related files or commit:** `<path or hash>`  |  **Slot status:** Not used

| # | Field | Whose words | Entry |
|---|---|---|---|
| 1 | Interaction ID | Developer | AI-003 |
| 2 | Date | Developer | |
| 3 | AI tool used | Developer | ☐ Claude  ☐ Gemini  (model shown by the tool, if any: ) |
| 4 | Task objective | Developer | |
| 5 | Original prompt (verbatim) | Developer (what I sent) | |
| 6 | AI response summary | AI output, summarised by me | |
| 7 | Developer review (and decision) | Developer | |
| 8 | Mistake or limitation identified | Developer | ☐ None identified  ☐ Mistake  ☐ Limitation<br>What was wrong:<br>How I noticed:<br>Evidence:<br>Label (Section 2.2): |
| 9 | Correction made | Developer | |
| 10 | Verification method | Developer | Status: |
| 11 | Final outcome | Developer | |
| 12 | Developer learning | Developer | |

---

### AI-004: `<short title>`

**Phase:** `<one of the ten in Section 5>`  |  **Related documents:** `<section references>`  |  **Related files or commit:** `<path or hash>`  |  **Slot status:** Not used

| # | Field | Whose words | Entry |
|---|---|---|---|
| 1 | Interaction ID | Developer | AI-004 |
| 2 | Date | Developer | |
| 3 | AI tool used | Developer | ☐ Claude  ☐ Gemini  (model shown by the tool, if any: ) |
| 4 | Task objective | Developer | |
| 5 | Original prompt (verbatim) | Developer (what I sent) | |
| 6 | AI response summary | AI output, summarised by me | |
| 7 | Developer review (and decision) | Developer | |
| 8 | Mistake or limitation identified | Developer | ☐ None identified  ☐ Mistake  ☐ Limitation<br>What was wrong:<br>How I noticed:<br>Evidence:<br>Label (Section 2.2): |
| 9 | Correction made | Developer | |
| 10 | Verification method | Developer | Status: |
| 11 | Final outcome | Developer | |
| 12 | Developer learning | Developer | |

---

### AI-005: `<short title>`

**Phase:** `<one of the ten in Section 5>`  |  **Related documents:** `<section references>`  |  **Related files or commit:** `<path or hash>`  |  **Slot status:** Not used

| # | Field | Whose words | Entry |
|---|---|---|---|
| 1 | Interaction ID | Developer | AI-005 |
| 2 | Date | Developer | |
| 3 | AI tool used | Developer | ☐ Claude  ☐ Gemini  (model shown by the tool, if any: ) |
| 4 | Task objective | Developer | |
| 5 | Original prompt (verbatim) | Developer (what I sent) | |
| 6 | AI response summary | AI output, summarised by me | |
| 7 | Developer review (and decision) | Developer | |
| 8 | Mistake or limitation identified | Developer | ☐ None identified  ☐ Mistake  ☐ Limitation<br>What was wrong:<br>How I noticed:<br>Evidence:<br>Label (Section 2.2): |
| 9 | Correction made | Developer | |
| 10 | Verification method | Developer | Status: |
| 11 | Final outcome | Developer | |
| 12 | Developer learning | Developer | |

---

### AI-006: `<short title>`

**Phase:** `<one of the ten in Section 5>`  |  **Related documents:** `<section references>`  |  **Related files or commit:** `<path or hash>`  |  **Slot status:** Not used

| # | Field | Whose words | Entry |
|---|---|---|---|
| 1 | Interaction ID | Developer | AI-006 |
| 2 | Date | Developer | |
| 3 | AI tool used | Developer | ☐ Claude  ☐ Gemini  (model shown by the tool, if any: ) |
| 4 | Task objective | Developer | |
| 5 | Original prompt (verbatim) | Developer (what I sent) | |
| 6 | AI response summary | AI output, summarised by me | |
| 7 | Developer review (and decision) | Developer | |
| 8 | Mistake or limitation identified | Developer | ☐ None identified  ☐ Mistake  ☐ Limitation<br>What was wrong:<br>How I noticed:<br>Evidence:<br>Label (Section 2.2): |
| 9 | Correction made | Developer | |
| 10 | Verification method | Developer | Status: |
| 11 | Final outcome | Developer | |
| 12 | Developer learning | Developer | |

---

### AI-007: `<short title>`

**Phase:** `<one of the ten in Section 5>`  |  **Related documents:** `<section references>`  |  **Related files or commit:** `<path or hash>`  |  **Slot status:** Not used

| # | Field | Whose words | Entry |
|---|---|---|---|
| 1 | Interaction ID | Developer | AI-007 |
| 2 | Date | Developer | |
| 3 | AI tool used | Developer | ☐ Claude  ☐ Gemini  (model shown by the tool, if any: ) |
| 4 | Task objective | Developer | |
| 5 | Original prompt (verbatim) | Developer (what I sent) | |
| 6 | AI response summary | AI output, summarised by me | |
| 7 | Developer review (and decision) | Developer | |
| 8 | Mistake or limitation identified | Developer | ☐ None identified  ☐ Mistake  ☐ Limitation<br>What was wrong:<br>How I noticed:<br>Evidence:<br>Label (Section 2.2): |
| 9 | Correction made | Developer | |
| 10 | Verification method | Developer | Status: |
| 11 | Final outcome | Developer | |
| 12 | Developer learning | Developer | |

---

### AI-008: `<short title>`

**Phase:** `<one of the ten in Section 5>`  |  **Related documents:** `<section references>`  |  **Related files or commit:** `<path or hash>`  |  **Slot status:** Not used

| # | Field | Whose words | Entry |
|---|---|---|---|
| 1 | Interaction ID | Developer | AI-008 |
| 2 | Date | Developer | |
| 3 | AI tool used | Developer | ☐ Claude  ☐ Gemini  (model shown by the tool, if any: ) |
| 4 | Task objective | Developer | |
| 5 | Original prompt (verbatim) | Developer (what I sent) | |
| 6 | AI response summary | AI output, summarised by me | |
| 7 | Developer review (and decision) | Developer | |
| 8 | Mistake or limitation identified | Developer | ☐ None identified  ☐ Mistake  ☐ Limitation<br>What was wrong:<br>How I noticed:<br>Evidence:<br>Label (Section 2.2): |
| 9 | Correction made | Developer | |
| 10 | Verification method | Developer | Status: |
| 11 | Final outcome | Developer | |
| 12 | Developer learning | Developer | |

---

### AI-009: `<short title>`

**Phase:** `<one of the ten in Section 5>`  |  **Related documents:** `<section references>`  |  **Related files or commit:** `<path or hash>`  |  **Slot status:** Not used

| # | Field | Whose words | Entry |
|---|---|---|---|
| 1 | Interaction ID | Developer | AI-009 |
| 2 | Date | Developer | |
| 3 | AI tool used | Developer | ☐ Claude  ☐ Gemini  (model shown by the tool, if any: ) |
| 4 | Task objective | Developer | |
| 5 | Original prompt (verbatim) | Developer (what I sent) | |
| 6 | AI response summary | AI output, summarised by me | |
| 7 | Developer review (and decision) | Developer | |
| 8 | Mistake or limitation identified | Developer | ☐ None identified  ☐ Mistake  ☐ Limitation<br>What was wrong:<br>How I noticed:<br>Evidence:<br>Label (Section 2.2): |
| 9 | Correction made | Developer | |
| 10 | Verification method | Developer | Status: |
| 11 | Final outcome | Developer | |
| 12 | Developer learning | Developer | |

---

### AI-010: `<short title>`

**Phase:** `<one of the ten in Section 5>`  |  **Related documents:** `<section references>`  |  **Related files or commit:** `<path or hash>`  |  **Slot status:** Not used

| # | Field | Whose words | Entry |
|---|---|---|---|
| 1 | Interaction ID | Developer | AI-010 |
| 2 | Date | Developer | |
| 3 | AI tool used | Developer | ☐ Claude  ☐ Gemini  (model shown by the tool, if any: ) |
| 4 | Task objective | Developer | |
| 5 | Original prompt (verbatim) | Developer (what I sent) | |
| 6 | AI response summary | AI output, summarised by me | |
| 7 | Developer review (and decision) | Developer | |
| 8 | Mistake or limitation identified | Developer | ☐ None identified  ☐ Mistake  ☐ Limitation<br>What was wrong:<br>How I noticed:<br>Evidence:<br>Label (Section 2.2): |
| 9 | Correction made | Developer | |
| 10 | Verification method | Developer | Status: |
| 11 | Final outcome | Developer | |
| 12 | Developer learning | Developer | |

---

## 7. If You Have Fewer Than Two Real Mistakes

The assignment asks for **at least two** real cases where the AI was wrong. **Do not invent them.** Instead:

| Step | Action |
|---|---|
| 1 | **Review the AI output harder.** Use the review questions in Section 5 and the review checklist in CODING_STANDARDS Section 19. Compare against the approved documents line by line. |
| 2 | **Run everything.** Many mistakes only show when code is run (outdated syntax, wrong library options, missing packages, failing tests). TECH_STACK risk R6 warns that assistants often produce outdated code, and several library behaviours are marked "to verify" in the design documents. These are **places to check**, not mistakes that have already happened. |
| 3 | **Cross-check with the second tool.** Ask the other AI (Claude or Gemini) to review a piece of code. Record the result honestly, whatever it finds, including "nothing". |
| 4 | **Test the unhappy paths** in TESTING_PLAN (bad files, wrong token, missing consent, simultaneous uploads). Real failures there are legitimate log material. |
| 5 | **Still short?** Record exactly what you have, and write a short honest note in Section 9. A truthful log with fewer examples is better than a fabricated one. Tell your assessor, and do not hide it. |

---

## 8. Mistakes Register (Fill In at the End From Section 6)

One row per **real mistake** logged above. Leave empty if there are none.

| Interaction ID | Label (Section 2.2) | One-line description | How noticed | How fixed | Fixed and re-verified? (Yes / Partly / No) |
|---|---|---|---|---|---|
| | | | | | |
| | | | | | |
| | | | | | |

---

## 9. Final Review (Written at the End, by the Developer)

### 9.1 My AI Use Statement

Write in your own words, in a few sentences, **what you used AI for**, **which parts you wrote or decided yourself**, and **what you did not delegate**.

> `<To be written by the developer. Do not generate with AI (rule R10).>`

### 9.2 Human Oversight Practices (Tick Only What You Really Did)

| Practice | Done? |
|---|---|
| I read every line of AI-generated code before keeping it | ☐ |
| I can explain every file and function in a live demo without help | ☐ |
| I compared AI output with the approved documents, not only with itself | ☐ |
| I ran the code and tests myself and kept the real output | ☐ |
| I used the other AI tool to review at least one important piece of work | ☐ |
| I never pasted secrets, tokens, the operator key or real personal data into a prompt | ☐ |
| I changed or rejected AI suggestions when they did not fit the requirements | ☐ |
| I recorded AI mistakes in this log when they happened, not afterwards | ☐ |
| Where I could not verify something, I said so | ☐ |

### 9.3 What I Would Do Differently

> `<To be written by the developer.>`

### 9.4 Known Gaps in This Log

List honestly anything the log does not cover (for example: "I used the AI in many short messages that are not logged here" or "fewer than two clear mistakes were found; see Section 7").

> `<To be written by the developer. Write "None" only if true.>`

---

## 10. Pre-Submission Checklist for This File

| # | Check | Done? |
|---|---|---|
| 1 | Between **5 and 10** key interactions are logged | ☐ |
| 2 | **At least two** entries describe a real AI mistake, each with **what was wrong, how I noticed, how I fixed it, evidence** (or Section 9.4 says honestly why not) | ☐ |
| 3 | Every prompt is **verbatim** and unedited | ☐ |
| 4 | Both **Claude and Gemini** are shown correctly (or only the one I really used) | ☐ |
| 5 | Dates are real, in time order, and **match the commit history** | ☐ |
| 6 | Every "Verified" or "tested" claim has **evidence** beside it (rule R5) | ☐ |
| 7 | AI suggestions and my decisions are **kept apart** in every entry | ☐ |
| 8 | Rows 7 and 12 are in **my own words** | ☐ |
| 9 | No secrets, tokens, keys, real names or real photos appear anywhere | ☐ |
| 10 | The index (Section 3), the phase sections (Section 5), the counts (3.1) and the register (Section 8) all agree with Section 6 | ☐ |
| 11 | Unused slots, template hints I no longer need, and every `<placeholder>` are removed | ☐ |
| 12 | Section 9 is complete | ☐ |

---

## Document Control

| Item | Detail |
|---|---|
| Document | AI_LOG.md |
| State | **Blank template. Contains no logged interactions.** |
| Template version | 1.0, 18 November 2026 |
| How the template was made | The **structure and guidance** of this template were drafted with AI assistance (Claude). **The template holds no content from any real interaction.** |
| Reviewed by developer | ☐ (tick when you have read and understood this template) |
| Related project documents | IMPLEMENTATION_PLAN.md (rule 6: record AI mistakes when they happen), CODING_STANDARDS.md (15.4, 19), TESTING_PLAN.md (case IDs for evidence), SECURITY_PRIVACY.md (what never goes in a prompt) |
