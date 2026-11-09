# Surfaces

Radii, alignment, elevation, outlines and hit areas.

## Concentric radii

```
outerRadius = innerRadius + padding
```

A card with 8px of padding around a child with a 12px radius takes a 20px radius. Equal
radii on parent and child make the inner surface look pinched, and it is the most common
single cause of an interface feeling subtly off.

```css
/* right */
.card       { border-radius: 20px; padding: 8px; }
.card-inner { border-radius: 12px; }

/* wrong */
.card       { border-radius: 12px; padding: 8px; }
.card-inner { border-radius: 12px; }
```

Past roughly 24px of padding the two surfaces no longer read as nested, so choose each
radius independently instead of forcing the arithmetic.

A useful scale, where the project has none: 28 for a hero card, 20 for a card or the
inner block of a sheet, 16 for a strip or an input, and a stadium shape for pills and
primary buttons.

## Optical alignment

Geometric centring is not visual centring. When it looks wrong, it is wrong.

**Icon beside text in a button:** about 2px less padding on the icon side.

```css
.button-with-icon { padding-left: 16px; padding-right: 14px; }
```

**A play triangle:** shift it right by about 2px. Its geometric centre sits left of its
visual centre.

**Asymmetric glyphs**, stars, arrows, carets: fix the SVG's viewBox or path so the icon
is centred in its own box. Every consumer then gets it right for free, where a margin at
one call site fixes one place and leaves the rest.

## Shadows for elevation, borders for structure

A border that exists only to imply depth is better as a layered translucent shadow:
shadows adapt to whatever is behind them, solid borders do not, which is why a bordered
card looks wrong the moment it lands on an image or a tinted surface.

```css
:root {
  --shadow-border:
    0 0 0 1px oklch(0 0 0 / 0.06),
    0 1px 2px -1px oklch(0 0 0 / 0.06),
    0 2px 4px 0 oklch(0 0 0 / 0.04);
  --shadow-border-hover:
    0 0 0 1px oklch(0 0 0 / 0.08),
    0 1px 2px -1px oklch(0 0 0 / 0.08),
    0 2px 4px 0 oklch(0 0 0 / 0.06);
}
```

Three layers: a 1px ring doing the border's job, a tight shadow for lift, a wider one for
ambient depth.

In dark mode, collapse to a single white ring. Layered depth shadows are invisible against
a dark surface and only cost paint time.

```css
--shadow-border: 0 0 0 1px oklch(1 0 0 / 0.08);
--shadow-border-hover: 0 0 0 1px oklch(1 0 0 / 0.13);
```

| Shadow | Border |
|---|---|
| Cards and containers with depth | Dividers between rows |
| Bordered buttons | Table cell boundaries |
| Dropdowns, modals, sheets | Input outlines, which are an accessibility affordance |
| Anything sitting on a varied background | Hairlines in dense layouts |
| A hover or focus lift | A selected or focused state |

## Card treatments are per surface

Theming cards globally repaints every card in the application, including the ones that
were already right. Style the surface that needs it.

## Image outlines

A 1px low-opacity outline gives images consistent depth beside bordered and shadowed
elements.

```css
img {
  outline: 1px solid oklch(0 0 0 / 0.1);   /* light */
  outline-offset: -1px;
}
```

Dark mode uses `oklch(1 0 0 / 0.1)`.

**Pure black and pure white only.** Never a near-black from the palette, no slate-900, no
zinc-900, no `#0a0a0a`. A tinted outline picks up the surface colour behind it and reads
as dirt along the image edge. The outline is a neutral separator, not a themed element,
so it never takes the accent colour either.

`outline` rather than `border` because it does not affect layout, and the negative offset
keeps it inset so the image stays its intended size.

## Hit areas

44x44 logical pixels on touch, at least 40x40 on a dense desktop surface, and 24x24 as
the hard accessibility floor.

Where the visible control should stay small, a 20px checkbox for instance, extend the
target with a transparent pseudo-element rather than growing the control:

```css
.checkbox { position: relative; width: 20px; height: 20px; }
.checkbox::after {
  content: "";
  position: absolute;
  top: 50%; left: 50%;
  transform: translate(-50%, -50%);
  width: 44px; height: 44px;
}
```

**Two hit areas must never overlap.** Where an extended target would collide, shrink it
to the largest size that does not. Give decorative layers, a glow or a gradient overlay,
`pointer-events: none`, so they never swallow a press meant for the control underneath.

## Spacing rhythm

Page rhythm comes from a scale, not from per-widget choices. A workable one: 28 above a
section heading, 16 below it, 16 for the gutter.

Sections carry no top padding of their own and no component carries its own horizontal
gutter, or the spacing compounds differently depending on what happens to be above.

## Press feedback

`scale(0.96)` on press, on anything pressable. Never below 0.95, which reads as
exaggerated rather than responsive. Pair it with a transform transition of 100 to 160ms,
and give it an opt-out for surfaces where the movement would distract.
