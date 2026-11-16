# Motion

Answer these in order. Most bad animation comes from starting at question four.

## 1. Should it animate at all?

| How often it is seen | Decision |
|---|---|
| Hundreds of times a day (keyboard shortcut, command palette) | No animation, ever |
| Tens of times a day (hover, list navigation, tab switch) | Remove it, or cut it hard |
| Occasional (modal, drawer, toast, sheet) | A standard animation |
| Rare or once (onboarding, a celebration, an explainer) | Delight is affordable |

**Never animate a keyboard-initiated action.** Someone using a shortcut is doing it for
speed, repeatedly, and any animation reads as lag. Raycast has no open or close animation
at all, which is correct for something opened a hundred times a day.

## 2. What is it for?

Every animation needs an answer. These are the valid ones:

- **Feedback.** A button scales down on press, confirming the interface heard it.
- **Spatial consistency.** A toast enters and leaves in the same direction, which is what
  makes swipe-to-dismiss feel obvious rather than learned.
- **State indication.** A control visibly becomes its new state.
- **Preventing a jarring change.** Things appearing or vanishing outright read as broken.
- **Explanation.** A marketing or onboarding animation showing how something works.

"It looks good" is a valid purpose only where the thing is rarely seen. On a frequent
interaction it is a tax paid every time.

## 3. Which easing?

- Entering or exiting → **ease-out**. It starts fast, so it feels responsive.
- Moving or morphing on screen → **ease-in-out**.
- Hover or colour change → **ease**.
- Continuous motion, a marquee or a progress bar → **linear**.
- Unsure → **ease-out**.

**Never ease-in on an interface element.** It starts slow, and the start is exactly when
attention is highest. A dropdown with `ease-in` at 300ms feels slower than the same
dropdown with `ease-out` at 300ms.

The built-in curves are weak. Use stronger ones:

```css
--ease-out: cubic-bezier(0.23, 1, 0.32, 1);      /* interface interactions */
--ease-in-out: cubic-bezier(0.77, 0, 0.175, 1);  /* on-screen movement */
--ease-drawer: cubic-bezier(0.32, 0.72, 0, 1);   /* iOS-like drawers */
--ease-crossfade: cubic-bezier(0.2, 0, 0, 1);    /* icon and state cross-fades */
```

## 4. How long?

| Element | Duration |
|---|---|
| Button press feedback | 100 to 160ms |
| Tooltip, small popover | 125 to 200ms |
| Dropdown, select | 150 to 250ms |
| Modal, drawer, sheet | 200 to 500ms |
| Explanatory or marketing | Longer is fine |

**Interface animations stay under 300ms.** Perceived speed is a real effect and not a
soft one: a faster spinner makes an identical load time feel shorter, and a 180ms select
feels more responsive than a 400ms one.

**Exit faster than enter.** An exit is the system responding, so it should be quick. An
enter can afford to be deliberate. The general rule: slow where someone is deciding, fast
where the system is answering. Hold-to-delete is the clearest case, 2s linear while
holding and 200ms ease-out on release.

## 5. How does it get interrupted?

**Transitions over keyframes for anything triggerable in quick succession.** A transition
can be retargeted mid-flight; a keyframe animation restarts from zero. Toasts, toggles
and rapidly-clicked controls all need transitions.

**Springs keep velocity through an interruption**, which is why they are right for
gestures someone may reverse mid-motion. Duration-based animations restart.

## 6. How does it leave?

An exit is designed, not an afterthought. Use a small fixed translation rather than
collapsing the full height, keep it softer than the enter, and keep the direction
consistent with the enter.

## Springs

Reach for a spring for drag with momentum, for anything that should feel physical, for an
interruptible gesture, and for decorative pointer-tracking. Reach for a duration
everywhere else.

```js
{ type: "spring", duration: 0.5, bounce: 0.2 }   // easier to reason about
{ type: "spring", mass: 1, stiffness: 100, damping: 10 }  // more control
```

Keep bounce between 0.1 and 0.3 where it is wanted at all, and at 0 for most interface
work. Bounce belongs to drag-to-dismiss and to deliberately playful surfaces.

Tying a value straight to pointer position feels synthetic because it has no momentum.
Interpolate it through a spring instead. This is decoration, so it is only correct where
decoration is correct: the same effect on a financial chart would be noise.

## Stagger

When several elements arrive together, offset them so the group reads as one movement
rather than a simultaneous pop. 30 to 80ms between items, or about 100ms for a staged
entrance where the sequence communicates hierarchy.

Stagger is decorative. Never block interaction while it plays, and never stagger a
frequent interaction.

## Cross-fading an icon or a state

Animate opacity, scale and blur rather than toggling visibility. Scale 0.25 to 1, opacity
0 to 1, blur 4px to 0. With a motion library, a spring with `duration: 0.3` and
`bounce: 0`. Without one, keep both icons mounted, one absolutely positioned, and
cross-fade with `cubic-bezier(0.2, 0, 0, 1)`.

## Blur to cover a transition that will not settle

When a cross-fade looks wrong at every duration and easing tried, the problem is that two
distinct objects are visible at once. A `blur(2px)` during the transition blends them so
the eye reads one thing changing rather than two things swapping. Keep blur under 20px,
because it is expensive, especially in Safari.

## Performance

**Animate transform and opacity.** They skip layout and paint. Height, width, padding,
margin, top and left trigger all three stages.

**Inherited custom properties are expensive.** Changing a CSS variable on a container
recalculates styles for every descendant. In a list with many rows, set the transform on
the element directly instead of updating a variable on the parent.

**CSS beats JavaScript under load.** CSS animations run off the main thread. Anything
driven by `requestAnimationFrame`, layout animations included, drops frames while the
browser is busy loading or parsing. Use CSS for predetermined motion and JavaScript for
dynamic, interruptible motion. Where programmatic control plus compositor performance are
both needed, the Web Animations API gives both.

**`will-change` only on transform, opacity and filter**, and only once first-frame
stutter has actually been observed. Never on `all`.

## Reduced motion

Reduced motion means less and gentler, not none. Keep opacity and colour transitions that
aid comprehension. Remove movement, parallax, autoplay and scale. Prefer making motion
opt-in, wrapped in `prefers-reduced-motion: no-preference`, over bolting on an override.

## Debugging

Replay at 10% speed, or temporarily multiply the duration by 5. Then look for: two states
visibly overlapping during a cross-fade, an abrupt start or stop, a transform origin in
the wrong place, and properties that are meant to move together drifting apart.

Step frame by frame to catch coordination problems between properties that are invisible
at full speed. Look again the next day, because fresh eyes catch what tired ones tuned
out. Test gestures on real hardware, never only in a simulator.
