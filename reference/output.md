# Output

Every suggesting run produces three things:

1. **Chat table**, ranked:

   | # | Title | Author | Why | Format | Library | Status |
   |---|---|---|---|---|---|---|

   Status is `Available now`, `N holds / M copies, ~D days`, or `Not owned`. Add a line under the table: "Availability checked <date time>; it changes." `wishlist` and `recs` are always separate tables.

2. **Tracker rows**: append each new pick as `Awaiting your pick` ([tracker.md](tracker.md)). Refresh `availability` and `checked_on` on rows already there.

3. **Artifact page**: when the Artifact tool is available, publish (or update, same file path for the same profile) a results page from `profiles/<name>/history/latest.html`: one card per pick with title, author, why, status, library, the Libby search link, and the approval status. Follow the artifact-design guidance; keep personal identifiers off the page beyond the profile name.

Then offer: approve picks, `export`, or the next mode.

## CSV export columns

Same as the tracker columns. With several libraries, add one `status_<key>` column per library.
