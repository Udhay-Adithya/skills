# Layout

Grouping, order, and holding up when the window, the language or the text size changes.

The numbers here apply only where the project has no spacing scale of its own. Where it
has one, use it. A value that merely differs from these starting points is not a finding;
a clip, an overlap, a broken mirror or a misread group is.

## Group with space, not lines

Space groups first. Background shapes second. Separator lines last, and only where space
alone cannot carry the structure.

The gap between groups is at least twice the gap within one. 8px inside a group means 16px
or more between groups. Get that ratio right and most separator lines become unnecessary.

## Align to shared edges

Pick a small set of alignment edges and put everything on them. One spacing step per level
of nesting, with 16px a reasonable default.

Most layouts that feel untidy have too many alignment edges rather than wrong ones.

## Order by importance

The most important content sits near the top and the leading edge. Within a row, the
identifying content leads; metadata and actions trail.

**Visual order matches source order.** Never move content out of sequence with `order`,
`row-reverse` or explicit grid placement. Keyboard and screen-reader users follow the
source, so reordering visually means two people are reading different screens and only one
of them can be right.

## Controls read as controls

Every interactive element gets a background shape, a border, an underline, or a consistent
placement zone such as a toolbar. A control styled like the static text beside it is not
discoverable, however obvious it looks to whoever built it.

## Borderless controls need more clearance

Without an established density system, start at 12px between adjacent bordered or filled
controls, and 24px between the visible glyphs of borderless text and icon controls,
counting each button's own padding.

Tighter is fine in a compact layout, as long as hit areas never overlap and the controls
stay distinguishable from each other.

## Content bleeds, controls float

Backgrounds and media extend to the viewport edges. Controls and text stay inside the
layout margins and the safe areas.

Full-width buttons in a content layout stay inside the margins, starting around 16px
inline on mobile. Edge-to-edge actions work only where they follow established platform
chrome and stay distinguishable from system UI.

Content scrolls beneath sticky chrome, so set the scroll padding to the sticky header's
height. Otherwise an anchored target or a focused element lands underneath it, which is
both a layout bug and a WCAG 2.4.11 failure.

Safe-area insets resolve to zero unless the viewport meta includes `viewport-fit=cover`,
which is why they so often appear to do nothing.

## Hint at hidden content

Off-screen or collapsed content needs a visible cue: the next item peeking 16 to 32px past
the scroll edge, a disclosure control, a count.

A truncated value someone actually needs has a path to the full text: an expand control, a
detail view, or a tooltip a keyboard can also reach.

## Hold structure until it breaks

Breakpoints come from the content, not from device presets. Keep the expanded layout until
it genuinely stops fitting, then collapse.

Prefer container queries for component-level adaptation, so a component adapts to the
space it is in rather than to the window it happens to be in. Test the smallest and the
largest supported size first; the middle usually takes care of itself.

## Plan for growth

Translated strings grow, and short ones grow proportionally more, which makes a one-word
button label the riskiest thing on the screen.

Size text containers with `max-width` and `min-height`, never a fixed width or height. Let
rows wrap. Test with pseudo-localisation and one long-string locale such as German.

Never park a critical action where resizing, zoom or scrolling can clip it. Keep it in the
normal flow or in stable chrome.

## Mirroring

Write direction-dependent layout as leading and trailing rather than left and right:
`margin-inline-start`, `padding-inline-end`, `inset-inline-start`, `text-align: start`.
Reserve physical left and right for physical geometry such as safe-area insets.

A few things have no logical form and need an explicit override in a right-to-left
context: `translateX`, `background-position`, and anything positioned by physical
geometry.

## Patterns to catch

| Pattern | Fix |
|---|---|
| A grid track or flex child holding long text or a wide table, overflowing | `minmax(0, 1fr)` on the track, `min-width: 0` on the flex child |
| `width: 100vw` | `width: 100%`. `100vw` includes the scrollbar and scrolls horizontally on desktop |
| `height: 100vh` on a full-height mobile pane | `100dvh`, so the toolbar and keyboard never cover the bottom |
| A fixed `height` on a box holding text | `min-height`, or `max-height` with `overflow-y: auto` |
| A media query inside a reusable component | A container query on its parent |
| `container-type: inline-size` on an element with no definite width | Give it a width; size containment collapses a shrink-to-fit box to zero |
| Safe-area insets with no `viewport-fit=cover` | Add it, or they resolve to 0 |
| A sticky header with no scroll padding | Set the scroll padding to its height |
| Visual reordering with `order` or `row-reverse` | Fix the source order |

## In Flutter

The same decisions, different levers. `ConstrainedBox` and `LayoutBuilder` for holding
structure, and `LayoutBuilder` is the closest thing to a container query: it measures the
space the widget is actually in rather than the window.

`Flexible` and `Expanded` are where overflow comes from, and the usual fix for a row that
overflows is a `Flexible` with a `TextOverflow.ellipsis` child rather than a fixed width.

`Directionality` and `EdgeInsetsDirectional` are the logical properties. Use
`EdgeInsetsDirectional.only(start:)` rather than `EdgeInsets.only(left:)` anywhere the app
will be mirrored.

`MediaQuery.textScalerOf(context)` carries the text scale factor, and a fixed-height
container holding text clips at large scale factors exactly as it does on the web. The fix
is the same: constrain with a minimum, not an exact height.
