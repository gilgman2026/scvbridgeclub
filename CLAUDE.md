# CLAUDE.md

Notes for working on this repo (`gilgman2026/scvbridgeclub`, formerly `Bridge-Seats`).

## What this is

"SCV Bridge Club Seating & Results": a single static page (`index.html`) hosted on GitHub Pages
(`https://gilgman2026.github.io/scvbridgeclub/`, deploy from `main` / root). It shows bridge game
seating and results read from a **public** Google Sheet, in the visitor's browser. No server, no build
step, no secrets, no dependencies. Keep it that way: vanilla JS, one file.

Related repo: `gilgman2026/BridgeScores` (a single `index.html` that reformats pasted STAC score reports).
The result parser here is a copy of that one.

## Data flow

Private working sheet --(IMPORTRANGE)--> **public sheet** (published to the web) --> this page.

- Only the public sheet's ID is in this repo (`SHEET_ID` at the top of the script). **Never put the
  private sheet's ID or link in this repo**: it contains contact data (emails, player pool).
- The public sheet has one tab per game plus a `Results` tab. Tab names are currently "Game 1 (teach)",
  "Game 2 (open)" and "Results". The organiser edits those tabs in place and does not rename them.
- The page discovers tabs by reading the published sheet's `/htmlview` page (names + gids). Every tab
  except `Results` is a game; the tab name is the game label. If discovery fails it falls back to
  `FALLBACK_TABS` in the script (gids 0, 2089762461 and 1628264295).
- Tab data is fetched as CSV from `.../gviz/tq?tqx=out:csv&gid=<gid>`. **Always fetch by gid.** With
  `sheet=<name>`, Google silently returns the *first* tab when the name doesn't match, with no error.
- The page only re-renders when the fetched data actually changed (signature compare in `load()`), and it
  restores the scroll position when it does. This was added after the owner reported being unable to scroll
  back up on iPhone Chrome (WebKit); redrawing the DOM mid-swipe was the suspected cause but could not be
  reproduced in Chromium, so it is unconfirmed. A "↑ Top" button appears after scrolling 300 px.
- Google allows these cross-origin fetches from `github.io`. The page polls every 60 s (`REFRESH_MS`).
  End-to-end lag after an edit in the private sheet is roughly 2-4 minutes (IMPORTRANGE + publish cache
  + poll).

## Seating rules (agreed with the owner; do not "improve" without asking)

- Find the header row containing both "North / South" and "East / West". The **Table column is the
  "Table" heading between those two**. Ignore everything else, including flights and any other Table
  column. (Older tabs had a second Table column on the left; Oct 2 style tabs have one.) Locate columns
  by heading text, not fixed positions: layouts differ between tabs.
- Each heading owns two name columns (heading column and the next one).
- **Pairs choose their own seats**, so the page shows a pair together under "North / South" and "East / West"
  and never an individual N, S, E or W. Blank names show as `TBD`.
- **List length:** for each of the four name columns, scan down from the first table; the list ends after
  `END_GAP` (= 3) empty cells in a row. A single blank in the middle is tolerated (not assigned yet). The
  **longest** of the four columns sets the number of tables shown. Anything after a 3-row gap (for example a
  waiting list further down) is not shown. The owner may tune `END_GAP`; it was strictly "first blank ends it"
  for a while, then changed back to 3.
- Cells contain zero-width spaces (`​`) when "empty": always clean cells before testing for blank.
- **Game date:** the first `m/d/yyyy` cell found scanning sheet rows 25-30, columns A-H (row by row, left to
  right), then stop. It is usually inside a name column, so the cell is blanked and never treated as a name.
  No date means the game is still shown, but results are not matched.
- The table numbering may skip numbers (for example no table 13). Show the sheet's numbers, not row positions.
- The default game is the next upcoming date (today or later), otherwise the most recent.

## Results rules

- The `Results` tab holds raw STAC reports, one report line per row in column A.
- Parsing reuses the BridgeScores code (`locateReports`, `parseDataLine`, ...) copied verbatim into the
  script. If STAC's format changes, fix it there and in BridgeScores by hand; they are intentionally duplicated.
- Deliberate differences from BridgeScores (all outside the copied block):
  - `restoreSubHeaderIndent`: pasting into a sheet cell drops the leading spaces on the `A B C` sub-header
    line, but ranks are placed by character position. The indent is restored (first "A" sits 3 characters left
    of "Section"). Verified only with the older single-group format; the newer "Overall Rank" format is untested.
  - `tidyTitle`: "Wednesday Morning Pairs Wednesday Mor Session September 30, 2026" is shown as
    "Wednesday Morning Pairs — September 30, 2026". Other titles fall back to the BridgeScores wording.
- A game's results are the reports whose title date matches the game's date (same y/m/d). There are
  normally two per date (N/S and E/W sections). No match shows "No results yet — this game hasn't been
  played, or the results haven't been posted."
- Names in results do not always match the seating names exactly (for example Harry / Harkirat, Murray /
  Murrays), and result pair numbers are not table numbers. Do not try to join the two by name.

## Find my seat

- Starts after 2 characters. Matches from the **left of the name** only (prefix match, case-insensitive),
  so "ru" finds Ruth Baker but "baker" finds nothing. The prompt asks for a first name; the no-match
  message is exactly "Not found, is that your first name?". Keep copy concise.
- Matches show table, pair side and partner, across all games; clicking one switches game and scrolls to the card.
  The same prefix rule highlights the pair in the results tables.

## Testing

No test framework. To check changes, use headless Chromium (pre-installed, `/opt/pw-browsers`) via Playwright
and intercept `https://docs.google.com/**` with `page.route`, fetching the real data with `curl` (the browser
cannot use the session proxy directly). Things worth checking after a change: Game 1 shows 7 tables and Game 2
shows 11 (as of the 2026-09-30 / 2026-10-02 data), the Sep 30 results render with ranks under the right A/B/C
columns, a phone-width screenshot, and no `pageerror`. In the Claude Code web sandbox, outbound access needs
`docs.google.com` and `*.googleusercontent.com` (Google redirects CSV downloads to a googleusercontent
subdomain); `github.io` is not reachable, so the live site cannot be checked from there.

## Git and deployment

- Push to `main`; GitHub Pages deploys from it. No PRs unless the owner asks.
- Renaming the repo changes the Pages URL and does not redirect the old one. Update the link in `README.md`.
- Commit messages end with the Co-Authored-By and Claude-Session trailer lines given for the session.

## Owner preferences

- Plain, concise wording on the page and in replies. Mobile-first: most players look at this on a phone.
- Public site, but keep private data out of the repo and out of the public sheet.
