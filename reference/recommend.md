# Building and ranking picks

## Candidate pool

Build 50-150 candidates for list modes (fewer for `tonight`), then filter. Sources, in rough order of trust:

1. Unread books by authors the reader rated 4-5.
2. Read-alikes of their 4-5 star books: same subgenre, tone, pace, era, narrator.
3. Their `to-read` shelf.
4. Open Library subjects for their top genres (below).
5. Well-known, well-reviewed books in the requested mood, from your own knowledge.

Then run the guard ([sources.md](sources.md#already-read-guard)) and check availability ([libby-catalog.md](libby-catalog.md)).

## Ranking

Rank by confidence. Weigh, in order: match to 5-star books and the moods asked for this run; tone and pace (what they abandon matters as much as what they love); format fit (audio-friendly: strong narration, not table- or footnote-heavy); on their to-read; availability within their wait tolerance. Every pick carries a one-line **why** that names the book or pattern it hangs on ("because you gave Project Hail Mary 5 stars").

Audience and length: drop titles whose catalog subjects include `Juvenile` or `Young Adult` unless the reader or the `for` recipient wants them. Drop audiobooks longer than `max_hours` when it is set.

Ratings are a signal, not a gate: use them to rank unless the user asks to filter by rating ("only authors I rated 4+").

## Modes

- **`recs`**: keep candidates at least one library owns in a chosen format. Default `picks_per_run`.
- **`check`**: no new candidates. Look up every `to-read` row; order available now, then by shortest wait, then not owned (say "owned in other format only" when true).
- **`wishlist`**: strong candidates no selected library owns in the chosen format. Keep them on a separate list from `recs`, always. Offer the purchase nudge: Libby **Notify Me** ([libby-actions.md](libby-actions.md#notify-me)).
- **`authors`**: for each author read, list their unread books the library owns. Before asking, state what the reader already read by that author (titles, series, ratings). Drop other editions of books already read.
- **`like <X>`**: X is a mood, genre, book or author. Seed from X and their 4-5 star books near X; short list (5-10) with 1-2 sentence descriptions.
- **`series`**: find every series in history (from titles, or Open Library / your knowledge). For each series with at least one read book, list **every unread book in it**, not only the next one, in series order, with library status. Optional filter the user can ask for: only series where they rated a book N+. Skip series they marked DNF unless asked.
- **`tonight`**: no interview beyond the mood. Work **catalog first**: query each library with the mood as keywords plus `showOnlyAvailable=true` ([libby-catalog.md](libby-catalog.md#lookup)), 40 per page, then guard and rank against taste. A knowledge-first pool fails here: in testing, 1 of 10 well-known funny sci-fi picks was available now (many audiobooks are Audible exclusives that libraries cannot buy). Return 3-5 picks; prefer shorter books (`formats[].duration`). If fewer than 3 survive, try a second phrasing of the mood, then offer ebooks or holds under 7 days.
- **`new`**: for authors rated 4-5 (or named), find books published in the last 12 months or announced as upcoming (Open Library `first_publish_year`, sorted new; plus your knowledge, marked as such with dates). Report library status; for unowned or upcoming ones, offer Notify Me.
- **`for <someone>`**: ask for 2-5 books the other person liked, their age range if a child, format, and occasion. Build from that alone; never read or write the user's profile or tracker unless asked. Offer to save as a separate profile.

## Open Library enrichment

Use for subjects, series hints, publication years and ISBNs. Rules from openlibrary.org/developers/api:

- Send a `User-Agent` naming the app, for example `book-me/1.0 (personal reading assistant)`. Add a contact email only if the user provides one for this purpose. Stay at or under 1 request per second without an email, 3 per second with one.
- Prefer one `search.json` call over many single lookups; cache results for the run.
- Open Library is sometimes slow or down (connection timeouts seen in testing). On a failure, wait a few seconds and retry once; if it fails again, carry on without it, mark the affected rows (for example "no ISBN found"), and say so in the report.
- `https://openlibrary.org/search.json?title=<title>&author=<author>&fields=key,title,author_name,first_publish_year,subject,isbn&limit=3`. Send title and author as separate fields: a combined `q=` missed "Mickey7 Edward Ashton" in testing.
- `https://openlibrary.org/search.json?author=<name>&sort=new&fields=title,first_publish_year&limit=20` for `new`.
- `https://openlibrary.org/subjects/<subject>.json?limit=50` (experimental) for genre candidates.
- `https://openlibrary.org/isbn/<isbn>.json` for one edition.
