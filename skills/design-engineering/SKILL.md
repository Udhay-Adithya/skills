---
name: design-engineering
description: >-
  Design an interface: what goes on the screen, what it is called, how it is grouped and
  ordered, which states exist and how each one is drawn, how it holds up when resized,
  translated or fed bad data, and whether anyone can reach it by keyboard. Reads the
  project's own design system first. Hands the finishing pass to attention-to-detail.
  Use when building a new screen, component or flow, when restructuring an existing one,
  when deciding what to show or where to put it, when a layout breaks at some width, or
  when reviewing an interface as a whole. Triggers on design, UI, interface, screen,
  page, view, component, widget, layout, hierarchy, information architecture, empty
  state, loading state, error state, skeleton, responsive, breakpoint, mobile, dark
  mode, theme, contrast, accessibility, keyboard, copy, label, microcopy.
---

# Design engineering

Deciding what the interface is: what appears, what it is called, how it is grouped and
ordered, which states it has, and whether it survives a small screen, a long translation
and a user with no mouse.

This is the design layer. The finishing layer is `attention-to-detail`, and this skill
depends on it: get the structure right here, then hand over for the craft.

## 1. Read the project first

Never design against a blank page when a design system already exists.

```bash
ls pubspec.yaml package.json 2>/dev/null
```

Then find and read, before deciding anything:

- **The theme.** Where colours, spacing and radii come from. `lib/core/theme/` in a
  Flutter project, `tailwind.config` or a `@theme` block or a `:root` on the web.
- **The nearest existing screen.** Whatever already does something similar is the
  strongest available statement of how this project does things. Extend it rather than
  inventing a sibling.
- **The shared components.** What already exists that nearly does this.
- **The copy.** Find the translation call and the locale files, and list the noun used for
  each object and the verb used for each action. New copy uses the terms already on that
  list.

A new pattern needs a reason. "The existing one did not fit" is a reason; not having
looked is not.

## 2. Structure

### Hierarchy first

The most important thing sits near the top and the leading edge. Within a row, the
identifying content leads and the metadata and actions trail.

Visual order matches source order. Never reorder with `order`, `row-reverse` or grid
placement, because keyboard and screen-reader users get the source order and will be
reading a different screen from everyone else.

### Group with space

Space groups first, background shapes second, separator lines last and only where space
cannot carry the structure. The gap between groups is at least twice the gap within one.

A screen that needs a line between every row is usually a screen whose grouping has not
been decided.

### One primary action per view

One primary action. Secondary actions go behind a menu once there are more than about
three. Prefer a short view that links deeper over one long view showing everything at the
top level.

**One loud element per screen.** If everything is saturated, nothing reads as urgent. The
loud thing is whatever needs action now, which means at most one.

### Controls must look like controls

Every interactive element needs a background, a border, an underline or a consistent
placement zone. A control styled like the text beside it does not read as a control, and
nobody clicks it.

### Nothing hidden without a cue

Off-screen or collapsed content needs a visible hint: the next item peeking past the
scroll edge, a disclosure control, a count. Content behind an invisible affordance does
not exist.

Full detail in `references/layout.md`, including how to hold a layout until it breaks,
what to do about translated strings that grow, and the mirroring rules.

## 3. Every state is a design decision

A screen is not one picture. At minimum it is five, and a real screen shows several at
once.

| State | The question |
|---|---|
| Idle | What is here before anything is asked for? |
| Loading | What is the shape of what is coming? |
| Empty | What would be here, and how does someone get it? |
| Error | What broke, and what can be done about it? |
| Too much | What happens at 48,000 rows, or a 60-character name with no spaces? |

**Loading looks like the content.** A skeleton in the shape of what is arriving, not a
centred spinner. Nine panels with nine spinners reads as broken.

**An error degrades its own panel**, never the page.

**An empty state is a design surface, not a fallback.** It is the first thing a new user
sees, and "No data" wastes it.

Designing only the success state is the most common failure in this whole skill, and it is
invisible until the day the data is wrong. `references/states.md` has each one, plus how
to stress a screen with worst-case data before shipping it.

## 4. Words are part of the design

Copy is not something written afterwards to fill the boxes.

