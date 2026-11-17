# Tracker

`profiles/<name>/tracker.csv` is the source of truth for every pick and decision. Read it at the start of any run that suggests or acts; never rely on memory or page variables for what was tagged, held or shelved.

## Columns

```csv
title,author,status,source_mode,list,format,library,availability,checked_on,why,action_on,notes
```

| Column | Meaning |
|---|---|
| `status` | See below |
| `source_mode` | Mode that suggested it (`recs`, `series`, `like: funny sci-fi`, ...) |
| `list` | `at_library`, `not_at_library`, `to_read_check` |
| `format` | `audiobook`, `ebook` |
| `library` | Key of the best library |
| `availability` | `Available now` or `N holds / M copies, ~D days` or `Not owned` |
| `checked_on` | Date availability was read |
| `why` | One-line reason |
| `action_on` | Date the last account action was verified |
| `notes` | Change log, review ids, reasons |

Write CSV with every field quoted.

## Statuses

| Status | Meaning | Set by |
|---|---|---|
| `Awaiting your pick` | Suggested, no decision | book-me |
| `Approved` | User said yes; act on it | user |
| `Tagged` | In the Libby tag, verified | book-me |
| `Tagged - review` | In the tag but something looks off | book-me |
| `Untagged` | Was tagged, removed on the user's yes | book-me |
| `On Hold` | Hold placed and verified | book-me |
| `Borrowed` | Loan verified | book-me |
| `Notify Set` | In Notify Me | book-me |
| `Shelved` | On the Goodreads shelf, audited | book-me |
| `Skipped` | Not now | user or book-me |
| `Already Read` | Never act on or recommend again | user |
| `Not Interested` | Never act on or recommend again | user |

Rules: a status the user set changes only on the user's word. Set an action status only after verifying it. Active statuses (`Awaiting your pick` through `Shelved`) keep a book out of new suggestions.

## Approvals

Follow `approval_style` from the profile:

- **batch** (default): show the list; the user answers all, none, or names the ones to keep or drop.
- **one-at-a-time**: per book or per author with AskUserQuestion; first say what they already read by that author or series.
- **tracker**: the user edits `status` in the file, then says "act on approved".

## Optional Claude Sheet

When the Docs connector is available and the user wants a shared, dropdown-friendly view, create a Sheets artifact with the same columns and a status dropdown. Read `guide(items=["topic.sheets"])` before editing it. Keep the CSV as the source of truth and sync the sheet from it, or switch the source of truth to the sheet and note that in the profile.

## Storage on claude.ai

The skill folder is read-only on claude.ai and chats do not share files. Pick the storage once, in this order, and record it in the profile as `storage`:

1. **Claude Sheet** (Docs connector available). One Sheets artifact named `book-me: <name>` with two tabs: `Tracker` (the columns above, status dropdown) and `Profile` (two columns, `field` and `value`; JSON values as text). Find it by listing the user's artifacts. Read and write it directly each run; nothing to re-upload. Read `guide(items=["topic.sheets"])` first.
2. **Project files** (no connector). The user keeps `profile.json` and `tracker.csv` in a Claude project and runs book-me in that project. Read them from the project files. At the end of every run that changed either one, hand back the updated files and remind the user to replace the project copies; an old tracker brings rejected books back.

In Claude Code, files under `profiles/<name>/` are the storage and this section does not apply.
