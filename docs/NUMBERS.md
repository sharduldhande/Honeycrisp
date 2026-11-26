# Honeycrisp in numbers

Every number below was measured on one machine; each carries how it was measured, how many runs,
and where it comes from. Nothing is estimated.

- **Machine:** Mac mini M1, 8 GB RAM, macOS 27.2 (26B123), one 1920×1080 display at 1× and 120 Hz.
- **Music:** the Music app that ships with macOS 27.2 (Info.plist `CFBundleShortVersionString` 1.7.2).
- **Account:** an Apple Music account (India storefront) whose library holds 2,353 songs,
  1,427 albums and 778 artists (the counts the tour logs).
- **Honeycrisp:** the published release build, commit `6a615c2` (release build plus the search
  suggestions dropdown switched off). Table 2's tour figures are from the release build
  (`e8e9e31`), which differs from `6a615c2` only in that dropdown.
- **Date:** 2026-11-16.

## 1. Honeycrisp vs Music (same Mac, same account)

Only rows where both apps were measured, the same way, on the same day.

| What | Honeycrisp | Music | Runs | How measured | Source |
|---|---|---|---|---|---|
| App size on disk | 22,516 KB (22.0 MB) | 66,140 KB (64.6 MB) | 1 each | `du -sk` of the whole `.app` bundle. Music is a system app on the sealed system volume; both also use system frameworks (MusicKit, AppKit) that neither bundle counts | measured 2026-11-16 by this task |
| Launch to first window on screen | median 692 ms (654–1,171) | median 1,316 ms (1,213–1,827) | 6 each | `open` the app, then poll the on-screen window list every 10 ms until the app has a normal window (≥ 600×400 pt). Each launch after both apps were quit. Launch to window, not to content | measured 2026-11-16 by this task |
| Memory footprint 69–70 s after launch, idle | median 217 MB (215–246) | median 203 MB (194–236) | 6 each | `footprint -p` (phys_footprint) 68 s after the first window (68.7–69.8 s after the launch request), nothing touched after launch, each app on the page it opens on | measured 2026-11-16 by this task |
| CPU time used by launch plus idle, to 69–70 s | median 6.3 s (6.1–6.3) | median 3.9 s (3.7–4.3) | 6 each | `ps -o cputime` at the same moment, same runs | measured 2026-11-16 by this task |

The launch row is the build before LAUNCH1; 1j has the launch against a bare AppKit app, before
and after. The launch, memory and CPU rows come from 6 screen-lock holds, each launching both apps one after
the other (order alternated between holds, 21:41–22:05 CET). No compiler was running at any start;
1-minute load average at each launch 1.45–3.30, except Music's first launch (22.0) and second (6.6),
which are its two slowest launches (1,827 and 1,280 ms). Honeycrisp's slowest (1,171 ms) was its first
launch of the session. In these runs Honeycrisp's covers were answered from local files with no image
request sent (`HONEYCRISP_IMG1_DRY=1`, confirmed by its "IMG1 stats at quit" line: 0 sent in all 6);
Music drew its own covers. Honeycrisp opens on its Songs list; Music opened on the page it last
showed (not set by this task). Honeycrisp's higher CPU time includes its library load (all 2,353
songs, albums, artists and playlists) at every launch.

## 1b. Click to first paint of a page (Honeycrisp vs Music, warm)

