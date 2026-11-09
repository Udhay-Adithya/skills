# Type, colour and icons

## Text wrapping

| Case | Use |
|---|---|
| Headings and short titles | `text-wrap: balance` |
| Paragraphs, descriptions, captions, list items, card text | `text-wrap: pretty` |
| Long text past about 10 lines, code, preformatted | Neither |

`balance` equalises line lengths, and browsers only apply it up to about 6 lines in
Chromium and 10 in Firefox because the algorithm is costly. Putting it on article body
text does nothing at all, which is worse than useless because it looks handled.

`pretty` only stops the last line being a single orphaned word, works at any length, and
is the right default for most running text.

## Font rendering

macOS renders text heavier than intended. Apply smoothing once, at the root:

```css
html {
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}
```

Other platforms ignore it, so it is safe everywhere. Applied per element it produces
headings and body text at visibly different weights.

This does not change the typeface. Smoothing, wrapping and figure style are rendering
details, and they never justify introducing a new or paid font family. Use the project's
type system unless changing it is the actual task.

## Tabular figures

Any number that changes in place gets equal-width digits, or the layout jitters on every
update.

```css
.counter { font-variant-numeric: tabular-nums; }
```

| Tabular | Not tabular |
|---|---|
| Counters, timers | A static number in prose |
| Prices that update | A large decorative number |
| Numeric table columns | Phone numbers, postcodes |
| Animated number transitions | Version strings |

Some faces, Inter among them, widen and recentre the digit 1 under this property. That is
expected and usually what is wanted, but look at it.

## Currency and time

**Currency groups the way its locale groups.** Indian rupee amounts group in lakh and
crore, not thousands, which `NumberFormat` with `en_IN` handles and hand-rolled grouping
does not.

**Timestamps render in the product's zone, not the device's.** A server storing UTC and a
business whose day ends at midnight IST means a reading rendered in the viewer's local
zone misattributes every record near a boundary.

**The financial year is April to March** where the business runs on one, and a range
picker matches the server's labels exactly rather than inventing its own.

Each of these lives in exactly one file. A second implementation is how two screens come
to disagree about the same number.

## Colour

**Every colour comes from a token.** A literal value in a component is wrong in one of
the two modes, always, and the only question is which.

**Status colours are a defined set.** Most design systems ship a destructive colour and
no success colour, which is why codebases fill up with improvised greens. Define healthy,
attention and broken once, for both modes, and have every badge, chip and chart series
read from there.

**One saturated element per screen.** If everything is loud, nothing is. The loud thing is
whatever needs action now.

**Charts need a bridge file.** A charting library knows nothing about the theme, so
exactly one file maps the active scheme onto the library's colour inputs. Series colours
picked at the call site drift apart immediately.

**Colour is never the only cue.** Status needs a redundant signal, an icon, a label or an
underline, both because 1.4.1 requires it and because colour alone fails for a large
number of people.

### Contrast

| Criterion | Applies to | Minimum |
|---|---|---|
| 1.4.3 AA | Text | 4.5:1 |
| 1.4.3 AA | Large text, 24px or 18.67px bold and up | 3:1 |
| 1.4.11 AA | Component boundaries, states, meaningful graphics | 3:1 |

Inactive controls and logos are exempt. Measure the rendered pair, not the token values,
because opacity and overlay change the result. Where a pair fails, report the pair and the
criterion and leave the colours alone unless changing them is the task: a palette change
made mid-task ripples further than the diff suggests.

## Icons

**Stroke weight tracks the adjacent text.**

| Adjacent text | Stroke on a 24px grid |
|---|---|
| Regular 400, 14 to 16px | 1.5px |
| Medium or semibold, 500 to 600 | 2px |
| Bold 700, or emphasised standalone | 2.5px |

One stroke weight per icon set on a surface, and never two icon libraries on one surface.
Size an inline icon against the text's cap height, roughly 1em to 1.25em.

**One asset, recoloured per state.** Draw with `currentColor` and let colour and opacity
carry hover, selected and disabled. Separate assets per state drift apart. Strip hardcoded
fills and strokes when importing an icon.

**Outline is the default, fill marks the active state.** An active tab, a saved bookmark, a
liked heart. Cross-fade between the two variants rather than swapping instantly, using the
values in `motion.md`.

**Design at render size.** Check every icon at the smallest size it will actually appear,
often 16px. Prefer a simplified glyph for small sizes over a detailed one scaled down, and
use the set's native grid sizes rather than arbitrary fractions.

**Right to left.** Flip directional arrows, navigation chevrons, indent controls and
directional send glyphs. Do not flip logos, checkmarks, clocks, or media playback
controls. Analyse a composite icon part by part, since an overlay often keeps its corner
while the base glyph mirrors.

Every icon-only control needs an accessible name, and a purely decorative icon is hidden
from assistive technology.
