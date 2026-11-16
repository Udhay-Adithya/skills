# Flutter

The craft decisions, in Flutter terms. `flutter-riverpod-mvvm` owns the architecture,
`design-engineering` owns the theme, the states and accessibility, and this file owns how
the result looks and feels.

## Easing translates exactly

Flutter's `Cubic` takes the same four control points as CSS `cubic-bezier`, so the curves
carry across unchanged:

```dart
abstract final class Ease {
  static const out = Cubic(0.23, 1, 0.32, 1);        // entering, exiting
  static const inOut = Cubic(0.77, 0, 0.175, 1);     // moving on screen
  static const drawer = Cubic(0.32, 0.72, 0, 1);     // sheets, iOS feel
  static const crossfade = Cubic(0.2, 0, 0, 1);      // icon and state swaps
}
```

`Curves.easeOutCubic` is a reasonable stock substitute. `Curves.easeIn` on anything
entering or exiting is the same mistake as `ease-in` on the web, and
`Curves.bounceOut` belongs almost nowhere.

Durations match the web table: 100 to 160ms for press feedback, 125 to 200ms for a
tooltip, 150 to 250ms for a dropdown or popover, 200 to 500ms for a sheet or dialog, and
under 300ms for anything on a frequently used path.

## Implicit first, explicit when it must be

`AnimatedContainer`, `AnimatedOpacity`, `AnimatedAlign`, `AnimatedScale`,
`AnimatedSwitcher` and `TweenAnimationBuilder` cover most interface motion and are
interruptible by construction: a rebuild with a new target retargets from the current
value, which is exactly the behaviour CSS transitions have and keyframes do not.

Reach for `AnimationController` when the animation needs to be driven, reversed, repeated
or scrubbed, when several properties must share one clock, or when a gesture drives it.
A controller needs `SingleTickerProviderStateMixin` and a `dispose`, and forgetting the
`dispose` is a leak the analyzer will not always catch.

`AnimatedSwitcher` with a `FadeTransition` plus `ScaleTransition` is the icon cross-fade:
scale 0.25 to 1, opacity 0 to 1, about 300ms, no bounce.

## Press feedback

```dart
AnimatedScale(
  scale: _pressed ? 0.96 : 1,
  duration: const Duration(milliseconds: 140),
  curve: Ease.out,
  child: child,
)
```

Driven by `GestureDetector`'s `onTapDown`, `onTapUp` and `onTapCancel`. `onTapCancel`
matters: without it a press that turns into a scroll leaves the control stuck small.

Where the surface already uses `InkWell`, the ripple is the feedback and a scale on top of
it is too much.

## Hit areas

Material's floor is 48x48 logical pixels, which is stricter than the web's 44, so use 48
on touch surfaces. `MaterialTapTargetSize.padded` is the default on most Material
controls and is what produces the invisible padding around a small `IconButton`; leaving
it at `shrinkWrap` to tighten a layout is how a control ends up untappable.

To grow a small visible control, wrap it rather than resizing it:

```dart
SizedBox(
  width: 48,
  height: 48,
  child: Center(child: SizedBox.square(dimension: 20, child: visibleControl)),
)
```

A `Stack` child that overlays a control intercepts its taps. Wrap a decorative overlay in
`IgnorePointer`, which is Flutter's `pointer-events: none`.

## Radii and elevation

```dart
BorderRadius.circular(20)   // outer, with 8 of padding
BorderRadius.circular(12)   // inner
```

Same arithmetic: outer equals inner plus padding. Keep the scale in one file rather than
typing numbers at call sites.

For elevation, a layered `BoxShadow` list beats `Material`'s `elevation`, which applies
a Material-specific shadow that will not match a shadcn surface:

```dart
const shadowBorder = [
  BoxShadow(color: Color(0x0F000000), blurRadius: 0, spreadRadius: 1),
  BoxShadow(color: Color(0x0F000000), blurRadius: 2, offset: Offset(0, 1)),
  BoxShadow(color: Color(0x0A000000), blurRadius: 4, offset: Offset(0, 2)),
];
```

In dark mode, collapse to a single light ring, which is better expressed as a
`Border.all` with a low-opacity white than as a shadow nothing can see.

An image gets its hairline from `foregroundDecoration` on the `Container`, so it sits
inside the bounds without changing layout:

```dart
Container(
  foregroundDecoration: BoxDecoration(
    border: Border.all(color: const Color(0x1A000000)),
    borderRadius: BorderRadius.circular(12),
  ),
  child: Image.network(url, fit: BoxFit.cover),
)
```

Pure black at 10% in light, pure white at 10% in dark. Never a tinted near-black.

## Typography

**Tabular figures**, for anything that updates in place:

```dart
const TextStyle(fontFeatures: [FontFeature.tabularFigures()])
```

Put it on the theme's number styles rather than at each call site.

**There is no `text-wrap: balance` or `pretty`.** The levers are `softWrap`, `maxLines`
and `overflow`, plus `TextOverflow.ellipsis` where truncation is correct. A heading that
must not orphan a word needs either a narrower `maxWidth` or a hand-placed break, and
neither is worth faking with a custom line-breaking pass.

**Font smoothing does not apply.** Flutter rasterises text itself, so there is no
equivalent of `-webkit-font-smoothing` and nothing to set.

`ConstrainedBox(maxWidth: 960)` or similar keeps a wide web layout readable; text running
the full width of a desktop browser is unreadable regardless of the type choices.

## Skeletons

A loading skeleton is plain containers sized like the content, never a
`CircularProgressIndicator` in a `Center`. Build it from the same spacing and radius
tokens as the real content, so the two cannot drift apart, and check that the layout does
not jump when the data lands.

`design-engineering`'s states reference owns which states exist and what each one says.
This is only how to draw them.

## Performance

- `const` constructors wherever the widget takes no runtime values. It is the cheapest
  rebuild saving available.
- `RepaintBoundary` around something that animates inside an otherwise static subtree, so
  the animation does not repaint its siblings every frame.
- Prefer `AnimatedOpacity` and `FadeTransition` over the `Opacity` widget in an animation.
  `Opacity` forces a `saveLayer`, which is expensive per frame.
- `BackdropFilter` and `ImageFiltered` are costly, especially on web. A blur behind a
  full-screen sheet will show on a mid-range phone.
- `ListView.builder` for anything that could get long, never a `Column` in a
  `SingleChildScrollView` built from a mapped list.
- Watch only the providers a widget needs, and `select` a field rather than watching a
  whole object, or every unrelated change rebuilds the subtree.
- Profile in release or profile mode. Debug-mode frame times are not evidence of
  anything.

## Web and mobile from one codebase

A Flutter web target changes what is affordable. Shadows and blur cost more, scroll
physics differ, and hover states exist, which they do not on a phone. Gate hover
affordances on the pointer actually being a mouse rather than assuming, and check a
release web build rather than trusting the debug one.