- **A button label starts with a verb** naming the action. "Send", "Save draft", "Delete
  project". Never "OK" or "Let's go".
- **An error says how to fix it, beside the thing that failed.** Not at the top of the
  page, not after the form is submitted again.
- **Link text makes sense out of context**, because screen-reader users navigate by a list
  of links. "Read the billing docs", never "Click here", and never two bare "Learn more"
  on one page.
- **A toggle is labelled for its on state.** "Send read receipts", so the off state is
  inferable. "Don't send read receipts" makes the off position a double negative.
- **One voice and one vocabulary.** If it is Archive in the menu it is not "Move to
  storage" in the toast. Tone flexes with the stakes: warm for empty states and
  onboarding, neutral for settings, plain and calm for errors, serious for anything about
  data loss.
- **Build strings whole**, with placeholders and the locale's plural rules. A sentence
  assembled from fragments around a variable cannot be translated, because word order
  changes.

## 5. Should it move at all

This is a design decision; everything after it is craft.

| How often it is seen | Decision |
|---|---|
| Hundreds of times a day (a keyboard shortcut, a command palette) | No animation, ever |
| Tens of times a day (hover, list navigation, tab switch) | Remove it, or cut it hard |
| Occasional (a modal, a drawer, a toast) | A standard animation |
| Rare or once (onboarding, a celebration) | Delight is affordable |

Then ask what it is for. Feedback, spatial consistency, showing a state change, preventing
a jarring appearance, or explaining something. "It looks good" is a valid purpose only
where the thing is rarely seen.

Once the answer is yes, the curve, the duration and the interruption behaviour all belong
to `attention-to-detail`.

## 6. It has to work

`references/accessibility.md` is not a late checklist. Two things are structural and
belong in the design rather than after it:

**Every pointer path needs a keyboard path.** Decided when the interaction is designed,
because a drag with no alternative has to be redesigned, not patched.

**Meaning never rides on colour alone.** Status needs a second cue: an icon, a label, an
underline.

Both are cheap at design time and expensive afterwards.

## 7. Hand off the finishing

When the structure is settled, the states are drawn, the copy is written and it works by
keyboard, the design work is done and the craft work starts.

Hand to `attention-to-detail` for radii, optical alignment, shadows, spacing rhythm,
typography rendering, icon weight, hit areas, press feedback and the exact motion values.
That skill carries the stack-specific files for Flutter and for the web.

Do not do its job here, and do not skip it. A well-structured screen with careless
details still reads as unfinished.

## Review output

State the scope, the stack, the design system found, and anything deliberately not
inspected. Then show what was looked at:

| Area | What was inspected | Result |
|---|---|---|
| Structure | files, screens, states | findings count, `Clear`, or `Not reviewed` plus why |

Cover structure, states, copy, colour and theme, and accessibility. The craft pass is
reported separately by `attention-to-detail`, so say which of the two this is rather than
blurring them.

| Severity | Location | Before | After | Why |
|---|---|---|---|---|

- `path/to/file:line`, or the exact screen and component.
- `Why` names what breaks and for whom.
- One row per root cause, listing every location it appears in.

**Severity.** `HIGH` makes something unreachable, unreadable or misleading: a flow with no
keyboard path, content clipped at 320px or 200% zoom, a truncated value with no way to the
full text, meaning carried by colour alone, an undesigned error state on a path that can
fail. `MEDIUM` damages hierarchy, reading order or adaptability. `LOW` is isolated.

Close with the checks actually run. Anything not run is `Not verified` with what remains.
`Block` while any `HIGH` stands, otherwise `Approve` and leave the rest as work to do.
Approving coverage that was not inspected is worse than reporting the gap.

## References

| File | Holds |
|---|---|
| `references/layout.md` | Grouping, alignment, reading order, responsive structure, growth and clipping, mirroring |
| `references/states.md` | Idle, loading, empty, error and too-much, plus stressing a screen with worst-case data |
| `references/color-and-theme.md` | Tokens, the status palette, dark mode, charts, contrast |
| `references/accessibility.md` | Focus, keyboard paths, accessible names, announcements, zoom, reduced motion |

For the finishing pass, use the `attention-to-detail` skill.
