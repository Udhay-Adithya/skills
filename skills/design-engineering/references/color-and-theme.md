# Colour and theme

## Every colour comes from a token

A literal colour value in a component is wrong in one of the two modes, always. The only
question is which one, and it is usually the one nobody checked.

This is the single rule in this file that is worth enforcing without exception, because it
is the one that cannot be fixed later cheaply: a hundred components each holding their own
greys cannot be re-themed at all.

## Status colours are a defined set

Most design systems ship a destructive colour and no success colour. That gap is why
codebases fill up with improvised greens, each a slightly different one, each picked at a
call site by someone in a hurry.

Define the set once, for both modes, in the theme: healthy, attention, broken, and
whatever else the product genuinely has. Every badge, chip, row highlight and chart series
reads from there. Nothing invents a colour locally.

In a shadcn project that means a `ThemeExtension` registered where the Material theme is
built, so it is reachable from anywhere the theme is.

## One loud element per screen

If everything is saturated, nothing reads as urgent.

The loud thing is whatever needs action now. On a dashboard that means at most one panel
in alarm colours and the rest muted, and it means the muted ones have to be genuinely
muted rather than merely less red.

A screen where three things compete for attention has, in practice, no primary action.

## Both modes ship together

Dark mode is not a later phase. Both schemes ship from the start and every screen is
checked in both, because checking in both is the only thing that reliably catches a
hardcoded colour.

Retrofitting dark mode onto a product that shipped light-only is several weeks of finding
colours one screen at a time, and it is entirely avoidable.

Shadows behave differently between the modes and this is usually got wrong: a layered
depth shadow is invisible on a dark surface and costs paint time for nothing. Dark mode
wants a single light ring instead, which is a different treatment rather than the same one
with different values.

## Charts need exactly one bridge

A charting library knows nothing about the theme. Exactly one file maps the active scheme
onto the library's colour inputs, and every chart reads from it.

Series colours picked at the call site drift apart immediately, and then two charts on one
screen disagree about what blue means.

## Contrast

| Criterion | Applies to | Minimum |
|---|---|---|
| 1.4.3 AA | Text | 4.5:1 |
| 1.4.3 AA | Large text, 24px or 18.67px bold and up | 3:1 |
| 1.4.11 AA | Component boundaries, states, meaningful graphics | 3:1 |

Inactive controls and logos are exempt.

**Measure the rendered pair, not the token values.** Opacity, a translucent overlay and a
background showing through all change the result, so a token pair that passes in the
palette can fail on the screen.

Where a pair fails, report the pair and the criterion it misses, and leave the colours
alone unless changing them is the actual task. A palette change ripples much further than
the diff suggests, and making it mid-task is how one contrast fix becomes a visual
regression somewhere nobody looked.

## Colour is never the only cue

Status needs a redundant signal: an icon, a label, an underline, a position.

This is 1.4.1, and it is also simply correct: colour alone fails for a large number of
people, and it fails for everyone in a screenshot, a print, a projector and bright
sunlight.

A red row and a green row that differ only in colour are the same row to a meaningful
fraction of users.

## Theming surfaces

Treat a card's styling as belonging to the surface it is on, not to a global card theme.
Theming cards globally repaints every card in the application, including all the ones that
were already right, and the blast radius of a small change becomes the whole product.

## In Flutter

`ShadTheme.of(context).colorScheme` in a shadcn_ui project, `Theme.of(context)` in a
Material one. Never `Colors.red`, `Colors.green` or a `Color(0xFF...)` literal in a
widget.

The status set is a `ThemeExtension`, registered once for both brightnesses and read
through a static accessor rather than reached into at each call site.

`ShadApp.router` with a `materialThemeBuilder` is the root. It derives a Material theme
from the active `ShadThemeData` and hangs it above the app, so Material widgets appearing
inside third-party packages, an `fl_chart` tooltip or a date picker, land in a coherent
theme instead of default blue. That builder is also where the status extension is
registered, which is the piece most often missed: without it `StatusColors.of` returns
nothing at exactly the moment a badge needs a colour.

## On the web

Tailwind 3 keeps tokens in `tailwind.config.js` under `theme.extend`, and dark mode is the
`dark:` variant. Tailwind 4 moves them into a `@theme` block in CSS.

Writing a `@theme` block into a Tailwind 3 project does nothing, silently, so check which
is installed before adding a token. The declared range in `package.json` is not the
installed version; read the installed one.
