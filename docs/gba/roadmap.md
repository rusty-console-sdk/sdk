# Roadmap

Finished items stay in their section with a check mark, so each section shows how far along it is. The milestones are in order, and each one ends with ROMs that pass on every backend in play. Native is always in play. mrustc joins once the spike says go, and after a no-go it stays experimental and drops out of every "both backends" item below. Native never waits on mrustc, so the spike's open items run alongside milestone 1 instead of holding it up.

The goal is an SDK that feels like a small [raylib](https://www.raylib.com): a frame loop, calls like `draw_sprite` and `draw_text` every frame, art loaded from converted files, and math types for movement and collision. Those calls go to the sprite and tile hardware, not to pixels, because clearing the mode 3 frame buffer alone costs about half a frame before anything is drawn. The game writes a copy of sprite memory (OAM) in RAM, and the frame loop copies it to the hardware during the vertical blank.

## Milestone 0: the spike

- [x] Startup code, linker script and a ROM that boots in mGBA.
- [x] Mode 3, the buttons, `wait_for_vblank`.
- [x] `log!` and panics through mGBA's debug registers.
- [x] `just` recipes for check, build and test, headless mGBA, remote builds.
- [x] This site.
- [ ] mrustc: build `core` 1.90 for a custom target and boot the hello ROM through `arm-none-eabi-gcc`.
- [ ] Record ROM size and frame cost for both backends, and decide go or no-go on mrustc.

## Milestone 1: foundations

- [ ] A test runner over libmgba and test ROMs under `tests/`, every test on both backends.
- [ ] Interrupts: the trampoline in `rt`, `VBlankIntrWait`, `wait_for_vblank` that sleeps.
- [ ] DMA, including a fill for `Mode3::clear`.
- [ ] A frame loop: ending a frame sleeps until the vertical blank, then runs the copies queued for it. A frame counter for timing and animation. The copies for sprites, palettes and scrolling come with them in milestone 2.
- [ ] Button edges: pressed and released this frame, and key repeat for menus.
- [ ] Fixed-point math: an `Fx` type with the usual operators, with host tests run from the root by a new `just gba::test-host` recipe.
- [ ] `log!` that is safe to interrupt: mGBA's string buffer is shared, so a handler that logs would corrupt a line in progress.
- [ ] Panics on screen as well as in the log.
- [ ] Memory functions in `rt` as assembly, shared by both backends.
- [ ] A test that every `.iwram` function really is in IWRAM.
- [ ] CI: `just gba::check` and `just gba::test` on every push.

## Milestone 2: graphics

- [ ] Sprites drawn each frame into the RAM copy of OAM, which the frame loop copies over, with flipping, priority, and affine rotation and scaling. Frame sequences for animation.
- [ ] Tiled backgrounds with scrolling through the frame loop. Regular backgrounds hold at most 64 by 64 tiles and affine ones 128 by 128, so bigger maps stream in as the view moves.
- [ ] Palettes, with changes queued for the frame loop.
- [ ] A VRAM allocator for tiles and palette banks, so loading art returns a typed handle and games never pick tile indices by hand.
- [ ] Text: a built-in 8 by 8 font on its own background layer, `draw_text`, and a writer that works with `write!`.
- [ ] The asset converter: PNG images to tiles and palettes, and maps from the Tiled editor to tilemaps. A `macro_rules!` macro wraps the output of `include_bytes!` in a typed handle inside a `#[repr(align(4))]` wrapper, since DMA and the halfword and word copies to VRAM need aligned sources.
- [ ] Frame comparisons against goldens.
- [ ] A small game in `gba/examples/` (Breakout or similar) with sprites, a background, text and input, to try the API out while it is still cheap to change.

## Milestone 3: game loop essentials

- [ ] `Vec2`, `Rect` and collision checks: rectangle against rectangle, point against rectangle, circle against rectangle.
- [ ] A sine table, an angle type and a small, fast random number generator.
- [ ] A 2D camera: one offset applied to background scrolling and sprite positions together.
- [ ] Mode 4 with page flipping, and shape drawing (rectangles, lines, circles) in IWRAM for prototypes and debug overlays.
- [ ] Windows and blending, with screen fades.
- [ ] Timers.
- [ ] An optional allocator over EWRAM.

## Milestone 4: sound and saves

- [ ] Sound effects on the PSG channels: tones and noise.
- [ ] Direct Sound with a mixer in IWRAM, and WAV files in the asset converter.
- [ ] Music: a small tracker format or a MOD player.
- [ ] Saves: SRAM, then flash and EEPROM.

## Milestone 5: a real game

- [ ] A small complete game in `gba/examples/`, built on both backends.
- [ ] `cargo gba-sdk`, so a game outside this repository builds the same way.
- [ ] Publish `gba-sdk` on crates.io.
- [ ] A run on hardware with a flash cart for everything timing-sensitive.
