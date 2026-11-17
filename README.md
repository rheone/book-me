# book-me

book-me is a Claude Code skill that finds books and audiobooks for a reader. It works from their reading history, checks the picks against their Libby libraries, and tracks what they approve.

| Version | Author | License |
| :-- | :-- | :-- |
| 1.0.0 | Robert H. Engelhardt (<rheone@gmail.com>) | [Apache 2.0](LICENSE) |

[How it works](#how-it-works) · [Examples](#examples) · [Modes](#modes) · [Libby tags](#libby-tags-read-this-first) · [Files](#files) · [Privacy](#privacy-and-terms)

## Background

book-me started as one long recommendation run for one reader. That run produced handoff notes about what worked and what broke along the way. Admittedly, most of those notes are about what broke. This skill turns those notes into a tool that works for any reader, with the hard lessons built in.

## Getting started

1. Install the skill. See [Install in Claude Code](#install-in-claude-code) or [Install on claude.ai](#install-on-claudeai).
2. Ask for a book, for example "What should I listen to next?", or type `/book-me` in Claude Code.
3. Answer the interview questions. book-me asks one question at a time and shows a recommended answer.
4. Give the account IDs that book-me asks for. book-me verifies each ID before it saves it.
   - **Libby:** for each library, open it in Libby on the web and paste the URL, for example `https://libbyapp.com/library/<key>`.
   - **Goodreads:** paste your profile URL, for example `https://www.goodreads.com/user/show/<id>-<name>`.
5. Give it a reading history. The fastest start is a Goodreads export:
   1. On the Goodreads desktop site, open **My Books > Tools > Import and export**.
   2. Click **Export Library**, wait for the dated link, then download the `.csv` file.
   3. Give book-me the path to the file.

book-me then saves a profile, shows ranked picks with library availability, and adds each pick to the tracker.

> [!NOTE]
> Claude starts book-me by itself when you ask for book or audiobook picks, share a reading export, or ask about Libby or Goodreads. In Claude Code you can also type `/book-me`.

## Install in Claude Code

1. Copy the `book-me` folder to one of these locations:
   - `~/.claude/skills/book-me` to use it in every project.
   - `.claude/skills/book-me` inside a project to use it only in that project.
2. Start a new Claude Code session. Claude Code loads skills when a session starts.
3. Type `/book-me`, or ask for book picks.

Profiles are saved in `book-me/profiles/<name>/` and stay between sessions.

## Install on claude.ai

1. Make a zip file of the `book-me` folder. The zip must contain `book-me/SKILL.md`.
   - *MUST* leave out every `profiles/<name>/` folder. Those folders hold personal reading data.
   - On Windows, right-click the folder and select **Compress to ZIP file**.
2. On claude.ai, open **Settings > Capabilities**.
3. Turn on **Code execution and file creation**. Skills need it.
4. In the **Skills** section, click **Upload skill** and select the zip file.
5. Turn on the skill. If an older book skill is in the list, such as `goodreads-libby-audiobook-recs`, turn it off or delete it so that only one skill answers book requests.
6. Optional: if Libby availability checks fail, add these domains to the allowed list in the code execution settings:
   - `thunder.api.overdrive.com`
   - `openlibrary.org`

Skills that you upload to claude.ai also sync to Claude Code.

### Keep your profile on claude.ai

On claude.ai the skill files are read-only, and a new chat does not remember the last one. book-me keeps your profile and tracker in one of two places. It picks the first one that is available and records the choice in your profile.

| Storage | Needs | Upkeep |
| :-- | :-- | :-- |
| Claude Sheet | The Claude Docs connector | None. book-me reads and edits the sheet directly. |
| Claude project files | Nothing extra | Replace two project files after each run. |

#### Claude Sheet

book-me makes one sheet named `book-me: <name>` with two tabs: `Tracker` and `Profile`. You can set a book's status in the `Tracker` tab yourself, and book-me reads it on the next run. Nothing needs to be uploaded.

#### Claude project files

1. Create a project, for example "Books".
2. Run book-me in a chat inside that project. On the first run, book-me interviews you and gives you `profile.json` and `tracker.csv`.
3. Add both files to the project files.
4. At the end of each run, book-me gives you updated copies. Replace the old project files with them.

> [!WARNING]
> If you skip step 4, the next chat uses your old tracker. Books you rejected in the last run can come back.

### Limits on claude.ai

- Tagging, holds, borrows, and Goodreads imports need Claude in Chrome. Without it, book-me gives you the steps to do by hand.
- The `/book-me` command and mode names such as `series` work only in Claude Code. On claude.ai, say what you want, for example "Find the unread books in series I started."

## How it works

Each run follows the same six steps:

1. **Profile.** book-me loads `profiles/<name>/profile.json`. If no profile exists, it runs the interview.
2. **Mode.** book-me takes the mode from your request, or asks you to pick one.
3. **Account IDs.** If the mode uses Libby or Goodreads and the profile does not have a verified ID for it, book-me asks for the ID before it continues.
4. **Reference.** book-me reads only the reference files that the mode needs.
5. **Guard.** book-me removes books you read, did not finish, rejected, or by authors you block.
6. **Output.** book-me shows a ranked table, adds rows to the tracker, and publishes a results page.

book-me accepts these reading history sources. You can combine them.

| Source | What it gives | How to get it |
| :-- | :-- | :-- |
| Goodreads CSV | Ratings, shelves, did-not-finish | My Books > Tools > Import and export > Export Library |
| StoryGraph CSV | Ratings, moods, did-not-finish | Manage Account > Export StoryGraph Library |
| Libby timeline | Borrow history, no ratings | Shelf > Timeline > Actions > Export Timeline |
| Typed list | A few favorites and moods | Type them when book-me asks |

## Examples

In Claude Code, type the command. On claude.ai, or in Claude Code, you can type the request in plain words instead.

| Claude Code command | Plain request | Result |
| :-- | :-- | :-- |
| `/book-me` | "Recommend some audiobooks for me." | Interview on the first run, then ranked picks your library owns |
| `/book-me check` | "What on my want-to-read list can I borrow now?" | Each to-read book marked available now, wait time, or not owned |
| `/book-me tonight funny and short` | "Give me something light I can start listening to tonight." | Three to five books you can borrow right now |
| `/book-me like Project Hail Mary` | "Find books like Project Hail Mary." | Picks that match the tone and pace of that book |
| `/book-me like cozy mysteries` | "I am in the mood for a cozy mystery." | Picks for a mood or genre |
| `/book-me series` | "Which books am I missing in the series I started?" | Every unread book in each series, with library status |
| `/book-me series rated 4+` | "Only series where I gave a book 4 stars or more." | The same list, filtered by your ratings |
| `/book-me authors` | "What else by authors I have read is at my library?" | Unread books per author, with what you already read |
| `/book-me new` | "Did my favorite authors publish anything new?" | Recent and upcoming books, with library status |
| `/book-me wishlist` | "Good picks my library does not have?" | A separate list, with an offer to set Libby Notify Me |
| `/book-me for my niece, 12, likes Percy Jackson` | "Pick a gift book for a 12-year-old who loved Percy Jackson." | Picks for that reader. Your profile does not change. |
| `/book-me taste` | "What does my reading history say about my taste?" | A short taste report, saved to your profile |
| `/book-me tag` | "Add the approved picks to my Libby tag." | Backup check, dry run, tagging, then verification |
| `/book-me hold` | "Place holds on the ones I approved." | Dry run with hold limits, then holds and verification |
| `/book-me shelve` | "Put my picks on a Goodreads shelf." | Dry run, CSV import, then an audit of the shelf |
| `/book-me export` | "Save the results of this chat." | Markdown, CSV, and a results page |
| `/book-me offload` | "Clean up my tracker." | Finished rows move to a dated report |
| `/book-me fresh` | "Start over with a new interview." | Profile and tracker archived, then a new interview |

A typical first session:

```text
/book-me
  > Interview: profile name, Goodreads profile URL, Libby library URL, audiobooks, moods
  > Ranked table of 10 picks, added to the tracker as "Awaiting your pick"
all but 3 and 7
  > Picks 3 and 7 set to Skipped, the rest set to Approved
/book-me tag
  > Libby backup check, dry run of 8 titles, then tagging and verification
/book-me export
```

## Modes

In Claude Code, type the mode after the skill name, for example `/book-me series`. Anywhere, you can also describe what you want.

| Mode | What it does |
| :-- | :-- |
| `recs` | Picks that your libraries own, in your formats |
| `check` | Availability for every book on your to-read shelf |
| `wishlist` | Strong picks that no library owns, with an option to tell the library you want them |
| `authors` | Unread books by authors you already read |
| `like <X>` | Picks for a mood, a genre, or "like this book" |
| `series` | Every unread book in each series you started |
| `tonight` | Three to five books you can borrow now |
| `new` | New and upcoming books by your favorite authors |
| `for <someone>` | Picks for another reader. Your profile does not change. |
| `taste` | A short report of what you love, what you abandon, and how you rate |
| `tag` / `untag` | Add picks to a Libby tag, or remove them |
| `hold` / `borrow` / `notify` | Place holds, borrow, or set Libby Notify Me |
| `shelve` | Import picks to Goodreads shelves |
| `export` | Save this chat's results as Markdown, CSV, and a results page |
| `offload` | Move finished tracker rows into a dated report |
| `fresh` | Archive your profile and tracker, then start from the interview |

## Approvals and account changes

book-me reads from Goodreads, StoryGraph, and Libby without asking. It changes an account only after you approve.

For each change, book-me does these steps:

1. Shows a dry run that lists each book and the action.
2. Waits for your OK.
3. Makes the change through the site's own buttons.
4. Verifies the result.
5. Updates the tracker and reports what changed, including mistakes.

By default you approve a whole list at once. Answer `all`, `none`, or name the books to keep or drop. You can change this to one book at a time, or to editing the tracker file, in your profile.

## Libby tags: read this first

> [!WARNING]
> Libby keeps tags on each device, not in an account. A reset deletes the tags on that device, and a tag export cannot be imported back. Back up before book-me changes any tag.

Before the first tag change in a session, book-me walks you through a backup check:

1. Pick your **main device**: the device that has the tags you want to keep.
2. On the main device, create a recovery passkey. Then export each tag and the timeline.
3. Choose a workflow:
   - **Linked devices (recommended).** Link the browser to the main device once with a setup code or the passkey. After that, tags you make in the browser sync to every linked device.
   - **Browser workspace, then transfer.** Copy the main device into the browser and tag there. Then, on each target device, export its tags, reset it, and recover it from the browser. *Note that the reset deletes any tags made on that device after the copy.*
4. Confirm that the browser shows the same tags as the main device.

These facts come from Libby Help (help.libbyapp.com, pages 6004, 6023, 6070, and 6137). Timeline data never syncs between devices.

## Tracker

Each profile has a `tracker.csv` file with one row per pick. The `status` column records every decision.

| Status | Meaning |
| :-- | :-- |
| `Awaiting your pick` | Suggested, no decision yet |
| `Approved` | You said yes. book-me acts on it. |
| `Tagged`, `On Hold`, `Borrowed`, `Notify Set`, `Shelved` | Done and verified |
| `Tagged - review` | Tagged, but something looks wrong |
| `Untagged` | Removed from the tag after you said yes |
| `Skipped` | Not now |
| `Already Read`, `Not Interested` | Never suggested again |

book-me never changes a status that you set.

## Files

```text
book-me/
├── SKILL.md              Run steps, mode menu, rules for every run
├── README.md             This file
├── LICENSE               Apache License 2.0
├── reference/            Detail that book-me loads only when a mode needs it
│   ├── profile.md        Interview, profile fields, taste report
│   ├── sources.md        Reading each export, merging, the already-read guard
│   ├── recommend.md      Candidate list, ranking, logic for each mode, Open Library
│   ├── libby-catalog.md  Availability lookups, format IDs, several libraries
│   ├── libby-safety.md   Libby backup check and the two workflows
│   ├── libby-actions.md  Tag, untag, hold, borrow, Notify Me
│   ├── goodreads.md      Shelf import, ISBN lookup, audit, automation levels
│   ├── tracker.md        Tracker columns and statuses
│   ├── output.md         Chat table, CSV, results page
│   ├── history.md        export, offload, and fresh
│   └── browser.md        Claude in Chrome tips
└── profiles/
    ├── README.md
    ├── template.json
    └── <name>/           One folder per reader: profile.json, tracker.csv, history/
```

`SKILL.md` stays short on purpose. book-me reads a reference file only when the current mode needs it.

## Privacy and terms

- **Profiles are personal.** The `profiles/<name>/` folders hold reading history. Share only `SKILL.md`, `README.md`, `reference/`, and `profiles/template.json`.
- **Libby.** book-me uses Libby only through its own web pages in your signed-in browser. Libby's private API is off limits. OverDrive suspends accounts that use it.
- **Libby availability.** Availability comes from an OverDrive catalog service that the Libby web app uses. The service is not documented. If it fails, book-me gives you the Libby search link instead.
- **Goodreads.** The Goodreads Terms of Use exclude robots and data extraction tools. By default book-me uses only the CSV export and import. In your profile you can turn on `assisted` or `full` automation. book-me explains the risk once before it turns either one on.
- **Open Library.** book-me uses the Open Library API for genres, series, and ISBNs. It stays within the published rate limits and identifies itself with a User-Agent.

## Requirements

- Claude Code, or claude.ai with code execution turned on.
- Claude in Chrome, for account changes and the browser shortcuts. Without it, book-me works from files and gives you the steps to do by hand.
- A Libby library card, for availability and Libby actions.
- The Artifact tool, for results pages. Without it, book-me writes Markdown and CSV only.

## Credits

book-me is written by Robert H. Engelhardt (<rheone@gmail.com>). The workflow comes from handoff notes written during the first long run, and from research against Libby Help, the OverDrive developer documentation, Goodreads Help, and the Open Library API documentation.

## Version

The version is `1.0.0`. It is set in the `metadata.version` field of `SKILL.md`. Change the version there when you release a new copy of the skill.

## License

Copyright 2026 Robert H. Engelhardt.

book-me is licensed under the Apache License, Version 2.0. See [LICENSE](LICENSE) for the full text, or <https://www.apache.org/licenses/LICENSE-2.0>.

Libby, OverDrive, Goodreads, StoryGraph, and Open Library are trademarks of their owners. book-me is not affiliated with or endorsed by them.
