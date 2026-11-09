# Testing

Tests here are not a coverage number. The cases below are the ones that have actually
broken this kind of app, so they are the ones that get tests.

```bash
flutter test
flutter test test/features/accounts/repository/accounts_repository_test.dart
```

## What gets tested, always

**Repository failure branches.** Every `return Left(...)` and every `catch` in a
repository needs a test. That layer is the only place unknown runtime behaviour becomes
a typed `Failure`, and an untested branch there surfaces as a blank panel with no
explanation. Use `MockClient` from `package:http/testing.dart` to return canned
responses: a 200, each handled status code, malformed JSON, a timeout, a
`ClientException`.

**Every server error shape.** `ServerFailure.fromResponse` gets a test per shape, plus
one for a body that is not JSON. The highest-value test in the repository.

**ViewModel state sequences.** Build a `ProviderContainer` with the repository provider
overridden by a fake, `addTearDown(container.dispose)`, listen, and assert the whole
sequence: `null`, `AsyncLoading`, then `AsyncData` or `AsyncError` carrying the right
`Failure` subtype.

**Out-of-order responses.** For any ViewModel with a refresh or a filter, a test where a
slow first request resolves after a fast second one, asserting the older result is
dropped. This is the bug the sequence guard exists to prevent and it is invisible
without a test.

**Pure functions, table-driven.** Time-zone conversion, financial-year and
month-boundary window maths, currency formatting, duration formatting, percentage
deltas against a zero previous period. Month boundaries and divide-by-zero are where
dashboards print nonsense.

**Shared widgets in `core/common/widgets`.** Pump inside `ProviderScope` and the app's
theme wrapper, since every widget reads the theme and fails without it. Assert all four
states render, in both brightnesses.

## What does not get tested

Generated files. Trivial constructors and `toJson`. Per-page goldens: pages change too
often to pay for themselves, so goldens cover only the shared primitives, in both
colour schemes.

## The widget pump helper

Every widget reads the theme, and the ones that show a state read the status palette off
it, so a widget test has to pump the whole wrapper. Running each case in both
brightnesses is the point: a hardcoded colour is wrong in one of the two modes, always.

```dart
Future<void> pumpInApp(
  WidgetTester tester,
  Widget child, {
  Brightness brightness = Brightness.light,
  List<Override> overrides = const [],
}) {
  return tester.pumpWidget(
    ProviderScope(
      overrides: overrides,
      child: ShadApp(
        theme: AppTheme.light,
        darkTheme: AppTheme.dark,
        themeMode: brightness == Brightness.dark
            ? ThemeMode.dark
            : ThemeMode.light,
        materialThemeBuilder: AppTheme.materialTheme,
        home: Scaffold(body: Center(child: child)),
      ),
    ),
  );
}
```

Then loop the brightnesses rather than writing each case twice:

```dart
void main() {
  for (final brightness in Brightness.values) {
    group('in ${brightness.name} mode', () {
      testWidgets('the loading state draws blocks and no spinner', (
        tester,
      ) async {
        await pumpInApp(
          tester,
          const PanelSkeleton(lines: 4),
          brightness: brightness,
        );

        expect(find.byType(PanelSkeleton), findsOneWidget);
        expect(find.byType(CircularProgressIndicator), findsNothing);
      });

      testWidgets('the error state shows the failure message', (tester) async {
        await pumpInApp(
          tester,
          const PanelError(failure: NetworkFailure('No internet connection.')),
          brightness: brightness,
        );

        expect(find.text('Could not reach the server'), findsOneWidget);
        expect(find.text('No internet connection.'), findsOneWidget);
      });
    });
  }
}
```

## Rules

- No test touches the network. A test that needs a real API is an integration test,
  lives in `integration_test/`, and runs against a locally seeded server.
- Fakes over mocks for repositories: a fake returning a canned `Either` is clearer than
  a mock with stubbed methods. Use `mocktail` at the `http.Client` boundary and for
  verifying a call was made.
- One behaviour per test, named for the behaviour and not for the method.
- A bug fix lands with the test that would have caught it, in the same commit.
