# Goodreads: shelving picks

Goodreads closed its API to new keys in December 2020. Its Terms of Use exclude "data mining, robots, or similar data gathering and extraction tools". The CSV export and import are features for the registered user, so book-me automates those by default; other in-page scripting is opt-in.

## Automation levels

Stored as `goodreads_automation` in the profile; ask the first time `shelve` runs.

| Level | What book-me does |
|---|---|
| `csv` (default) | Export and import CSVs, by clicking the page's own buttons or by handing the user a file. ISBNs from Open Library. Audit from a fresh export. |
| `assisted` | Adds in-page scripts on the user's own pages: Goodreads ISBN lookup, reading shelf list pages, removing rows with the page's Remove link. |
| `full` | All of the above without pausing between steps, still behind one dry run and OK per batch. |

Before switching to `assisted` or `full`, say once: this goes beyond the export/import features, the Terms of Use discourage automated tools, and the risk is on the user's account. Record their choice.

## Steps

0. **Account check.** Read `goodreads.user_id` from the profile; `<id>` below means that value. Missing or unverified: collect it first ([profile.md](profile.md#interview)). When working in the browser, confirm the signed-in account matches it (the My Books link contains `/review/list/<id>`); if it does not, stop and tell the user which account is signed in.
1. **Rows.** Take tracker rows with status `Approved` (or the list the user names). Separate shelves: picks a library owns -> `goodreads_shelves.at_library`; not owned -> `not_at_library`. Never mix them.
2. **ISBNs.**
   - `csv`: Open Library `search.json?title=<title>&author=<author>&fields=title,author_name,isbn&limit=3`. From the matching work's `isbn` list, pick a 13-digit ISBN starting `9780` or `9781` (English-language groups). The first ISBN in the list is often a foreign edition (testing picked French `9782...` editions), and Goodreads would shelve that edition. No English ISBN: show the row as "no ISBN" in the dry run and ask.
   - `assisted`/`full`: on goodreads.com `fetch('/book/auto_complete?format=json&q='+encodeURIComponent(title+' '+author))`, pick the result whose author last name matches and title starts the same, then `fetch('/book/show/'+bookId)` and read `"isbn"` or `"isbn13"`.
   - Last resort: the library edition's ISBN from Libby catalog `formats[].identifiers`. Audiobook ISBNs often land on the wrong Goodreads book; the audit catches it.
   - Reject any match whose title contains set, collection, box, omnibus, summary, analysis, study guide, workbook or conversation starters.
3. **Dry run.** Show title, author, ISBN, target shelf, and warn: every imported row also lands on **Want to Read**. Wait for the OK.
4. **Build one CSV per shelf**, exactly this header (Goodreads' sample, odd spacing included):

   ```
   Title, Author, ISBN, My Rating, Average Rating, Publisher, Binding, Year Published, Original Publication Year, Date Read, Date Added,Shelves, Bookshelves, My Review
   ```

   Quote Title and Author; plain digits in ISBN (no `="..."` wrapper); `Shelves` = `to-read` (blank sends every row to **Read**, which corrupts later runs); `Bookshelves` = the shelf name, hyphenated (spaces separate shelf names; missing shelves are created); rating and review blank (they overwrite existing ones). Under 500 rows per file, UTF-8.
5. **Import.** At `https://www.goodreads.com/review/import`, "Import from a File". By script:

   ```js
   const dt=new DataTransfer();
   dt.items.add(new File([csvText],'picks.csv',{type:'text/csv'}));
   document.querySelector('input[type=file]').files=dt.files;
   // then click the first "Import Books" button (the file form's)
   ```

   Or save the CSV in `profiles/<name>/history/` and tell the user to upload it. Speed is about 15-20 books per 30 s. Poll by re-reading the "Import Results" block (still processing / successfully imported / already on your shelves / could not import). Submit the second shelf only after the first finishes.
6. **Audit (always).** Get the shelf's contents (fresh export at `csv`; shelf list page `/review/list/<id>?shelf=<shelf>&view=table&per_page=100&sort=date_added&order=d` at `assisted`/`full`, looping pages). Compare with the intended list by normalized title prefix. Flag wrong editions (study guides, sets), author mismatches, duplicates, rows on the wrong shelf, anything that landed on Read.
7. **Change log.** Record each added book (title, shelf, time; review id `review_<n>` when visible) in the tracker notes and the run report, so undo is possible.
8. **Fix.** `csv`: tell the user which rows to remove by hand. `assisted`/`full`: on the fully loaded list page, `window.confirm=()=>true; document.getElementById('review_<n>').querySelector('a.deleteLink').click();`, wait about 3 s, verify the row is gone. Re-import with another edition's ISBN. Raw POSTs to shelf endpoints fail; use the page's controls.
9. **Report** what changed, including mistakes.

## Facts worth knowing

- Import never changes the exclusive shelf of a book already shelved; it only adds the custom shelf.
- Re-importing the same file is harmless.
- Default shelves (Want to Read, Currently Reading, Read, Did Not Finish) are exclusive; a book has exactly one exclusive shelf.
- Listopia lists are public and every add is a vote; keep picks on private shelves. Shelf visibility follows the profile's privacy setting.
- Goodreads does not keep shelf order, so rank lives in the tracker, not the import.
