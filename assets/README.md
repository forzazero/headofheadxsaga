# Art assets (optional, drop-in)

Put PNG files here and restart the server — each one replaces its procedural
placeholder. Missing files fall back automatically, so art can land one piece
at a time.

| File | Size | Format |
|---|---|---|
| `character.png` | 64×72 | Sprite sheet: 4 columns (walk frames: contact, pass, contact, pass) × 3 rows (facing **down**, **up**, **side**). Each cell is 16×24. The side row faces **right**; left is mirrored automatically. **Grayscale pixels (R≈G≈B) get tinted with the player's palette color** (luminance preserved) — paint the body gray to make it recolorable, use real colors for parts that must not tint. Transparent background. |
| `floor.png` | square (16–64 px) | One floor tile, repeated per cell. A subtle checker is overlaid automatically. |
| `brick.png` | square | Texture for destructible bricks (box faces). |
| `solid.png` | square | Texture for border walls and pillars. |
| `powerups.png` | 512×512 | 8×8 atlas of 64×64 icons, row-major in `Powerup` enum order: extra bomb, flame, gold flame, skate, kick, punch, grab, jelly / trigger, spooge, skull, bomb-down, fire-down, speed-down, pierce, heart / egg, mine, homing, bombthrough, wallthrough, fireball (remaining cells empty). Background is chroma-key magenta (`#ff00ff`) — the renderer punches it to transparent. |

## Shipped art, not drop-in

`limestone.jpg`, `limestone-normal.png`, `limestone-height.png` (64² each) are
the temple's destructible block — the one texture in the renderer that is scanned
rather than procedural (docs/3DASSETS.txt §1.2). They are not overrides: nothing
generated looks like them, and the piece falls back to its procedural coursed
masonry only if a file is missing.

**All three or nothing.** A photographic albedo with no normal map is a flat
photo; a normal map with no albedo is grey plastic. `loadStoneMaps()` treats an
incomplete set as absent.

The suffixes mean what they say, which the source files did not: `-normal` is the
tangent-space normal map (mean RGB ≈ 128/128/205), `-height` is greyscale. Check
a replacement the same way before dropping it in — the reviewer
(`./start.sh --review`, temple → brick) shows each channel on its own.

`classic-crate-1.png`, `classic-crate-2.png`, `classic-crate-3.png` (64² each)
are the classic arena's destructible crates (docs/3DASSETS.txt §4.4) — one whole
face of the block's box each, drawn as-is with no tiling and no baked maps. The
cell hash picks one per block, so a wall of destructibles is a stack of different
crates instead of one repeated tile.

**All three or nothing.** Two crates and a procedural placeholder mixed into the
same wall reads as a bug, not as degraded art, so `loadCrates()` treats an
incomplete set as absent and every block falls back to the procedural wood crate.

A replacement only has to be a 64² square face; it is magnified with
NearestFilter (pixel art stays crisp) and minified through the mip chain, so keep
the pixel grid honest and avoid a border that would look like a seam when four
crates sit side by side. The reviewer (`./start.sh --review`, classic → brick)
opens each one on its own.

`kick-burst.png` (64²) is the comic POW drawn where a bomb starts sliding
(docs/3DASSETS.txt §3.1). A GREYSCALE MASK on transparency, not finished art: the
renderer tints it amber for an ordinary kick and hot orange for a fireball one,
and the core/band/rim ramp baked into the greys (255 / 190 / 130) is what becomes
a lit centre with a darker inked edge under either tint — the same
luminance-preserving trick `character.png` uses for player colour.

It is the one file here with NO procedural fallback, which is deliberate:
everything else replaces something the client can draw for itself, so a miss is
degraded art. This is decoration over an event the audio already announces, so a
miss is simply no burst — nothing to be all-or-nothing about, and nothing that
has to be implemented twice and kept in agreement.

Do not hand-edit it. `bun packages/tools/src/paint-kick-burst.ts` writes it and
`--check` verifies the committed file still matches the code that claims to have
produced it; `packages/client/test/kickfx.test.ts` runs that check in the suite.
Change the drawing, re-run the painter, commit both.