Both apps were measured by one method, a screen recording, so no in-app timer is involved. Each
page had already been visited once in that session. The figures are medians (range) of 5 runs per
app, in ms from the mouse-down (the worst run is the range's upper end).

| Page (one click in the sidebar or list) | Honeycrisp: content on screen | Honeycrisp: complete | Music: content on screen | Music: complete |
|---|---|---|---|---|
| Home | 92 (80–98) | 92 (80–98) | 154 (149–165) | 167 (158–204) |
| Songs (2,353 songs) | 45 (42–57) | 45 (42–57) | 111 (102–116) | 111 (102–116) |
| Albums (1,427 albums, cover grid) | 44 (36–52) | 44 (36–52) | 111 (109–120), titles with empty cover boxes | 126 (117–144), covers in |
| An artist in the library's Artists view (Aaliyah, detail pane) | 71 (62–72) | 71 (62–72) | 100 (97–138) | 100 (97–138) |
| An album page, one click on its Home tile ("Meddle (2016 Remaster)")² | 163 (158–167) | 167 (165–175) | 241 (237–250); an empty page from 82 (78–92) | 395 (239–407) |
| New² | 100 (92–110) | 100 (92–110) | 141 (138–149) | 141 (138–149) |
| Radio² | 72 (63–82); its top row of 6 live-station covers stayed blank³ | 72 (63–82)³ | 142 (138–159) | 142 (138–159) |
| Artists (778, list and last-selected artist)² | 49 (40–61) | 49 (40–61) | 153 (147–158) | 166 (164–174) |
| Recently Added² | 57 (47–67); some covers missing³ | 57 (47–67)³ | 122 (117–133), titles with empty cover boxes | 125 (121–133), covers in |

² Second set, 22:49–23:05 CET: 6 runs per app (Music's album page: 5, because its first run
started on the album page itself). 1-minute load average 1.64–5.34, no compiler running.
For the album page the previous page was Home itself: Songs, then Home, then Home's back button
(Music keeps the album open under Home), then the measured click. Music first paints an empty
page for the album (title bar only), so its "content on screen" is the first frame with real
content: more than 3 % of the region more than 24 grey levels away from the frame's median grey.
The empty page's time is given as well.
³ Honeycrisp's covers came only from its disk cache (`HONEYCRISP_IMG1_DRY=1`, nothing sent). Covers it
didn't have stayed blank or dark, so "complete" here means "nothing else arrived". With network
covers Honeycrisp's "complete" on these two pages would be later. Music fetched what it fetches.

**How these were measured** (measured 2026-11-16, 22:34–22:45 CET, by this task):
- **Tool:** a small Swift tool, using only public API, records the display with ScreenCaptureKit
  (`SCStream`, 1920×1080, `minimumFrameInterval` 1/120 s). Every frame's presentation timestamp is
  host time. The same tool posts the click with `CGEvent` and reads the host clock just before the
  mouse-down, so the click and the frames share one clock. The mouse-up follows 8.3 ms (one frame)
  later. The pointer is moved to the target 700 ms before the click.
- **Frame interval:** frames arrived 8.33 ms apart (120 Hz), so each figure is good to one frame.
  ScreenCaptureKit only delivers a frame when the screen changes.
- **Same conditions for both apps:** both windows were at the same place and size (1440×900 pt at
  240,60), with nothing playing and Music's queue panel closed. The apps took turns, page by page,
  and which app went first alternated between the 5 runs. Each run was one launch of both apps
  inside one screen-lock hold. Every page was visited once before it was measured, and the
  previous page was opened 1.5 s before each measured click.
- **Region:** only the page's content area is compared (x 445–1670, y 65–878 on screen). This
  leaves out the sidebar, whose selection changes at once, and the floating player. For the artist,
  only the detail pane (x 745–1670) is compared, because the list's selection highlight would count
  otherwise. Pixels are sampled every 3rd pixel, and a pixel counts as changed when its grey level
  moves by more than 12 of 255.
- **"Content on screen":** the first frame where more than 0.5 % of the region differs from the
  frame before the click.
- **"Complete":** the last frame in the 2.5 s after the click where more than 0.3 % of the region
  changed from the frame before it. Pixels still changing in the last 600 ms (an animation that
  never settles) are left out. That only happened on Music's Home: its animated "Get Up!" cover
  moves about 0.1–0.4 % of the region in every frame and is not counted as loading.
- **Checked by eye:** the before, first-content and complete frames were looked at for every page
  in both apps (kept in the task's scratch folder). In both apps the first changed frame already
  shows the new page's text and covers, except Music's Albums. That grid shows its titles over
  empty cover boxes for 1–2 frames, then the covers; both moments are given.
- **Covers and load:** Honeycrisp's covers came from its own disk cache (`HONEYCRISP_IMG1_DRY=1`, 0 image
  requests sent in all 6 launches). Music drew covers from its own caches. One of Honeycrisp's Home
  covers was missing (drawn dark) because it wasn't in the cache. The 1-minute load average at the
  clicks was 1.56–3.05, and no compiler was running.

### A playlist page (warm)

The library playlist "Desi Hits" was opened with one click in the sidebar, coming from Songs,
which was opened 1.5 s before. Same recording method and region as above. Both apps were in one
launch, the playlist had been opened once before, and there were 5 runs per app, alternating,
00:10–00:12 CET on 2026-11-17 (1-minute load average 2.3–2.8, no compiler running).

| Page | Honeycrisp: content on screen | Honeycrisp: complete | Music: content on screen | Music: complete |
|---|---|---|---|---|
| Playlist "Desi Hits" (sidebar) | 98 (96–100) | 98 (96–100) | 354 (346–359) | 413 (404–426) |

- **Checked by eye:** both apps drew the playlist page, with its tracks, in one frame. Music's
  "complete" is its toolbar buttons (top right) coming in later. Until then, Music kept Songs on
  screen unchanged, with no empty page.
- **Covers:** Honeycrisp's came from its disk cache with 0 image requests sent.
- **Sidebar:** Honeycrisp's Playlists section in the sidebar was expanded for this and left expanded.

### Cold: first visit after launch

The recording method is the same as table 1b. Each run is a fresh launch of both apps (18 s
after `open`, then the first click on the page). Figures are the median (worst) in ms from the
mouse-down.

| Page | Honeycrisp: content on screen | Honeycrisp: complete | Music: content on screen | Music: complete | Runs |
|---|---|---|---|---|---|
| Albums (cover grid), first visit after launch | 44 (62) | 44 (62) | 141 (168) | 156 (199) | Honeycrisp 6, Music 5 |

- **Which runs count:** Honeycrisp always opens on Songs. Music opens on whatever page it last showed:
  Home, Albums, Search or Artists in these six launches. Its one launch that opened on Albums is
  left out.
- **Pages not given cold:**
  - Songs, because Honeycrisp opens on it.
  - Home, because Music opened on it.
  - Artists, because Music opened on it in 3 of 5 launches, so only 2 cold Music runs exist.
  - Catalog pages opened from Home's tiles: those tiles differ from launch to launch in both apps.
    Catalog pages by a fixed route are in the next table.
- **Covers and load:** Honeycrisp's covers came from its disk cache, with 0 image requests sent in all
  launches (`HONEYCRISP_IMG1_DRY=1`). Music drew from its own cache. Runs 23:19–23:26 CET, 1-minute
  load average 1.8–3.0.

### Cold: catalog pages (network-bound) by a fixed route

Same recording method. Each run is a fresh launch of both apps, and each page was measured once
per launch, on its first visit. There were 5 launches, 23:45–23:55 CET, and which app went first
alternated. Figures are the median (worst) in ms from the mouse-down. "Content" is the first frame
of the new page with its text in. "Complete" means all text and covers are in (see the notes for
Music's artist page).

| Click | Honeycrisp: content | Honeycrisp: complete | Music: content | Music: complete |
|---|---|---|---|---|
| Artist name on the catalog album "Meddle (2016 Remaster)" → catalog artist page "Pink Floyd" | 533 (803) | 533 (803) | 666 (669), text over empty cover boxes | 692 (716), covers in⁴ |
| First "Essential Albums" tile on that artist page → catalog album "The Wall" | 166 (186)⁵ | 180 (446)⁵ | 542 (555) | 709 (722) |
| Library album → its catalog album ("Meddle (2016 Remaster)")⁶ | 132 (142) | 135 (159) | 813 (817) | 982 (988) |

**The route.** In both apps the library album "Meddle (2016 Remaster)" was opened from Recently
Added. Honeycrisp opened it with one click on its tile. Music opened it by an AX press on its tile,
because a synthetic click only selects a tile in Music's grids. Then each app went to the
catalog version of that album, its artist page and an album on that page:
- **To the catalog album:** in Honeycrisp, one click on "Show Complete Album". Music shows no such
  link on this album, so its "…" menu was opened (not timed) and the timed click was on "Show
  in Apple Music". Music's time includes the menu closing. Its menu stayed on screen until
  453 (459) ms, and then an empty page showed until the content came.
- **To the artist:** a click on the album's artist link.
- **To the album:** the artist page was scrolled by the same wheel events (10 × 40 px) in both
  apps. Then came a click on the first "Essential Albums" tile, "The Wall" (found by AX in Music).
  A still before every timed click confirmed that the right page and tile were on screen.

**What each app had from before (not cleared).** These are first visits after launch, not first
visits ever. Both apps had opened these pages in the scouting runs 2–12 minutes before.
- Honeycrisp's log says it drew all three pages from its own disk copy of the page in every launch,
  except "The Wall" in the first launch, which it had never opened. That run is the worst Honeycrisp
  figure in the album row: the header (cover, title, buttons) came at 88 ms over an empty track
  list, and the tracks at 446 ms.
- Music's disk caches can't be inspected or cleared here, so what Music had cached is unknown.

⁴ Music's artist page then fades in its header video, which keeps playing, so its last changed
frame is not a load time. "Complete" is the first frame with all covers in. That was 25–33 ms
after the text (one launch: 50 ms). The header was black until the video came in.
⁵ Honeycrisp showed covers from its disk cache only (`HONEYCRISP_IMG1_DRY=1`, 0 image requests sent in
all 5 launches). Covers it didn't have stayed dark: one of the four Essential Albums tiles, and
some tiles in the "More by Pink Floyd" and "Albums" rows. With network covers, Honeycrisp's
"complete" on these pages would be later.
⁶ The two apps used different controls for this row, a link in Honeycrisp and a menu item in Music.

### Search: typing an artist's name

Same recording method. Each run was a fresh launch of both apps, with one search per app, the
first after launch. There were 5 launches, 00:00–00:05 CET on 2026-11-17, and which app went first
alternated. "coldplay" was typed into each app's search field as key events, one letter every
150 ms, with the same tool that records the screen. The two apps start a search differently:
- **Honeycrisp** searches as you type, so its clock starts at the last letter.
- **Music** searches only on Return, so its clock starts at Return, 1.5 s after the last letter.

Figures are the median (worst) in ms. The region is the left part of the results (x 445–1225,
y 110–768).

| | Honeycrisp (from the last letter) | Music (from Return) |
|---|---|---|
| First change on screen | 50 (51): the Apple Music part of the results dims, with a spinner by its heading | 86 (94): the page goes empty |
| Results on screen | the library and Apple Music results for "coldpla" were already on screen before the last letter; the final Apple Music results: 783 (884) | 821 (927) |
| Last change | 783 (884) | 821 (961) |

- **Per run:** Honeycrisp's final results came at 84, 84, 783, 801 and 884 ms. Music's results came
  at 486, 493, 821, 828 and 927 ms.
- **Honeycrisp's two quick runs:** in the two launches where Honeycrisp searched first, the final
  Apple Music results matched what "coldpla" had already shown, and the dimming lifted at 84 ms.
- **Honeycrisp's three slower runs:** a different set of Apple Music results (with "Coldplay
  Essentials") replaced the dimmed one at 309–409 ms. At 783–884 ms that set was replaced by the
  final one, which was the same set shown before the last letter. So in those runs the Apple Music
  results changed twice after the last letter. Honeycrisp's library results ("In Your Library",
  "Songs") did not change after the last letter in any run.
- **Music:** it showed an empty page from 70–94 ms until its results, which came in one frame
  (two runs had a second, small change 26–34 ms later).
- **Covers and load:** Honeycrisp's covers came from its disk cache (`HONEYCRISP_IMG1_DRY=1`, 0 image
  requests sent). The 1-minute load average was 2.7–5.0, and no compiler was running.
