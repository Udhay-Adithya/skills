---
name: design-engineering
description: Build and review interfaces that feel right: motion that earns its place, concentric radii, optical alignment, shadows for elevation, theme-driven colour, tabular numbers, real hit areas, and keyboard paths. Reads the project's own stack and design tokens first and expresses every fix in that system. Use when building or changing any screen, component, widget, style, theme, animation, transition, hover or press state, when something "feels off" or needs polish, and when reviewing frontend or UI code. Triggers on UI, interface, frontend, component, widget, screen, design, polish, feels off, make it feel better, animation, transition, easing, duration, spring, stagger, hover, focus, press, border radius, shadow, elevation, spacing, typography, icon, colour, contrast, theme, dark mode, accessibility, keyboard, hit area, skeleton, empty state.
---

# Design engineering

Interfaces do not feel good because of one thing. They feel good because a hundred small
decisions each went the right way, and almost none of them are ones the user will ever
consciously notice. That is the point.

This skill holds the decisions. Read the project first, then work.

## 1. Read the project before changing anything

Never apply a value from this skill directly. Find the project's own system and express
the change in it. Introducing a second styling system to land a polish fix is worse than
the imperfection it fixes.

```bash
ls pubspec.yaml package.json Cargo.toml 2>/dev/null       # which stack
```

**Flutter** (`pubspec.yaml`): read `references/flutter.md`. Find the theme directory,
usually `lib/core/theme/`. Learn the existing radius, spacing and colour tokens before
writing a number.

**Web** (`package.json`): read `references/react-web.md`. Check which of `tailwindcss`,
`motion`, `framer-motion`, `sonner` and `@radix-ui/*` are present, and at what version.
Find the token source: `tailwind.config`, a `@theme` block, or a `:root` in the global
stylesheet.

Then, in either case, answer these before writing a line:

- Where do colours come from, and is there a token for the one needed?
- What is the radius scale, and the spacing scale?
- Is there a dark mode, and how is it switched?
- What already exists that does nearly this? Extend it rather than adding a sibling.

A project with a token for something and a hardcoded value beside it has a bug, not a
style. A project with no token for something needs the token added once, in the theme,
not inlined at the call site.

## 2. Pick the mode

**Build.** Making something new or changing something that exists. Work through the
relevant reference, apply the decisions as the code is written.

**Review.** Judging existing work. Slow the interface down: replay motion at 10% speed
in the browser's Animations panel, or raise the duration 5x temporarily in Flutter, and
walk every state, idle, hover, focus, active, loading, empty, error. What looks wrong at
10% speed is what feels subtly wrong at full speed. Report in the format in section 6.

**Stress.** Feeding the worst case through: a 60-character name with no spaces, an
unbreakable email, a one-letter name, a missing field, a count of 0 and a count of
48,000, a label in Tamil or German, an emoji, a negative number, a date at a month
boundary. Render it behind a toggle rather than committing fixture data, and report
every element that broke with its fix.

## 3. The enforcement rule

**Fix what the task touches. Report the rest.**

A hardcoded colour in the widget being edited gets fixed, in the same commit, because it
is already in the diff. The same hardcoded colour in forty other files gets listed at
the end with its locations and does not get touched. That keeps the diff reviewable and
keeps one logical change per commit.

The exception is a `HIGH` finding (section 6): an interaction nobody can reach by
keyboard, a control with no accessible name, text that fails contrast. Say so
immediately rather than filing it in a list at the bottom.

## 4. Decisions that hold everywhere

These are stack-independent. The references carry the mechanics.

**Motion must earn its place.** How often will someone see this animation? Hundreds of
times a day, as with a keyboard shortcut or a command palette, means no animation ever.
Tens of times a day means reduce it hard. Occasional, as with a modal or a toast, means
a standard animation. Rare, as with onboarding, is where delight is affordable. An
animation whose only purpose is that it looks good, on something seen often, is a cost
with no return. See `references/motion.md`.

**Nothing appears from nothing.** Never animate in from zero scale. Start at 0.95 and
pair it with opacity. Nothing in the physical world pops into being at zero size, and an
element that does reads as broken rather than fast.

**Concentric radii.** Outer radius equals inner radius plus the padding between them. A
parent and child sharing one radius is the single most common reason an interface feels
subtly wrong. Past about 24px of padding, treat the surfaces as unrelated and choose each
radius on its own.

**Optical over geometric alignment.** When centred looks off, it is off. An icon beside
text wants about 2px less padding on the icon side. A play triangle wants shifting right.
Fix an asymmetric glyph in the SVG or the icon widget rather than padding around it
forever.

