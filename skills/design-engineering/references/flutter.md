# Flutter

The same decisions, in Flutter terms. Pairs with the `flutter-riverpod-mvvm` skill, which
owns the architecture; this file owns how it looks and feels.

## Colour comes from the theme

`ShadTheme.of(context).colorScheme` in a shadcn_ui project, `Theme.of(context)` in a
Material one. Never `Colors.red`, `Colors.green` or a `Color(0xFF...)` literal in a
widget.

Status colours are a `ThemeExtension` registered once, for both brightnesses, because
shadcn's scheme has `destructive` and no success. Read it with a static accessor rather
than reaching into the extension at each call site.

`ShadApp.router` with a `materialThemeBuilder` is the root: it derives a Material theme
from the active `ShadThemeData` so Material widgets inside third-party packages, an
`fl_chart` tooltip or a date picker, land in a coherent theme instead of default blue.
That builder is also where the status extension is registered.

Charts get one bridge file mapping the active scheme onto the chart library's inputs.

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

## State, loading and errors

Four states, drawn exhaustively, as the architecture skill sets out. The visual rules:

- Loading is a skeleton in the shape of the content. Blocks sized like the text and rows
  that are coming, not a `CircularProgressIndicator` in a `Center`.
- An error in one panel degrades that panel.
- An empty state says what would be there and how to get it.
- Test the skeleton against the real content and check the layout does not jump when the
  data lands. A skeleton of the wrong height is worse than none.

## Accessibility

- `Semantics(label:)` on anything whose meaning is visual only. An `IconButton` takes a
  `tooltip`, which also supplies the semantic label.
- `ExcludeSemantics` for decoration, `MergeSemantics` to make a row of label and control
  announce as one thing.
- Reduced motion is `MediaQuery.of(context).disableAnimations`. Branch on it and
  cross-fade instead of sliding. `accessibleNavigation` tells you a screen reader is
  active, which is when a custom gesture needs its button alternative.
- Focus on web and desktop is real: `FocusTraversalGroup` to fix a traversal order,
  `Shortcuts` and `Actions` or `CallbackShortcuts` for key handling, and a visible focus
  ring. Flutter's default focus highlight is faint, so check it against the surface rather
  than assuming it is there.
- Text scaling comes through `MediaQuery.textScalerOf(context)`. Fixed-height containers
  holding text clip at large scale factors, so constrain with `minHeight`.

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
