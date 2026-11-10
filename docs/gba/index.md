# gba-sdk

gba-sdk is a Game Boy Advance SDK for Rust. A game is a `no_std` crate built for `thumbv4t-none-eabi`; the SDK supplies the startup code, the linker script, typed access to the hardware and an API on top of it.

```rust
#![no_std]
#![no_main]

use gba_sdk::entry;
use gba_sdk::input::{Button, Buttons};
use gba_sdk::video::{wait_for_vblank, Color, Mode3};

entry!(main);

fn main() -> ! {
    let mut screen = Mode3::enable();
    let mut shown = None;
    loop {
        let color = if Buttons::read().held(Button::A) {
            Color::GREEN
        } else {
            Color::rgb(4, 8, 20)
        };
        if shown != Some(color) {
            screen.clear(color);
            shown = Some(color);
        }
        wait_for_vblank();
    }
}
```

## What it gives you

- **A ROM that boots.** [The runtime](runtime.md) holds the header, startup and linker script, so a game is a function. `entry!` names it.
- **Typed hardware access.** Registers are `Reg<u16>` or `Reg<u32>` with volatile reads and writes, never references, so the compiler can't reorder or elide them. Byte writes, which VRAM, palette RAM and OAM mishandle, don't exist for those regions.
- **An API that grows by hardware area.** [Video](video.md) and [input](input.md) today. The [roadmap](roadmap.md) builds toward something like a small raylib: a frame loop, sprites, tiled backgrounds, text, converted assets, fixed-point math and collision, then sound and saves.
- **[Logging](logging.md) through mGBA**, and panics that land in the same log.
- **Two backends, one source.** The native backend is `rustc`. A second one through [mrustc](https://github.com/thepowersgang/mrustc), which emits C for `arm-none-eabi-gcc`, is planned; see [Backends](backends.md).

## Status

gba-sdk is at its first milestone: the example above builds, boots in mGBA and reacts to a button. The API covers mode 3 and the buttons. Everything else on the [roadmap](roadmap.md) is still to come, and nothing has been tested on hardware yet.
