# Icons

Weight, state, size and direction.

## Stroke weight tracks the adjacent text

| Adjacent text | Stroke on a 24px grid |
|---|---|
| Regular 400, 14 to 16px | 1.5px |
| Medium or semibold, 500 to 600 | 2px |
| Bold 700, or emphasised standalone | 2.5px |

An icon beside text carries that text's optical weight, and a hairline glyph next to bold
text reads as a rendering fault rather than a style.

One stroke weight per icon set on a surface, and never two icon libraries on one surface.
Two sets will disagree about grid size, stroke, corner radius and optical padding, and no
amount of per-icon adjustment reconciles them.

Size an inline icon against the text's cap height, roughly 1em to 1.25em.

## One asset, recoloured per state

Draw with `currentColor` and let colour and opacity carry hover, selected and disabled.
Strip hardcoded fills and strokes when importing an icon.

```html
<svg fill="none" stroke="currentColor" stroke-width="2">...</svg>
```

Separate assets per state drift apart: someone updates the default and not the hover, and
the icon changes shape slightly on hover for the rest of its life.

## Outline by default, fill for active

| Variant | For |
|---|---|
| Outline | The default: toolbars, list rows, inline with text |
| Fill | Selected or active: the current tab, a saved bookmark, a liked heart |

Cross-fade between the two variants rather than swapping instantly, with the values in
`motion.md`.

## Design at render size

Check every icon at the smallest size it will actually appear, which is often 16px. A
detailed glyph scaled down turns to mud, and the fix is a simplified glyph for small
sizes, not a sharper render.

Use the set's native grid sizes, 16, 20, 24, rather than arbitrary fractional scales. SVG
rather than a raster asset.

## Optical centring

An icon's geometric centre is rarely its visual centre. A play triangle sits left of
centre and needs shifting right. Stars, arrows and carets each carry their weight
unevenly.

Fix it in the SVG's viewBox or path so every consumer gets it right. A margin at one call
site fixes one place and leaves the rest wrong.

## Right to left

| Flip | Leave alone |
|---|---|
| Back and forward arrows, navigation chevrons | Logos and brand marks |
| Indent and text-alignment controls | Checkmarks |
| Directional send glyphs | Clocks, cups, pencils |
| Speaker waves tied to reading direction | Media playback controls |

Analyse a composite icon part by part: an overlaid badge often keeps its corner while the
base glyph mirrors.

## Names

Every icon-only control needs an accessible name, and the name is the action rather than
the picture: "Delete project", never "Trash icon". A purely decorative icon is hidden from
assistive technology instead.
