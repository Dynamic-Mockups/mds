# MockAnything Measurements — Coordinate System Reference

## Measurement Fields

| Field | Origin | Reference point | x+ | y+ |
|---|---|---|---|---|
| `printArea.uv.position` | Canvas bottom-left `(0,0)` | PA **center** | right | up |
| `printArea.uv.dimension` | — | width/height as fraction of canvas | — | — |
| `printArea.xy.position` | Canvas bottom-left `(0px,0px)` | PA **center** | right | up |
| `printArea.xy.dimension` | — | width/height in pixels | — | — |
| `artwork.uv.position.absolute` | Canvas bottom-left `(0,0)` | Artwork **center** | right | up |
| `artwork.uv.position.relative` | PA bottom-left corner `(0,0)` | Artwork **center** | right | up |
| `artwork.uv.dimension` | — | width/height as fraction of canvas | — | — |
| `artwork.xy.position.absolute` | Canvas bottom-left `(0px,0px)` | Artwork **center** | right | up |
| `artwork.xy.position.relative` | PA bottom-left corner `(0px,0px)` | Artwork **center** | right | up |
| `artwork.xy.dimension` | — | width/height in pixels | — | — |

## Sanity Check Boundaries

| Condition | Expected value |
|---|---|
| PA centered on canvas | `printArea.uv.position = {x:0.5, y:0.5}` |
| Artwork centered on PA | `artwork.uv.position.relative = {x: paW/2, y: paH/2}` |
| Artwork at PA top-right | `artwork.uv.position.relative ≈ {x: paW, y: paH}` |
| Artwork at canvas center | `artwork.uv.position.absolute = {x:0.5, y:0.5}` |

## Note on Warp

`artwork.uv` and `artwork.xy` values are **approximations** — they assume a flat surface.
With non-zero warp strength the 3D decal projection introduces deviation between the
visual position and the calculated UV value. Set warp to `0` for exact measurements.
