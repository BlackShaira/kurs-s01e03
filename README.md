# #4471, split into the causes the support kb computes

Ticket **#4471** at the support desk holds **412 reports** about wrong dates, merged as one issue
and marked known. Its own table always printed two causes. This repo files one ticket per cause,
with how many reports it covers, and adds the page that works the split out to the kb.

## The tickets

| cause on the kb | reports | ticket | how the desk closed it |
|---|---|---|---|
| `parser` | 379 | [issue #1](../../issues/1) | fixed, in `fix(dates): parse dd/mm for eu locales` |
| `hint` | 33 | [issue #2](../../issues/2) | won't fix: `booking-v3` was retired |

#4471 was closed on 5 October. Both tickets here are closed to match, each with the desk's own
resolution. The counts are the ticket as it was merged, and closing it does not change them.

379 + 33 = 412, every report counted once. The two causes are the two in
`src/lib/kb/causes.ts`: `export const CAUSES = ['parser', 'hint']`.

Five reports inside `parser` describe a date one day out rather than a swap. They are the desk's
own `oneDayMove` list in `src/lib/data/desk-notes.json`. They are not a third cause and they get
no ticket: `readAs` counts them where they are filed and footnotes them there, which is what the
kb does. The page names all five.

## The page

- source: `src/routes/kb/incidents/4471/split/+page.svx`
- as the desk will see it: `rendered/kb/incidents/4471/split/index.html`
- signed in its frontmatter, and it ends with `<RenderStamp {signature} />`, as the kb's README
  asks. Rendered desk ref: `DR-VMC5-QD10`.

Every number on the page is computed by `src/lib/kb/SplitTable.svelte` from the desk's own export
with the desk's own rules. No number is typed into a paragraph.

## What is the export and what is ours

`src/` is the kb export, unchanged, except:

| file | what we did |
|---|---|
| `src/routes/kb/incidents/4471/split/+page.svx` | new: our page |
| `src/lib/kb/SplitTable.svelte` | new: the component that counts it |
| `src/routes/kb/incidents/4471/+page.svx` | one paragraph added, pointing at the page above |

`src/lib/data/` is untouched: every report still has `mergedInto: 4471`, so the desk's view of
#4471 counts the same before and after.

The export ships `src/` only, with no `package.json`, so the `.svx` cannot be built here. The
rendered copy is produced by a renderer that reads the page source and reuses the kb's own logic
and the layout's own markup, so the words have one source.
