# Libby actions: tag, untag, hold, borrow, Notify Me

All through Libby's own UI in the user's signed-in browser ([browser.md](browser.md)). Libby has no public write API. Its private hosts (`vandal.libbyapp.com`, `sentry.libbyapp.com`) reject anything but the app and OverDrive enforces with account suspension, so fetches to them and reuse of the sign-in token are out of bounds.

Every action follows the write gate: dry run listing each title and the action, the user's OK, then act, verify, update the tracker ([tracker.md](tracker.md)), report.

## Which library

Use only library keys from the profile. Each action names its library in the dry run. When a title is at several libraries, act at the one the dry run showed, and confirm the browser's Libby has a card for it (Menu > Cards); if not, stop and ask the user to add the card.

## What to act on

Only tracker rows with status `Approved` (or every row in the current list if the user says "all"). Never act on `Already Read`, `Not Interested` or `Skipped` rows.

## Open a title

1. Navigate to the search link from [libby-catalog.md](libby-catalog.md#search-link-for-the-user-or-the-browser).
2. Wait about 5 s, take a small screenshot (scale 0.5).
3. Check the first result's title and author. With several editions, use the one showing `Borrow` or `Place Hold` in the right format. If the library has none, skip it and say so.

Positions come from a fresh screenshot every time; viewports change between runs. Wait 2 s and zoom before re-clicking anything: a double click can toggle a tag back off.

## Tag

1. On the result card click `Save`. The "Save Title to a Tag" dialog opens. A small tag chip where `Save` would be means the title is already in a tag: skip it.
2. Click the tag's circle. Zoom to confirm it is filled. The dialog's cover and count lag one title behind; trust the count going up by one.
3. Quick Saving still opens the dialog for every book; leave the setting alone.

## Untag

Unchecking the circle in the dialog **does not save**. Instead: name the titles and wait for the user's yes; open the tag page; swipe the book's row left (drag from about 90% to 30% of the row width); click the red trash button; wait; reload and confirm the count dropped. One book at a time.

## Verify tags

Reload `libbyapp.com/tags` and open the tag; the header count can lag. To list titles, scroll the page and every scrollable element to the end repeatedly (400 px steps, 350 ms apart, 60-70 times) and collect text matching `^Audiobook: '(.+)'\. Cover image\.$` (or `eBook:`). Compare with the intended list. A row with no title is a failed catalog lookup, not a lost tag. Only then set `Tagged` with the date. Suggest the user checks one linked device too.

## Hold

1. Dry run must show, per title: library, current wait, and the user's hold count vs the library's hold limit (Libby > Menu > Cards shows limits; ask the user to read it if not visible).
2. Click `Place Hold`, confirm the card if asked, and accept or adjust the notification settings the dialog offers.
3. When several holds would land at once, offer **Suspend** (formerly "Deliver Later"): the hold keeps its place in line but is not delivered until unsuspended.
4. Verify on Shelf > Holds; set tracker status `On Hold` with the date and wait estimate.

## Borrow

1. Dry run shows: library, loan count vs loan limit, loan length, and for ebooks in a U.S. library whether to read in Libby or deliver to Kindle (Kindle is U.S.-only).
2. Click `Borrow`, then confirm `Borrow!`. Verify on Shelf > Loans; set `Borrowed`.

## Notify Me

The purchase nudge for picks a library does not own. Find the title (Libby may need a "deep search" to show unowned titles), then Save it to the **Notify Me** smart tag. Libby shares the interest anonymously with the user's libraries and alerts the user if a library buys it. Not every library supports it. Verify on the Notify Me tag; set `Notify Set`.
