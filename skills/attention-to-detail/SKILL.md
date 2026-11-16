---
name: attention-to-detail
description: >-
  The finishing pass on an interface: spacing, concentric radii, optical alignment,
  shadows against borders, typography rendering, tabular figures, icon weight and state,
  image outlines, hit areas, press feedback, and the exact easing and duration of every
  animation. Reads the project's own styling system and expresses each fix in it. Use
  after building or changing any interface, when something looks almost right but not
  quite, when polishing, when a layout feels cramped or loose, or when reviewing
  frontend or UI code for craft rather than correctness. Triggers on polish, detail,
  feels off, looks cheap, not quite right, spacing, padding, margin, border radius,
  rounded corners, shadow, elevation, alignment, typography, font, text wrapping,
  tabular numbers, icon, stroke weight, hit area, tap target, press state, hover state,
  easing, duration, transition, animation timing, stagger.
---

# Attention to detail

Almost nothing here gets noticed on its own. That is the point. A hundred small decisions
each going the right way is what separates software that works from software that feels
finished, and the person using it will never be able to name a single one of them.

> All those unseen details combine to produce something that's just stunning, like a
> thousand barely audible voices all singing in tune.

This is the finishing pass. `design-engineering` decides what goes on the screen; this
decides whether it looks like somebody cared.

## Express every fix in the project's system

Never paste a value from here directly. Find how the project already expresses this and
use that: Tailwind classes in a Tailwind project, the theme file in a Flutter project,
plain CSS where that is the convention.

Introducing a second styling system to land a polish fix is worse than the imperfection
it fixes, because now there are two places to look and they will disagree.

```bash
ls pubspec.yaml package.json 2>/dev/null
```

Flutter goes to `references/flutter.md`, web to `references/react-web.md`. Read the theme
or token file before writing any number. A project with a radius scale and a hardcoded
radius beside it has a bug, not a style; a project with no scale needs the token added
once, in the theme, not inlined at the call site.

## Slow it down

Reviewing at full speed finds almost nothing. Replay motion at 10% in the browser's
Animations panel, or raise the duration 5x temporarily in Flutter, and walk every state:
idle, hover, focus, active, loading, empty, error, disabled.

What looks wrong at 10% speed is what feels subtly wrong at full speed. This single habit
finds more than any checklist.

Then look again the next day. Fresh eyes catch what tired ones have tuned out, and this is
the one step that cannot be rushed or delegated.

## The details

### Shape

**Concentric radii.** Outer equals inner plus the padding between them. A parent and child
sharing one radius is the single most common reason an interface feels subtly wrong. Past
about 24px of padding the surfaces stop reading as nested, so choose each independently.

**Optical over geometric alignment.** When centred looks off, it is off. An icon beside
text wants about 2px less padding on the icon side. A play triangle wants shifting right.

**Shadows for elevation, borders for structure.** A border that only implies depth becomes
a layered translucent shadow, which survives any background. A border that separates
things stays a border.

**Image outlines are pure black or pure white at 10%.** A near-black from the palette
picks up the surface behind it and reads as dirt along the edge.

### Space

**Space groups, lines separate, and space goes first.** The gap between groups is at least
twice the gap within one. Reach for a separator line only where space alone cannot carry
the structure.

**One spacing scale, one step per level of nesting.** Numbers chosen per component is how
two cards that should match end up 2px apart.

**Borderless controls need more clearance than bordered ones**, because their hit area is
invisible and their glyphs are all the user has to aim at.

### Type

**Balance headings, pretty everything else, neither on long text.**

**Tabular figures on anything that changes in place**, or the layout jitters on every
update.

**Font smoothing once, at the root.** Per element it produces visibly different weights.

**Truncate only where the full value is reachable** by some other path.

### Icons

**Stroke weight tracks the adjacent text:** 1.5px beside regular, 2px beside semibold.
One icon set per surface, never two.

**One asset drawn with `currentColor`,** states from CSS. Outline by default, fill for
active.

**Check every icon at 16px**, which is where a detailed glyph turns to mud.

### Feel

**`scale(0.96)` on press**, on anything pressable. Never below 0.95, which reads as
exaggerated. This is the cheapest single thing that makes an interface feel responsive.