`dust-puff.png` (128², four 64² silhouettes in a 2×2 atlas) is the dust a
hurry-up wall throws when it lands (docs/3DASSETS.txt §3.2). A GREYSCALE MASK on
transparency like the kick burst, and tinted the same way — but its greys carry a
LIGHT rather than an ink ramp: a dome lit from the upper-left, 150 in its own
shadow up to 255 on the lit shoulder. The shader multiplies tint by ramp, so a
puff reads as a lump of dust; tint alone would be a flat sticker.

Four silhouettes because a drop sheds nineteen particles at once and the eye
picks a repeat out of nineteen identical stamps immediately. The shader also
rolls each particle by its own angle, so four drawings cover a burst.

Missing, a wall simply lands without dust — the block still falls and its shadow
still gathers, so this is decoration over an event the collapse already
announces. Do not hand-edit it: `bun packages/tools/src/paint-dust-puff.ts`
writes it and `--check` verifies the committed file still matches the code that
claims to have produced it; `packages/client/test/blockdrop.test.ts` runs that
check in the suite. Change the drawing, re-run the painter, commit both.

`locked.png` (64², straight RGBA — no mask, no tint) is the lock-on reticle,
drawn over the head of whoever a wingbomb has locked onto (§3.4.3). It is sized
to 80% of the 1.15 billboard and drawn with depth testing off, so it is always
readable and never quite covers the player it is identifying — the answer it
gives is "who", and a mark that hid its own target would destroy it.

The second file here with no procedural fallback, and unlike the kick burst that
is not comfortable: a miss costs INFORMATION rather than decoration. A homing
bomb walks at its prey for seconds before it goes off, and this is the only
thing on screen that says who it is walking at.

More than one hunter on the same player is still ONE reticle — two crosshairs on
one head read as neither — with a small red count at the bottom left, composited
onto the sprite at runtime (`game/lockon.ts`, render3d `lockTexture`). Replace
the art freely; keep the middle open enough to see a face through, and leave the
bottom-left corner clear for the badge.

`classic-solid.png` (64²) is the classic arena's indestructible wall — a riveted
metal panel worn on every face of the block's box (docs/3DASSETS.txt §4.5),
drawn as-is with no tiling and no baked maps, filtered exactly like the crates.

**One panel, repeated on purpose.** The walls are the arena's frame — the border
ring and the pillar lattice — and a frame reads as one structure only if it
repeats; variety there would compete with the destructibles, which are what a
player actually has to scan. So there is no variant set here and nothing to be
all-or-nothing about: missing, every wall falls back to the procedural red brick
and the reviewer (classic → solid) says so.

`boss-metal-normal.png` (128²) is the boss arena's corrugated-steel RELIEF —
the tangent-space normal map baked from `Horror_Metal_06-128x128`
(docs/3DASSETS.txt §4.2). It ships alone, with no albedo beside it, because the
paint is generated from it: four rusted panels per arena, all sharing this one
map, so the ribs land in the same place on every block while no two blocks rust
alike.

Replacing it re-skins the whole boss arena, and a replacement only has to be a
square tangent-space normal map of something ribbed — the paint follows whatever
relief it is given. It is point-sampled down to 64² on the way in (PANEL_PX in
`src/metal.ts`), which is the size a panel is actually painted at, so detail
finer than every other texel of this map is detail the arena never sees. Missing,
the theme synthesises a plain 14-rib corrugation instead and the reviewer (boss →
brick) says so. Check what a candidate does to the paint before committing to it:

```
bun packages/tools/src/preview-metal.ts --out /tmp/metal.png
```

## Generated, not drop-in

`jerseys.png` + `jerseys.json` are **build artifacts** — do not hand-edit them.
They are produced from the kit reference art in `sprites/country/` by:

```
bun packages/tools/src/bake-jerseys.ts [--verbose]
```

The atlas holds three 8×8 cells per kit (front, back, side) and the manifest
carries each kit's slug, country, and the sampled colors the compositor paints
arms/legs with. Re-run the bake after changing `sprites/country/index.json` or
any source PNG. If both files are absent the game simply has no jerseys and
every character falls back to a palette body.

