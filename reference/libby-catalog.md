# Libby catalog lookups (read-only)

Availability comes from OverDrive's `thunder` catalog API, the backend the Libby web app reads. It needs no sign-in. It is **undocumented and unofficial** (not on developer.overdrive.com; OverDrive's official APIs are open only to partner organizations). So: keep calls light, cache per run, and when it fails, say availability could not be checked and offer the Libby search link instead.

## Library keys

The key is the last part of `https://libbyapp.com/library/<key>` (open the library in Libby web and copy the URL). Verify each: `GET https://thunder.api.overdrive.com/v2/libraries/<key>`; 404 = wrong key. The `<name>.overdrive.com` subdomain often does not exist; ask the user for the Libby URL instead of guessing.

## Format ids

- Audiobook: `audiobook-overdrive,audiobook-mp3`
- Ebook: `ebook-overdrive,ebook-kindle,ebook-epub-adobe,ebook-epub-open`

Confirm against a result's `formats[].id` if a lookup returns nothing for a title you expect.

## Lookup

```
https://thunder.api.overdrive.com/v2/libraries/<key>/media?query=<title author>&format=<ids>&perPage=3
```

Fields: `title`, `firstCreatorName`, `isAvailable`, `holdsCount`, `ownedCopies`, `estimatedWaitDays`, `detailedSeries.seriesName`, `subjects[].name` (genres, and audience such as `Juvenile Fiction`, `Young Adult Fiction`), `formats[].duration` (audiobook length `hh:mm:ss`), `formats[].identifiers` (type `ISBN` = that edition's ISBN).

Add `&showOnlyAvailable=true` to get only titles borrowable right now. With a mood or genre as `query` (for example `humorous science fiction`), this returns an available-now candidate pool straight from the catalog. Tested 2026-11-16; undocumented like the rest of this API.

Run from a page or tool that can fetch it (CORS allows `libbyapp.com` and `goodreads.com` tabs; WebFetch or a script also work). In-page helper:

```js
const FORMATS={audio:'audiobook-overdrive,audiobook-mp3',ebook:'ebook-overdrive,ebook-kindle,ebook-epub-adobe,ebook-epub-open'};
const norm=s=>(s||'').toLowerCase().replace(/[^a-z0-9]/g,'');
async function lib(key,title,author,fmt){
 const u='https://thunder.api.overdrive.com/v2/libraries/'+key+'/media?query='+encodeURIComponent(title+' '+author)+'&format='+FORMATS[fmt]+'&perPage=3';
 const j=await fetch(u).then(r=>r.json());
 const m=(j.items||[]).find(i=>norm(i.firstCreatorName).includes(norm(author.split(' ').pop()))&&norm(i.title).startsWith(norm(title)));
 return m&&{key,avail:m.isAvailable,holds:m.holdsCount,copies:m.ownedCopies,wait:m.estimatedWaitDays,series:m.detailedSeries&&m.detailedSeries.seriesName};}
```

Batch with `Promise.all` in groups of about 10.

Pass `title` without the series suffix and subtitle (`Children of Time`, not `Children of Time (Children of Time, #1)`). Match on the **whole** normalized title: a short prefix matches sibling books in a series (a 9-letter prefix matched "Children of Strife" for "Children of Time" and reported a book the library does not own in audio as available).

## Reading the result

- `isAvailable: true` = borrow now.
- `false` with holds = wait; show `N holds / M copies, ~D days`.
- `false` with 0 holds = all copies checked out; short wait.
- No match = not owned in that format. Check the other format once before saying "not owned".

## Several libraries

Query every library in the profile. Per book, show the best option (available now first, then lowest wait) and name that library; in CSV output add one status column per library. Partner-library lending can have lower limits or no holds.

## Search link (for the user, or the browser)

`https://libbyapp.com/search/<key>/search/audiobooks/query-<url-encoded title and author>/page-1` (use `ebooks` for ebooks). The `mediatype-audiobook` path form gets rewritten; this one works.
