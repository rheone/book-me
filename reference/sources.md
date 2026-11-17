# Reading history sources

Every source is turned into one list of normalized **history rows**: `{title, author, rating (0-5, 0 = unrated), state, shelves[], series, date}` where `state` is `read`, `to-read`, `reading`, `dnf`, or `borrowed`.

Default intake is a **file the user downloads and gives a path to**. The browser shortcut is optional and needs Claude in Chrome (see [browser.md](browser.md)).

## Goodreads CSV

User steps (desktop site): My Books > Tools > Import and export > **Export Library**, wait for the dated link, download the `.csv`. Large libraries take minutes.

Browser shortcut (allowed at every automation level, it is the user's own export): on `https://www.goodreads.com/review/import` click Export Library, wait 30-60 s until `a[href*='review_porter/export']` appears, then fetch and parse it in the page:

```js
const a=[...document.querySelectorAll('a')].find(a=>/review_porter\/export/.test(a.href));
const txt=await fetch(a.href).then(r=>r.text());
const rows=[];let row=[],f='',q=false;
for(let i=0;i<txt.length;i++){const c=txt[i];if(q){if(c=='"'){if(txt[i+1]=='"'){f+='"';i++}else q=false}else f+=c}else if(c=='"')q=true;else if(c==','){row.push(f);f=''}else if(c=='\n'){row.push(f);rows.push(row);row=[];f=''}else if(c!='\r')f+=c}
if(f||row.length){row.push(f);rows.push(row)} // last row when the file has no trailing newline
const h=rows[0],ix=n=>h.indexOf(n);
window.__b=rows.slice(1).filter(r=>r.length>10).map(r=>({t:r[ix('Title')],a:r[ix('Author')],r:+r[ix('My Rating')],s:r[ix('Exclusive Shelf')],sh:r[ix('Bookshelves')]}));
```

Mapping:
- Find columns by header name, never position; the column list is not officially documented, so check the real header first.
- `Exclusive Shelf`: `read` -> read, `to-read`, `currently-reading` -> reading, `did-not-finish` -> dnf. A custom exclusive shelf: ask what it means once and save the answer in the profile.
- `My Rating` 0 on a read book = read, unrated (neutral).
- `Bookshelves` -> shelves. ISBNs are wrapped as `="978..."`; strip the wrapper.
- Series often sits in the title: `Title (Series Name, #3)`. Split it into `series` and the number.

## StoryGraph CSV

User steps: StoryGraph > profile > Manage Account > Export StoryGraph Library, download the CSV. Find columns by header (expect title, authors, read status, star rating out of 5 with halves, moods, DNF). Map `read status` to `state`; keep moods as extra taste signal. Round half stars only for display.

## Libby timeline export

User steps: Libby > Shelf > Timeline > Actions > **Export Timeline** > Spreadsheet (a file) or Table (a web page). The file type is not documented; read whatever arrives (CSV, XLSX, or a pasted table).

- Rows become `state: borrowed` with the date. No ratings: weight borrowed-and-returned-late or borrowed-twice as mild likes, single short borrows as neutral.
- The timeline is per device and does not sync. If it looks thin, suggest Timeline > Actions > **Recover Card History** on that device first.
- A Libby **tag export** (Tags > tag > Actions > Export Tag) also reads here: rows become `state: to-read` with shelf = tag name.

## Typed list or genres only

Ask for 3-10 books or authors loved, any disliked, and moods. Store them in `typed_favorites`. Treat loved ones as rating 5, disliked as 2. This is enough for `recs`, `like`, `tonight` and `for`.

## Merging several sources

Match rows by normalized title (lowercase letters and digits, series suffix and subtitle removed) plus author last name. On conflict keep the strongest signal: a rating beats none, `dnf` beats `to-read`, the newest date wins.

## Already-read guard

Run on every candidate list before showing it. A **title match** means: same author last name, and the normalized titles (lowercase letters and digits, series suffix and subtitle removed) are equal, or one starts with the whole of the other. Never match on a short fixed prefix: it ties sibling books together ("Harry Potter and the..." would drop the whole series after book 1).

Drop a candidate when any is true:

- It is in history as `read`, `reading`, `dnf` or `borrowed`.
- Its author is in `authors_never`, or in `authors_skip_unless_asked` and not named this run.
- It is in the tracker with status `Already Read`, `Not Interested`, or any active status (see [tracker.md](tracker.md)); `Skipped` rows may return only if the user asks for them.
- It is in an offloaded record (`profiles/<name>/history/*-offload.csv`) with any status except `Skipped`.
- It is another edition, omnibus, box set, or retitled version of something above (watch for "Collection", "Box Set", "Books 1-3", and UK/US alternate titles).
- It is a study guide or summary ("Summary of", "Analysis of", "Conversation Starters", "Workbook").

`to-read` books are not dropped; they are strong candidates and get a "on your to-read" note.
