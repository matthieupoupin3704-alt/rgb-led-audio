# Silkscreen artwork

Vector art for the F.Silkscreen layer, generated with `potrace` + PIL from raster sources.

## Main constraint

JLCPCB prints down to **0.15 mm** feature size. Anything finer is silently deleted during fabrication, so a halftone whose dots fall below that threshold comes back as a grey smudge rather than an image. Every file here is characterised against that limit.

| File | Paths | Min size | Verdict |
|------|-------|----------|---------|
| `stars1_solid.svg` | 8 | any | Solid shapes, zero risk. Prints at any scale. |
| `hands_clean.svg` | 23 | ~20 mm | Clean silhouettes, very legible. |
| `signature_v2.svg` | 40 | 40 mm | Name / project / version block, 52 × 20 mm as drawn. |
| `flower_D_medium.svg` | 100 | 30 mm | Granular texture, fragile at smaller sizes. |
| `marb_mid.svg` | 443 | 25 mm | Marbling, strong graphic result, large solid areas. |
| `shape_B_bold.svg` | 870 | 25 mm | Square halftone, dilated to clear the threshold. |
| `lily_halftone.svg` | 879 | 30 mm | **The only true halftone that survives.** Dots land at 0.25 mm. |

## Why most halftones fail

Re-screening an existing halftone at a printable dot pitch destroys the image: the source screen is much finer than 0.15 mm, so coarsening it enough to print erases the structure that made it readable. `lily_halftone.svg` works only because its source was already coarse.

For a halftone look, start from a coarse source; otherwise use solid shapes.

## Importing

`File → Import → Graphics`, target layer **F.Silkscreen**, respect the minimum size in the table.

Placing artwork under a component means it will not be visible on the finished board. Check the 3D view before ordering.

## What's on the current board

`lily_halftone.svg` plus the signature block. This is why `rgb-led-audio.kicad_pcb` is 35 MB: the halftone stores thousands of polygons directly in the board file. Swapping it for `stars1_solid.svg` would drop the file to roughly 2 MB.