- **Why the previous attempt failed:** the earlier attempt's key events reached neither app. Its
  tool cleared the field with a Cmd-flagged "a" key event, and that left Cmd held down in the
  system's key state, so later letters arrived as Cmd shortcuts. These runs cleared the field
  with Backspace instead, and checked before and after each run that no modifier key was down.

### What is on screen while a page loads

These states were classified by looking at the recorded frames of the runs above. Times are in ms
from the click (or from the last letter or Return), as a median, with the range in brackets. "Old
page" means the page that was there before the click stays on screen, unchanged.

| Page | Honeycrisp | Music |
|---|---|---|
| Catalog artist page, cold | old page until 533 (409–803), then the whole page in one frame. No empty page, spinner or placeholder seen | empty (black) page from 106 (103–108) to 666 (637–669). Then text over empty cover boxes until 692 (662–716), then covers |
| Catalog album from the artist page, cold | old page until 166 (88–186), then the whole page. In the one launch where Honeycrisp had never opened this album: header (cover, title, buttons) over an empty track list from 88 to 446 | empty page (toolbar only) from 113 (102–117) to 542 (504–555) |
| Library album → catalog album, cold | old page until 132 (118–142) | the menu stays open until 453 (440–459). Then an empty page until 813 (803–817) |
| Search, first after launch | the Apple Music part of the results dims, with a spinner by its heading, from 50 (42–51) to 783 (84–884). In 3 of 5 runs a different result set showed in between (see Search) | empty page from 86 (70–94) to 821 (486–927) |
| Album page from a Home tile, warm (table 1b) | old page until 163 (158–167). None seen | empty page from 82 (78–92) to 241 (237–250) |
| Albums grid, warm (table 1b) | none seen | titles over empty cover boxes from 111 (109–120) to 126 (117–144) |
| Albums grid, cold (first visit after launch) | none seen | titles over empty cover boxes in 3 of 5 runs, for 9, 17 and 84 ms. In one run the covers came with the first changed frame. In one run (Music had opened on Home) the kept frames don't settle it |
| Home, warm (table 1b) | none seen | none seen |
| Playlist from the sidebar, warm | none seen (old page until 98) | old page until 354 (346–359), then the page; its toolbar buttons follow 59 ms later |

- **What was looked at:** for the runs above, every changed frame up to the first 120 was kept and
  looked at. For the rows from table 1b and the cold Albums row, the earlier task kept 8 frames
  per run: before the click, first change, the 1st and 3rd frame after it, first content, last
  change and the frame before it, and the final frame. Those frames and the per-frame change log
  were read again here. A state shorter than one frame (8.3 ms) can't be seen.
- **Honeycrisp's empty cover tiles** (dark squares) in these runs are covers that weren't in its disk
  cache while nothing was sent (`HONEYCRISP_IMG1_DRY=1`). They are not loading placeholders and are not
  counted here.

## 1c. Scrolling: frames on screen during the same scripted scroll (Honeycrisp vs Music)

Both apps got the same scroll, recorded by the same tool as table 1b. Each figure is the median
and the worst of 6 runs per app (3 scrolling down and 3 back up).

| List | Honeycrisp: 120 Hz frames missed while the list moves | Honeycrisp: longest gap | Music: frames missed | Music: longest gap |
|---|---|---|---|---|
| Songs (2,353 rows) | 8.1 % (worst 10.9 %) | 16.7 ms (worst 16.7) | 9.6 % (worst 10.8 %) | 16.7 ms (worst 16.7) |
| Albums cover grid (1,427 albums) | 10.4 % (worst 19.3 %) | 16.7 ms (worst 25.0) | 38.4 % (worst 44.9 %) | 25.0 ms (worst 41.7) |

**What was posted:** 120 scroll-wheel `CGEvent`s (pixel units, 60 px each), one every 8.33 ms
on absolute deadlines (1.0 s in all, 7,200 px), with the pointer still over the list at 1000,500
on screen. The events have no momentum phase, so the list stops when they stop. Before each run
the page was opened from the sidebar. Window, place and region were the same as in table 1b. The
runs were 23:08–23:14 CET, with a 1-minute load average of 1.9–3.7 and no compiler running.

**How it is counted:** ScreenCaptureKit delivers a frame only when the screen changes.
- **Frame counts:** a frame counts as presented when more than 0.03 % of the list region differs
  from the frame before it. The list is "moving" from its first changed frame to its last changed
  frame no later than 50 ms after the last event. The expected count is the number of 8.33 ms slots
  in that window; "missed" = (expected − presented) / expected. The longest gap is the longest time
  between two presented frames in that window.
- **After the stop:** the last change came, as a median (worst): Honeycrisp Songs 11.2 (14.2) ms, Albums
  12.7 (18.1) ms; Music Songs 14.8 (16.4) ms, Albums 14.7 (16.3) ms after the last event. For the grid this includes any covers drawn late.
- **Blank tiles:** every 20th moving frame of the grid runs (5–6 per run) was looked at. No
  placeholder tiles stood out in either app, but a dark cover can't be told from an empty one at
  this sampling. Blank tiles were not counted automatically.
- **Limits:** a recording shows the frames that reached the screen, not each app's own frame
  times. A frame the capture itself dropped would count as missed for either app. Both lists
  scrolled through covers already cached: Honeycrisp from its disk cache with nothing sent
  (`HONEYCRISP_IMG1_DRY=1`, 0 requests), Music from its own cache.

## 1d. Memory at the end of a scripted session

Each app ran the same scripted session alone: about 4.5 minutes of pages, scrolling, a search
and 3 minutes of playback. Its memory was read 12 s after playback was paused.

**Build `82d14a4`** (release build). 5 runs per app, the apps taking turns (Honeycrisp first in runs 1, 3
and 5, Music first in 2 and 4), on 2026-11-17, 11:38–12:34 CET:

