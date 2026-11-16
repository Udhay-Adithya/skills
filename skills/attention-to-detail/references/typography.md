# Typography

How text renders. None of this changes what the words are or which typeface carries them.

## Wrapping

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

## Rendering

macOS renders text heavier than intended. Apply smoothing once, at the root:

```css
html {
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}
```

Other platforms ignore it, so it is safe everywhere. Applied per element it produces
headings and body text at visibly different weights, which is the bug it is usually added
to fix.

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

## Line length and measure

Running text past roughly 75 characters a line is hard to track back from the end of one
line to the start of the next. On a wide desktop layout that means a `max-width` on the
text container, not on the page.

This is the one typography rule that is structural rather than cosmetic: text running the
full width of a 27-inch browser is genuinely harder to read, whatever else is right.

## Truncation

Truncate only where the full value is reachable another way: an expand control, a detail
view, or a tooltip that a keyboard can also reach. A truncated value with no path to the
whole thing is lost data wearing an ellipsis.

Use the single-character ellipsis the platform provides rather than three full stops, and
truncate at the end for most content. Truncating in the middle is right for a filename or
an id, where both ends carry meaning.

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
to disagree about the same number, and that disagreement is always found by a user rather
than a test.

## Capitalisation and punctuation

One case policy per element type, applied to every instance of that type. Sentence case
where the project has no policy. "Save Changes" beside "Discard changes" reads as
carelessness even to someone who could not say why.

Prefer real punctuation in rendered copy, a proper apostrophe and a single ellipsis
character, and leave `text-transform` out of anything that will be translated: uppercasing
is not reversible and breaks in several languages.
