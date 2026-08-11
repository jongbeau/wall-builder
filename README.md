# Wall Builder

An isometric siege defence game, viewed from a distance. Two rival kingdoms send invaders swarming
across your ground, and you hold them off by building a brick wall — laid out across the field, not
just stacked up.

**▶ Play it: <https://jongbeau.github.io/wall-builder/>** — landscape, works on a phone.

No build step, no dependencies, no install. Or open `index.html` in a browser locally.

```bash
open index.html
```

Some browsers restrict `localStorage` on `file://`, which would drop your high score. To keep it:

```bash
python3 -m http.server 4173
```

Then visit <http://localhost:4173>.

Two versions live here:

| File | What it is |
|---|---|
| `index.html` | The isometric game. Build across ~200 tiles; invaders pathfind around your walls. |
| `classic.html` | The original side-view version. One lane, build upward only. Simpler and still fun. |

## Playing

**Landscape only** — the kingdom needs the width. On a phone, turn the device sideways (if nothing
happens, your rotation lock is on).

The world is **shaped to your screen** rather than letterboxed into a fixed 16:9 box, so nothing is
wasted on black bars and the tiles grow to fill whatever room there is. The board runs the full
height too: there is no sky and no HUD bar, just a readout and two buttons floating over the field's
far edge.

Measured on the usable area each phone actually gives a landscape web page, after the Dynamic Island
and home indicator take their cut:

| Phone | Usable box | Tiles | Tile spacing | Wasted |
|---|---|---|---|---|
| iPhone 13 mini | 712×354 | 210 | 32.6 px | none |
| iPhone SE | 667×375 | 199 | 33.0 px | none |
| iPhone 17 | 756×381 | 210 | 34.4 px | none |
| iPhone 17 Pro Max | 838×419 | 210 | 38.1 px | none |

Every phone gets roughly the same number of tiles; a bigger screen buys bigger tiles rather than
more of them, so the game plays the same everywhere. Palette buttons stay at or near the 44 px touch
target throughout (43.3 px at the very smallest, 51 px on a Pro Max).

The proportions are measured at load. A phone is normally held in portrait, so the page usually
loads that way and measures the screen stood up — which is the wrong shape. When you then rotate,
the game reloads once with the right measurement. It can only do that before a game starts, so
nothing is ever lost, and rotating mid-siege just pauses as usual. Consecutive reloads are capped in
case the measurement wobbles, and the cap resets as soon as the world fits.

Archer posts face away from the keep, since that is where their arrows go. The figure is drawn live
rather than baked into the sprite so it can be mirrored per tile.

One thing worth knowing about pointing: the pick follows what you can **see**. Clicking the raised
face of a wall targets that wall, not the ground it hides — about 9% of the board once a full-height
ring is up. The highlighted diamond always shows the tile you will actually build on.

| | Touch | Desktop |
|---|---|---|
| Lay a brick | tap where you want it | click where you want it |
| Lay a run quickly | drag across the ground | drag across the ground |
| Pick which half of a tile | tap that side of it | click that side of it |
| Choose a tool | palette on the right | palette, or `1`–`5` |
| Repair / Raze | palette | `R` / right-click |
| See through walls | Peek, in the palette | `V` |
| Pause | Pause, in the palette | `Space` |
| Mute | Sound, in the palette | `M` |

Everything you can press lives in the right-hand column, which sits outside the board — the rest of
the screen is playable ground.

The brick goes exactly where you touch — no offset, no confirm step. You aim at a **ground tile**
and bricks stack automatically, so you never aim at a height. Which of the tile's two slots you get
is decided by which side of the tile centre you touched.

**Drag to lay a run.** Every slot the drag crosses gets built, with no cooldown between them, and
the path between move events is filled in — so a fast swipe leaves a solid line rather than a dotted
one. Gold is the only limit. This works the same on touch and mouse.

## Sound

A patriotic march plays under the siege, and switches to something faster and darker while a surge
is on. Bricks give way with a crack and rubble; invaders go down with a dry impact; anything that
reaches the keep lands a low boom.

All of it is **synthesised at runtime** with the Web Audio API — no audio files, because the game is
a single self-contained page. The march is scheduled against the audio clock rather than the frame
loop, so the beat does not jitter when the frame rate does.

