# States

A screen is not one picture. Designing only the success state is the most common failure
there is, and it stays invisible until the day the data is wrong.

## The five

### Idle

What is on screen before anything has been asked for. Often this collapses into loading,
but not always: a search screen before a query, a form before submission, a dashboard with
a date range nobody has chosen yet.

Where idle is a real state, it says what to do next.

### Loading

**A skeleton in the shape of the content.** Blocks sized like the text and rows that are
coming, not a centred spinner.

A screen with nine panels and nine spinners reads as broken rather than busy. Each panel
loads and resolves on its own.

Check the skeleton against the real content and confirm the layout does not jump when data
lands. A skeleton of the wrong height is worse than no skeleton, because it promises a
shape it does not deliver.

For anything that usually resolves in under about 300ms, showing nothing beats a flash of
skeleton. A loading state that appears and vanishes is noise.

### Empty

The most under-designed state in software, and the first one a new user sees.

"No data" wastes it. An empty state says what would be here, and how to get it:

- What this surface is for, in one line.
- The action that fills it, as a real control.
- Where relevant, the distinction between "nothing yet" and "nothing matches your
  filters", which are completely different problems with completely different fixes.

That second distinction is the one most often missed. Someone staring at an empty list
because of a filter they forgot has a different next step from someone who has genuinely
never added anything.

### Error

**An error degrades its own panel**, never the page. One failing request on a dashboard
leaves the other eight panels working.

The message says what broke and what can be done, in the vocabulary of whoever is reading
it. An admin tool can be specific about what failed; a consumer app cannot and should not
try.

An error message never carries a raw exception, a stack trace, a URL with a token in it,
or another tenant's data. Those go to the log.

Where retrying is sensible, the retry is a control in the error state, not something the
user has to discover by reloading.

### Too much

The state nobody designs and every real dataset eventually produces. See the stress pass
below.

## Which states a surface actually has

Not every surface has all five, and inventing states nobody can reach is its own waste.
Work it out from the data:

- Can it be empty? Then it has an empty state.
- Can the request fail? Then it has an error state, and the answer is always yes for
  anything over a network.
- Is it paginated or unbounded? Then it has a too-much state.
- Does it take more than a moment? Then it has a loading state.

A dashboard panel typically has all five. A static label has none.

## The stress pass

Before a surface ships, feed it the worst case. Render it behind a toggle rather than
committing fixture data, and report what broke with the fix for each.

| Input | What it finds |
|---|---|
| A 60-character name with no spaces | Containers that cannot wrap, overflow, a broken row |
| An unbreakable email or URL | The same, in the place people forget |
| A one-letter name | Avatars and initials, centring, minimum widths |
| A missing optional field | Null rendering as "null", or a gap with no label |
| A count of 0 | "0 items" against a proper empty state, singular against plural |
| A count of 48,000 | Pagination, number formatting, grouping, a column that no longer fits |
| A negative number, and a zero denominator | Percentage deltas, divide-by-zero, a minus sign that wraps |
| Text in Tamil, Hindi or German | Growth, line height, clipping, a one-word button that is now three |
| An emoji, and a right-to-left string | Line height, alignment, mirroring |
| A date at a month or year boundary | Window arithmetic, off-by-one, the wrong period label |
| The same screen at 320px and at 200% zoom | Clipping, overlap, horizontal scroll |

The last three rows are where real bugs hide rather than cosmetic ones. Month boundaries
and divide-by-zero are how dashboards come to print confident nonsense.

## In Flutter

The states map onto the architecture directly, and `flutter-riverpod-mvvm` owns that
mapping: `null` for idle, `AsyncLoading`, `AsyncData`, `AsyncError`, switched over
exhaustively so the compiler catches a missing branch.

The design work is making sure each of those four branches draws something considered,
rather than three real screens and an `Text('Error')`.
