# lite-xl (fork)

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Workflow

When the user asks to commit/push, check whether CLAUDE.md or README.md need updates to reflect the changes made in the session. Update them in the same commit if so. Only update these files at commit time, not during development.

## Project

Personal fork of [lite-xl](https://github.com/lite-xl/lite-xl) with fractional display scaling support for Wayland/HiDPI.

## Fork changes

Three commits on top of upstream master:

1. **Fix integer overflow in ren_draw_text glyph positioning** - `GlyphMetric.y0` is `unsigned int`, which causes C unsigned promotion in the `target_y` arithmetic. With float `surface_scale`, the promotion chain wraps negative intermediates to ~4 billion, overflowing to `INT_MIN` and clipping the top of all rendered text. Fix: cast to `int` before arithmetic.

2. **Fractional display scaling support** - Change `RenSurface.scale` and `RenFont.scale` from `int` to `float` throughout the rendering pipeline. Remove the integer-only assertion in `query_surface_scale`. Fall back to `SDL_GetWindowDisplayScale` when pixel/point sizes are equal (Wayland fractional scaling). Round font pixel sizes instead of truncating.

3. **Auto-disable subpixel antialiasing at fractional scales** - Subpixel (LCD) rendering produces color fringing at non-integer scales because the RGB subpixel grid doesn't align. Store the user's requested antialiasing mode and automatically fall back to grayscale when the scale is fractional. Restore subpixel if the scale becomes integer.

## Building

Requires `-Drenderer=true` for scaling support (off by default on Linux in upstream):

```
meson setup build -Drenderer=true -Dportable=true
ninja -C build
```

For portable builds, symlink the data directory:

```
ln -sfn ../../data build/src/data
```

Then run: `./build/src/lite-xl`

## Syncing with upstream

```
git fetch origin
git merge origin/master
```

The `origin` remote points to `lite-xl/lite-xl`. The `fork` remote points to this repo on GitHub.

## Key files

- `src/renderer.c` - Font rendering, text drawing, glyph positioning. The integer overflow fix and subpixel fallback live here.
- `src/renderer.h` - `RenSurface` struct (scale field changed to float).
- `src/renwindow.c` - Window surface creation, scale detection, clip rect scaling. The `query_surface_scale` and `scaled_rect` changes live here.
- `data/core/style.lua` - Default font sizes and antialiasing settings. Defaults to subpixel; the C code overrides to grayscale at fractional scales.
- `data/core/node.lua` - Tab bar layout and drawing.
- `data/plugins/scale.lua` - User-facing zoom (Ctrl+mousewheel). Independent of the display scale.

## Architecture notes

- The `LITE_USE_SDL_RENDERER` define (from `-Drenderer=true`) switches between two rendering paths. Without it, there is no HiDPI scaling support on Linux.
- `SCALE` (Lua global) controls UI/font sizing from env vars (`LITE_SCALE`, `GDK_SCALE`, `QT_SCALE_FACTOR`). This is independent of the C-level `surface_scale` which handles pixel density. Don't set both or you get double scaling.
- Font metrics (`height`, `baseline`) are computed from `font->size` (logical), not pixel size. The drawing code multiplies by `surface_scale` to get pixel positions. The rounding from `FT_Set_Pixel_Sizes` can cause small mismatches at fractional scales.