**It plays through the iPhone ring switch.** That needs `navigator.audioSession.type = "playback"`
(Safari 16.4+), and it is feature-detected — where it is unavailable, audio simply follows the
switch as usual.

Overriding that switch takes away the control you would normally use to silence a page, so the game
provides its own: **Sound**, at the bottom of the palette, or `M`. The choice is remembered. Audio
also cannot start until you touch the screen, which is a browser rule, not a choice.

## The idea

Invaders always take the cheapest route to your keep. That one rule is the whole game.

- **Leave a gap and they will find it.** They walk right past your wall and through the hole, without
  ever swinging at it. A wall with a gap is a corridor — which is useful, if it runs past your archers.
- **Seal them out and they smash the weakest tile instead.** With no open route they converge on
  whichever part of your wall is cheapest to break — the thinnest, most damaged, softest spot.
  A wall is only as strong as its worst tile.

So you are choosing between two strategies: an airtight ring that concentrates every attacker onto a
few tiles you must frantically repair, or a longer maze that keeps them walking in the open where
your archers can work.

Bricks are real bricks — a 2:1 rectangle covering half a tile — and **every tile holds two of
them**, side by side. That is ~370 buildable slots across roughly 200 tiles, and the exact numbers
shift a little with the shape of your screen.

It also gives you a choice of thickness. One brick in a tile is a thin, cheap wall; it still blocks
the tile completely. Fill both slots and you have a proper thick wall with twice the stone to chew
through. Height is separate again: four courses is what stops climbers, whichever thickness you
built.

Bricks always turn their long face toward the keep, so walls coursed around it run with the grain.
Exact facing is impossible on any square grid — a tile straight out from the keep would want a brick
at 45 degrees — so it snaps to the nearer axis. The board therefore changes coursing along the
diagonal running out from the keep, the way real brickwork turns a corner.

Two more rules do the rest:

1. **Support.** Bricks stack; you cannot float them.
2. **Collapse.** Break the bottom brick and everything above falls, taking damage on landing — so a
   tall stack can cascade.

Bricks are at full strength the moment they are laid. They used to cure over a couple of seconds,
but at this scale the only tell was a few pixels of tint, and an invisible rule is worse than no
rule. Cost and the build cooldown still stop you conjuring a fortress mid-breach.

Repairing is deliberately better value per gold than laying a fresh brick, so patching under
pressure is the rewarded play.

## Knowing your enemy

Invaders arrive from two far edges. A **third front opens at 4:00**, flanking from the side. They
are drawn small on purpose — at this distance you read the battle by where the crowds are, not by
individual soldiers.

Each type gets its own stretch of the siege before the next arrives:

| | Appears |
|---|---|
| Grunt | from the start |
| Climber | 1:00 |
| Sapper | 2:05 |
| Ram | 3:15 |
| Catapult | 4:40 |
| Warlord | 6:40 |

| Unit | What it does | What beats it |
|---|---|---|
| **Grunt** | Batters the base of a wall tile | Anything |
| **Climber** | Goes *over* anything 3 high or less, ignoring your maze entirely | Build **4 high** |
| **Sapper** | Detonates, gutting a 2×2 footprint two courses deep | Build **thick**; kill it on the approach |
| **Ram** | Slow, very tough, enormous damage | Depth, spikes, focused fire |
| **Catapult** | Halts out of reach and lobs rocks at your **tallest** stack | Kill it — or don't put archers on your high ground |
| **Warlord** | Speeds up and strengthens everything near it | Focused fire |

Note the tension: 4-high walls stop climbers, but tall stacks are exactly what catapults aim at.

Only **two invaders** can attack one tile at once — the rest queue. That is what makes a wall worth
building, and why a breach is sudden: the moment a tile falls, the queue pours through.

## About the camera

The view is fixed — no rotation, no zoom, so the whole board is always on screen. The cost is
inherent to that choice: **a tall stack hides the tile diagonally behind it.** No block height that
still reads as a wall avoids this (fully hiding that tile only needs `height × block ≥ tile depth`).

**PEEK** is the fix. It fades your walls and lets you build on the ground they cover, which takes the
number of unreachable tiles to zero in every layout tested.

## Tuning

Every balance number is in the `CONFIG`, `BRICKS` and `INVADERS` objects at the top of `index.html`.
Nothing is buried in game logic.

The dials that matter most:

