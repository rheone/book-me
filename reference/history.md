# History: export, offload, fresh

All three write to `profiles/<name>/history/`, with file names starting `YYYY-MM-DD-`. Each readable document is produced in three forms: **Markdown**, **CSV**, and an **artifact page** (when the Artifact tool is available).

## Readable document layout

```markdown
# book-me <kind> — <profile name>, <date>

Summary: N picks (A approved, T tagged, H on hold, ...). Sources used. Libraries checked.

## <Section per list or mode>
| Title | Author | Status | Why | Library / availability | Action date |

## Changes made to accounts
(Libby tags, holds, borrows, Notify Me; Goodreads imports and removals, with review ids)

## Notes
(taste observations, skipped items, anything that failed)
```

The CSV carries the same rows with the tracker columns. The artifact page shows the same content, grouped by section, readable on a phone.

## `export`

Save **this chat's results** (every list shown this session, decisions taken, account changes) as `<date>-export.md`, `<date>-export.csv`, and the artifact page. Leaves the tracker as is. Give the user the file paths and the page link.

## `offload`

Move **finished** rows out of the tracker into a readable record, so the tracker stays short.

1. Finished = `Tagged`, `Untagged`, `Borrowed`, `Shelved`, `Skipped`, `Not Interested`, `Already Read`, older than a cutoff the user picks (default: everything finished).
2. Dry run: show counts per status and the cutoff; wait for the OK.
3. Write `<date>-offload.md`, `<date>-offload.csv`, and the artifact page.
4. Keep the guard working: the guard reads `history/*-offload.csv` as well as the tracker, so `Already Read` and `Not Interested` titles are never suggested again.
5. Rewrite `tracker.csv` with only the remaining rows. Report counts moved and kept.

## `fresh`

Start over from the interview while keeping the past.

1. Ask: fresh for this profile (archive it) or a new profile beside it.
2. Archiving: offload the whole tracker as above, then move `profile.json` to `history/<date>-profile.json`, then create an empty tracker.
3. Ask whether to keep the never-recommend lists (`authors_never`, `Already Read`, `Not Interested` history). Default keep: the guard still reads them.
4. Run the interview in [profile.md](profile.md).
