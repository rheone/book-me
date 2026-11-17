---
name: book-me
description: Book and audiobook recommendations from a reader's history, checked against their Libby libraries. Use when the user asks what to read or listen to next, wants picks like a book, author, series or mood, shares a Goodreads, StoryGraph or Libby export, asks what on their to-read list is available, or wants Libby tags, holds, borrows or Goodreads shelves for picks.
license: Apache-2.0
metadata:
  author: Robert H. Engelhardt
  email: rheone@gmail.com
  version: "1.0.0"
---

# book-me

Recommend books and audiobooks from a reader's history, check them against their Libby libraries, track decisions in a tracker, and act on approved picks in Libby or Goodreads.

Paths below are relative to this skill's folder. `reference/` holds the detail; open a file when the step names it.

## Run

1. **Profile.** Look for `profile.json` in `profiles/<name>/`, then in the project's or conversation's uploaded files. One found: load it and ask only "anything changed?". Several: ask which. None, or the user said `fresh`: run the interview in [reference/profile.md](reference/profile.md). Where the skill folder is read-only (claude.ai), use the storage in [reference/tracker.md](reference/tracker.md#storage-on-claudeai): a Claude Sheet when the Docs connector is available, otherwise project files with updated copies handed back at the end of the run.
2. **Mode.** Take the mode from the user's words or arguments; when unclear, offer the menu below with AskUserQuestion (default: `recs` plus `check`).
3. **Account IDs.** Before any step that uses Libby (availability, tag, hold, borrow, notify), the profile must hold at least one verified library key. Before any step that uses Goodreads (export shortcut, `shelve`, audit), it must hold a verified Goodreads user ID. If one is missing, collect and verify it as in [reference/profile.md](reference/profile.md#interview) before going on. Use only the IDs the user gave.
4. **Load** the reference files the mode lists, and follow them.
5. **Guard.** Before showing any pick, run the already-read guard in [reference/sources.md](reference/sources.md#already-read-guard).
6. **Output.** Show results and update the tracker as [reference/output.md](reference/output.md) says. End by offering the next useful mode and `export`.

## Modes

| Mode | What it does | Load |
|---|---|---|
| `recs` | New picks the user's libraries own, in their formats | sources, recommend, libby-catalog |
| `check` | Every to-read book: borrowable now, wait, or not owned | sources, libby-catalog |
| `wishlist` | Strong picks no library owns; offer the purchase nudge | sources, recommend, libby-catalog, libby-actions |
| `authors` | Unread books by authors already read, per author | sources, recommend, libby-catalog |
| `like <X>` | Picks by mood, genre or "like <book/author>" | sources, recommend, libby-catalog |
| `series` | Every unread book in series the user has started | sources, recommend, libby-catalog |
| `tonight` | 3-5 picks borrowable right now for a stated mood; no interview | sources, recommend, libby-catalog |
| `new` | New and upcoming releases by favorite authors | sources, recommend, libby-catalog |
| `for <someone>` | Picks for another reader (gift, club, kid); the profile is left untouched | recommend, libby-catalog |
| `taste` | Readable taste report, saved to the profile | sources, profile |
| `tag` / `untag` | Add or remove approved picks in a Libby tag | libby-safety, libby-actions, tracker, browser |
| `hold` / `borrow` / `notify` | Place holds, borrow, or set Notify Me on approved picks | libby-actions, tracker, browser |
| `shelve` | Import approved picks to Goodreads shelves | goodreads, tracker |
| `export` | Save this chat's results as Markdown, CSV and an artifact page | history, output |
| `offload` | Move finished tracker rows into a dated readable report | history, tracker |
| `fresh` | Archive the profile and tracker, then start from the interview | history, profile |

Every file named in Load lives at `reference/<name>.md`. Tracker work always loads [reference/tracker.md](reference/tracker.md).

## Rules for every run

- **Write gate.** Anything that changes an account (Libby tag, hold, borrow, Notify Me; Goodreads import or delete) starts with a dry run: list exactly what will change, then wait for the user's explicit OK. Afterwards, verify, and report what changed, including mistakes.
- **Libby backup gate.** Before the first Libby tag or untag in a session, walk the user through [reference/libby-safety.md](reference/libby-safety.md): tags live on devices, a reset wipes them, and the user picks a safe workflow and confirms a backup. Offer to guide them step by step.
- **Libby through its own UI.** Use Libby's buttons in the user's signed-in browser. Libby's private hosts and sign-in token stay untouched: OverDrive suspends accounts that use them.
- **Status ownership.** A tracker status the user set is final; change it only when the user says so.
- **Snapshot.** Availability and wait times are a snapshot; say when they were checked.
- **Profile upkeep.** At the end of a run, offer to save changes to the profile (libraries, formats, likes, avoids, last run).