- `CONFIG.path.wallBase` / `wallHpScale` — how much invaders hate breaking walls versus walking. Raise
  `wallBase` and they will detour further; lower it and they smash through sooner.
- `CONFIG.combat.frontage` — attackers per tile. The single most powerful lever on difficulty.
- `CONFIG.spawn.growth` / `exponent` — how fast the siege escalates. It ramps forever.
- `CONFIG.board.rect` — the screen area tiles must fall inside. The board is the set of tiles landing
  in this rectangle, not a diamond, which is the whole point: a diamond can only cover half its
  bounding box, so the screen corners were going to waste. Filling a rectangle instead buys ~50% more
  area, and therefore much bigger tiles for the same count. `y0` is solved for at load, from the HUD
  height plus a stack of clearance.
- `CONFIG.board.uw` / `uh` / `zh` — starting tile geometry only. `fitBoard()` overrides them at load
  with the **largest** tile that still yields ~200 tiles in the rectangle, so a wider screen spends
  its extra room on bigger tiles rather than more of them.
- `CONFIG.board.keepPlan` — the keep's footprint in tiles. A difficulty dial in disguise: a bigger
  keep has more tiles touching it, which multiplies both the wall you must maintain and the number of
  attackers that can reach it at once. Change it and `economy.income` has to follow. The building
  itself is drawn at a fraction of this (`PLOT` in `drawKeep`) with the remainder shown as grounds,
  so its size on screen can be tuned without moving the balance.
- `Sound.musicBus` / `sfxBus` gain — the music/effects balance, currently 0.34 against 0.85 so the
  march sits under the siege rather than over it.
- `CONFIG.unitScale` / `fxScale` — how small invaders and debris are drawn. Invaders are drawn at
  nominal proportions and scaled as a whole, so line weights shrink with them. Hitboxes keep a
  world-space floor so arrows still connect.
- `CONFIG.board.slotsPerTile` — bricks per tile. 1 makes each brick fill its tile as a slab; 2 gives
  real brick proportions and doubles the buildable positions. The tile stays the unit of pathing and
  collision either way.
- `CONFIG.brick.fill` — how much of its slot a brick covers. Short of 1 so a mortar joint shows.
- `CONFIG.spawn.*` and `INVADERS[type].unlock` — the pacing. `growth` and `exponent` set how fast
  the siege builds; `hpRampPer` and `combat.dmgRampPer` set how fast invaders inflate. Lower growth
  and longer ramp periods make a gentler game without ever capping the escalation.

For reference, a bot that seals the 8-tile ring and builds archer posts survives about **400
seconds** with 250–330 kills. That is the floor to beat.

## Debugging

`window.__game` is exposed:

```js
__game.audit()          // invariants: dead bricks, over-height stacks, stuck invaders, frontage
__game.fastForward(60)  // run 60s of simulation instantly
__game.routeOpen()      // is there still a walking route to the keep?
__game.computeFlow()    // the raw pathfinding field
```

## Known limits

- Verified in a desktop browser at phone-landscape sizes (667×375, 844×390, 956×440) with synthesized
  pointer events, and played on a real phone. Sustained framerate across devices is still unconfirmed.
  Tile spacing is 33-43 CSS px depending on screen shape — essentially at the 44 px touch guideline on
  a wide phone, a little under it on a narrow one.
- The ring-switch override is written against `navigator.audioSession` and feature-detected, but the
  browser used for testing does not implement that API, so **only a real iPhone can confirm music
  plays with the switch on.** Everything else about the audio was verified by measuring the output:
  the march reaches RMS 0.053, effects layer to 0.17, the surge mix is louder than the calm one, and
  muting drops output to exactly zero.
- The world's aspect is measured once at load. Rotating a phone shows the landscape prompt, and a
  desktop window resized to a very different shape will letterbox rather than rebuild the board,
  because rebuilding the grid mid-game would throw away the wall you built.
- The keep sits on the board's near corner, so eight tiles touch it. Sealing that ring is still the
  strongest opening; the rest of the board earns its place as archer ground and as maze space.
- Walls are deliberately flat at this distance (a full-height stack is only a little taller than a
  tile is deep). That keeps the camera's blind spot small, but it does make walls read more as low
  ramparts than as towers.
- Brick coursing changes along the diagonal that runs out from the keep, since that is where "which
  way is the keep" changes axis. Walls crossing that line visibly change direction. It reads as
  deliberate masonry, but it is a seam.
