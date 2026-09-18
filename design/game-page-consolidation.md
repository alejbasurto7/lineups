# Game page: merging five tabs into three

Analysis of the game page as it ships today (`/teams/:teamId/games/:gameId`), and a
consolidated information architecture.

Note on scope: this repository holds the published build only, so the audit below was
read out of `assets/index-*.js` rather than the app source. Component names in
parentheses are the minified identifiers, kept so the findings can be traced back.

## What the page does today

The layout (`NE`) renders a header, a five-item tab bar (`uC`) and an outlet. The index
route redirects to `lineup`.

| Tab | Route | Component | What it draws |
| --- | --- | --- | --- |
| Attendance | `/attendance` | `IE` | "n of m attending"; every player with All / Absent and one toggle per segment |
| Positions | `/positions` | `VE` | Copy from last game; Goalie / Defense / Midfield / Forward buckets; an Anywhere bucket; Clear all |
| Lineup | `/lineup` | `eA` | Segment strip S1–Sn (`Xk`); warnings (`Zk`); the field; the bench (`zk`); Generate / Clear unpinned |
| Summary | `/summary` | `oA` | "n below their fair share"; a player × segment table with a Total column |
| Stars | `/stars` | `nA` | "n of m stars given"; four star buttons per player; a comment sheet |

## Where the page repeats itself

1. **Attendance and Summary are the same grid.** Both are players × segments. Attendance
   writes booleans into it, Summary reads slot labels back out of it. Two tabs, two scroll
   positions, one matrix.
2. **The Total column and the `BELOW_TARGET` warning state the same fact.** Summary shows
   `2` against a red row; the Lineup warning list says "Anton C. plays 1 segment (fair
   share 2)". Same number, two tabs.
3. **Three tabs each draw the whole roster.** Attendance, Positions and Stars all render
   every player with a different control strip, and each carries its own empty state
   ("Nobody is attending yet." appears in two of them).
4. **Segment is local state.** Lineup keeps the segment in `?s=`; Attendance's toggles and
   Summary's columns are segment-indexed too, but do not read it. Pick S3 on Lineup, open
   Summary, and you have lost your place.
5. **Availability is edited in one tab and consumed in three.** Turning a player off on
   Attendance silently produces an `UNAVAILABLE` warning on Lineup and blank cells on
   Summary, with no link back to the switch that caused it.
6. **Positions is setup wearing a tab.** It is per-game roster metadata that overlaps the
   player profile's preferred positions, touched once before generating, and it holds a
   fifth of the navigation.
7. **Four counts in four places.** "12 of 14 attending", "2 warnings", "2 below their fair
   share", "5 of 12 stars" — none visible from any of the others.
8. **The first tab is not the landing tab.** The index redirects to `lineup`, which is
   third in the bar.
9. **The bar mixes phases.** Attendance and Positions are pre-game, Stars is post-game.
   Five flat peers means two or three are dead weight whenever you open the page.
10. **Five tabs do not fit.** The bar is a non-scrolling flex row; on a 360 px phone the
    labels are cramped and there is no overflow affordance.

## The proposal

**Squad · Lineup · Stars**, plus one persistent segment rail and one status line.

### The rail is the page's only segment control

`Whole game | S1 | S2 | S3 | S4`, directly under the header. Everything else on the page is
drawn on those same four columns: the grid's headers, and each Squad row's availability
ticks. One motif, three appearances, always aligned — which is what makes the merge
legible rather than merely shorter.

### Squad = Attendance + Positions

One row per player: name and number, four segment ticks (who is here), one chip on the
right (where they play today, or `Any`). Header carries `Copy Sep 13 vs Gunners` and
`Everyone in`. Two tabs become one list, and the position data sits next to the
availability it interacts with.

### Lineup = Lineup + Summary

The rail chooses the view instead of a second tab:

- a segment → the field and the bench, as today;
- Whole game → the grid, with availability and playing time in the same cells.

Cell states: a slot label on a green fill (on the field), a dot on white (here, sitting
out), grey (not available). The Total column becomes `played/fair share`, and every player
chip on the field and the bench carries a four-tick fair-share meter, so the count lives
where you act on it rather than one tab away.

### Stars keeps its tab, and its timing

Giving a star is its own post-game moment with its own comment sheet, so it stays a
destination. What changes is when: it is the tab the game page lands on once the game is
in the past, and it reuses the Squad row instead of drawing a third roster.

### One status line

Three pills under the header — attendance, checks, stars — each a link to where it is
fixed. A pill is hidden on the screen that owns it, so Squad does not show the attendance
count above a list of attendance.

### One rule for the warnings list

A warning is listed only when the view cannot show it. `BELOW_TARGET` leaves the list: the
grid totals and the amber bench chips already say it. What stays is what a view cannot
draw — no goalie available, a duplicate, an out-of-position cover, a lock that cannot be
met, a player sitting two segments in a row.

### Smaller fixes carried along

- The tab bar moves to the foot of the screen. Three items, thumb reach, and it stops
  competing with the back link.
- Every control is at least 44 px, including the grid cells.
- The gold Sportsmanship star gains a darker outline; `#eab308` on white is about 1.9:1
  and fails the 3:1 floor for a graphical object.

## What this costs

Removing `/attendance`, `/positions` and `/summary` as routes breaks any bookmark or
shared link pointing at them; they should redirect to `/squad` and `/lineup?s=all`. The
grid is the one screen that gets denser, so it is the first thing to test on a small
phone with a fourteen-player roster.
