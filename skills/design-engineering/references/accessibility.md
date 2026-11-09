# Accessibility

Two walks. Keyboard only, where every flow completes without a pointer. Then a screen
reader, where every control announces a name, a role and its state.

When in doubt, take the platform's native control over a custom rebuild, and remove ARIA
rather than adding it.

## Criteria, not conventions

A finding cites a WCAG 2.2 Level A or AA criterion by number, or names a concrete task
that someone using assistive technology cannot complete. Everything else is a
recommendation and is never `HIGH`. That includes AAA criteria, APG conventions, the
single-`<h1>` rule, and the 44px and 40px target sizes, which are heuristics on top of
the 24x24 requirement.

## Native elements first

The first rule of ARIA is not to use ARIA where a native element exists. `<button>` for
an action, `<a href>` for navigation. A real link has to support Cmd-click, Ctrl-click
and middle-click, which a div with a handler never will.

## Focus

Style `:focus-visible`, not `:focus`, so keyboard users get a ring and pointer users
usually do not. The browser's own indicator is a good default and beating it is harder
than it looks.

A custom ring needs 3:1 against every colour it crosses, including the element's own
`currentColor`, under 1.4.11. 2px solid with a 3:1 change of contrast is the 2.4.13
target. `outline: none` without a verified replacement is always a bug, and forced-colors
mode must keep the system colours.

A focused element must never end up hidden behind a sticky header or footer, which is
2.4.11. Pad the scroll container by the height of the sticky chrome.

## Keyboard

Every pointer interaction needs a keyboard path. Follow the APG patterns: Escape closes
an overlay, arrow keys move within a composite widget, Tab moves between widgets, Enter
and Space activate a button.

Only `tabindex="0"` to join the natural order and `tabindex="-1"` for programmatic focus.
A positive value breaks the order of the whole page. A composite widget uses roving
tabindex, the active item at 0 and the rest at -1, or `aria-activedescendant` where focus
has to stay in an input, as in a combobox.

## Focus trapping and restoration

Prefer `<dialog>` with `showModal()`: it makes the background inert and handles Escape
without help. A custom overlay sets `inert` on the background and `role="dialog"` plus
`aria-modal="true"` on itself. Either way, move focus in on open and return it to the
trigger on close.

A client-side route change resets nothing on its own. Update the document title and move
focus to the new view's heading or main landmark.

## Forms

Every input gets a real `<label>`, bound by `for` or by wrapping. A placeholder is never a
label: it vanishes exactly when it is needed. The label and the control share one hit
target, with no dead zone between a checkbox and its text.

Add a correct `autocomplete` token, a descriptive `name`, and the `type` and `inputmode`
that summon the right keyboard. Never block paste, because people paste passwords and
one-time codes.

Validate on submit. Do not disable the submit button until the form is valid: it leaves
someone with no way to find out what is wrong. Mark failing fields `aria-invalid="true"`,
point `aria-describedby` at the error text, and move focus to the first invalid field.

## Accessible names

An icon-only button needs a descriptive label. Where a control has visible text, that text
must appear in its accessible name, and ideally start it, so that voice control works.
Decorative elements get `aria-hidden="true"`, which never goes on anything focusable or on
an ancestor of anything focusable.

## Announcements

Three mechanisms, three jobs:

- `aria-describedby` for validation tied to a specific field.
- `role="status"`, a polite region, for non-urgent updates not tied to a control: toasts,
  result counts.
- `role="alert"` for urgent untied errors, and nothing else.

A repeated polite announcement needs a stable empty region that is already in the DOM
before its text changes. A region mounted together with its first message is often not
announced at all.

## Images

Decorative images take `alt=""`. Informative images describe the meaning, not the
picture. Functional images, an icon that is itself the button, describe the action.

## Structure

Headings describe their sections and form a coherent outline. One visible `<main>`
landmark. Where repeated navigation precedes it, a "Skip to content" link is the first
focusable element. Anchored headings get `scroll-margin-block-start` so the anchor does
not land them under the header.

## Motion and timing

Wrap motion in `prefers-reduced-motion: no-preference` so it is opt-in. Under reduced
motion, replace slides and scales with opacity cross-fades and remove parallax and
autoplay entirely.

Anything moving, blinking or updating on its own for more than 5 seconds needs a visible
pause control, which is 2.2.2. A toast carrying an action or an error stays until it is
dismissed, because a timeout on it is a dropped message.

## Zoom and reflow

Text survives 200% resize, which is 1.4.4, and the page reflows at 320px with no
horizontal scrolling, which is 1.4.10. Text containers take `min-height`, never a fixed
`height`. Never set `maximum-scale=1` or `user-scalable=no`. For iOS input zoom, the fix
is a 16px font size on the input, not disabling zoom for everyone.

## Drag

Every drag interaction needs a single-pointer alternative that reaches the same result, a
button or a menu that moves the item, under 2.5.7.

## Patterns to catch on sight

| Pattern | Fix |
|---|---|
| `outline: none` with no `focus-visible` replacement | Restore the ring, or add a verified custom one |
| `<div onClick>` or `<span onClick>` | `<button>`, or `<a href>` when it navigates |
| `role="button"` with no key handler | Use the native element, or implement the key map |
| Any positive `tabindex` | Fix the DOM order, use 0 |
| `disabled` on submit until valid | Keep it enabled, validate on submit |
| A live region mounted with its text | Render the empty region first, then update it |
| `aria-live="assertive"` on a success toast | `role="status"` |
| `<img>` with no `alt` attribute at all | `alt=""` if decorative, otherwise describe the purpose |
| An `aria-label` that omits the visible text | Start the name with the visible text |
| `aria-hidden` on an ancestor of a control | Remove it, or make the subtree inert |
| `user-scalable=no` | Remove it |
| A tooltip on a natively disabled control | Text beside it, or `aria-disabled` so it stays focusable |
| Drag with no button alternative | Add a single-pointer path to the same result |

## Severity

`HIGH` on sight: a missing accessible name, a missing focus indicator, a pointer path
with no keyboard path, motion ignoring reduced motion, content lost at 320px or 200%, and
meaning carried by colour alone.

## Verification

Without a browser: accessible names on every interactive element, key handlers on
non-native controls, focus styles present, reduced-motion guards, labels bound to inputs.

With a browser: tab the whole flow and check the order, read computed names and roles out
of the accessibility tree, confirm a visible indicator at every stop, and run an automated
audit for what it does catch.

Report every check that could not be run as `Not verified`, and say what remains.
