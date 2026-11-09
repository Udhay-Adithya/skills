# React on the web

Tailwind, Motion and Radix. Check the installed versions before applying anything here,
because several of these rules are version-dependent.

```bash
node -e "const d={...require('./package.json').dependencies,...require('./package.json').devDependencies};console.log(d)"
cat node_modules/motion/package.json | grep '\"version\"'
```

The declared range is not the installed version. Read the installed one.

## Tailwind 3 against Tailwind 4

**Tailwind 3** keeps tokens in `tailwind.config.js` under `theme.extend`, and dark mode
is the `dark:` variant driven by a class or a data attribute set in the config.

**Tailwind 4** moves tokens into a `@theme` block in CSS and needs no config file.

Writing a `@theme` block into a Tailwind 3 project does nothing, silently. Check which one
is installed before adding a token.

Arbitrary values are the escape hatch, not the habit: `rounded-[20px]` once is fine,
`rounded-[20px]` in nine components means the scale needs an entry.

## Motion: the version matters

From Motion 14.2, the shorthand props (`x`, `y`, `scale`, `rotate`) are
hardware-accelerated on HTML elements. **Before 14.2 they run on `requestAnimationFrame`
on the main thread**, which drops frames whenever the browser is busy loading, parsing or
painting.

```jsx
// on motion < 14.2: main thread, janks under load
<motion.div animate={{ x: 100 }} />

// hardware accelerated on every version
<motion.div animate={{ transform: "translateX(100px)" }} />
```

On a project below 14.2, either upgrade or use the full transform string on anything that
animates while the page is doing other work. A navigation or tab transition during a route
load is the exact case where this shows.

Some Motion animations stay on the main thread in every version: layout animations
(`layout`, `layoutId`), anything with `onUpdate` or `transformTemplate`, drag inertia, and
any value driven every frame by a motion value such as `useTransform`. Where a layout
animation janks during page loads, a plain CSS animation is the fix, because CSS runs off
the main thread.

**Import path.** `motion` and `framer-motion` are the same library under two names.
Match whatever the surrounding files import; mixing both in one project ships two copies.

```jsx
import { motion, AnimatePresence, useReducedMotion, useSpring } from "motion/react";
```

**`initial={false}` on `AnimatePresence`** stops enter animations firing on first mount,
which is what makes a list appear to explode on page load. Check it has not killed an
entrance that was wanted.

**`useReducedMotion`** to branch, rather than overriding in CSS afterwards:

```jsx
const reduce = useReducedMotion();
const hidden = reduce ? { opacity: 0 } : { opacity: 0, x: "-100%" };
```

## Radix

Radix ships behaviour, not looks: focus trapping, dismissal, typeahead, roving tabindex
and correct ARIA are already handled. Rebuilding any of that by hand is how keyboard
support gets lost.

**Animate from `data-state`**, which Radix sets on the content element:

```css
.content[data-state="open"]   { animation: enter 150ms var(--ease-out); }
.content[data-state="closed"] { animation: exit 120ms var(--ease-out); }
```

**Use the transform-origin Radix computes.** A popover should scale from its trigger, not
from its own centre, and Radix exposes the right origin per component:

```css
.popover-content {
  transform-origin: var(--radix-popover-content-transform-origin);
}
```

The same variable exists for dropdown-menu, tooltip, hover-card and select, named after
the component. A modal dialog is the exception and keeps a centred origin, because it is
not anchored to anything.

Radix also exposes `--radix-popover-content-available-height` and the trigger width,
which are what a scrollable menu near the viewport edge needs instead of a hardcoded
max-height.

**`asChild`** merges Radix's props onto your own element rather than nesting another one,
which is how a `<Link>` becomes a menu item without a wrapper div in between.

## CSS that replaces JavaScript

**`@starting-style`** animates entry with no mount-state hook:

```css
.toast {
  opacity: 1;
  transform: translateY(0);
  transition: opacity 400ms var(--ease-out), transform 400ms var(--ease-out);

  @starting-style {
    opacity: 0;
    transform: translateY(100%);
  }
}
```

This replaces the `useEffect(() => setMounted(true), [])` pattern. Keep the old pattern
where the browser support matters.

**Percentage translation** is relative to the element's own size, so `translateY(100%)`
moves a thing by exactly its own height whatever that height turns out to be. It is how
Sonner positions toasts and how Vaul parks a drawer offscreen, and it is more robust than
any pixel value.

**`clip-path: inset()`** animates, and it is the cleanest way to do a reveal, a
comparison slider, or hold-to-delete:

```css
.overlay { clip-path: inset(0 100% 0 0); transition: clip-path 200ms ease-out; }
.button:active .overlay { clip-path: inset(0 0 0 0); transition: clip-path 2s linear; }
```

Slow on the press, fast on the release.

## Hover on touch devices

A touch device fires hover on tap, so an unguarded hover effect sticks after a tap:

```css
@media (hover: hover) and (pointer: fine) {
  .card:hover { transform: scale(1.02); }
}
```

Tailwind's `hover:` variant does **not** include this guard by default in v3. For anything
beyond a colour change, write the media query.

## The Tailwind shorthands worth remembering

| Want | Class |
|---|---|
| Balanced heading | `text-balance` |
| No orphan in body text | `text-pretty` |
| Font smoothing, on `<html>` | `antialiased` |
| Stable digits | `tabular-nums` |
| Press feedback | `active:scale-[0.96] transition-transform` |
| Image hairline | `outline outline-1 -outline-offset-1 outline-black/10 dark:outline-white/10` |
| Shadow transition | `transition-[box-shadow]` |

`transition-transform` covers transform, translate, scale and rotate. `transition-all` is
never the answer: it animates properties nobody intended, including ones that trigger
layout.

## Performance

Animate transform and opacity. Everything else, height, width, padding, margin, top,
left, goes through layout and paint on every frame.

Setting an inherited custom property on a container recalculates styles for every
descendant. In a drawer with many rows, write the transform on the element rather than
updating `--swipe-amount` on the parent.

`will-change` only on transform, opacity and filter, only after seeing first-frame
stutter, and never on `all`.

Keep blur under 20px. It is expensive, and Safari especially so.

## Gestures

Dismiss on velocity, not only on distance. A quick flick should be enough:

```js
const velocity = Math.abs(swipeAmount) / elapsedMs;
if (Math.abs(swipeAmount) >= THRESHOLD || velocity > 0.11) dismiss();
```

Damp past the boundary rather than stopping dead. Capture the pointer once a drag starts,
so it continues when the pointer leaves the element. Ignore additional touch points after
the first, or switching fingers mid-drag jumps the element.
