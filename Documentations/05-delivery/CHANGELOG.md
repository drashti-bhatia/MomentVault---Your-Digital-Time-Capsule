# Changelog

All notable changes to **MomentVault: Visitor Photo Slideshow** are recorded in this file.

The format follows [Keep a Changelog 1.1.0](https://keepachangelog.com/en/1.1.0/).
Version numbers follow [Semantic Versioning 2.0.0](https://semver.org/spec/v2.0.0.html), adapted for this project as explained in [How Version Numbers Are Assigned](#how-version-numbers-are-assigned).

> **Status of this file.** No version of the application has been released yet.
> Every section below is intentionally empty. An entry is added only when the change
> has really been made, committed, and checked (a passing test or a recorded manual check).
> Planned work belongs in `IMPLEMENTATION_PLAN.md`, not here.

---

## [Unreleased]

Changes that are committed but not yet part of a numbered release go here.
At release time, this content moves into a new version section (see [Release Procedure](#release-procedure)).

### Added

- Nothing yet.

### Changed

- Nothing yet.

### Fixed

- Nothing yet.

### Removed

- Nothing yet.

### Security

- Nothing yet.

---

<!--
TEMPLATE FOR THE NEXT RELEASE (copy, fill in, and place directly below [Unreleased]).
Delete this comment block only when the first real release is written.

## [0.1.0] - YYYY-MM-DD

### Added
- <what a user or developer can now do, in one plain sentence>

### Changed
- <what behaves differently from before>

### Fixed
- <what was broken and is now correct>

### Removed
- <what no longer exists>

### Security
- <what was hardened or what vulnerability was closed>
-->

## How to Write Entries

| Rule | Detail |
|---|---|
| **Write for people, not machines** | Describe the effect ("Photos larger than 5 MB are rejected with a clear message"), not the code ("Added `if size > MAX`"). |
| **One change, one line** | Keep each entry to one or two sentences. Link a commit, issue, or document where it helps. |
| **Only real, verified changes** | Never list a feature as done because it is planned or because code was generated. Add it after it runs. |
| **Newest first** | Versions are listed with the newest at the top. Dates use ISO 8601 (`YYYY-MM-DD`). |
| **Do not copy commit messages** | A changelog is a summary for readers. The Git history already holds the detail. |
| **Be honest about failures** | If something was tried and reverted, record it under **Removed** or **Fixed**, not nowhere. |

### What Each Section Means

| Section | Use it for | Examples of the kind of entry (illustrations only, not real entries) |
|---|---|---|
| **Added** | New features, endpoints, pages, tests, documents, or settings | "Upload page accepts a name and photos." |
| **Changed** | Changes to existing behaviour, defaults, limits, wording, or structure | "Default slide duration changed from 10 to 8 seconds." |
| **Fixed** | Bug fixes, including mistakes in AI-generated code that were found and corrected | "Fixed a crash when two uploads used the same file name." |
| **Removed** | Features, files, settings, or endpoints that no longer exist | "Removed the thumbnail endpoint." |
| **Security** | Anything that affects safety or privacy: validation, headers, token handling, consent, data deletion, dependency vulnerability fixes | "Photo responses are no longer cached publicly after consent is withdrawn." |

**Security entries have special rules:**

- Describe the improvement, not an exploit recipe. Do not publish details that help an attacker against a deployed copy.
- Never include a real token, key, password, address, or photo in this file.
- A change that touches consent, withdrawal, or deletion should always get a Security entry, even if it also appears under Added or Changed.

**Optional sections.** Keep a Changelog also defines **Deprecated** (things that will be removed later). It is not used here by default. Add it only if a feature is marked for removal.

---

## How Version Numbers Are Assigned

Version numbers have the form **MAJOR.MINOR.PATCH** (for example `0.3.1`).

### The Basic Rules

| Part | Increase it when | Example |
|---|---|---|
| **MAJOR** (first number) | A change breaks existing use: the API contract changes in an incompatible way, the database needs a manual migration, or stored data is no longer readable. | `1.4.2` to `2.0.0` |
| **MINOR** (second number) | A new feature or a visible improvement is added without breaking anything that already worked. | `1.4.2` to `1.5.0` |
| **PATCH** (third number) | Only bugs or small safe corrections are fixed, with no new features. | `1.4.2` to `1.4.3` |

When a higher part increases, the parts to its right reset to zero.

### How This Project Applies the Rules

1. **Start at `0.1.0`, not `1.0.0`.** A `0.y.z` version means the application is still being built and its API may change. Use `0.1.0` for the first release that contains a verified, working slice of the application.
2. **While below `1.0.0`:**
   - Increase **MINOR** for new features and also for changes that would otherwise be breaking.
   - Increase **PATCH** for bug fixes only.
3. **Release `1.0.0` only when all of these are true:**
   - The MVP required behaviours work (name and photo upload, slideshow on a display, returning-visitor playback).
   - The privacy, consent, file-validation, size-limit, and simultaneous-upload areas from the assignment are implemented and tested.
   - The required repository documents are complete and accurate.
   - The version is the one you submit or demonstrate.
4. **After `1.0.0`,** apply the MAJOR, MINOR, and PATCH rules in the table above strictly.
5. **Documentation-only changes** do not need a new version on their own. Record them under **Unreleased** and include them in the next release.
6. **Security fixes** get at least a PATCH release if the project has already been released or deployed.

### Choosing the Number: A Quick Check

Ask these questions in order and stop at the first "yes":

1. Does this change break existing use or stored data? **Increase MAJOR** (or MINOR while below `1.0.0`).
2. Does it add or change a feature people will notice? **Increase MINOR.**
3. Is it only a bug fix or a small safe correction? **Increase PATCH.**

### Pre-release Labels (Optional)

If you want to mark a version that is not ready for final use, add a label after a hyphen, for example `0.4.0-rc.1`. This is optional and not needed for the assignment.

---

## Release Procedure

Follow these steps each time a version is released. Do not skip the checks.

1. **Check the facts.** Confirm that the tests relevant to the changes were actually run and their results are recorded.
2. **Review `[Unreleased]`.** Remove anything that is not true yet. Merge duplicate entries and make wording clear.
3. **Choose the version number** using the rules above.
4. **Create the version section.** Copy the template above, set the version and today's date (`YYYY-MM-DD`), and move the reviewed entries into it.
5. **Reset `[Unreleased]`.** Leave each subsection with "Nothing yet."
6. **Update the links** at the bottom of this file (see below).
7. **Update the version** wherever it is declared (for example the backend and frontend package settings), once those files exist.
8. **Commit** with a message such as `docs(changelog): release v0.1.0`.
9. **Tag the commit** in Git with the same number: `git tag -a v0.1.0 -m "Release 0.1.0"`, then `git push origin v0.1.0`.

---

## Link References

Replace `<REPO_URL>` with the address of the public GitHub repository when it exists.
Add one line per released version, newest first. These lines stay as placeholders until a release is really made.

<!--
[Unreleased]: <REPO_URL>/compare/v0.1.0...HEAD
[0.1.0]: <REPO_URL>/releases/tag/v0.1.0
-->

When the first release is made, the first link becomes:

```
[Unreleased]: <REPO_URL>/compare/v0.1.0...HEAD
[0.1.0]: <REPO_URL>/releases/tag/v0.1.0
```

For later releases, each version compares with the one before it:

```
[0.2.0]: <REPO_URL>/compare/v0.1.0...v0.2.0
```
