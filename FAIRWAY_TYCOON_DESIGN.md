# Fairways Tycoon — Design & Code Guide

A SimGolf-style golf game: design courses, run a golf club, host tournaments and play your own holes. Single HTML file, three.js 3D rendering, installable from GitHub Pages as a home-screen app. Personal project, **not going to the App Store**. **Primary platform: iPhone** (bottom-sheet menus under 600px wide, 44px+ touch targets, 16px inputs so iOS doesn't zoom); also works on PC.

---

## 1. Feature list (current)

### Branding
Game name **Fairways Tycoon**. The logo (golf ball, flag, clubhouse, striped fairway) is embedded on the start screen as a base64 JPEG (`img.logo`) and used for the home-screen icons (apple-touch-icon 180, 192, 512, maskable 512 on green, favicon 32). The save key stays `fairway-tycoon-v1` and the file name stays fairway-tycoon.html.

### Tee boxes
Each hole draws up to three tee sets (`teeSpots`): back = the hole's tee tile (blue markers), middle ~1–1.4 tiles toward the pin/bend (white markers on a tee pad), forward ~2–2.9 tiles (red markers); short holes get fewer, and a set is skipped if it would sit on water, trees, rocks, a building or a green. Golfers pick a set (`pickTeeSet`): tour/event players and your pro play the back tees, grinders and strong players (skill ≥ .78) the back, seniors and beginners (skill < .4) the forward tees, everyone else the middle (`setGolferTee` on arrival, each new hole, edits). Listed yardage is from the back tees.

### Pin positions
Hole editor → **Pin position on the green**: Front / Middle / Back (relative to the tee) / 🎲 Random, free, using the connected green tiles around the pin (`greenTilesOf`, `setPin`). **Change every pin each morning** (`S.pinDaily`, saved) moves every pin to a random spot on its green at the start of each day (not during events or cups).

### Shaped greens (🏗️ Build → ⛳ Green shapes ›)
Stamp tools `gs_oval`, `gs_round`, `gs_kidney`, `gs_peanut`, `gs_star`, `gs_heart`, `gs_lshape` (`GSHAPES`, ~10–23 tiles each within a 9×9 area), $60 a tile, owned land only, skipping water, rocks, buildings, tees and existing green. ↻ Rotate turns the stamp in 45° steps (`S.shapeRot`). One undo step per stamp. The smooth terrain shader rounds the tile edges.

### Golfers using facilities
Visitors who warm up walk to the range and/or putting green first (`g.warm`), stand in a range bay or on the practice green (`practiceSpot`) for 5–8 s, then go to the first tee; members' practice visits do the same for longer. Range balls fly out from the bays toward the targets (`stepRangeBalls`, pooled meshes). Mid-round detours now take time (state `using`): golfers stand at the snack bar or drink stand facing it, sit on benches, and disappear into the restroom for a moment, then carry on.

### Carts in groups
Within a group, slots pair up (0+1, 2+3), so a foursome uses at most 2 carts. Only the first player of each pair rents (and pays for) a cart; the partner shares it. The partner rides as a passenger (`g.inCart`, hidden walker, `setPassenger` shows them in the right-hand seat of the driver's cart; driver sits on the left) whenever the driver is riding nearby, then walks the last bit to their own ball.

### Golfer outfits
Members and your pro have `hatStyle` (cap, back = backward cap, visor, bucket, none), `hatColor` (`HAT_COLORS`) and `bottoms` (shorts or pants) next to `shirt`/`pants`, edited in their profile (Shorts or pants + color, Hat style + color). Old members get a random style on load (`fixMember`); the pro defaults to a green cap and pants. Visitors get random looks (`randHatStyle`, `randBottoms`). 3D: `addHat()` builds each hat on the walker and the cart rider; shorts = short pant leg, bare lower leg and a white sock. The profile avatar draws the same (`hatSVG`). Changing an outfit redraws that golfer on the course right away.

### Par 3 courses (LEVELS 9 and 10, `par3:true`, `isPar3()`)
**Oak Hollow Par 3** (parkland, $12,000 incl. $9,000 build budget, $20 fee) and **Sea Breeze Par 3** (seaside, pot bunkers, $15,000 incl. $10,000, $22 fee). Rules: every hole 100–250 yards (`PAR3_MAX`, also when moving tees/pins), no doglegs (tool hidden), facilities limited to snack bar, drink stand, bench, restroom, pro shop and putting green (`PAR3_FAC`; landmarks still allowed; no range, restaurant or cart barn). Only local amateur invitations. Never count toward the golf tour (`tourQual`). Variety rating uses length spread and elevation instead of par mix; quick rounds bring 30% more golfers. Deer at Oak Hollow, gators at Sea Breeze.

### Your own golf tour (empire screen → 🏌️ Your golf tour)
Courses qualify with 18 holes and ≥3.5★ (`tourQual`, uses each save's stored `stars`). Tiers (`TOUR_TIERS`): 4 courses Regional Tour, 6 National Tour, 8 World Tour (the top level). Start it from the empire screen (name it). `store.tour` (empire-wide, saved and cloud-synced) = { name, tier, season, sched:[{course, done, winner, score}], players:[48 tour pros + your club pro once he plays], history }. Name lists' tour pros are used first. Each scheduled course gets a never-expiring gold invitation in Club → Events (`syncTourOffer` on course load): field 24/30/36 from the tour (season leaders + random), your pro can enter, hosting pays tier base ($15k/$30k/$60k) + gate, purse $60k/$150k/$400k, prestige 3/4/6. Results award season points (`TOUR_PTS`: 500, 300, 190…, `tourResult`); when every stop is played, the points leader is crowned, history saved, and the next season starts — moving up a tier if you now have more qualifying courses.

### Members: groups and practice visits
Visitors always arrive in groups of 2–4. Members who play tee off in groups of 2–4 too (`planMembers`). Other members (30% of non-players, if you have a range, putting green, pro shop or food) come just to practice (`startPractice`, state `practice`): they visit 1–3 facilities (range and putting green first), swing or putt there for a while, pay (range balls, shop, food, scaled by facility level), get slightly better (range: driving/accuracy/irons; putting green: putting/chipping) and happier, then go home. Shown as "Member, practicing" in Golfers.

### Play mode putting
No putting: when your ball stops on the green (or within 1.1 tiles of the pin off a bunker/water/trees), `autoPutt()` adds 1 putt (≤6 ft), 2 putts (≤35 ft) or 3 putts and finishes the hole ("Two putts from 19 ft."). The putter is no longer in the club list.

### Clubhouse location
Build tray → 🏠 **Clubhouse** (`houseTap`): tap the top-left of a clear 2×2 spot on owned land (with room for the door on the right). Free until the first hole exists, then $2,500. Saved as `S.house` (default `CHX, CHY`); `DOOR` and the clubhouse model follow it (`houseAt`, `setDoor`).

### Golfer feedback (📋 Golfers → 💬 Feedback)
Remarks are mapped to topics in `FBK` (complaints with a fix tip, praise); `say()` tallies them in `S.fb` (decays 20% a day). Every finished golfer leaves a review (`leaveReview`: 1–5 stars from mood, a quote from their own remarks, score) in `S.reviews` (last 30). The tab shows average stars, top complaints with tips, what golfers love, and recent reviews.

### Wildlife
Ephemeral animals (`animals`, `ensureAnimals` after each prop rebuild, `stepAnimals` every frame): ducks paddling on water (any setting), alligators in water on warm settings (Coastal Links, Azalea National, Sandhills, Lagoon Isles) that sometimes bask on the bank, deer near trees on Meadow Run, Pine Barrens, Azalea, Cliffside, Sandhills and Lakeshore that graze and run from golfers. Golfers react ("Look, deer!", "Aww, ducks!", "Not going in after that one. Gator!"), which counts as Wildlife praise.

### Facility upgrades
🏗️ Build → **⬆️ Upgrades ›** tray (`TRAYS.upg`, `upgradeCards`): one card per facility you own (numbered if you have several) showing `Lv 1→2 · $price` or `Max level ✓`, plus one card per Equipment shed item (`buyGear`, shared with Club → Staff). Cards you can't afford are dimmed; the Build tray's Upgrades button shows how many are affordable. Tapping a facility card moves the camera to it and opens its sheet (`openFacEd`) with what the next level does and an Upgrade button; tapping an equipment card asks to buy it. (Tapping a facility with ✏️ Edit still opens the same sheet.) Levels 1–3 (`f.lv`). Upgrade cost: 1.2× then 2× the build price. Earnings and effects ×1 / ×1.6 / ×2.3 (`FAC_EARN`; more customers, higher spend, bigger range/putting skill boosts, snack/drink stands draw from farther away), upkeep ×1 / ×1.5 / ×2.2 (`FAC_UPK`). Upgraded buildings are drawn bigger with gold pennants. Landmarks don't upgrade.

### Team cups (Ryder Cup–style match play) — Club → Events → Team cups
- **Members’ Team Cup** (8 members, 9 holes): members split into two balanced teams (snake draft by skill), default Team Blue vs Team Red; costs $400.
- **Inter-club Cup** (18 holes, 4 members, another owned course with 4+ members): your members host that course's members; result also lands in the other course's Honors.
- **Challenge Cup** (18 holes, 4 members): your best members vs a fictional rival club (`RIVALS`) with similar skill.
Setup: team names editable, 2–8 players a side. Singles match play over every hole; matches end early when decided ("won 3&2", `cupCheckDecided`); a win = 1 point, halved = ½. Teams wear blue and red. Course closed to the public during the cup; live score on the event chip and in Events. Results go to Honors and a results sheet; prestige +0.75 / +1.5; hosting income for inter-club and challenge cups. 7-day cooldown each (`S.cupLast`).

### Pace, messages and news
Days last 400 sim seconds; golfers walk 2.6 tiles/s (carts 4.6), pre-shot routine 1.2 s, ball flights 1.1 s + 0.08 s per tile, so an 18-hole round (~20 s per hole, ~355 s) fits in a day. Pace rating and "taking forever" thresholds are calibrated to that. Toasts stay up 2.6–12 s depending on length, show a 📰 icon, and every message goes into `S.news` (saved, last 40). Tap a message or the day label to open **📰 Club news**, grouped by day.

### Buying land
Unowned land is muted in the shader (tile G channel 128) and outlined with a gold dashed line while a tray is open or in land mode (`OWN`, `OWN_EDGES`, `computeOwn`, `drawOwnEdges`). Build tray → 🗺️ **Buy land** switches to **land mode** (`openLand`): the camera zooms out, every parcel for sale is drawn as a gold area with a name + price label (`drawLand`), tapping one selects it (pulsing gold) and the bottom bar (`#landBar`) shows its size and price with Buy / Done. 8 parcels (`PARCELS`: N/S/E/W strips, 4 corners, ~$55 per tile + $2,000 for strips); on coastal settings the sea side is "Open water". Bought parcels are saved in `S.parcels` and add to property value. Painting, holes, facilities and doglegs only work on owned land (`ownedAt`, `notOwned`).

### Hole editor
Build tray → ✏️ **Edit hole**: tap a tee or pin to open the hole sheet (highlighted in gold). Move tee / Move pin ($100; a pin off the green adds one green tile; keeps 100–700 yds), Straighten dogleg, Play earlier / later (swaps hole order, records and golfers), Delete hole (tee box back to rough). Undo works for all edits.

### Crew coverage & equipment shed
`crewShort()` works out how many more groundskeepers/greenskeepers are needed (first cut counts as half a fairway tile). Shown as a card at the top of Club → Staff (coverage bars + one-tap "Hire them" = `hireNeed`), a pulsing ⚠️ chip in the HUD when short-handed or condition < 50% (opens Staff), a daily warning in the end-of-day toast, and a toast when you paint more grass than the crew can handle (`crewCheckAfterPaint`). **Equipment shed** (`GEAR`, saved as `S.gear`, bought in Staff): Mower fleet (2 levels, crew +25%/+50%), Irrigation (2 levels, water bills −40%/−65%, turf +1/+2 a day), Greens roller (greens +1.5 a day), Maintenance shed (equipment wears half as fast). Playtest: a 6-hole course with the starting crew fell to 11% turf and 2.6★ in 30 days; following the warning kept it at 100% and 4.0★.

### Scenery: creeks, shrubs, tree species, landmarks
- **Creek** (Landscape, $35/tile, `thin` tool = always 1×1): paints water; any narrow water (≤2 water neighbors) automatically gets pebble banks and reeds.
- **Shrubs** (`T.SHRUB` = 18, $25): round bushes on mulch; play like flower beds; count toward scenery, hazards and beauty.
- **Trees ›** sub-tray (`TRAYS.trees`, parent `land`): Oaks, Pines, Maples, Cherry blossom, Willows, Palms, Cypress. Species per tile in `TS` (saved as `species`, `SPEC` list); "Mixed trees" keeps the setting's forest mix.
- **Landmarks ›** sub-tray (`TRAYS.marks`): facility kind `scenery` — Windmill 2×2 $6,000 (sails turn with the wind, `spinners`), Lighthouse 1×1 $5,000, Gazebo 2×2 $2,500, Fountain 1×1 $2,000, Founder statue 1×1 $1,200. Each landmark within 6 tiles of a tee or pin adds big scenery to that hole (`landmarksNear`), boosts the Beauty rating (+0.7 each, max 2.5), and golfers react ("What a view!"). Not counted in the electricity bill.

### Weather & wind
`S.weather` = { k: sunny/cloudy/windy/rain, mph, dir, temp } rolled each new day from `S.forecast` (end-of-day toast says tomorrow's weather). Odds depend on the setting (desert mostly sunny, links/dunes windier). Effects (`WX`): golfer demand ×1.2 sunny / ×1 cloudy / ×.85 windy / ×.45 rain; rain = walking 15% slower, half roll, turf +2 and greens +1, water bills ×0.2, golfers a bit grumpier; sunny = happier. Wind drifts every full shot (AI and your pro) by mph/18 × distance, reduced up to 45% by Course management (`windDrift`). Visuals: dimmer light for cloud/rain, rain streaks overlay (`drawWeather`), flags point downwind and flap faster. HUD day label shows icon and temperature (title has wind/golfers); play mode shows a wind box above the meter with an arrow relative to your aim.

### Livelier golfers
Visitors arrive in **groups of 1–4** (`grp`, `slot`): they stand side by side on the tee and wait for each other before teeing off. Chatter comes from `LINES` pools (~200 distinct lines in a 30-day test vs. one line making up 39% before): outcome lines, weather lines, scenery lines, end-of-round lines, and group banter ("Nice birdie, Chip!" via `mateSay`/`pendingSay`). Birdies or better trigger a hop-for-joy celebration (`g.cel`).

### Terrain look (smooth shapes, crisp grid while painting)
The ground is colored by a shader (`terrainMat` with `onBeforeCompile`, `TERRAIN_FRAG_HEAD/BODY`). `rebuildTerrain` writes tile data into `tileTex` (R type, G fringe flag, B facility pad code 1 range/2 putting/3 other, A tie-break priority) and colors into `palTex` (row 0 base colors incl. dryness; row 1 edge/lip color + strength; slots 20–23 fringe and pads). Each pixel scores the 3×3 tiles around it with a smooth kernel (radius 1.05) and takes the winning type, so painted squares read as rounded natural shapes with crisp anti-aliased edges; bunkers get a lip, pot bunkers a dark sod wall, greens a collar. Mowing patterns are procedural in the shader. Heights still come from the blended height grid (SUB=4). The white tile grid (opacity .5) and tile cursor show only while a Build/Landscape tray is open.

### Bunker and rough types (tiles 12–17)
| Tool (tray) | Tile | Cost | Plays |
|---|---|---|---|
| Pot bunker (Build) | `T.POT` | $40 | very hard: ~28–50% distance, big errors; small, deep, dark sod rim |
| Grass-faced (Build) | `T.GBUNK` | $35 | hard: ~38–68%; grassy lip |
| Waste area (Build) | `T.WASTE` | $15 | playable sandy scrub: ~78–93%, some roll; scattered scrub tufts |
| First cut (Landscape) | `T.FCUT` | $20 | light rough next to fairways: ~92–97%, extra roll |
| Deep rough (Landscape) | `T.DEEP` | $10 | ~50–70%, error up; dark grass tufts |
| Fescue (Landscape) | `T.FESCUE` | $15 | ~40–65%, big errors; tall golden tufts |
Helpers: `isBunker(t)` (sand, pot, grass-faced, waste), `rollOf(t)`. All count as hazards in hole design (first cut doesn't); bunker types satisfy tour "sand" requirements; golfers react ("A pot bunker. Ugh.", "Lost in the fescue…"). Uses Sand skill for bunker types and Recovery for rough types. Saves are unaffected (new tile numbers only).

### Polish & platform (step 7)
- **No sound by design.**
- **Landscape** (`@media (orientation: landscape) and (max-height: 520px)`): the map fills the left side; hint, tray (wrapping, scrollable) and a two-column dock sit in a 228px right panel; HUD pills sit over the map; side safe areas respected.
- **Animations:** golfer swing (club pivot at the hands: backswing → downswing → follow-through during the 0.5s address and the start of ball flight), ball flight trails (overlay), smooth zoom buttons (`smoothZoom`), sheets slide up, trays slide in when opened. All respect reduced motion.
- **Performance:** 1024 shadow map on phones; when paused and idle for 1.5s the scene draws at ~15fps (`lastInteract`, `idleN`).
- **iPhone hardening:** no page pinch-zoom (`gesturestart`), no rubber-band scroll, no long-press callout, persistent storage requested.
- **Backup / restore** (empire screen): copy the whole save store as a code; paste a code to restore.
- **Cloud saves (Firebase)** (empire screen → ☁️ Cloud saves; section `cloud saves (Firebase)`): email/password sign-in (Google sign-in is unreliable in iPhone home-screen apps). Firebase compat SDK v10.12.2 loads from gstatic only when a config exists (`FB_CONFIG` constant, or pasted config saved in localStorage `fairway-tycoon-firebase`; `parseFbConfig` accepts the console snippet). Firestore doc `saves/{uid}` = `{ data: JSON of store, savedAt, device, courses }`. `saveStore()` stamps `store.savedAt` and queues an upload 6s later when signed in (auto toggle). On sign-in or app open (`cloudStartup`, `cloudSync`): no cloud doc → upload; empty device → load cloud; otherwise if times differ, a sheet asks "Use the cloud save" or "Keep this device's". Manual Save to cloud now / Load cloud save / Sign out / Forgot password. Only works in the GitHub version (the Claude artifact blocks outside connections). Rules are in README.md.
- **Sign-in screen first** (section `sign-in screen`): on launch `startFlow()` shows `#signin` (logo, email, password, Sign in, Create account, Forgot password?, Play without signing in). If the device was signed in before (`FB_ON`), it shows "Signing you in…", restores the Firebase session, syncs (`cloudSync`) and goes straight to the empire screen; if Firebase can't load (offline) it still lets a previously signed-in player in (`cloud.offline`) and syncs on the `online` event. `enterGame()` hides the sign-in screen and shows the empire screen. The empire screen shows an account line (`#acctLine`) and a 👤 Account button (opens the cloud sheet: auto-save toggle, save/load now, sign out → back to sign-in). Guests see "Not signed in" and 👤 Sign in. Remembered email: localStorage `fairway-tycoon-cloud-email`. The GitHub service worker also caches the Firebase compat SDK files so offline sign-in restore works.
- **Offline / airplane mode** (tested): the home-screen GitHub version runs fully offline after one online visit (service worker caches index.html, vendor/three.min.js, icons, manifest, and Google Fonts once loaded). Building, simulation, play mode and saving all work offline. Cloud saves: `loadScript` refuses when `navigator.onLine` is false (10s timeout otherwise); uploads while offline set `cloud.pending`; a `window 'online'` listener finishes connecting or uploads pending progress. The Claude-hosted link needs internet.
- **Load safety net:** a small script before the game shows the error on the start screen if loading fails (or THREE is missing after 4s) and keeps the save backup button working; `window.__ftReady` is set when startup finishes. Save conversion runs in try/catch.
- **GitHub package:** three.js is bundled locally (`vendor/three.min.js`, MIT license included) and cached by the service worker so the home-screen app works offline; if `vendor/three.min.js` is missing, index.html falls back to cdnjs, and the service worker caches files one by one so a missing file can't block updates. The Claude artifact still loads three.js from cdnjs.

### Controls layout (bottom dock)
The bottom of the screen is a five-button dock: 🏗️ **Build**, 🌳 **Landscape**, 🏌️ **Play**, 🏆 **Club** (red badge = waiting invitations), 📋 **Golfers** (scorecards). Build and Landscape open a tray above the dock (`TRAYS`, `openTray`, `closeTray`, `S.tray`):
- Build tray: Hole, Dogleg, Fairway, Green, Bunker, Cart path, Bridge, and Facilities › (opens the facilities tray, which has ‹ Build to go back).
- Landscape tray: Trees, Flowers, Pond, Raise, Lower, Clear.
- Each tray starts with a ✓ Done button and ends with the Brush toggle (paint tools only). Undo/Redo float on the map (bottom-left) while a tray is open.
With no tray open the game is in **look mode** (`S.tool = 'move'`): one finger pans, taps open golfer cards. New courses start in look mode. In play mode the dock shows Scorecard, Restart and Quit round.

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
- **Hole tool:** tap tee spot, then pin spot (100–700 yds). Builds the tee box and a single green tile at the pin; the player paints the rest of the green (holes with a green under 5 tiles get a "Green is small" note and less design score). Par: ≤250 yds = 3, ≤470 = 4, else 5. Cost $200 + green tiles.
- **Paint tools:** Fairway $30, Green $60, Bunker $25, Cart path $15, Trees $20, Flowers $35, Pond $60, Clear $10 (water→$40, rock→$80). Brush 1×1 or 3×3. Clearing a tee or pin removes that hole.
- **Move tool** for one-finger panning.
- **Elevation:** Raise / Lower tools ($40 per tile per level, levels 0–8, one level = 0.22 tiles ≈ 5.5 yds; `E` Int8Array, `ELH`, `EMAX`, `elevAt`). Water and the clubhouse stay flat; painting water flattens a tile. New courses get natural hills per setting (`HILLS`, aligned with `LEVELS`; `genElev`). Rendering adds elevation into the blended terrain heights. Holes show "plays as" yardage (`info.plays`, `info.drop`); par uses plays yardage; elevated tees (2+ levels down to the green) add design score. Shots: AI golfers compensate for target elevation, carry is shortened uphill / lengthened downhill based on the landing spot; play mode applies the same physics and the hint shows plays-as distance.
- **Dogleg tool** ($100): tap near a hole's route to put a bend point there (`h.bend`, 2+ tiles from tee and green, total ≤ 720 yds); tap the bend again to straighten for free. Hole length, plays-as yardage, par, fairway coverage and hazard checks follow the bent route (`routeOf`, `routeLen`); crossing checks test every segment pair (`segX`). Bends sharper than ~15° count as a dogleg (`info.dog` 'left'/'right', +0.7 design score on par 4/5). AI golfers also search shot targets toward the corner while they're before the bend, so they follow the fairway around it (or cut it if they can land in a good spot). Route lines and a bend dot draw in build mode.
- **Fringe:** rough/fairway tiles touching a green become fringe (`FR`, `isFringe`, `lieAt`): drawn as a lighter collar and played like fairway (chip/putt friendly). The play hint names it.
- **Bridge tool** ($80, water tiles only, stored in `BR`, saved as `bridges`): wooden deck with rails over water; counts as cart path for member/senior cart access. Clear removes a bridge ($10); painting a tile to anything else removes it.
- **Hole signs:** every tee gets a sign showing hole number, par and plays-as yards (canvas texture, rebuilt when the hole changes).
- **Undo / Redo** (first two buttons in the tool bar): snapshot-based history of grid, elevation and holes (`checkpoint`, `undo`, `redo`, 40 steps). One checkpoint per paint stroke or hole. Undo refunds what that action cost; redo charges it again. Cleared when you switch courses; locked during events.
- Hole design score (0–10) from fairway coverage, nearby hazards, green quality, scenery, tee box, length, and crossing other holes. Course star rating = average design + par variety + golfer satisfaction.

### Simulation
- Day = 40 sim seconds; speeds pause / 1× / 4×. Daily upkeep; bankrupt below −$2,000.
- Public golfers arrive based on stars, holes, green fee vs fair price and prestige. They pay the green fee and play every hole with shot AI (club distance by skill and lie, target picking, errors, roll, water/OB penalties, putting model), then rate the experience. Thought bubbles ("Birdie!", "Splash…", "Where's the fairway?").
- Lies: tee/fairway best; rough, bunker, trees, native area, flowers worse; water = penalty drop; rocks/clubhouse/off-map = out of bounds.

### Facilities (Build tray → 🏛️ Facilities ›)
Placed by tapping (footprint centered on the tap; needs clear ground away from water, tees, greens, rocks and buildings). Footprint tiles become `T.HOUSE` (out of bounds) with a facility pad color, and are flattened to the average elevation. `S.facilities = [{type,x,y}]`, footprint index map `FO`. Clear on a facility sells it for half. Daily upkeep is added to course upkeep. Each distinct facility type adds +0.08 stars (max +0.5).
| Facility | Size | Cost / day | Effect |
|---|---|---|---|
| Snack Bar 🌭 | 1×1 | $1,500 / $20 | Hungry or thirsty golfers within 4.5 tiles of the green or next tee walk over, buy food ($6–16), hunger/thirst reset, +3 mood |
| Drink Stand 🥤 | 1×1 | $800 / $10 | Thirsty golfers buy a drink ($4–7), thirst reset, +2 mood |
| Bench 🪑 | 1×1 | $150 / $1 | Tired golfers (energy < 75) rest: +25 energy |
| Restroom 🚻 | 1×1 | $1,000 / $12 | Once per round, +3 mood |
| Pro Shop 🛍️ | 2×2, one | $6,000 / $60 | On arrival, chance by spending trait to buy ($10–60) |
| Putting Green ⛳ | 2×2, one | $3,000 / $30 | 50% practice: +0.04 putting, +0.02 chipping for the round |
| Driving Range 🎯 | 2×5, one | $8,000 / $50 | Chance to pay $8 to warm up: +0.02 driving, +0.03 accuracy |
| Cart Barn 🚗 | 2×2, one | $5,000 / $40 | Visitors rent carts ($15; seniors 85%) when cart paths exist; renters ride like members |
| Restaurant 🍽️ | 2×2, one | $9,000 / $90 | After the round, chance to eat ($15–45), +5 mood (counts toward the course rating) |
Golfers walk to "need" facilities between holes (`planDetour`, `g.detour`, `useFacility`), arrival extras happen at the first tee (`arriveExtras`), the restaurant in `finishRound`. Golfers have a `spend` trait (0–1) and track `spent` (shown on their card). Attribute boosts are `g.bonus`, added by `A()`. Visitors no longer ride carts unless they rent one; members still ride free.

### Golf empire (start screen = empire screen)
The 9 course settings are **properties**. `store.empire.owned` (in the main save store) lists owned indexes; Meadow Run is owned free, and any setting with a save is owned (old players keep everything). Each property keeps fully independent saves: money, staff, members, facilities, rating, loan.
- **Empire screen** (`showLevels`): stats (rank, properties owned, empire value, cash on hand), "Your properties" (club name, day, holes, stars, cash, value, loan; Continue / Build here / Start over), "Land for sale" sorted by unlock value.
- **Buying land:** price = land cost (`LAND`) + the setting's build budget (`LEVELS[i].cash`), paid from any started property's cash (pick in the card). Every property is for sale from the start; you just need the cash in one of your clubs (`UNLOCK` is no longer used; sale list sorted by price). The buyer's books record "Land purchases".
- **Values:** `propertyValue()` = land (min $10k) + $1,500/hole + 60% of facility cost + stars × holes × $700 + $400/member + $800/prestige; saved as `value` and `stars` with each save. Empire value = Σ (value + cash − loan). Ranks: Course Owner, Club Group (2), Golf Company (3+, $120k), Golf Resorts Group (5+, $300k), Golf Empire (7+, $600k).
- **Loans** (Club → Club tab): borrow $10k/$25k/$50k up to 50% of property value; repay 115% over 60 days in daily payments (`S.loan = {balance, daily}`, `loanDay()`), or pay off early.
- **Send money** between started properties (Club tab), recorded as "Sent to other courses".
- **Bankruptcy** now only loses that one property (unless it's your only one).
- **Unlocks:** facilities need course stars (Putting Green 1★, Pro Shop 1.5★, Cart Barn 2★, Driving Range 2.5★, Restaurant 3★; tray shows 🔒); Superintendent needs 4 holes. Bigger tour invitations already scale with holes, stars and prestige.

### Course rating breakdown (tap ★ in the HUD, or Goals → Rating breakdown)
`courseRatings()` scores nine categories 0–10; stars = weighted sum / 2 (`RATING_W`) + up to +0.3 for distinct facility types:
Hole design 25% (avg hole quality) · Challenge 10% (actual strokes over par per hole from played holes, best around +0.8; labeled Easy/Fair/Tough/Brutal) · Beauty 10% (scenery near tees/greens, elevated tees, flower beds) · Variety 10% (par mix, doglegs, elevation changes, length spread) · Fairness 10% (hole warnings: missing fairway, crossing holes, pin off green, too short/long) · Green quality 10% (green size + greens condition) · Condition 10% (`condScore`) · Pace of play 5% (sim seconds per hole from the last 25 rounds, `S.paceLog`, shown as hours for 18) · Golfer happiness 10% (recent satisfaction). The sheet shows bars and tips for the two weakest categories. Stored on `S.course.r`.

### Your rounds (Play): golfer view + swing meter
Controls are one compact bottom row (◀, club ‹ name/yards ›, ▶, Swing) plus a 🗺️/🏌️ view toggle under the meter; in landscape the Scorecard/Restart/Quit buttons stack in the side panel. Camera sits behind the ball (FOV 52°, `playCamStep`, `camPos`/`camLook`), looking down the aim line; 🗺️ Hole view toggles back to the overhead map (pan by dragging). Aim with ◀ ▶ (hold) or by dragging sideways; auto-aims at the pin or the dogleg corner. Clubs (`CLUBS`): Driver, 3 Wood, 5/7/9 Iron, Wedge, Sand Wedge, Putter, auto-picked for the distance (‹ › to change). Three-tap swing (`swingTap`, `stepMeter`, `hitP`) on a slim **vertical meter on the right edge** (`.vmeter`, fills bottom to top; red overswing zone above the white 100% line; a label beside the marker shows yards/feet during power and Now!/Wait…/Late! on the way down): tap Swing (or anywhere during the swing) → power fills in 1.6s (up to 110%) → tap to set power → marker drops back in 1.25s → tap on the green sweet spot at the bottom for accuracy (early = push right, late = pull left; sweet spot width grows with Driving accuracy/Chipping/Putting). Keyboard: Space swing, ←/→ aim, ↑/↓ club. Lie, elevation, roll, penalties and coins work as before; golfer swing pose follows the meter.

### Record book (Club → Honors)
`S.records = { rounds:{[holeCount]: {toPar,total,name,kind,country,day}}, holes:[{best,par,name,kind,day}], aces:[] }`. Course record per layout size, best score on each hole (resets if that hole's par changes, shifts when a hole is removed), hole-in-one list; toasts for new course records and aces. Updated from AI golfers, event players and your pro's rounds.

### Names
**Your name lists** (Club → Club tab): two text boxes, one name per line, stored empire-wide in `store.names = { pros:[], members:[] }`. New tour pros (`proPlayer`) and new members (`newMember`) take unused names from these first (`poolPick`); pros can carry a country code ("Name, ESP" or "Name (ESP)", `parseNameLine`). Otherwise `randName(code, fun=.35)` makes names: 35% original golf puns (`FUN_FULL`, `FUN_FIRST` × `FUN_LAST`: Chip Shotwell, Sandy Trapp, Rusty Irons, Bo Gee, Holly Inone…), the rest country/regular names. The built-in pools don't include real people; Andrew adds whoever he wants through the lists.

### Golfer traits
Skill attributes (`ATTRS`, coin-upgradable): Driving, Driving accuracy (tee/long shots), **Iron play** (`ir`, approach shots at the pin from 60+ yds), Chipping, Sand, Recovery, Putting, **Course management** (`cm`: better target choices with less randomness, more respect for water, better elevation adjustment). Personality traits (not skills): **Patience** (`patience`, bad holes hurt mood more for impatient golfers; holes taking 14+ sim seconds annoy them: "This is taking forever.") and **Spending** (`spend`). Shown as "Patient, big spender" on golfer cards and member profiles. Old members/pro get `ir` = old accuracy and a `cm` value.

### Staff & maintenance (🏆 Club → Staff tab)
**Course condition** (`S.cond`: turf, greens, bunkers, equipment, 5–100%) changes once a day in `maintDay()`. Crew capacity vs. workload (`maintCalc()`): each Groundskeeper handles ~60 fairway/tee tiles; each Greenskeeper ~25 green tiles and ~20 bunker tiles (groundskeepers help a little); a Superintendent makes the crew 30% more effective; worn equipment (decays daily, Repair button, cost grows with wear) cuts effectiveness up to 40%. Daily change ≈ +8×coverage − 5 for turf (greens +9×coverage − 6, bunkers +6×coverage − 4), plus watering (Low −2 / Normal 0 / High +1.5), fertilizer (+1.5 turf, +1 greens) and minus golfer traffic. Desert courses lose 2 turf a day unless watering is High.
**Effects:** course stars + clamp((condition − 70)/60, −0.8, +0.3) where condition = turf 40% + greens 40% + bunkers 20% (`condScore`); golfer mood (condition − 65)/12 per hole plus complaints ("These greens are bumpy.", "The fairways need a mow.", "Nobody raked these bunkers."); putting success × (0.82 + 0.18 × greens%); grass color dries toward straw-brown below ~72%.
**Roles** (`ROLES`, daily wage): Superintendent $90 (max 1), Groundskeeper $35, Greenskeeper $40, Pro Shop Clerk $30 (needs Pro Shop, +40% shop sales), Chef $35 (needs a food facility, +35% food & drink), Cart Attendant $25 (needs Cart Barn, +15% rental chance), Marshal $30 (golfers walk 12% faster, pick up at par+3). New courses start with a groundskeeper and a greenskeeper; old saves get the same starter crew.
**Daily bills:** salaries, water (per fairway/tee/green tile by watering level), fertilizer, electricity ($15 + $8 per facility), repairs; all in Finances. The end-of-day message shows the condition % and warnings.

### Economy
Money pays for everything you build. It comes in from green fees (each public round), food & drink, pro shop sales, range fees, cart rentals, weekly member dues (every 7 days), hosting events (tour invitations pay a hosting fee plus ticket sales; club events collect entry fees) and your club pro's prize money. Lesson fees from your pro's member lessons also come in. It goes out to building and daily upkeep. Every transaction is booked by category (`book(cat, amt)`) into `S.books` (this week) and `S.lastBooks` (last week, rolled every 7 days); the Club tab shows a Finances table. Tapping the money in the HUD opens it.

### Club pro (you) and the coin system
`S.pro`: name, nationality, shirt/pants and six attributes. **Play round mode is your pro playing** (uses the pro's attributes and outfit).
- **Attributes** (`ATTRS`, stored in `.at`, 0.1–1.0 for members, up to 1.1 for the pro): Driving `dr` (distance), Accuracy `ac` (full shots), Chipping `ch` (shots within ~60 yds of the pin), Sand `sd` (bunker distance/accuracy), Recovery `rc` (rough, trees, native, flowers), Putting `pt`. Read with `A(g,k)` (falls back to `pow/acc/put/skill` for AI golfers); lie effects from `lieAdj(lie, g)`. Handicap = average of attributes.
- **Coins** (`S.coins`, shown in the HUD; tap to open the pro): earned by the pro only. Play round: ace 15, eagle 6, birdie 3, par 1 per hole, +5 for finishing and +5 more if under par. Lessons: 🪙4 + $40 fee each, 3 per day (`LESSONS_PER_DAY`), member gets +1 in the chosen attribute and a mood boost. Tournaments: 1st 40, 2nd 25, 3rd 20, top 10 12, otherwise 5.
- **Spending coins:** +1 to any attribute of the pro or any member (Profile sheet). Cost rises with level (`coinCost`: about 2 coins at 50, 6 at 75, 9 at 100).
- No money is spent on the pro anymore. Enter the pro in pro-level invitations (senior, pro tour, major) with the toggle in event setup; they play as an AI golfer with a gold name tag. Finishes pay a share of the purse to the club (1st 18%, 2nd 11%, 3rd 7% … 10th 1.8%). Tour record: starts, wins, top 10s, earnings and a results log.

### Tour invitations (random events)
Outside tournaments arrive as random invitations (`S.offers`, max 3; ~40% chance each day once you have a hole; each expires after 6–14 days). Each lists course requirements with ✓/✗ and "(X now)" progress, and can only be accepted when every requirement is met. Tiers (`TIERS`): County amateur (3–4 holes, 2★, plus water/bunkers/par 3), Collegiate (6 holes, 2.5★, par 5, bunkers on 3 holes), Legends Senior Tour (6 holes, 3★, cart paths on every hole; pro can enter), Atlantic Pro Tour with fictional sponsors (6–9 holes, 3.5★, water on 2 holes, par 5, 2,000 yards; pro can enter, $20k purse), United Links major (9 holes, 4★, prestige 5, water on 3, two par 3s, two par 5s, 3,200 yards; $60k purse). Bigger tiers become more likely as holes, stars and prestige grow. Requirement checks: `REQ` (holes, stars, members, prestige, par3, par5, water, sand, carts, yards).

### Club management (🏆 Club button)
- **Staff tab:** course condition bars, crew coverage, watering/fertilizer/repairs, daily costs, staff list (let go), hire cards.
- **Club tab:** club pro card, Pro lessons (pick member + attribute, 3 per day), Finances table, club name, Club Championship trophy name, stats, reigning champion, what each golfer type thinks.
- **Members tab:** member cards (shirt-color badge with flag, personality, handicap, match record, mood). Tap to open the **member profile sheet**: name, nationality (14 flag chips + None), personality, six attributes upgraded with coins (Driving, Accuracy, Chipping, Sand, Recovery, Putting; handicap derives from their average), shirt and pants colors (avatar preview), and record (rounds, best round to par, match W-L-H, titles, head-to-head vs each member). Weekly dues (collected every 7 days) vs "fair dues". Members apply when rating/dues are good; unhappy members quit; members play free rounds on their own schedule.
- **Golfer types (personalities):** Weekend Golfer (wants wide fairways, hates water), Grinder (loves hazards/tough holes, bored by easy ones), Senior (shorter, rides carts, hates long walks), Social Golfer (loves scenery, cares less about score), Big Hitter (+15% distance, less accurate, loves par 5s). Applies to public visitors (weighted mix), members and guests; drives satisfaction and thought bubbles. Club tab shows "What golfers think" mood per type.
- **Member matches:** pick two members (Events tab); they play match play while the course stays open; live status ("X 2 up thru 5"); results update records/head-to-head; match log on Honors tab.
- **Cart paths:** members ride 3D carts (faster) when start and destination are within 2 tiles of a path; cart access between holes makes them happier, long walks without carts make them unhappy.
- **Events tab:** 📨 Invitations (random tour events, see above), Member matches, Club events: Club Championship (members, for your named trophy) and Member-Guest Invitational (member + guest teams, best ball; guests may join the club), with requirements and cooldowns. Editable field: names and countries. Course closes to the public and locks building during events. Live leaderboard, leader banner, green name tags on the top 3, results screen.
- **Countries:** 14 common pro-golf nations (USA, CAN, MEX, ENG, SCO, IRL, ESP, ITA, SWE, RSA, AUS, JPN, KOR, CHN) with matching name pools; flag emojis (falls back to 3-letter codes on Windows). Shown in setup, leaderboard, name tags, scorecards, Honors.
- **Honors tab:** every champion with day, event and score.

### Scorecards (📋 button or tap a golfer)
List of everyone on the course; individual hole-by-hole cards (birdies red, bogeys blue), current hole, total to par, mood, member handicap, country. "Follow on the map" camera tracking.

### Play mode (🏌️ Play round)
You play as your club pro: max distance from Driving, error from Accuracy / Chipping (near the green) / Putting, lie penalties from Sand and Recovery. Earns coins (see above). Slingshot aiming (pull back and release), power and carry shown, landing circle, putting in feet on greens, penalties, scorecard and results.

### Graphics & camera (SimGolf-inspired look, original art)
**Crisp square tiles:** every tile is one terrain type with hard edges and per-tile mowing patterns (`rebuildTerrain` builds non-indexed geometry, `SUB=4` sub-cells per tile for smooth heights, normals from the height grid). While a Build/Landscape/Facilities tray is open, a white tile grid (`gridLines`) and a tile cursor (`S.cursor`; shows brush or facility footprint) appear.
three.js r128 with a near-isometric camera (FOV 14°, pitch 0.84, starts zoomed in; Fit shows the whole course). Terrain: 4 vertices per tile (`SUB=4`), crisp type blending, bold mowing patterns (diagonal fairway stripes, wide rough bands, checkered greens), dark dirt beds under trees, deep blue water, sunken bunkers, raised greens/tees/rocks. The course sits inside a **countryside ring** (`buildRing`, `RING=14` tiles): forest on land sides (2-tile clearing at the edge so trees never hide the course), open water on coastal sides; rebuilt only when the setting changes. Trees: per-setting forest mixes (`FOREST`, aligned with `LEVELS` order; append-only) of round, pine (3-tier), willow, blossom, autumn, cypress, palm, longleaf and cactus; 2–3 trees per tile. Grand clubhouse (main hall, tower, two wings, porch with columns, awnings, chimneys, planters, flagpole). Sun shadows, dusk shift, 3D golfers/carts, waving flags. 2D overlay for route lines, aim guide, badges, thought bubbles, name tags.
**Floating HUD** over the map (glass pills: club/goals, money, stars, coins, day, fee, speed). **Golfer info card** (tap a golfer): avatar, type, quote based on mood, likes, mood bars (Fun, Energy, Hunger, Thirst), hole-by-hole strip, recent thoughts. Golfers get hungrier/thirstier/tired each hole (`energy`, `hunger`, `thirst`), which lowers mood when high — ready for refreshment amenities. Every visitor has a generated name.
Controls: pinch/scroll zoom, two-finger or Move-tool pan, twist / right-drag / Q,E to rotate, arrows pan, +/− zoom, ⟲ ⟳ and Fit buttons.

---

## 2. Code map (sections inside the single `<script>`)

| Section marker | What's in it |
|---|---|
| (top) | Constants: map `COLS=50, ROWS=62` with the starting property `BX0=8, BY0=8, BW=34, BH=46` in the middle; clubhouse at `CHX=BX0+16, CHY=BY0+44`. `DAY=400` sim seconds (about 6½ min per day at 1×; ▶▶ is 3×), `MAXG=60`. `migrateSave` converts 18×26 and 34×46 saves (offsets keep the clubhouse in place; coastal settings fill the outer sea side with water); feature counts scale by `AF`, camera fit by `MK`; terrain `SUB=3`, `YD=25` yards per tile, `DAY=40`, `MAXG=30`, tile types `T`, `TOOLS`, `UPKEEP`, `ALL_TOOLS`, `NATIVE_STYLE`, `LIEVAL`, `LEVELS`, name pools, `COUNTRIES`, `EVENTS`, `SHIRTS`, helpers (`clamp`, `lerp`, `gauss`, `fmt`, `rng`, `hash`, `pick`) |
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
- **Hole:** `{ tee:{x,y}, pin:{x,y}, bend?:{x,y}, plays, strokes, info:{ yards, par, q, notes, cart } }`
- **Golfer:** `{ id, x, y, bx, by, hole, strokes, state:'walk'|'address'|'flight', type, skill, at? (attributes) or pow/acc/put (AI), color, pants, skin, hat, sat, card[], name?, mid? (member id), ep? (event player), mp?/side? (member match), country?, cart? }`
- **Member:** `{ id, name, type, country, at:{dr,ac,ch,sd,rc,pt}, skill (avg), lessons, shirt, pants, skin, sat, rounds, best, titles, rec:{w,l,h}, h2h:{[oppId]:{name,w,l,h}} }` (old saves upgraded by `fixMember`)
- **Match:** `{ id, a, b (member ids), ca[], cb[] (hole scores), done:{a,b} }`
- **Event player:** `{ name, skill, pow, acc, put, type?, country?, mid?, team?, guest?, card[], started, done, sat }`
- **Offer:** `{ id, tier, name, org, blurb, field, skill, type?, prestige, purse, pro, base, gate, reqs:[[key,n]], expires }`
- **Club pro:** `{ name, country, at:{dr,ac,ch,sd,rc,pt}, skill, shirt, pants, skin, starts, wins, top10, earnings, log[] }`
- **Save (per level index):** `records, grid, elev, bridges, facilities, holes, money, earned, day, fee, recent, won, members, mid, dues, prestige, eventLast, history, clubName, trophy, matchLog, matchId, typeSat, pro, offers, offerId, books, lastBooks, coins, lessonsLeft, staff, staffId, cond, water, fert, loan, stars, value`; empire-wide: `store.empire.owned`

---

## 3. Roadmap (from the full game spec)
1. ✅ Construction upgrade: undo/redo, hills and elevation.
2. ✅ Better holes: dogleg holes (bend point), fringe collar on greens, bridges, hole signs.
3. ✅ Facilities: pro shop, snack bar, drink stand, restaurant, driving range, putting green, cart barn, restrooms, benches; revenue from shop, food and drink, range and cart rentals; fix hunger/thirst/energy.
4. ✅ Staff and maintenance: superintendent, groundskeepers, greenskeepers, clerks, chefs, cart attendants, marshals; salaries; course condition (turf, greens, bunkers, equipment) with watering, fertilizer and repairs; water and electricity costs.
5. ✅ Ratings breakdown (design, challenge, beauty, variety, fairness, green quality, condition, pace of play, happiness) and deeper golfer traits (iron play, course management, patience, spending).
6. ✅ Golf empire: buy additional properties with independent books, loans, money transfers, progression unlocks.
7. ✅ Polish: animations, landscape layout, performance, backups, offline package (no sound by design; not going to the App Store).

## 4. Ideas not built yet
- Amenities: snack carts, refreshment stands (fix hunger/thirst), benches (energy), restrooms, driving range, pro shop, decorative buildings (windmill, lighthouse, gazebo, fountain)
- Staff (greenskeepers, marshals) and course condition
- Multi-round tournaments with a cut, weather and wind
- Elevation tools (raise/lower land), island greens
- Sync saves across devices (would need an online service)
