# Fairways Tycoon — Design & Code Guide

A SimGolf-style golf game: design courses, run a golf club, host tournaments and play your own holes. Single HTML file, three.js 3D rendering, installable from GitHub Pages as a home-screen app. Personal project, **not going to the App Store**. **Primary platform: iPhone** (bottom-sheet menus under 600px wide, 44px+ touch targets, 16px inputs so iOS doesn't zoom); also works on PC.

---

## 1. Feature list (current)

### Branding
Game name **Fairways Tycoon**. The logo (golf ball, flag, clubhouse, striped fairway) is embedded on the start screen as a base64 JPEG (`img.logo`) and used for the home-screen icons (apple-touch-icon 180, 192, 512, maskable 512 on green, favicon 32). The save key stays `fairway-tycoon-v1` and the file name stays fairway-tycoon.html.

### v54: graphics, smooth pass + play-mode look + shot effects (golfer v2 is a separate test page)
- **Smoothness:** play mode advances the swing meter and ball flight with up to 0.1 s per frame (`dtp`), so a slow frame no longer turns into slow motion (the simulation still uses 0.05). **Adaptive resolution** (`stepFps`, `dprCap`): when the frame rate stays under 45 for 2 s (while playing or the sim is running), pixel ratio steps down by 0.25 to a minimum of 1.25; above 58 for 8 s it steps back up to 2. **📈 Frame rate** chip (📋 Golfers screen, `store.opts.fps`) shows fps and the current pixel ratio. In play mode, far-off work is skipped (animals, beverage carts, range balls, ropes, scoreboards; event crowds still update).
- **Camera:** smoothing is frame-rate independent (`1 − e^(−dt·8)`), and on full shots (longer than ~60 yds, not putts) a **chase cam** follows the ball in the air from behind and above, looking toward the landing spot, then glides back behind the ball.
- **Play-mode look** (only while playing, not in overview; `setPlayLook`, `stepPlayLook`): a gradient **sky dome** with sun glow and 14 cloud sprites, distance **haze** (`playFog`, 40–150, rain 22–95), sky/haze colors by weather; **focused shadows** that follow the ball (±14 instead of ±58, much sharper); water gets stronger ripples and shine. The builder view is unchanged.
- **Everywhere:** trees and the outer forest **sway** in the wind (`swayMat`, vertex shader on canopy parts only; gentle normally, stronger when windy or raining); water has slowly moving **ripples** (`rippleTex` bump map, subtle from above).
- **Shot effects:** a white **ball tracer** for your shots (fades 1.4 s after landing, `drawTracer`, `P.lastFl`), and **landing puffs** (`puffAt`): turf bits on fairway/rough, a sand spray in bunkers, light green dust on greens.
- **Golfer v2 test page** (separate artifact https://claude.ai/artifact/1LXnotgHGNWwYwvnZvz6aL, source in outputs/golfer-test/index.html): new golfer with a lathe-shaped torso, smooth limbs drawn between joints, gloved hand, cap with brim, ears and a sketch-style face texture; animations: full swing (address, backswing, top pause, downswing, impact, follow-through with the ball flying off, swing done by rotating the arms and club in a tilted swing plane plus shoulder/hip turn and wrist hinge), walk with arm swing, birdie fist pump, idle; slow-mo, outfit swap and drag-to-spin; today's golfer stands alongside for comparison. Not in the game yet.

### v53: school colors and conference seasons
- **School colors** (`SCHOOL_INFO`: shirt color, cap color and a 3-letter tag for each of the 14 schools, e.g. Westfield WFU green/gold, Riverbend RBU purple/gold). College players wear their school's shirt and cap on the course (`schoolDress`, passed to `makeGolfer` as `color`/`hat`/`hatStyle:'cap'`) and on their sketch faces (`playerFace`). Leaderboards show a colored school tag after each name (`schoolTag`), and team standings show tags too.
- **Conference season** (per course, saved as `S.conf`, `ensureConf`): each course has one conference (`name` from `CONFS`) with 4 fixed schools, each with a 5-player roster (`players`: name, country, year 1–4, skill). The **Conference Championship** invitation (`conf:true`, tier `college`) appears automatically every 28 days once the course has 6 holes (`maybeConfOffer` from `clubDay`; random conference invites are off), stays 14 days, and its field is each school's top 4 (`confField`). After the event (`confAfter`): team and individual champions recorded in `C.history` (season, team + score, individual + school + score, runner-up), titles counted, defending champions set (`C.def`); "retains the title" / "defends the individual crown" messages; then the season turns over: every player improves 0.02–0.04, seniors graduate (named in the results), freshmen fill each roster back to 5. The setup screen shows a season box (season number, defending team and individual champions with tags, most titles) (`confInfoHtml`); Honors shows the conference history table (`confHistoryHtml`).
- **Nationals:** your conference's defending champion gets a place in the National Collegiate Championship field.
- Club tabs now scroll sideways and use a slightly smaller font, since 6 tabs no longer fit on a phone.

### v52: college championships and clubhouse placement
- **College golf** replaces the generic "Collegiate Invitational" with two invitation tiers (TIERS keys `college` — kept so old saved offers still work — and new `ncaa`):
  - **Conference Championship** (`college`): "[Coastal Plains / Great Lakes Athletic / Mountain Ridge / Bluegrass Collegiate / Pacific Shores / Prairie Valley Conference] Championship", 4 schools × 4 players, skill 0.68–0.9, needs 6 holes, 2.5★, a par 5 and 3 bunkers, prestige 1.5.
  - **National Collegiate Golf Championship** (`ncaa`, fictional "National Collegiate Golf Association"): 6 schools × 4 players, skill 0.78–0.98, needs 9 holes, 3.5★, prestige 3, two par 5s and 2,600 yards; prestige 3, bigger pay and gate; offered once you have 9 holes and prestige 3.
  - Players are generated college players with a school (`collegeField`, from 14 fictional `SCHOOLS`, each team with a small strength offset), not tour roster players. **Team scoring**: each school counts its best 3 of 4 scores to par (`collegeTeams`); team standings show in the live Events view and the results sheet above the individual leaderboard, the results text names the team champion, and Honors records a "(team)" entry.
- **Clubhouse placement on new courses:** starting a new course puts the clubhouse in the middle of your land (`centerHouse`, nearest clear 2×2 spot), and closing the welcome card switches to placement (`startHousePlacement`: Holes tray, Clubhouse tool selected and scrolled into view, whole course in view). A pulsing gold ring marks the clubhouse while the tool is selected; tap anywhere to move it (free until the first hole, `houseTap`), then ✓ Done. Shared helpers `houseSpotOk`, `setHouseAt`.

### v51: name lists fixed (and everything else in the save file)
- **Bug:** `loadStore` rebuilt the store with only `unlocked` and `saves`, so every time the app opened it dropped the **name lists**, the golf tour (`store.tour`), the tour player roster (`store.roster`) and settings (`store.opts`: Easy swing, Comments); the empire list was rebuilt from saves. Cloud loading (`applyCloud`) and backup restore had the same problem. All three now keep the whole object (`{ ...s, unlocked, saves }`).
- Name boxes save as you type (0.6 s after the last keystroke) and when you leave the box.
- **Tour player names join the tour**: when you finish editing the Tour player names box (or close the Club), `syncRosterNames` puts each name on the roster, replacing the least-played players (they keep their skill, stats reset), marks them `mine`, and brings back removed players who are re-added; names you delete from the list that never played are given new generated names. A toast says how many were added.
- Country codes work as "Sam Smith, ESP", "Sam Smith (ESP)" or "Sam Smith ESP" (a final all-caps 3-letter word that's a known country code).

### v50: tour players, rankings, awards and more
- **Tour players (one roster):** `store.roster` (empire-wide, 90 players, skill 0.55–1.15, seeded from the "Tour player names" list first, renamed from "Tour pro names"). Every invitation picks its field from the roster (`proPlayer` → `rosterPick`, by event skill range, favoring players who've played before and rank higher), so the same players keep coming back (test: 8 of 12 in a third event had played before). The user-run golf tour also draws its 48 players from the roster (`rid`). Players can be renamed or removed ("Remove from the tour" sets `gone`).
- **Rankings** (🏆 Club → **Ranks** tab, `rankingsHtml`): 🌍 Tour players — world ranking points from every non-team event (`recordTourResults`: base by event size amateur 20 / pro 60 / open 150 / tour 220 × finish factor 1, .6, .45, .35… ; all points ×0.97 after each event), events, wins, top 10s, scoring average to par, best round, prize money; your pro is included (`S.pro.rk`). Tap a player for their card (`openRosterCard`). 🏠 Members — rank by handicap, scoring average, best round, titles, rounds or vs-pro record; full rounds update `m.st` (n, tp, best).
- **Remove members:** member profile → "Remove from the club".
- **Course awards** (Roller Coaster Tycoon style, every 28 days, `monthlyAwards`): Most Beautiful Course, Best Value in Golf, Happiest Golfers, Best Conditioned Course, Best New Design (each +0.5 prestige and 15% more visitors for 7 days, `awardMul`), or Worst Value / Most Neglected (15% fewer visitors). Listed in Honors (`awardsHtml`, saved `S.awards`).
- **Facilities turn** (`f.rot`, 0–3 for square buildings, 0/2 for long ones; grandstands still face the nearest green): new facilities face the nearest cart path within 7 tiles (`autoRot`); tap a facility or landmark → **↻ Turn it**. Golfers use the front door (`facFront`).
- **Carts drive the course:** members and renters ride whenever there's a cart barn or any cart path (`cartsAvail`). They follow paths when start and end are near a connected path, otherwise drive straight over fairway and rough, never through water, rocks, trees or buildings, and park short of greens, tees and bunkers, walking the last bit (`cartPark`, `NO_CART`). Renting no longer needs paths. The "No cart path out here?" complaint only appears when no carts are available.
- **Walkers carry their clubs:** ~60% pull a trolley (bag, handle, two wheels), the rest carry a shoulder bag (`makePerson`).
- **Beverage cart upgrades** (Upgrades tray card, `BEV_UP`, `S.bevLv`): Lv2 Snack menu $4,000 (sales ×1.4, hunger fully reset, grill on the cart), Lv3 Bar & grill cart $9,000 (sales ×1.9, 25% faster, serves from 2.3 tiles, canopy).
- **Members of more than one course:** member profile → **🤝 Also a member at** (`m.also`). Those members visit your other courses (as away members, ~20% chance a day), play free, and that course earns a reciprocal fee of 40% of its dues per away member each week; happy members (mood > 78) sometimes join another of your courses on their own.
- **Gallery crowds** are bigger (fans ÷ 5, at least 24, up to 260), figures ~20% larger, spread from tee to green, and also appear for team cups.

### Comment bubbles (less clutter)
`say()` always records the line (Feedback topics and the golfer's recent log), but whether a bubble pops up depends on **💬 Comments on the map** (📋 Golfers screen, saved as `store.opts.chatter`): **All** (old behavior), **Some** (default), **Big moments** (only lines matching `BIG_SAY`: ace, albatross, eagle, birdie, stadium roars, gators, splashes, out of bounds, "Not coming back", "What a round", "brutal", course records, sheep) or **Off**. In Some mode: at most 2–5 bubbles on screen (fewer the busier the course: 6 − golfers/10), at least 1.1 game-seconds between new routine bubbles, routine lines shown with probability 10/golfers (min 20%), big moments can go one over the cap. When drawing (any mode but All), big moments draw first and a bubble that would overlap one already drawn is skipped. The golfer you're following always shows. Bubbles last 2 s (2.6 s for big moments). Test, ~15 golfers: All peaked at 7 on screen (avg 1.6), Some at 3 (avg 0.7), Big moments at 2.

### 🗺️ Courses sheet (switch or buy without leaving)
Club tab → **🗺️ Switch courses or buy a new one**, or Goals → **🗺️ Switch or buy courses** (`openCourses`, overlay `#courses`). Saves the current course first. **Your courses**: mini map, club name, day, holes, stars, cash; the current one is highlighted ("Here"), others have **Go** (or **Build** if not started), which calls `startLevel(i, cont)`. Blocked during an event, team cup or your own round. **Buy another course**: every unowned course, cheapest first, with price and build budget; **Pay from** any owned course with money, including this one (deducted from `S.money` and booked as land), others from their save (`buyCourseHere`). The empire screen still works as before.

### Swing meter v2 (v45): hold, let go, tap
Replaces the tall vertical meter (hidden with CSS, `.vmeter`). Modeled on PGA Tour 2K's hold-and-release 3-click power and Golf Clash's needle-on-a-target accuracy.
- **Aim arrow (v49, Andrew's idea):** accuracy is now a **swinging arrow at the ball** instead of a needle bar (`drawSwingArrow`, drawn in `drawAim` during the `acc` phase; the `#smAccRow` bar is hidden). After you let go of power, a white arrow swings back and forth across a fan on the ground (triangle wave, constant speed, starts on a random side, `m.adir`; it no longer auto-fires, you tap when ready). Fan zones: gold = Perfect (±0.18 × sweet width), green = Great (±½), light = Good (±1), dark = the full swing (±`ACC_A`). The shot direction is exactly where the arrow points: offset = needle value × 0.9 × 1.1 × lie error (`arrowK`, `hitP` uses `off = acc*.9`), so the arrow is truthful (test: arrow 16.7° → ball 16.1°; center → 0.9°, PERFECT!). Better pro skill = slower swing and wider zones; overswing speeds it up. Tap Swing or anywhere on the course to stop it.
- **Layout (v48, after feedback that the panel was too bulky):** no panel. A slim 16 px **vertical power bar** on the right edge (`#smeter`, top ≈30% to just above the play bar; fills bottom-up; red overswing cap above the white 100% line; white 🎯 line = power to reach the pin, red if out of range; green tick = last power), with a small tag beside it that shows the pin distance when idle ("🎯 275y") and the live yards while holding. A small **aim bar** (`#smAccRow`, ~170 px) sits just above the Swing button and only shows while aiming and while the ball is in the air. The result ("PERFECT!") flashes at ~26% from the top (`#smRes`). A tiny **🎯 Pro / 😌 Easy** toggle sits beside the top of the bar (`#smMode`). Hints describe hold → let go at the 🎯 line → tap the needle.
- *(v45 original, replaced)* Panel just above the play bar: a horizontal **Power** bar (0–110%, white line at 100%, red overswing zone), a **🎯 pin marker** at the power that reaches the pin with the current club (red if out of range), a live yardage line ("212 yds · 🎯 pin 275 yds", or "about 78% power" before you swing), and an **Aim** bar with nested zones: Good (2× sweet width, light), Great (sweet width, green) and Perfect (0.36× sweet width, gold) (`sweetW` still scales with the pro's skill for the shot).
- **Controls** (`swingDown`, `swingUp`, `lockPower`, `swingTap`, `stepMeter`): press and hold Swing → power fills (and bounces back down at 110% instead of auto-firing); release → power locked, the needle sweeps from left (early → pushed right) to right (late → pulled left) at a speed set by skill (`meterTimes().down`), 4× faster per unit of overswing; tap (Swing button or anywhere on the course) to hit. If the needle reaches the end it hits late automatically. Quick taps (<220 ms press) still work as tap-start, tap-set-power, tap-hit. Keyboard Space works as taps.
- **Results** (`hitP` tiers): PERFECT! (scatter ×0.4), Great! (×0.75), Good (full scatter), or Pushed right / Pulled left (aim error grows past the Great zone, as before). Shown as a big colored pop-up above the meter (`showSwingResult`); after the shot, a green tick on the power bar and a white (or red) tick on the aim bar mark where you stopped, until the next swing.
- **Easy swing** (link under the meter, saved for the whole game as `store.opts.easySwing`): hold and let go for power only; accuracy is rolled from the pro's skill, mostly inside the Great zone ("Nice swing!").
- Button text follows the phase: "Hold to swing" → "Let go!" (or "Tap to set") → "TAP!".

### Swing meter feedback and live score (play mode)
- The meter now scales with your pro's skill for the shot (`shotSkill`: putting, chipping inside ~60 yds, irons inside 200, otherwise driving accuracy): power fill 1.35 + 0.55×skill seconds, return 1.05 + 0.45×skill (`meterTimes`), and the green sweet spot is 2% + 7.5%×skill of the meter (`sweetW`) — roughly 5% for a beginner, ~10% for a top pro.
- After each swing the meter keeps a gold mark at the power you set and a green (or red if missed) mark where you stopped for accuracy, with a label like "52% · Pure!" (`P.lastPow`, `P.lastAcc`, `P.lastWord`), until you start the next swing.
- A live score chip at the top (`#liveScore`) shows hole, score to par, holes played and current stroke; tapping it opens the scoreboard (`showCard`): hole, par, yards, score (birdie+ red circle, eagle double circle, bogey blue square, double+ double square), running total, totals, course record and your pro's best round for this layout (`S.pro.bestRounds`, a new best is announced).

### Pro shop pricing
Tap the pro shop (in look mode, or via Upgrades / ✏️ Edit) → **🏷️ Prices**: golf balls, gloves, logo hats, club polos, drivers (`SHOP_ITEMS`, saved `S.shopPrices`, weekly `S.shopSold`). Fair prices (`fairShop`) rise with stars and prestige. Every arriving golfer (and practicing member) browses once (`shopVisit`): each item's buy chance = base × spending habit × shop level/clerk × clamp(1.8 − 0.8 × price/fair). Revenue per golfer peaks around +10% over fair (~$19 vs ~$18 at fair); above +40% golfers grumble ("Pro shop prices are steep!", a Feedback complaint) and sales fall. Tapping any non-landmark facility in look mode opens its sheet.

### Tournament scoreboard (🌳 Landscape → Landmarks → 📊 Scoreboard)
2×1 landmark, $4,000, one per course. A canvas-texture board on posts (`makeScoreboard`, `drawBoard`, refreshed every second by `stepBoards`): during events it shows the live top 5 (name, to par, thru); during team cups the team score and matches finished; otherwise course record, last champion, holes and rating, day and weather, golfers on course. Counts as a landmark for beauty.

### Golfer portraits
Every player gets an automatic ink-style sketch face (`faceSVG`), seeded from their name (`nameSeed`) so the same person always looks the same; nothing is stored in saves. Face shape, hair style and color (gray for seniors and some older players), brows, eyes, nose, beards, glasses or sunglasses, freckles and a pastel background come from the seed; skin tone, shirt and hat style/color come from the player's outfit when they have one (members default to a green cap, like their body avatar). Expression follows mood (≥65 smile, <45 frown, else neutral); eyes blink (CSS, off with reduced motion). Helpers: `memberFace`, `golferFace`, `proFace`, `nameFace`, `playerFace` (member/pro/name lookup). Shown in: members list (44 px, flag badge), member/pro profile (72 px beside the outfit), golfer cards (64 px), Golfers list, reviews in Feedback, event leaderboards (single players), tour standings, team cup lineups and Honors. Cached by name+outfit+mood+size (`faceCache`).

**Your pro's face (🪙 coin chip or Club → your pro → 🎨 Your face):** pick skin tone, face shape (oval, round, long, wide), hair (short, side part, buzz, curly, long, ponytail, bald) and hair color, eyes, eyebrows, nose, facial hair, glasses or sunglasses, expression (smile, calm, game face), freckles, smile lines and background; 🎲 Surprise me and ↺ Start over. Choices are saved as `S.pro.face` (only the parts you picked; the rest still come from the name) and override the recipe via `faceParts`; skin also sets `S.pro.skin` for the body and 3D golfer. The profile header (face preview) stays pinned while scrolling.

### Play as your pro (events and member matches)
- **Events:** in the setup screen of any stroke-play event you host (invitations and the Club Championship), "🏌️ Enter your pro", then "🎮 Play the pro's round yourself". Teeing off starts your round first (`startRound({ ev })`, the entry is marked `you` and skipped by `eventStep` spawning); when you finish, your card goes on the event (`P.ev.card`), and the rest of the field plays after you return to the course. Quitting early fills the remaining holes with par + 2. Prize money (if the event has a purse), coins and the pro's tour record work as before.
- **Member matches:** member profile → "⚔️ Challenge [name] to a match" (`challengeMember`). Match play: after each of your holes the member's score is simulated from their skill (`oppScore`), the hole is won, lost or halved, and the match ends when one side can't be caught (e.g. 3&2). The live score chip shows "vs Name · 2 UP"; the scorecard adds their column with ✅ ❌ ➖ and the match status. Results: their record vs your pro (`m.vsPro`, shown in their profile), coins (win 8, halve 4, lose 2), and the member's mood goes up (more if they beat you).

### Bridges
Bridge tool is now tap-to-tap: tap the bank where it starts, then the far bank (`bridgeTap`). It snaps to a straight or 45° line, must cross water the whole way between two land tiles, up to 11 water tiles, $80 per water tile; both end tiles become cart path ($15) so it joins your paths. Stored as spans in `S.bridges` (saved) plus `BR` tiles for cart routing. Drawn as one continuous arched deck with rails and posts (`buildBridgeMeshes`); old single-tile bridges still draw the old way.

### Event tickets and gallery crowds
- Event income is now hosting fee + **ticket sales** + **concessions** (`gateEst`, `eventIncome`). Fans = event size × rating × prestige × weather (sunny 1.1, cloudy 1, windy 0.9, rain 0.6) × 1.15 if your pro is entered, times a price factor clamp(1.6 − 0.6 × price/fair). Fair ticket = 5 + 2.5 × stars + 0.4 × prestige (`fairTicket`). Revenue peaks near the fair price. Concessions: $4 per fan with a snack bar or drink stand, +$3 with a restaurant. Club Championship fans (family and friends) come free and only buy concessions; team events have no gallery.
- Setup screen: **🎟️ Ticket price** −/+ with fair price, expected fans, tickets and concessions (`ticketHtml`, `S.setup.ticket`). The estimate is fixed at tee off (`S.event.gate`); results list fans, ticket price and totals; the event chip shows 👥 fans.
- **Gallery:** up to 140 spectators (fans ÷ 8) drawn as instanced figures around greens and along the edges of holes (not on fairways, greens, sand, water, paths or buildings), ~62% gathered at the focus hole: the hole you're playing when your pro plays it yourself, otherwise the leader's hole (`placeCrowd`, `updateCrowd`, re-placed when the focus hole changes). They jump and cheer when you make a birdie or better in your event round ("👏 The gallery roars!").

### Beverage cart (staff)
Club → Staff → **🍹 Beverage Cart** (role `bevcart`, $30/day wage, up to 3, needs 3+ holes). Each one is a 3D drink cart (white body, blue cooler, red-and-white umbrella, driver in a gold cap) that leaves the clubhouse door and drives to the hungriest/thirstiest golfer it can reach (`bevTarget`), pulling up beside them off the line of play, using your cart paths via `cartRoute` when the start and end are near a connected path (`stepBevs`, runs in sim time; `syncBevs` draws it). It sells to any golfer within ~1.7 tiles at most once every 3 holes: $5–15 (×1.3 if they were hungry or thirsty), counted in Food & drink (chef bonus applies), resets thirst, cuts hunger by 45, +2 to +4 happiness, and a line like "Bev cart! Perfect timing." (a Wildlife-style praise topic, "Beverage cart"). It parks when nobody's out and stays in during events, where each cart (up to 3) instead adds $2 per fan to event concessions. Happiness tips suggest one for 6+ hole courses without food on the course. Test: one cart served 15 golfers and sold $199 in a day on a 4-hole course with ~20 golfers.

### Gallery ropes, grandstands and the stadium hole
- **Gallery ropes** go up automatically when an event starts and come down when it ends (`updateRopes`, keyed on `S.event` + hole layout): yellow ropes on white stakes ~1.9 tiles either side of each hole's route (starting past the tee, stopping before the green) and a ring ~2.4 tiles around each green, open toward the approach. Skipped over water, greens, tees, paths, rocks and buildings. Instanced meshes. Fans stand outside the ropes.
- **🏟️ Grandstand** (Landscape → Landmarks, 3×2, $9,000, $20/day; kind scenery but `stand:true`, so no beauty bonus): a 4-row bleacher with a roof that turns to face the nearest green (`standYaw`). During events it fills with fans (~85% of 36 seats drawn, `standSeats`) and sells premium seats: fans × 0.7 fill stands up to 260 each, at 1.6× the ticket price, or 2.5× for stands within 7 tiles of the stadium hole's green (`standRevenue`). Each grandstand adds 7% more fans (up to 6), and a stadium hole with stands adds another 15%. Setup screens and results show seats filled and seat money.
- **🏟️ Stadium hole** (✏️ Edit hole → Stadium hole): make one hole the stadium hole (`h.stadium`, saved), and **Build the stadium** auto-places up to 3 grandstands around its green on clear rough (not fairway or paths) behind and beside it (`stadiumSpots`, `buildStadium`). A quarter of the course gallery gathers there. When you play your pro in an event, a birdie there makes "🏟️ The stadium ERUPTS!", a bogey gets heard about.
- In testing (3-hole course, 3.5★): one stadium of 3 stands took an amateur event from ~$4,000 to ~$13,700 (384 fans, 269 seats, $8,743 in seats), so stands pay for themselves in about 3 events.

### Save fixes (v43)
v41/v42 saved bridge spans under the same key as the bridge tile map (`bridges`), which could break bridges after a reload. Spans now save as `bridgeSpans`; loading recovers spans saved the old way and rebuilds the tiles. Holes now also save and undo `gshape`, `grot` and `stadium`; undo keeps bridge spans.

### Rivals
Members tab → **⚔️ Rivals** (top 8 members who've played your pro, ranked by wins minus losses): face, W-L-H from their side, last result and day, streaks of 2+ (🔥 wins / 🧊 losses), and a **Rematch** button. Match results now also store `vsPro.last`, `vsPro.day` and `vsPro.streak`. After a match the scorecard's Play again button becomes **⚔️ Rematch**.

### Bridge styles and cart bridges
Grass & paths tray → with the Bridge tool selected, a **Style** button cycles 🪵 Wood ($80/water tile), 🪨 Stone ($140), 🏙️ Modern ($120) (`BSTYLES`, `S.bridgeStyle`, saved per bridge as `b.style`). All are cart-width (0.74 tile) with path-colored ramps at both ends that run into your cart paths. Wood: plank deck, rails and posts. Stone: thicker pale paved deck, solid side walls and corner pillars, higher arch. Modern: concrete deck, thin steel rails, white side arches with hangers.

### Water, cart access, sorting, saving
- Water landings (ponds and creeks) were already a penalty + drop at the edge; now they also show a **splash** (rings and droplets, `splashAt`/`drawFx`) for golfers and in your own rounds, with a "💦 Splash!" message.
- ✏️ Edit hole shows **🛺 Cart access**: how far the nearest path is from the tee and the green (must be within 3 tiles) and whether those paths connect.
- Members tab: **Sort** by Newest, A–Z, 😀 Happiest, 😠 Unhappiest, 🎯 Best golfers.
- **💾 Quick save** button (top of the zoom buttons) saves now and uploads to the cloud right away if signed in; a small message confirms ("☁️ Saved to the cloud ✓", or saved on this phone). The game now also saves whenever you leave or switch away from the app, and every 30 seconds after building changes. A small "☁️ Saved ✓" appears after automatic cloud uploads.

### Club attention, event value, member cap
- The red number on 🏆 Club counts **attention items** (`clubAttention`): invitations waiting, crew short-handed, equipment under 40%, unhappy members (<45 mood), dues far above fair. The Club tab opens with a red **Needs your attention** box (each item has a button that jumps to the right tab), and each tab shows its own red count.
- Every event card (invitations, club events, team cups) and every setup screen shows an **estimate** (`eventMath`): how long it runs (field × 1.6 s + holes × your seconds per hole), your normal daily public income (green fees + food + shop + range + carts, from last week), what closing costs, and the estimated net plus prestige. Green if it pays, red if it costs.
- Member cap = 11 per hole, max 200 (`memberCap`), explained in the Members tab.

### Cart paths
A spot has cart access within ~3 tiles of a cart path or bridge (`NEAR`, radius 3). Carts now **drive along the path** (`cartRoute`: BFS over path + bridge tiles, cached until the course changes): from the ball to the nearest path tile, along the path, then off to the next ball. A hole only counts as having cart access (and stops the "Wish I had a cart path" / "No cart path" complaints) when its tee and green are both near the **same connected** path network.

### Hole editor: green shape
✏️ Edit hole → **Green shape** (shape buttons with pictures) reshapes that hole's green around the pin (`reshapeGreen`), automatically turned to face the tee; **↻ Turn it** rotates 45°. New tiles cost $60; old green outside the new shape becomes fairway. Saved on the hole as `gshape`/`grot`.

### Build menus
🏗️ Build → Holes (Hole, Edit hole, Dogleg, Clubhouse, 🚩 All pins) · Grass & paths (Fairway, Green, Cart path, Bridge) · Bunkers (4 styles) · Green shapes · Facilities · Upgrades (badge = affordable) · Buy land. 🌳 Landscape → Rough · Trees · Plants (Shrubs, Flowers) · Water (Creek, Pond) · Hills (Raise, Lower) · Landmarks, plus Clear. Opening a top-level menu selects the Move tool so nothing gets painted by accident.

### Golfer happiness tips (📋 Golfers → 💬 Feedback, top; also a button in the Rating sheet)
`happyTips()` checks the course and lists up to 7 prioritized fixes: fee too high for the rating (fair ≈ 12 + stars×10 + holes×2), condition/crew, no food/drink, no restroom, no benches, long walks between holes (named), holes without cart paths, plain holes and holes needing fairway (named), slow pace, bare scenery, the unhappiest golfer type and what it likes, and repeated complaints.

### Kilbrae Old Links (LEVELS 11, `links:true`)
Scotland/Ireland-style seaside links: few trees, fescue, many pot bunkers, rocks, hilly, olive-gold grass. Weather leans wet (≈16% sunny, 36% cloudy, 24% windy, 24% rain) and cool (46–64°). A flock of 12 sheep grazes the rough (`SHEEP_LV`, kind `sheep`: wander slowly, shuffle away from golfers); golfers say "Watch out for the sheep!" (Wildlife praise). $28,000 land + $22,000 build budget.

### Upgrade visuals
Each facility level adds real detail (`facModel`): range – extra bays and a ball dispenser, then a covered roof, floodlights and another target; pro shop – awning and displays, then a second floor; restaurant – more patio umbrellas, then an upper floor; snack/drinks – umbrella table and cooler, then more seating; putting green – extra flags, then benches and lights; cart barn – more parked carts, then an extension; bench – shade umbrella, then a pergola; restroom – planters, then a bigger roof and lamp.

### Tee boxes
Tee tiles are now flat and mown-fairway colored, and each hole draws one long raised tee strip (from just behind the back tee to the farthest forward set; short holes get a small one) with up to three marker sets (`teeSpots`): back = the hole's tee tile (blue), middle ~0.75–0.85 tiles toward the pin/bend (white), forward ~1.4–1.7 tiles (red); short holes get fewer, and a set is skipped if it would sit on water, trees, rocks, a building or a green. Golfers pick a set (`pickTeeSet`): tour/event players and your pro play the back tees, grinders and strong players (skill ≥ .78) the back, seniors and beginners (skill < .4) the forward tees, everyone else the middle (`setGolferTee` on arrival, each new hole, edits). Listed yardage is from the back tees.

### Pin positions
Hole editor → **Pin position on the green**: Front / Middle / Back (relative to the tee) / 🎲 Random, free; 🏗️ Build → Holes → **🚩 All pins** does the same for every hole at once (`openPinsAll`), using the connected green tiles around the pin (`greenTilesOf`, `setPin`). **Change every pin each morning** (`S.pinDaily`, saved) moves every pin to a random spot on its green at the start of each day (not during events or cups).

### Shaped greens (🏗️ Build → 🟢 Green shapes ›)
Tool buttons show a picture of the actual shape (`shapeSVG`, follows the current rotation). Stamp tools `gs_oval`, `gs_round`, `gs_kidney`, `gs_peanut`, `gs_star`, `gs_heart`, `gs_lshape` (`GSHAPES`, ~10–23 tiles each within a 9×9 area), $60 a tile, owned land only, skipping water, rocks, buildings, tees and existing green. ↻ Rotate turns the stamp in 45° steps (`S.shapeRot`). One undo step per stamp. The smooth terrain shader rounds the tile edges.

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
