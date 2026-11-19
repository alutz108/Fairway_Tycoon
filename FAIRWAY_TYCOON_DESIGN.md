# Fairway Tycoon — Design & Code Guide

A SimGolf-style golf game: design courses, run a golf club, host tournaments and play your own holes. Single HTML file, three.js 3D rendering, installable from GitHub Pages. **Primary platform: iPhone** (bottom-sheet menus under 600px wide, 44px+ touch targets, 16px inputs so iOS doesn't zoom); also works on PC.

---

## 1. Feature list (current)

### Course settings (start screen)
All 9 settings are unlocked; each has an optional challenge (holes, star rating, green-fee earnings) and its own save, club name, members and trophy history. Each card shows a mini map preview.

| # | Setting | Style / inspiration | Special terrain |
|---|---------|--------------------|-----------------|
| 0 | Meadow Run | Open farmland (starter) | Round trees, ponds |
| 1 | Pine Barrens | Sandy pine forest (Pine Valley-like) | Pines, sandy waste areas |
| 2 | Coastal Links | Atlantic beach & dunes (Kiawah-like) | Ocean on left, beach, fescue |
| 3 | Azalea National | Classic Southern parkland (Augusta-like) | Pines, azalea flower beds, winding creek, lush grass |
| 4 | Cliffside Links | Pacific clifftop (Pebble/Cypress-like) | Ocean + rock cliffs, cypress trees |
| 5 | Sandhills | Sandhills & wiregrass (Pinehurst-like) | Large waste areas, longleaf pines |
| 6 | Lagoon Isles | Florida lagoons (Sawgrass-like) | Many ponds, palms |
| 7 | Desert Canyon | Desert target golf (Scottsdale-like) | Whole map starts as desert, cactus, boulders |
| 8 | Lakeshore Dunes | Great Lakes dunes (Whistling Straits-like) | Lake along top, pot bunkers, fescue |

### Building
- **Hole tool:** tap tee spot, then pin spot (100–700 yds). Auto-builds tee box + green. Par: ≤250 yds = 3, ≤470 = 4, else 5. Cost $200 + green tiles.
- **Paint tools:** Fairway $30, Green $60, Bunker $25, Cart path $15, Trees $20, Flowers $35, Pond $60, Clear $10 (water→$40, rock→$80). Brush 1×1 or 3×3. Clearing a tee or pin removes that hole.
- **Move tool** for one-finger panning.
- Hole design score (0–10) from fairway coverage, nearby hazards, green quality, scenery, tee box, length, and crossing other holes. Course star rating = average design + par variety + golfer satisfaction.

### Simulation
- Day = 40 sim seconds; speeds pause / 1× / 4×. Daily upkeep; bankrupt below −$2,000.
- Public golfers arrive based on stars, holes, green fee vs fair price and prestige. They pay the green fee and play every hole with shot AI (club distance by skill and lie, target picking, errors, roll, water/OB penalties, putting model), then rate the experience. Thought bubbles ("Birdie!", "Splash…", "Where's the fairway?").
- Lies: tee/fairway best; rough, bunker, trees, native area, flowers worse; water = penalty drop; rocks/clubhouse/off-map = out of bounds.

### Club management (🏆 Club button)
- **Club tab:** club name, Club Championship trophy name, stats, reigning champion.
- **Members tab:** member cards (shirt-color badge with flag, personality, handicap, match record, mood). Tap to open the **member profile sheet**: name, nationality (14 flag chips + None), personality, ability sliders (Power, Accuracy, Putting; handicap derives from their average), shirt and pants colors (avatar preview), and record (rounds, best round to par, match W-L-H, titles, head-to-head vs each member). Weekly dues (collected every 7 days) vs "fair dues". Members apply when rating/dues are good; unhappy members quit; members play free rounds on their own schedule.
- **Golfer types (personalities):** Weekend Golfer (wants wide fairways, hates water), Grinder (loves hazards/tough holes, bored by easy ones), Senior (shorter, rides carts, hates long walks), Social Golfer (loves scenery, cares less about score), Big Hitter (+15% distance, less accurate, loves par 5s). Applies to public visitors (weighted mix), members and guests; drives satisfaction and thought bubbles. Club tab shows "What golfers think" mood per type.
- **Member matches:** pick two members (Events tab); they play match play while the course stays open; live status ("X 2 up thru 5"); results update records/head-to-head; match log on Honors tab.
- **Cart paths:** members ride 3D carts (faster) when start and destination are within 2 tiles of a path; cart access between holes makes them happier, long walks without carts make them unhappy.
- **Events tab:** Member matches section, Club Championship (members, for your named trophy), Member-Guest Invitational (member + guest teams, best ball; guests may join the club), County Amateur Invitational (County Golf Association), Atlantic Pro Classic (Atlantic Pro Tour), National Links Open (United Links Association). Requirements (holes, stars, members), cooldowns, hosting income, prestige. Editable field: names and countries. Course closes to the public and locks building during events. Live leaderboard, leader banner, green name tags on the top 3, results screen.
- **Countries:** 14 common pro-golf nations (USA, CAN, MEX, ENG, SCO, IRL, ESP, ITA, SWE, RSA, AUS, JPN, KOR, CHN) with matching name pools; flag emojis (falls back to 3-letter codes on Windows). Shown in setup, leaderboard, name tags, scorecards, Honors.
- **Honors tab:** every champion with day, event and score.

### Scorecards (📋 button or tap a golfer)
List of everyone on the course; individual hole-by-hole cards (birdies red, bogeys blue), current hole, total to par, mood, member handicap, country. "Follow on the map" camera tracking.

### Play mode (🏌️ Play round)
Slingshot aiming (pull back and release), power and carry shown, landing circle, putting in feet on greens, penalties, scorecard and results.

### Graphics & camera
three.js r128: rolling vertex-colored terrain with smooth type blending, sunken bunkers and water, raised greens/tees/rocks, diorama soil skirt, sun shadows, dusk lighting shift, instanced low-poly trees per setting (round, pine, cypress, palm, longleaf, cactus), azalea bushes, boulders, 3D clubhouse, waving flags, 3D golfers with walking legs and carts, balls in real arcs. 2D overlay canvas for route lines, aim guide, badges, thought bubbles and name tags.
Controls: pinch/scroll zoom, two-finger or Move-tool pan, twist / right-drag / Q,E to rotate, arrows pan, +/− zoom, ⟲ ⟳ and Fit buttons.

---

## 2. Code map (sections inside the single `<script>`)

| Section marker | What's in it |
|---|---|
| (top) | Constants: grid `COLS=18, ROWS=26`, `YD=25` yards per tile, `DAY=40`, `MAXG=30`, tile types `T`, `TOOLS`, `UPKEEP`, `ALL_TOOLS`, `NATIVE_STYLE`, `LIEVAL`, `LEVELS`, name pools, `COUNTRIES`, `EVENTS`, `SHIRTS`, helpers (`clamp`, `lerp`, `gauss`, `fmt`, `rng`, `hash`, `pick`) |
| `state` | Grid `G` (Uint8Array), state object `S`, save/load (`saveGame`, `store`), `tile()`, `at()`, cart access map `NEAR` / `hasCart()` |
| `map generation` | `genMap(L)` seeded terrain per setting (coast, creek, trees, flowers, ponds, waste, sand, rocks, clubhouse) |
| `hole analysis` | `holeInfo(h)` (yards, par, quality, notes, cart), `recompute()` (course stars) |
| `3D rendering (three.js)` | Renderer, lights, terrain mesh (`rebuildTerrain`, `heightAt`), props (`STYLE`, `buildProps`), holes (`syncHoles`), golfers (`makePerson`, `syncGolfers`), camera (`view`, `zoomAt`, `panBy`, `rotateBy`, `toWorld`, `toScreen`, `layout`), overlay helpers, `draw(now)` |
| `simulation` | Spawning (`makeGolfer`, `spawn`, `nextInterval`), golfer AI (`stepGolfer`, `swing`, `putt`, `fullShot`, `land`, `holeOut`), `endDay`, `sim(dt)` |
| `goals` | Challenge check |
| `building` | `paintAt`, `holeTap`, `removeHole` |
| `play mode` | Player round: `P` state, `aimP`, `shootP`, `landP`, scorecard |
| `club: members & events` | `TYPES`, `PANTS`, `SHIRT_PICK`, `taste()`, `fixMember`, `memberGolfer`, members & dues (`clubDay`), member matches (`startMatch`, `matchText`, `finishMatch`), tournaments (`openSetup`, `teeOff`, `eventStep`, `standings` incl. best-ball teams, `finishEvent`), club UI `renderClub`, member profile sheet (`openProfile`, `renderProfile`), `nameTag` |
| `scorecards` | `openScores`, `renderScores`, `golferAt`, follow |
| `input` | Pointer/touch/mouse/keyboard: paint, tap, pan, pinch, rotate, aim |
| `UI` | Hints, tool bar `buildTools`, HUD, goals modal, level select (`showLevels`, `preview`), `startLevel` |
| `loop` | `frame()` → sim steps, camera follow, `draw()` |

### Key data shapes
- **Hole:** `{ tee:{x,y}, pin:{x,y}, plays, strokes, info:{ yards, par, q, notes, cart } }`
- **Golfer:** `{ id, x, y, bx, by, hole, strokes, state:'walk'|'address'|'flight', type, skill, pow, acc, put, color, pants, skin, hat, sat, card[], name?, mid? (member id), ep? (event player), mp?/side? (member match), country?, cart? }`
- **Member:** `{ id, name, type, country, pow, acc, put, skill (avg), shirt, pants, skin, sat, rounds, best, titles, rec:{w,l,h}, h2h:{[oppId]:{name,w,l,h}} }` (old saves upgraded by `fixMember`)
- **Match:** `{ id, a, b (member ids), ca[], cb[] (hole scores), done:{a,b} }`
- **Event player:** `{ name, skill, pow, acc, put, type?, country?, mid?, team?, guest?, card[], started, done, sat }`
- **Save (per level index):** `grid, holes, money, earned, day, fee, recent, won, members, mid, dues, prestige, eventLast, history, clubName, trophy, matchLog, matchId, typeSat`

---

## 3. Ideas not built yet
- Amenities: snack carts, benches, restrooms, driving range, pro shop
- Staff (greenskeepers, marshals) and course condition
- Multi-round tournaments with a cut, weather and wind
- Elevation tools (raise/lower land), island greens
- Sync saves across devices (would need an online service)