**Shadows for elevation, borders for structure.** A border that exists only to suggest
depth becomes a layered translucent shadow, which works on any background because it is
transparent. A border that separates things, a divider, a table cell edge, an input
outline, a selected state, stays a border.

**Colour comes from the theme, always.** A literal colour value in a component is wrong
in one of the two modes, every time. Status colours in particular are a token set, not
an improvisation: most design systems ship a destructive colour and no success colour,
and the answer is to define the status palette once rather than to invent a green at the
call site.

**One loud thing per screen.** If everything is saturated, nothing reads as urgent. The
loud element is whatever needs action now, which means at most one.

**Dark mode is not a later.** Both schemes ship together and every screen is checked in
both, because that is the only way a hardcoded colour is ever caught.

**Numbers that change are tabular.** Any counter, timer, price or table column gets
tabular figures, or the layout jitters as the value changes.

**Real hit areas.** 44x44 logical pixels on touch, at least 40x40 on a dense desktop
surface, 24x24 as the absolute accessibility floor. Extend a small visible control with
a transparent layer rather than making the control bigger. Two hit areas must never
overlap, and a decorative layer must never swallow the press meant for the control under
it.

**Every pointer path needs a keyboard path.** No exceptions, and this is the one bullet
here that is a legal requirement rather than a matter of craft. See
`references/accessibility.md`.

**Loading looks like the content.** A skeleton in the shape of what is coming, not a
centred spinner. A screen with nine panels and nine spinners reads as broken. An error in
one panel degrades that panel and never the page.

**Every state is designed.** Idle, loading, empty, error, and the one with far too much
data. The empty state says what would be there and how to get it. A real screen shows
several of these at once.

## 5. What not to do

Steering by prohibition makes the forbidden thing more available, not less, so this list
is short and each line names the positive target too.

| Instead of | Do this |
|---|---|
| `transition: all` | Name the exact properties |
| Animating height, width, padding, margin, top, left | Animate transform and opacity |
| `ease-in` on an interface element | `ease-out`, or a custom curve |
| A literal colour in a component | A theme token |
| A spinner as the loading state | A skeleton shaped like the content |
| A placeholder as a label | A real label bound to the input |
| Disabling submit until a form is valid | Keep it enabled, validate on submit |
| A near-black like slate-900 for an image outline | Pure black or pure white at 10% |
| Animation as the only feedback for a state change | Pair it with colour, an icon or a label |

## 6. Review output

State the mode, the exact scope, the stack, the styling convention found, and anything
deliberately not inspected. Then show what was actually looked at:

| Area | What was inspected | Result |
|---|---|---|
| Motion | files, components, states | findings count, `Clear`, or `Not reviewed` plus why |

Cover motion, surfaces, typography and colour, accessibility, and performance. Never
imply an uninspected surface was reviewed.

Findings go in one table per principle, never as separate "Before:" and "After:" lines:

| Severity | Location | Before | After | Why |
|---|---|---|---|---|

- `path/to/file:line` for `Location`, or the exact screen and component where there is no
  source file.
- `Why` names the principle broken and the effect on the person using it.
- One row per root cause, listing every location it appears in. Forty instances of one
  mistake is one finding.

**Severity.** `HIGH` makes an interaction unreachable, unreadable, misleading, or
repeatedly disruptive. `MEDIUM` is a real usability or consistency problem. `LOW` is
isolated polish.

**Then, honestly:** list the commands actually run and what they showed. Anything not
run is labelled `Not verified` with what remains. Close with `Block` while any `HIGH`
stands, `Needs changes` for `MEDIUM` or `LOW`, and `Approve` only with nothing actionable
left. Approving coverage that was not inspected is worse than reporting the gap.

With nothing to report, say "No actionable findings", give the verification, and approve.
Never pad a report to look thorough.

## References

| File | Holds |
|---|---|
| `references/motion.md` | The animation decision order, easing curves, durations, springs, stagger, interruptibility, performance |
| `references/surfaces.md` | Radii, optical alignment, shadows, image outlines, hit areas, elevation |
| `references/type-and-color.md` | Wrapping, smoothing, tabular figures, icon weight and state, tokens, contrast |
| `references/accessibility.md` | Focus, keyboard paths, accessible names, reduced motion, zoom, announcements |
| `references/flutter.md` | All of the above translated to Flutter, shadcn_ui and Riverpod |
| `references/react-web.md` | All of the above for React, Tailwind, Motion and Radix |
