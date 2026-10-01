# SCV Bridge Club Seating & Results

A static web page (GitHub Pages) that shows bridge game seating and results from a public Google Sheet.

Live site: https://gilgman2026.github.io/scvbridgeclub/

## How it works

- `index.html` is the whole site: no build step, no server, no secrets.
- The page reads a **public** Google Sheet (the one published to the web), not the private working sheet. The sheet's ID is the `SHEET_ID` line at the top of the script in `index.html`.
- Every tab except `Results` is treated as a game; the tab name is shown as the game's label.
- Each game tab is read like this:
  - The **Table** column is the one between the *North / South* and *East / West* headings. Everything else (flights, other columns) is ignored.
  - Each heading has two name columns. Pairs choose their own seats, so a pair is shown together as "North / South" or "East / West" with no individual direction. Blank names show as TBD.
  - For each of the four name columns, count down from the first table to the first blank cell. The **longest** of those four runs sets how many tables are shown. Names below a blank are not shown until the gap is filled.
  - The game date is the first `m/d/yyyy` cell found in sheet rows 25-30, columns A-H.
- The `Results` tab holds raw STAC score reports, one report line per row in column A. Each report's date (from its title line) is matched to the game's date and shown under the seating. No match shows "No results yet".
- The result parser is a copy of the one in [BridgeScores](https://github.com/gilgman2026/BridgeScores). If the report format changes, update both by hand.
- The page refreshes every minute.

## Public sheet setup

The public sheet pulls the seating tabs and `Results` from the private sheet with `IMPORTRANGE`, so nothing private (emails, player pool) is published. Keep the tab names in the private sheet stable and edit them in place.

## Hosting

Settings → Pages → Deploy from a branch → `main` / `(root)`.
