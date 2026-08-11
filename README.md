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

The world is **shaped to your screen** rather than letterboxed into a fixed 16:9 box, so a wide phone
gets a wider board instead of black bars — and the tiles grow to match. On a 2:1 phone that is ~40
CSS px between tile centres, against ~30 on a 16:9 one.

| | Touch | Desktop |
|---|---|---|
| Lay a brick | tap where you want it | click where you want it |
| Lay a run quickly | drag across the ground | drag across the ground |
| Pick which half of a tile | tap that side of it | click that side of it |
| Choose a tool | palette on the right | palette, or `1`–`5` |
| Repair / Raze | palette | `R` / right-click |
| See through walls | PEEK button | `V` |
| Pause | PAUSE button | `Space` |

The brick goes exactly where you touch — no offset, no confirm step. You aim at a **ground tile**
and bricks stack automatically, so you never aim at a height. Which of the tile's two slots you get
is decided by which side of the tile centre you touched.

**Drag to lay a run.** Every slot the drag crosses gets built, with no cooldown between them, and
the path between move events is filled in — so a fast swipe leaves a solid line rather than a dotted
one. Gold is the only limit. This works the same on touch and mouse.

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
them**, side by side. That is 360 buildable slots on a 14x14 board.

It also gives you a choice of thickness. One brick in a tile is a thin, cheap wall; it still blocks
the tile completely. Fill both slots and you have a proper thick wall with twice the stone to chew
through. Height is separate again: four courses is what stops climbers, whichever thickness you
built.

Bricks always turn their long face toward the keep, so walls coursed around it run with the grain.
Exact facing is impossible on any square grid — a tile straight out from the keep would want a brick
at 45 degrees — so it snaps to the nearer axis. The board therefore changes coursing along the
diagonal running out from the keep, the way real brickwork turns a corner.

Three more rules do the rest:

1. **Support.** Bricks stack; you cannot float them.
2. **Collapse.** Break the bottom brick and everything above falls, taking damage on landing — so a
   tall stack can cascade.
3. **Curing.** A fresh brick starts at a quarter strength and takes ~2.5s to set. You cannot patch a
   breach instantly; you have to buy the time first.

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
- `CONFIG.board.planX` / `planY` / `uw` / `uh` / `zh` — board size and tile geometry, all coupled.
  Both axes are at their limit at 14×14: width is `planX × uw` against the palette, height is
  `planY × uh` plus a full stack of clearance above the far corner. Growing the board means shrinking
  the tile, and tile spacing is what a thumb needs — 23.5 CSS px here, which is why touch commits on
  release.
- `CONFIG.board.keepPlan` — the keep's footprint in plan units. A difficulty dial in disguise: a
  bigger keep has more tiles touching it, which multiplies both the wall you must maintain and the
  number of attackers that can reach it at once. Change it and `economy.income` has to follow.
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
  Tile spacing is 30-40 CSS px depending on screen shape — under the 44 px touch guideline on narrow
  phones, but close to it on wide ones.
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