**Hit areas: 44x44 on touch, 40x40 on dense desktop, 24x24 as the hard floor.** Extend a
small control with a transparent layer rather than growing it. Two hit areas never
overlap, and a decorative overlay never swallows the press beneath it.

**Gate hover effects behind a real pointer.** A touch device fires hover on tap, so an
unguarded hover effect sticks after a tap.

### Motion

**Exit faster than enter.** The enter can be deliberate; the exit is the system
responding.

**Never animate in from zero scale.** Start at 0.95 with opacity. Nothing in the physical
world pops into being at zero size.

**Under 300ms for anything on a frequent path.** Perceived speed is real: a 180ms select
feels more responsive than a 400ms one, and a faster spinner makes an identical load feel
shorter.

**Transitions, not keyframes, for anything triggerable in quick succession.** A transition
retargets mid-flight; a keyframe restarts from zero.

**Stagger 30 to 80ms between items**, and never block interaction while it plays.

Everything about motion, including the exact curves, is in `references/motion.md`.

## What not to do

| Instead of | Do this |
|---|---|
| `transition: all` | Name the exact properties |
| Animating height, width, padding, margin, top, left | Animate transform and opacity |
| `ease-in` on an interface element | `ease-out`, or a custom curve |
| The same radius on a parent and its child | Outer equals inner plus padding |
| A separator line between every row | Space, and a line only where space fails |
| `scale(0)` as an entrance | `scale(0.95)` with opacity |
| A near-black for an image outline | Pure black or pure white at 10% |
| `will-change` left on permanently | Add it after seeing stutter, on transform, opacity or filter only |
| Two icon libraries on one surface | One set, one stroke weight |

## Scope

**Fix what the task touches. Report the rest.**

A hardcoded radius in the component being edited gets fixed, because it is already in the
diff. The same mistake in forty other files gets listed with its locations and left alone.
That keeps the diff reviewable and keeps one logical change per commit.

A detail finding is never a reason to block a change. Nothing here makes software fail; it
makes it feel unfinished. Where something genuinely breaks an interaction, that is
`design-engineering`'s accessibility reference, and it outranks everything on this page.

## Review output

State the scope, the stack, the styling convention found, and anything deliberately not
inspected. Then show what was actually looked at:

| Area | What was inspected | Result |
|---|---|---|
| Shape | files, components, states | findings count, `Clear`, or `Not reviewed` plus why |

Cover shape, space, type, icons, feel and motion. Never imply an uninspected surface was
reviewed.

Findings go in one table per area, never as separate "Before:" and "After:" lines:

| Severity | Location | Before | After | Why |
|---|---|---|---|---|

- `path/to/file:line`, or the exact screen and component where there is no source file.
- `Why` names the detail broken and what it costs. "Nested corners should be concentric"
  is a reason; "looks better" is not.
- One row per root cause, listing every location. Forty instances of one mistake is one
  finding, not forty.

**Severity here tops out lower than elsewhere.** `MEDIUM` is a noticeable usability or
consistency problem, such as a tap target that is genuinely hard to hit or numbers that
shift as they update. `LOW` is isolated polish, which is most of this. Reserve `HIGH` for
something that actually prevents use, and when one appears, say so immediately rather than
filing it in a table.

Close with the checks actually run and what they showed. Anything not run is labelled
`Not verified` with what remains.

**Say when there is nothing.** "No actionable findings" is a real and common result. Never
pad a report to look thorough: a list of inventions trains the reader to skim the next
one.

## References

| File | Holds |
|---|---|
| `references/surfaces.md` | Radii, optical alignment, shadows, image outlines, hit areas, spacing rhythm, press feedback |
| `references/typography.md` | Wrapping, rendering, tabular figures, measure, truncation, currency and time |
| `references/icons.md` | Stroke weight, state, render size, optical centring, mirroring, names |
| `references/motion.md` | The easing curves, durations, springs, stagger, interruptibility, performance, debugging |
| `references/flutter.md` | All of it in Flutter, with shadcn_ui and Riverpod |
| `references/react-web.md` | All of it for React, Tailwind, Motion and Radix |