| What (MB, physical footprint) | Honeycrisp | Music |
|---|---|---|
| The app's own process, at the end | median 287 (277–297) | median 462 (449–504) |
| The app's own process, highest during the session | median 300 (286–307) | median 723 (691–762) |
| Player process: RemotePlayerService for Honeycrisp; Music plays inside its own process | 52 (51–53) | none |
| App plus its own helpers (player, Honeycrisp's `log` child: 3.2), at the end | median 342 (333–353) | median 462 (449–504) |

Per run, app's own process at the end (in brackets: the highest during the session):

| Run | Honeycrisp | Music |
|---|---|---|
| 1 | 297 (307) | 463 (715) |
| 2 | 294 (307) | 449 (691) |
| 3 | 287 (300) | 462 (757) |
| 4 | 277 (286) | 504 (723) |
| 5 | 285 (297) | 456 (762) |

The highest figure is the kernel's `ri_lifetime_max_phys_footprint` for the process, read at
the end. In the three runs sampled every 10 s, Music's highest came 30–40 s after launch,
during the pages (Albums, the artist, the album). It was 511–549 MB when playback started at
80 s and 454–506 MB 3 minutes later. Honeycrisp's highest came at 70–80 s (after the search), and
it was 277–287 MB at the end of playback.

**Earlier builds `e35e6ae` and `829caac`** (runs 1–2 on `e35e6ae`, run 3 on `829caac`, the
published build of the moment). 3 runs per app, the apps taking turns, on 2026-11-17,
04:28–05:55 CET:

| What (MB, physical footprint) | Honeycrisp | Music |
|---|---|---|
| The app's own process | median 263 (257–291) | median 462 (461–463) |
| Player process: RemotePlayerService for Honeycrisp; Music plays inside its own process | 51 (51–52) | none |
| Other own helper: Honeycrisp's `log` child | 3.2 (3.2–3.3) | none |
| App plus the processes above | median 318 (312–346) | median 462 (461–463) |

Per run (app's own process; the figure in brackets is app plus player plus `log`):

| Run | Honeycrisp | Music |
|---|---|---|
| 1 | 291 (346) | 463 |
| 2 | 257 (312) | 462 |
| 3 | 263 (318) | 461 |

**Agents both apps use:** Apple's AMPLibraryAgent, AMPArtworkAgent, AMPDeviceDiscoveryAgent and
itunescloudd ran during every session of both apps. Their footprints at the end were the same
next to either app: 28.3–28.7, 8.7, 3.6–3.7 and 11.5–11.9 MB. They are counted in neither
column.

**The route** (`resuse/session.sh` in the task's scratch folder): the same script for both apps,
the same click positions on the same window (1440×900 pt at 240,60), each app alone.
1. Launch, 15 s wait.
2. Pages, about 3 s each:
   - Home, Songs, Albums.
   - Artists, then the artist Aaliyah.
   - Albums, then the album "Aaliyah": Honeycrisp by a click on its tile; Music by an accessibility
     press, because a synthetic click in Music's grid only selects the tile.
   - The playlist "Desi Hits" from the sidebar.
3. Scrolling: Albums, two flings down (5,000 pt/s peak); Songs, two flings down (6,000 pt/s peak).
4. Search: the Search page, "coldplay" typed one letter every 160 ms (Music: then Return), 4 s wait.
5. Playback: the playlist "Desi Hits", a double-click on its first song, "Thodi Si Daaru", which
   plays for 180 s. Then a click on the player's pause button.
6. 12 s later (2 s after the click, then 10 s), the reading, then quit.

From launch to the reading took 273–277 s in all six runs.

**Positions:** the click positions for the playlist, the song row and Music's album tile were
found once, before the runs, with accessibility look-ups (`resuse/probe.sh`). No look-up ran
during a measured session, except Music's one press on the album tile.

**How memory was read:**
- **App's own process:** `footprint -p <pid>`, the "Footprint" line. This is the kernel's
  `phys_footprint`, the figure Activity Monitor shows as Memory.
- **Helpers:** `ri_phys_footprint` from `proc_pid_rusage` at the same moment, by a small sampler
  run for the whole session. On the app process the two methods agree within 1 MB (`footprint` prints whole MB).

**System state:**
- Swap used at the launches was 2,997–3,248 MB, except Music's third run (1,148 MB).
- The memory pressure level read "warn" (2) at Honeycrisp's second launch and "normal" (1) at every
  other launch and at every reading.
- 1-minute load average at the launches: 2.0–5.3. Other background processes were running on the Mac.

**Builds and covers:**
- **Honeycrisp:** the published release build, the release build. Runs 1 and 2 used commit
  `e35e6ae`. Run 3 used `829caac` (the next release build), which replaced it between the runs.
  Launched with `HONEYCRISP_IMG1_DRY=1`, so covers came only from its disk cache and no image request
  was sent ("IMG1 stats at quit": 0 sent in all three).
- **Music:** 1.7.2. It used its own caches, already used by earlier runs that night, and fetched
  whatever it fetches. Its requests were not counted.

**Mac:** Mac mini M1, 8 GB, macOS 27.2 (26B123), one display at 1920×1080, 1×, 120 Hz.

**Earlier dry run, not used:** a first run of the script (one per app, 03:35–03:40 CET) read
338 MB for Honeycrisp and 712 MB for Music, app process alone. It is not used here, for three
reasons:
- Its album step opened different items in the two apps.
- Its playback was about 12 s of a different song in each app.
- It made three accessibility look-ups in each app during the session.

Music's reading in that run was taken with the system at "warn".

## 1e. CPU time and energy over the same scripted session

These come from the same 5 runs per app as 1d's build `82d14a4` table: the same route, the same
sampler, launch to the reading (273–275 s).

| What (seconds of CPU time, whole session) | Honeycrisp | Music |
|---|---|---|
| The app's own process | median 13.1 (12.8–13.5) | median 27.2 (26.4–27.7) |
| App plus its own helpers (RemotePlayerService, `log`) | median 18.3 (17.9–18.6) | median 27.2 (26.4–27.7) |
| Energy, app plus its own helpers (J, kernel estimate) | median 22.1 (21.2–22.4) | median 24.5 (23.0–25.2) |
| WindowServer, shared (only one app ran at a time) | median 36.0 (35.7–36.5) | median 19.7 (19.1–20.0) |
| coreaudiod, shared | median 8.4 (8.3–8.4) | median 9.0 (9.0–9.0) |
| Apple's agents both apps use (AMPLibraryAgent, AMPArtworkAgent, AMPDeviceDiscoveryAgent, itunescloudd, mediaremoteagent, VTDecoderXPCService) | median 2.6 (2.5–2.7) | median 4.3 (3.2–4.4) |

Per run, CPU time of the app plus its own helpers, then WindowServer:

| Run | Honeycrisp | Music |
|---|---|---|
| 1 | 18.6 s, WindowServer 35.7 s | 26.4 s, WindowServer 19.1 s |
| 2 | 18.6 s, 35.8 s | 27.2 s, 19.7 s |
| 3 | 18.1 s, 36.0 s | 27.6 s, 19.6 s |
| 4 | 18.3 s, 36.5 s | 27.7 s, 20.0 s |
| 5 | 17.9 s, 36.0 s | 27.1 s, 19.9 s |

**Where the time went** (runs 3–5, sampled every 10 s; sums over the stretches, median of 3):

| Stretch | Honeycrisp app | Honeycrisp's RemotePlayerService | Music | WindowServer with Honeycrisp | WindowServer with Music |
|---|---|---|---|---|---|
| Launch, pages, scrolling, search (0–80 s) | 11.7 s | 1.2 s | 13.9 s | 28.5 s | 8.2 s |
| 180 s of playback (80–260 s) | 1.2 s | 3.4 s | 13.2 s | 2.5 s | 6.0 s |

- While the pages, scrolling and search ran, WindowServer used about 3.5 times the CPU time
  with Honeycrisp that it used with Music. This row does not favour Honeycrisp. What in Honeycrisp's windows
  makes WindowServer work was not looked into here; it is a follow-up item.
- While a song played, Music's process used about 11 times Honeycrisp's: Music decodes and plays in
  its own process. Honeycrisp hands playback to RemotePlayerService, Apple's player for MusicKit
  apps, which is counted in Honeycrisp's figures above.
- coreaudiod's time is the audio output itself, about 7.8–8.5 s over the 180 s with either app.

**How these were measured:**
- **CPU time:** user plus system time per process, from `proc_pid_rusage` (`RUSAGE_INFO_V6`).
  It was read by a small sampler (public API, no root) at the launch and every 10 s (runs 3–5;
  every 60 s in runs 1–2), and at the end. The app's process counts from its start. The
  helpers that were already running count from the launch.
- **Processes outside the user's account:** WindowServer and coreaudiod can't be read by that
  call without root. Their CPU time came from `ps` (10 ms resolution). They serve every app on
  the Mac, so they are shown apart, not added to either app.
- **Which processes are each app's:**
  - Found by comparing every process's CPU time before and after a part of the route, each app
    alone (the method notes have how).
  - Music: its own process only. It started no helper process of its own.
  - Honeycrisp: its process, its `log stream` child (reads the player's log for the stream format)
    and RemotePlayerService.
  - Apple's agents in the last row of the first table ran during both apps' sessions and are
    counted in neither app.
- **Energy:** the kernel's own estimate per process (`ri_energy_nj`), the one source readable
  without root (powermetrics needs root). It covers CPU work only, not the GPU and not
  WindowServer's share.
- **Cross-check:** on one run the sampler's CPU time matched `top`'s TIME column to 0.01 s, and
  its footprint matched `footprint -p` to 1 MB.
- **Conditions:** as in 1d. 1-minute load average 1.3–2.3 at the launches, swap 1,438–1,462 MB
  used, memory pressure "normal" throughout. Honeycrisp sent no image request (0 in all 5).

## 1f. Wake-ups and CPU at rest

Each app ran alone with its window visible and the pointer off it. Two minutes were measured
in each run:
- **Nothing played yet:** 60 s, starting 60 s after the launch.
- **Paused after playing:** 20 s of a song from the Songs list, a click on the player's pause
  button, then 60 s.

3 runs per app, the apps taking turns, build `82d14a4`, 2026-11-17, 12:34–12:59 CET.
Wake-ups per second, median (range):

| At rest | Honeycrisp app | Honeycrisp's RemotePlayerService | Music app |
|---|---|---|---|
| Nothing played yet: wake-ups/s | 0.27 (0.18–0.30) | 0.35 (0.35–0.40) | 1.48 (1.46–1.53) |
| Nothing played yet: CPU ms/s | 0.78 (0.77–7.12) | 0.12 (0.10–0.12) | 1.20 (0.75–1.30) |
| Paused after playing: wake-ups/s | 0.30 (0.27–0.30) | 0.68 (0.63–0.68) | 2.21 (2.13–2.23) |
| Paused after playing: CPU ms/s | 1.30 (1.27–1.37) | 0.55 (0.52–0.65) | 2.03 (1.92–3.02) |

The other processes, about the same with either app:
- itunescloudd: 0.25–0.43 wake-ups/s.
- AMPLibraryAgent: 0.08–0.18.
- AMPArtworkAgent: under 0.1.
- Honeycrisp's `log` child: 0.

**How measured:** wake-ups are `ri_interrupt_wkups` from `proc_pid_rusage`: how often the
process's threads were woken by an interrupt, which is mostly timers. The difference was taken
over the 60 s, and the sampler itself is not one of the processes. CPU ms/s is user plus system
time over the same 60 s.

**Not measured here, idle wake-ups as Activity Monitor shows them:** that count
(`ri_pkg_idle_wkups`, top's IDLEW) only counts a wake that takes the whole processor package out
of idle. On this Mac, with other background work running, the package was almost never idle. The
count was 0–0.03 a second for every process in every run, both apps alike. Separate laptop readings
(Honeycrisp 24, Music 9, idle, 2026-11-17) were taken that way, on a laptop at rest, and can't be
compared with these.

**Why Honeycrisp's figure was higher before:** on the build of 2026-11-16 (`e35e6ae`), at rest,
Honeycrisp had 2.03 wake-ups/s (2.01–2.03) and Music 2.26. Honeycrisp's matched the repeating timers in
its code, 1.9 a second between them:
- the player watchdog, every 1 s, also when nothing played;
- the cover cache's sweep, every 2 s;
- the idle check, every 5 s;
- the text cache's sweep, every 10 s;
- the player's safety re-read, every 15 s.

A 20 s `sample` of the app at rest found the sweep every 2 s as most of its CPU at rest. The item
IDLEWAKE1  made these run only when there is something to do, which gives the 0.27
above.

The first minute after launch is left out of the table: Honeycrisp was still loading the library
then. On `e35e6ae`, from 15 s after launch, its CPU at rest read 4.5–11.7 ms/s; 60 s after
launch, 1.8 ms/s.

## 1g. A 30-minute session: memory over time, and what Honeycrisp's memory holds

One run per app, build `82d14a4`, 2026-11-17: Honeycrisp 12:59–13:30 CET, Music 13:31–14:02 CET.
Each app ran alone:
- Launch, then the playlist "Desi Hits" from its first song: 10 songs, about 30 minutes, which
  played to the end of the session.
- Every 5 minutes, the 1d route's pages, scrolling and search again: 6 rounds.
- Pause at 30 minutes, then the reading 12 s later.

Honeycrisp ran with its memory log on (`HONEYCRISP_MEMLOG=1`, one line every 15 s) for the breakdown
below.

| Physical footprint of the app's own process (MB) | Honeycrisp | Music |
|---|---|---|
| 3 minutes after launch | 290 | 455 |
| 10 minutes | 283 | 590 |
| 20 minutes | 309 | 644 |
| 25 minutes | 323 | 664 |
| At the end (30 minutes) | 322 | 627 |
| Highest during the session | 361 | 790 |
| Honeycrisp's RemotePlayerService at the end | 55 | none |

| Whole session (CPU seconds; energy in J) | Honeycrisp | Music |
|---|---|---|
| App's own process, CPU | 48.3 | 273.9 |
| Own helpers, CPU (Honeycrisp: RemotePlayerService 45.9, `log` 2.2) | 48.0 | none |
| App plus own helpers, energy (kernel estimate, CPU only) | 54.8 | 147.2 |
| VTDecoderXPCService (Apple's video decoder) started during the session, CPU | 0.1 | 61.1 (46 MB at the end) |
| WindowServer, shared | 282.7 | 471.8 |
| coreaudiod, shared | 101.3 | 109.8 |

VTDecoderXPCService decodes video for whichever process asks. It started during each app's
session. Music shows animated covers (Home's "Get Up!", for one). It is listed apart and is not
added to Music's figures.

**Honeycrisp over the 30 minutes:** its footprint went up and down between 283 and 336 MB with the
pages it showed, and ended 32 MB above its 3-minute figure. Music's ended 172 MB above its own.

**What Honeycrisp's 322–336 MB is**, read at the end of the session from its memory log, a `MEMDBG`
dump, `footprint -p`, `vmmap --summary` and `heap -s`:

| Part | MB | What it is | Capped by |
|---|---|---|---|
| Objects in malloc, in use | 184 | The library as the app holds it (2,362 songs, 1,430 albums, 781 artists, 22 playlists), the bridge's copies of MusicKit's items (2,587 songs, 117 albums, 7,083 artwork templates), and the views of the pages kept alive (5,033 views, 7,475 layers in 3 page trees). By type: 284,645 strings (12.7 MB), 79,723 Swift strings (9.9 MB), 11,625 dictionaries (8.4 MB), 236,165 untyped blocks (53 MB). | nothing: it grows with the library and with the pages kept |
| Malloc pages free but still dirty | about 20 | 204 MB of "Malloc Small" pages are dirty, of which 184 MB hold objects. The malloc zones report 152 MB free inside them, mostly given back to the system ("reclaimable" 73 MB) | the system's own reuse |
| Small cover copies (thumbnails, 64 px) | 30.1 | 1,930: every library cover (1,680) plus up to 600 others | library size + 600 |
| Decoded covers | 22–59 | 22.4 MB at rest (39 shown, 388 kept for going back), up to 59 MB right after the Albums page and a fling | shown + 64 MB, 160 MB ceiling |
| Rendered text | 4–10 + 12 | Row and title bitmaps: 4.1 MB at rest, up to 10 MB after a fling. Tinted copies: 12 MB, at their cap | 4–12 MB; tinted 12 MB |
| Text slabs | 9.4 | the memory those text bitmaps live in (14.2 MB mapped, 4.8 MB given back inside them) | follows the text above |
| Code and data of the app and the system frameworks, CoreAnimation, stacks, other | about 30 | | |

**What RAM1 could shrink, and at what cost:** each "at most" is the part's measured size above,
an upper bound, not a measured saving. None of the costs were measured. Each is a follow-up item
to measure before and after, as RAM1 asks.

| Part | Lever | At most | Cost to check |
|---|---|---|---|
| Malloc pages free but dirty | `malloc_zone_pressure_relief` once the launch's library load is done, and after a page tree is freed | about 20 MB | CPU time of the call |
| Small cover copies | keep the 64 px copies on disk, memory-mapped, and decode only the ones on screen | 30 MB | a decode per cell shown; the scroll budgets |
| Decoded covers kept for going back | a smaller slack than 64 MB | 37 MB after a fling (59 − 22), 18.6 MB at rest (the evictable part) | covers decoded again from disk on going back; page timings |
| Tinted text copies | a smaller cap than 12 MB | 12 MB | re-rendering on hover and selection |
| Pages kept alive | let page trees out of sight go sooner (5,033 views, 7,475 layers) | not measured | page build time on going back |
| The library and the bridge's copies | one copy instead of two (the app's models and the bridge's cache of MusicKit's items) | not measured: needs a run with malloc stack logging | none at run time |

**Separate 2× Retina laptop reading (392 MB):** the reading at the first memory warning after 3 minutes of browsing,
2026-11-17, had artwork at 72.5 MB (418 covers), thumbnails at 41.6 MB (2,667) and text at 4.7 MB.
That is the same make-up as here. Covers there are decoded at 2× (four times the bytes of 1×),
and its library cover set is larger. The malloc part was not read there.

## 1h. Click to first paint under memory pressure

1b's warm pages, measured with 1b's tool, region and rules, with the Mac under memory pressure.
5 runs per app, both apps in one launch per run, the order of the apps alternating. Build
`82d14a4`, 2026-11-17, 14:07–14:26 CET. Medians (range), ms from the mouse-down:

| Page | Honeycrisp: content on screen | Honeycrisp: complete | Music: content on screen | Music: complete |
|---|---|---|---|---|
| Home | 51 (45–60) | 51 (45–60) | 146 (139–270) | 896 (889–1,053)¹ |
| Songs | 45 (41–51) | 45 (41–51) | 97 (91–103) | 97 (91–103) |
| Albums | 43 (36–51) | 43 (36–51) | 109 (99–143) | 124 (117–143) |
| The artist Aaliyah (detail pane) | 67 (63–72) | 67 (63–72) | 100 (92–106) | 106 (92–108) |
| Playlist "Desi Hits" (sidebar) | 134 (122–138) | 134 (122–138) | 395 (392–409) | 467 (460–471) |

¹ In every run Music's Home kept changing until about 0.9 s after the click. What changed was
not looked at frame by frame. Its "content on screen" is the comparable figure.

**How the pressure was made:**
- **Tool:** a small program (`resuse/mp.c`, public API only) allocated memory in 128 MB steps,
  one every 2 s. It filled every page with pseudo-random bytes, so the system's memory
  compressor could not shrink it.
- **When it stopped:** it read `kern.memorystatus_vm_pressure_level` before each step, and
  stopped growing at the first "warn" (2). It never reached "critical" (4); it was built to give
  memory back at once if it did.
- **What it held:** 2,688–3,200 MB, reaching "warn" after 45–54 s. It held that through the
  run, touching its memory every 10 s.
- **During the clicks:** the level read "warn" at every measured click, in every run. Swap in
  use was 1,829–2,769 MB, and grew in each run. The 1-minute load average was 2.8–11.2.
- **Order and release:** the apps were launched after the pressure was made, and their pages
  were each visited once before measuring. The program was then stopped. The level was back to
  "normal" (1) within 1 s in every run.

**Not the same as 1b's table:** 1b was measured on build `6a615c2`, without pressure, and
Honeycrisp's Home there was 92 ms (80–98). Build `82d14a4` builds Home before it is shown
(PREBUILT), so the two Honeycrisp Home figures differ by build as well as by pressure. Music's
figures can be set side by side:

| | Home | Songs | Albums | Artist | Playlist |
|---|---|---|---|---|---|
| Music without pressure (1b) | 154 | 111 | 111 | 100 | 354 |
| Music under pressure | 146 | 97 | 109 | 100 | 395 |

Under pressure, Music's figures were within 15 ms of 1b's, except the playlist page (41 ms
later). Honeycrisp was not measured without pressure on this build.

**Covers:** Honeycrisp sent no image request (0 in all 5 launches). Music used its own caches.

## 1i. The same session at 2× and 60 Hz (one run per app)

1d–1e's session once per app with the display set to 1280×800 pt at 2× (2560×1600 pixels) and
60 Hz, instead of 1920×1080 at 1× and 120 Hz. Build `82d14a4`, 2026-11-17, 14:33–14:44 CET.
- **Window:** 1200×750 pt at 40,30, because 1440×900 does not fit that screen. The Dock was
  hidden for the hold so the window fitted above it.
- **Click positions:** found again for this window by the same accessibility probe.
- **Afterwards:** the mode and the Dock were put back.

| What | Honeycrisp | Music |
|---|---|---|
| App's own process at the end (MB) | 290 | 515 |
| App's own process, highest during the session (MB) | 351 | 688 |
| App plus own helpers at the end (MB) | 348 | 515 |
| CPU time, app plus own helpers (s) | 18.7 | 29.0 |
| Energy, app plus own helpers (J) | 20.3 | 22.9 |
| WindowServer CPU time (s), shared | 22.6 | 27.3 |
| WindowServer during launch, pages, scrolling and search (0–80 s) | 10.6 | 7.7 |
| WindowServer during the 180 s of playback | 7.1 | 14.2 |

One run each, so no ranges.
- **Same direction as at 1×:** memory, CPU time and energy.
- **Different from 1×, WindowServer:** at 2× and 60 Hz its time during the interactive part was
  10.6 s with Honeycrisp against 7.7 s with Music. At 1× and 120 Hz it was 28.5 s against 8.2 s
  (1e). Over the whole session it was lower with Honeycrisp (22.6 s against 27.3 s).
- Honeycrisp sent no image request (0).

## 1j. Launch against a bare AppKit app (LAUNCH1)

How close Honeycrisp's launch is to an app that does nothing. Measured 2026-11-17 by LAUNCH1
(see OPTIMISATIONS.md for what changed), release builds: "before" is `82d14a4` (the
build in table 1), "LAUNCH1" is the branch that moved Honeycrisp's own launch work off the path to
its first frame.

**Method.** `open -n -a <app>`, then the on-screen window list polled every 5 ms until the app
has a normal window (≥ 600×400 pt), as table 1 (`numbers/winwait.swift`, plus the process's start
time from the kernel). Honeycrisp's own milestones come from its launch log (`HONEYCRISP_PERFLOG=1`): the
first frame whose page shows the library's rows, and the first frame where every visible cover
shows artwork. Times are ms from the process start unless they say otherwise. Each app quit
before the next launch; 7 rounds with the apps' order rotating; median (range). Honeycrisp's covers
came from its disk cache with 0 image requests sent. The floor apps:
- **Empty AppKit app:** one 1180×760 window with a label.
- **Same, linking Honeycrisp's frameworks:** the same app linked against the same 44 system libraries
  as Honeycrisp's binary (MusicKit, StoreKit, ScreenCaptureKit, SwiftUI, AVKit, iTunesLibrary, …).

**Warm, 1× 120 Hz** (1920×1080, 15:27–15:33, 1-minute load average 3–5, no compiler running):

| App | Window on screen | From the `open` call | Window with content | First screenful of covers |
|---|---|---|---|---|
| Empty AppKit app | 278 (273–295) | 295 (292–317) | 278 | – |
| Same, with Honeycrisp's frameworks | 263 (248–344) | 282 (266–378) | 263 | – |
| Honeycrisp before | 638 (628–700) | 657 (648–721) | 629 (621–696) | 646 (635–708) |
| Honeycrisp LAUNCH1 | 467 (464–985)¹ | 485 (480–1,046)¹ | 465 (459–978)¹ | 465 (459–978)¹ |

**Warm, 2× 60 Hz** (1280×800 pt, 2560×1600 pixels, 15:31–15:36):

| App | Window on screen | From the `open` call | Window with content | First screenful of covers |
|---|---|---|---|---|
| Empty AppKit app | 288 (272–291) | 306 (289–310) | 288 | – |
| Same, with Honeycrisp's frameworks | 275 (266–291) | 291 (283–308) | 275 | – |
| Honeycrisp before | 629 (617–660) | 647 (635–674) | 618 (615–637) | 634 (615–654) |
| Honeycrisp LAUNCH1 | 458 (445–469) | 477 (464–486) | 451 (443–459) | 451 (443–459) |

¹ The slowest LAUNCH1 launch was the first launch of a freshly copied build; the other six were
464–469 (see Cold).

**Dock click, 1×** (15:44–15:47; each app kept in the Dock alone, 5 clicks each, alternating; from
the mouse-down to the window on screen): before 662 ms (628–664), LAUNCH1 493 ms (487–498). From
the process start: 637 (597–639) and 465 (458–470).

**Cold, 1×** (16:28, load 4.0–4.6): the first launch ever of an app copy written 31 minutes
earlier, 3 copies per build, alternating. Not after a reboot (this Mac isn't rebooted), and the
Mac's file cache was not emptied (that needs `sudo purge`). Window with content: before 864 ms
(844–964), LAUNCH1 726 ms (720–844); from the `open` call to the window: 932 (912–1,038) and 785
(784–911). Both builds spend ~250 ms more before their own code runs on a never-run copy than
warm (the system checking a new binary).

**What it says.** Linking all of Honeycrisp's frameworks costs nothing measurable on this Mac (the
second floor row is no slower than the first). Honeycrisp is still ~190 ms behind the empty app at
1× (~165 ms at 2×), down from ~360 ms; nearly all of that is AppKit laying out and drawing
Honeycrisp's first screen (sidebar, toolbar, the Songs list's rows, the player). The covers of the
first screen now arrive in the same frame as the window. Memory is unchanged: 224 MB (224–232)
before and 224 MB (223–231) with LAUNCH1, 60 s after launch at rest (3 launches each).

## 2. Honeycrisp's own performance budgets (as the tour enforces them)

Honeycrisp's test tour drives the real app (real library, real Apple Music) and logs a
`SNAPCHECK` line per check; a check that misses its limit logs `FAIL`. These are the limits that
exist in code today. "Latest" is the release build's tour (release-build tour run on 2026-11-16, 18:30–19:00) unless it says otherwise; "area run" means the area run alone,
"full tour" the whole tour in one launch.

| Budget, in plain words | Limit | Latest measured | Now | Enforced by |
|---|---|---|---|---|
| Scrolling a 2,353-song list at speed: each scroll step's main-thread work fits one 120 Hz frame | ≤ 1 % of steps over 8.3 ms, none over 16.7 ms, and at least 1,000 rows/s | Songs: median p50 4.0 ms (p95 4.4), 0 steps over in 5 of 5 runs (isolated re-measure). area runs: 0 over in 3 of 4; the first run had 3 over 8.3 and 1 over 16.7 | passes (the one failing run was the first of four, see notes) | SCR2 (`scr2` area), `SCR2Checks.swift` |
| Same, on a 2,131-song playlist page | same | median p50 5.4 ms (p95 6.1), 0 steps over in 5 of 5 runs; under memory pressure p50 5.5 ms, 0 over in 5 of 5 (isolated re-measure). area runs: 0 over in 3 of 4; the first had 85 over | passes | SCR2, and ROWS1 under memory pressure (`mem1` area) |
| Same, on the 777-artist list | same | median p50 2.1 ms (isolated re-measure, 5 runs); 0 over in all 4 area runs | passes | SCR2 |
| Same, on a short (32-song) album page | same (on 6–8 steps, one slow step fails it) | median p50 5.7 ms, 0 over in 5 of 5 runs (isolated re-measure); area runs: fails in 2 of 4 (1,000 rows/s floor: 852 and 975 rows/s), ROWS1 1 over; Follow-up: 1 step over in each of 6 evening runs; the same album from the catalog (with shelves) fails 9 of 12 | **on the line**: AppKit's own work on a page whose header stays in view; no app-side fix found (tracked as an accepted platform ceiling) | SCR2 / ROWS1 |
| Fast fling: no row's text is drawn on the main thread mid-scroll | no frame spends over 3 ms drawing text on the main thread; under 2 % of texts wait; 0 wrong | 0 main-thread renders (worst frame 0.00 ms), 0.0 % waiting, Songs and Albums (full tour and 3 area runs) | passes | fling text, `ScrollChecks.swift` |
| A page's first visit (Songs, Albums, Artists, Playlists): main thread from the click to the first frame | ≤ 20 ms, first and second visits | full tour: Artists 22.2, Albums 16.3, Playlists 16.8, Songs 17.6 ms (first visits). Calmer re-measure (isolated re-measure, 5 runs): medians 13.3–21.3 ms per page, at least one page over 20 ms in every run. Older builds re-run this evening: 11.8–27.5 ms (see notes) | **fails** (by 1–3 ms; this Mac's evening state, older builds fail too) | first page visits, `ScrollChecks.swift` (`visitLimit`) |
| Album grid fling (~15,600 pt/s): covers on screen as drawn | under 2 % blank tiles | 0 % blank (prefetch on); worst case without prefetch 4 % colour-only, 0 % blank | passes | album fling, `main.swift` |
| Freeing cover memory while scrolling never stalls a frame | every trim ≤ 2 ms (a quarter frame at 120 Hz); never over the hard cap by more than 48 MB | worst trim 0.053 ms; 0 MB over the cap | passes | artwork trims, `ScrollChecks.swift` |
| Holding ↓ through 20 playlists in the sidebar: the main thread is never held long | longest hold ≤ 80 ms | full tour: 81 ms (FAIL); area run: 44 ms | **on the line** (fails by 1 ms in the full tour, passes alone) | PLAY1-E1 (`play1e` area), `Play1EChecks.swift` |
| Reloading the whole library doesn't hold the main thread | longest hold ≤ 150 ms | 68 ms (reload of 2,353 songs and the rest in 7,779 ms) | passes | PLAY1-D2, `Play1Checks.swift` |
| A player call with a time limit returns on time even while the main thread is busy | a 300 ms limit is honoured by 450 ms | claimed at 305 ms | passes | PLAY1-D1, `Play1Checks.swift` |
| Launch: the window with the library's rows from the process start (5 real launches started by the tour) | best launch < 500 ms and median < 550 ms; first screenful of covers median < 800 ms | best 482, median 500 ms at 1×; best 488, median 492 ms at 2× (LAUNCH1 branch, 2026-11-17 16:3x); `HONEYCRISP_LAUNCH1_OFF=1` best 635 | passes | LAUNCH1 (`launch1` area), `Launch1Checks.swift` |
| Home, cold: shelves in the frame of the click | the click's frame within 50 ms; worst shelf ≤ 4 ms | 44.0 ms; worst shelf 1.46 ms | passes | home cold start, `HomeChecks.swift` |
| Home, warm: its own layout when reopened | under 8 ms (half a frame) | 0.0 ms (from memory, from the disk copy, and kept) | passes | home warm open, `HomeChecks.swift` |
| Home refreshing while visible: only changed tiles touched | worst shelf update < 4 ms | 0.26 ms; 0 tiles moved | passes | home refresh, `HomeChecks.swift` |
| Home: Recently Played after a play | move ≤ 4 ms; reply applied ≤ 4 ms; slide 290 ms ± 60 (Music's) | 1.22 ms; 0.90 ms; 299 ms | passes | HOME4.6, `Home4Checks.swift` |
| New and Radio pages: each shelf's update; warm reopen layout | each shelf ≤ 4 ms; layout ≤ 8 ms | New worst shelf 3.58 ms; Radio 0.57 ms; Radio two-line shelf 0.82 ms | passes | new page, radio page, radio3 (`NewChecks.swift`, `RadioChecks.swift`, `Radio3Checks.swift`) |
| Top Charts page draws | warm ≤ 400 ms, cold (request included) ≤ 6,000 ms | warm 189 ms, cold 602 ms | passes | charts1, `CHARTS1Checks.swift` |
| Queue panel opening/closing beside the page: no empty gap | open: ≤ 60 % of the strip empty 40 ms in; close: ≤ 25 % | 46 %; 7 % | passes | queue panel gap (`jank` area), `JankChecks.swift` |
| A memory warning halves the caches; they come back after | text, covers, thumbnails at most halved; caps restored after pressure | text 13.3→6.7 MB, covers 56.4→28.2 MB, thumbnails 2,264→1,939; caps back (80→160 MB) | passes | memory, MEM1 caps recover (`mem1` area), `main.swift` |
| Pages not on screen hold no covers | 0 MB | 0.0 MB never shown, hidden, and kept past the cache | passes | MEM1 covers on pages not on show |
| Idle: spare cover memory is given back | ≤ 16 MB after idle | 47.4 → 15.9 MB (earlier build) | passes (earlier build) | CAPS-H3 (`caps` area), `CAPSChecks.swift` |

**Tour size:** the full tour logged 255 `SNAPCHECK` lines (counted with
`grep -c SNAPCHECK full.log`; this includes 41 per-area summary lines such as
"radio: 12 checks, 0 failed"), 3 of them `FAIL` (PLAY1-E1 above, one menu-item request check
and its area summary). The fast-scroll checks (SCR2, and ROWS1 in the `mem1` area) are not in the full tour; their
figures come from runs of those areas alone. The tour
has 108 areas: distinct `HONEYCRISP_SNAPSHOT_ONLY` names in `main.swift` (counted with
`grep -o 'SNAPSHOT_ONLY"] == "[^"]*"' main.swift | sort -u`); `TourAreas.swift` chains areas
in one launch and adds none.

## Method notes

- The tour times work on the app's main thread with its own clock (`CACurrentMediaTime`); it
  is not a screen recording. Its numbers can't be compared with a recording of Music, so they
  stay in table 2.
- 120 Hz gives a frame 8.3 ms; the scroll budget allows 1 % of steps over that and none over
  two frames (16.7 ms). The first two steps of a fling are left out (they make the table's first
  row views on a page just shown).
- Covers were answered locally in every run cited here (`HONEYCRISP_IMG1_DRY=1`): no image request
  left the Mac.
- The SCR2 area was run 4 times in a row; the first run was the slow one (3 Songs steps
  and 85 playlist steps over 8.3 ms), the next three passed or failed only on the short album.
  the isolated re-measure (5 runs each way, one run per screen-lock hold, started only with no
  compiler running) is the calmer figure.
- First page visits: the 20 ms limit failed this evening on the current build and on three older builds
  re-run beside it ( 11.8–27.5 ms per page, 2–3 runs each, load 2.0–5.7), while earlier the
  same day an earlier isolated run measured 11.7–16.0 ms in most runs (one page at 21.4 ms in one run) and
  20.0–29.4 ms in a pair run at load average ~4. It follows the machine's state, not one build.

## Not compared: Music's side was not measured

| What | Honeycrisp's own figure | How Honeycrisp measured it |
|---|---|---|
| Scroll smoothness | see table 2 (median p50 2.1–5.4 ms a step on long lists) | the tour's step timer on the main thread |
| Opening an album from the Albums grid | Honeycrisp opens it on one click (the recording measured 133 ms to content, 1 run) | in Music's grid a synthetic single or double click only selects the album, so the same action can't be timed in both. An album page opened from a Home tile is in table 1b |

Music's side of these needs a screen recording with input and frames on one clock; it was not
done for this file, and the tour's in-app timer is never compared with a recording of Music.
