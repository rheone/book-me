# Profile: interview, file, taste report

A profile stores one reader's setup so later runs skip the interview. One folder per reader:

```
profiles/<name>/
  profile.json   setup and taste (this file's schema)
  tracker.csv    one row per pick (see tracker.md)
  history/       exports, offloads, archived profiles (see history.md)
```

`<name>` is a short lowercase slug the user picks (`alex`, `family`).

## Interview

Ask with AskUserQuestion, **one question at a time, with selectable answers** and a recommended option first. Skip any question the user already answered in their request.

1. **Name** for the profile folder.
2. **Reading history source** (multi-select): Goodreads CSV, StoryGraph CSV, Libby timeline export, typed list of favorites / genres only. For each file source, ask for the file path or offer the browser shortcut (see [sources.md](sources.md)).
3. **Accounts in use** (multi-select): Libby, Goodreads, neither. Then collect the IDs for each one chosen; they are required, not optional:
   - **Libby library keys**: one or more, in preference order. Ask the user to open each library in Libby web and paste the URL (`https://libbyapp.com/library/<key>`); take the key from it. Verify each ([libby-catalog.md](libby-catalog.md#library-keys)) and re-ask on a 404. Note whether each is in the U.S. (Kindle delivery is U.S. only).
   - **Goodreads profile**: ask the user to paste their profile URL (`https://www.goodreads.com/user/show/<id>-<name>`, from clicking their avatar > Profile). Take the numeric `<id>`. Verify that `https://www.goodreads.com/user/show/<id>` loads and shows the name the user expects; re-ask if not. Store both the ID and the URL.
   Never guess an ID from a name, an email, or an earlier profile.
4. **Formats**: audiobook, ebook, or both.
5. **Moods and genres** wanted now, and anything to avoid. Offer genres inferred from the history plus "surprise me". Optional: liked narrators, max audiobook hours.
6. **Avoid list**: authors or series never to recommend (for example, ones read out). Separately, authors to skip unless asked.
7. **How many picks** (default 10) and **wait tolerance**: available now only, short holds (under ~2 weeks), any wait.
8. **Approval style**: batch then exceptions (default), one at a time, or edit the tracker file.
9. **Write targets, only if wanted**: Libby tag name (short: Libby rejects long names; `Recs` works, `scifi-fantasy-recs` was rejected), Goodreads shelf names, Goodreads automation level ([goodreads.md](goodreads.md#automation-levels)).

Then write `profile.json`, create an empty `tracker.csv` with the header from [tracker.md](tracker.md), and offer `taste`.

## profile.json

Start from [`../profiles/template.json`](../profiles/template.json). Fields:

| Field | Meaning |
|---|---|
| `name` | Folder slug |
| `sources` | List of `{type, path, exported_on}`; type is `goodreads`, `storygraph`, `libby-timeline`, `typed` |
| `typed_favorites` | Books or authors the user named, for cold start |
| `libraries` | `{name, key, us, verified_on}`; order is preference order. Required when Libby is used |
| `goodreads` | `{user_id, profile_url, verified_on}`. Required when Goodreads is used |
| `formats` | `audiobook`, `ebook` |
| `genres_want` / `genres_avoid` | Free text moods and genres |
| `authors_never` | Never recommend; the guard drops them |
| `authors_skip_unless_asked` | Dropped unless the user names them this run |
| `narrators_like`, `max_hours` | Audiobook preferences |
| `picks_per_run`, `wait_tolerance` | `now`, `short`, `any` |
| `approval_style` | `batch`, `one-at-a-time`, `tracker` |
| `libby_tag`, `libby_main_device`, `libby_backup_confirmed_on` | Set by the backup gate in libby-safety.md |
| `goodreads_shelves` | `{at_library, not_at_library}` |
| `goodreads_automation` | `csv`, `assisted`, `full` |
| `taste` | Saved taste report (below) |
| `storage` | `files` (Claude Code), `sheet` or `project` (claude.ai; see [tracker.md](tracker.md#storage-on-claudeai)) |
| `last_run` | ISO date |

## Taste report

Built from the history in [sources.md](sources.md). Write it for the reader, short and specific:

- **Loves**: top genres, tones and pace from 4-5 star books; repeat authors.
- **Abandons**: patterns in DNF and 1-2 star books (dense, slow, literary, and so on).
- **Rating habits**: star distribution; whether they rate generously; unrated reads count as neutral.
- **Read out**: authors whose available work they have finished; offer to add them to `authors_never`.
- **Series in progress**: series with unread books (feeds `series`).
- **Format habits**: audio vs ebook share when the source shows it.

Show it in chat, save a compact version under `taste` in the profile, and reuse it on later runs until the history changes.
