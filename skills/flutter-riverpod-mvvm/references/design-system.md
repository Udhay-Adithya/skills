# Design system and formatting

`shadcn_ui` is the design system. `ShadApp.router` is the root, with a
`materialThemeBuilder`: `ShadApp` derives a Material theme from the active
`ShadThemeData` and hangs it above the app, so Material widgets that appear inside
third-party packages (`fl_chart` tooltips, date pickers) land in a coherent theme
instead of default blue. `materialThemeBuilder` is also where a status palette is
registered as a `ThemeExtension`.

## Colour

**Every colour comes from the theme.** `ShadTheme.of(context).colorScheme`, never
`Colors.red` or `Colors.green` or `Colors.blue`. A hardcoded colour is wrong in one of
the two modes, always.

**Status colours are a theme extension, not an exception.** An app that shows health
needs healthy, attention and broken, and shadcn's scheme has `destructive` but no
success. Define them once in `core/theme/status_colors.dart`, for both modes, and have
every badge, chip and chart series read them from there. Nothing invents a green.

**One saturated element per screen.** If everything is loud, nothing is. On a dashboard
the loud thing is whatever needs action today, which means at most one panel is in alarm
colours and the rest are muted.

**Dark mode is not optional.** This gets opened at 1am. Both schemes ship from day one
and every screen is checked in both.

**Charts read their colours from `core/theme/chart_theme.dart`**, derived from the
active `ShadColorScheme`. `fl_chart` knows nothing about the theme, so exactly one file
bridges them.

## Shape and rhythm

**Radii** come from `core/theme/radii.dart`: 28 for hero cards, 20 for cards and the
inner blocks of sheets, 16 for strips and inputs, stadium for pills and primary buttons.

**Page rhythm** comes from `core/theme/spacing.dart`: 28 above a section header, 16
below it, 16 for the gutter. Sections supply no top padding of their own, and no widget
carries its own horizontal gutter.

**Inputs** are `ShadInputFormField` with the shared decoration. Do not hand-roll an
input treatment.

**Bespoke card treatments per surface, not a global card theme.** Theming cards globally
repaints every card in the app, including the ones that were fine.

**Tables are a real view, not a fallback.** A list of records is a sortable, paginated
table, and pagination is server-side. Never fetch every row to sort them in Dart.

## Formatting

**Currency is grouped the way its locale groups it.** For INR that is
`NumberFormat.currency(locale: 'en_IN', symbol: '₹')` in
`core/utils/format_inr.dart`: lakh and crore grouping, not thousands.

**Timestamps render in the product's zone, not the device's.** A server storing UTC and
a business whose day boundary is IST means a dashboard rendered in the viewer's local
zone misattributes every record near a month boundary. One file converts, and the UI
labels the zone.

**The financial year is April to March** where the business runs on one. Any range
picker matches the server's window labels exactly rather than inventing its own.

All three live in exactly one file each under `core/utils/`. A second implementation of
any of them is how two screens come to disagree about the same number.