Notes:
- Uploaded custom heads still composite on top of `character.png`'s head area.
- Hats (cap/crown/halo) still draw procedurally over any head.
- Keep pixel art crisp: everything is rendered with nearest-neighbor scaling.

## Baked art, from the model forge

`<slug>-albedo.png`, `<slug>-normal.png`, `<slug>-height.png` are a map set from
`bun packages/tools/src/model-forge.ts` (docs/3DASSETS.txt §8). The MODEL path
writes them beside a piece module in `packages/client/src/pieces/` and the two
are regenerated together — editing one without re-running the forge puts it out
of step with a geometry that has no way to say so. The TEXTURE path (§8.4) writes
the three on their own, for a piece that already has its geometry in code.

- `boss-solid-*` (64², from `assets/horror/po/Misc/Horror_Misc_14-128x128.png`)
  is the boss arena's indestructible wall: a cast filigree screen, forged through
  the TEXTURE path, so there is no piece module beside it — a theme draws it
  (docs/3DASSETS.txt §4.3). Forged at half the source's edge, to the same 64² the
  panels next to it are painted at; re-forging at another `--size` means moving
  `--strength` the opposite way, for the reason §4.3 gives. Only albedo and
  normal are drawn; the height is fetched with them because the set is
  indivisible, and shown by the reviewer.
  Missing, the wall falls back to the procedural glass box.

- `coal-cart-*` (1024², from `assets/coal_cart/scene.gltf`) is the coal cart
  (docs/3DASSETS.txt §8.7), the one piece-module set in the tree, and the one set
  here that is NOT baked: the albedo and normal are the artist's own maps, resized
  and (for the normal) renormalised, and the piece carries the artist's uv map to
  go with them (§8.10). `packages/client/src/pieces/coal-cart.ts` is its other half
  and the two only make sense together. There is NO `-height.png`: a glTF's third
  PBR map is metalness/roughness/occlusion, none of which is displacement, so the
  set is two files and no bumpMap is bound. `loadPieceMaps()` needs the albedo and
  the normal; the height is optional.
  Both maps are uploaded with `flipY: false`, because a glTF's uv origin is the
  TOP-LEFT while three's default is the bottom — mismatch it and every uv island
  samples another island, which shipped once and looked like the wheels had been
  painted with the bucket's plate (§8.10).
  The source's metallic-roughness map is deliberately unused: no environment map in
  this renderer means metalness renders black (§8.10).
  Missing, the cart draws in the vertex colours in its module, sampled from that
  same albedo, so the fallback is a rusty cart rather than a white one.
  A PROP, and a CANDIDATE: 5,307 triangles — the artist's geometry minus the 365
  faces no camera can reach (§8.9) — which is legal for scenery drawn once
  (`--target prop`, docs/3DASSETS.txt §6) and 26× what a tile may cost. Nothing in
  `THEMES` draws it, and dropping it into a solid/brick slot would put it in a
  600-instance pool; the reviewer's label says `prop` for that reason.
  These two maps are 4.2 MB together, which is affordable only because a
  candidate's maps are fetched by the reviewer alone — as is the 186 KiB module,
  which is in the reviewer's bundle and not the game's (§8.10, measured). Re-forge
  at `--map 512` or less before any theme draws this.

`old-pillar-*` and `greek-pillar-*` were candidates too, and were removed along
with their modules rather than left to be paid for. The forge itself is untouched —
re-forge from `assets/old_pillar/scene.gltf` with the command line in
docs/3DASSETS.txt §8.6 to get that one back; `assets/greek_pillar` has since been
deleted from the tree, so §8.5's example needs its source downloading again. What
old-pillar was actually good for survives in the hand-built temple columns, whose
proportions and drum joints are measured off it (docs/3DASSETS.txt §8.6).

**All three or nothing**, for the same reason as the limestone — `loadPieceMaps()`
treats an incomplete set as absent. What happens next depends on the piece: the
boss wall drops to its procedural glass, and a forged piece, having no procedural
stone under it, drops to the vertex colours baked into its own module. The
reviewer says which one you are looking at either way.
