---
name: flutter-riverpod-mvvm
description: Build Flutter features on Riverpod code generation and MVVM, with a repository layer that returns Either<Failure, T>, a sealed failure hierarchy, exhaustive four-state UI, and no service locator. Use when writing or reviewing any Flutter or Dart feature, adding a screen, a provider, a repository or a model, wiring networking or auth refresh, deciding where a file goes, or when a widget is doing work that belongs in a ViewModel. Triggers on Flutter, Dart, Riverpod, @riverpod, ProviderScope, ref.watch, AsyncValue, go_router, fpdart, Either, build_runner, json_serializable, shadcn_ui, MVVM, repository pattern.
---

# Flutter, Riverpod and MVVM

The architecture to follow in a Flutter app: four layers per feature, Riverpod as the
only dependency graph, failures as values rather than exceptions, and widgets that
cannot reach the network.

Read `references/layers.md` before adding a feature, `references/networking.md` before
touching a repository or an HTTP client, `references/testing.md` before writing a test,
and `references/design-system.md` before writing a widget.

## Stack

| Concern | Choice |
|---|---|
| State management | Riverpod with code generation (`@riverpod`) |
| Dependency injection | Riverpod providers and `ProviderScope` overrides. No GetIt, no service locator |
| Routing | `go_router` |
| Networking | `http.Client`, wrapped once for auth |
| Error handling | `fpdart`'s `Either<Failure, T>` |
| Serialization | `json_serializable` |
| Code generation | `build_runner` |
| UI | `shadcn_ui` |
| Charts | `fl_chart` |
| Session storage | `flutter_secure_storage` |
| Local database | `drift`, and only once a feature needs one |
| Formatting | `intl` |
| Tests | `flutter_test`, `mocktail`, `package:http/testing.dart` |

Versions live in `pubspec.yaml` and nowhere else. Add a package with `flutter pub add`
so its constraint is resolved when it is added, rather than copied out of a document
that has gone stale.

The four Riverpod packages (`flutter_riverpod`, `riverpod_annotation`,
`riverpod_generator`, `riverpod_lint`) pin each other. A caret range on one resolves
the lint plugin away. Pin all four exactly, and bump them together or not at all.

## Commands

```bash
flutter pub get
flutter run -d chrome
flutter analyze                                              # clean before every commit
flutter test
dart run build_runner build --delete-conflicting-outputs
dart format .
```

Run `build_runner` after touching any annotated class, then `flutter analyze`. A commit
whose generated files are stale is a broken commit.

## Where a file goes

```text
lib/
├── main.dart
├── init_dependencies.dart
├── core/
│   ├── common/widgets/      # shared presentational widgets
│   ├── constants/           # base url, timeouts, paths
│   ├── errors/failure.dart  # the sealed hierarchy
│   ├── models/              # models more than one feature reads
│   ├── network/             # the one client, wrapped once
│   ├── providers/
│   ├── router/
│   ├── theme/
│   └── utils/
└── features/
    └── my_feature/
        ├── model/           # data and serialization
        ├── repository/      # contract plus data source
        ├── viewmodel/       # state and feature logic
        └── view/
            ├── pages/       # full screens
            └── widgets/     # pieces of those screens
```

Feature code stays inside its feature. Moving something to `core/` is a decision made
when a second feature needs it, not a reflex when the first one is written.

## Order of decisions when adding a feature

1. **Model first.** What shape does the server actually return? Read the router or the
   schema. Never guess at an API shape.
2. **Contract second.** Write `abstract interface class MyRepository` with the
   operations this feature uses, typed, returning `Future<Either<Failure, T>>`.
3. **Implementation third.** One class per data source. It maps status codes and
   exceptions to `Failure` subtypes and decodes JSON. It is the only layer allowed to
   know that HTTP exists.
4. **ViewModel fourth.** It depends on the contract, never on `http`. It exposes typed
   state and never decodes JSON.
5. **View last.** `switch` over the state exhaustively, every branch drawn.

Skip a layer only when there is nothing for it to do. A screen that reads a value
derived purely from its arguments wants a plain `@riverpod` function with those
arguments as parameters, not a notifier with a `fetch` method; let Riverpod's own
caching do the work.

## The rules that matter

**No service locator.** Riverpod providers are the graph. Anything that must be built
asynchronously before the first frame (secure storage, a database, a warmed client) is
built in `initDependencies()` and handed to `ProviderScope` as an override. A provider
that can build itself synchronously does not belong there. Never reach a provider
through a global container; use `ref`.

**Failures are values, not exceptions.** A repository never throws across its contract.
It returns `Left(Failure)` for an expected failure and `Right(data)` for success. The
hierarchy is `sealed`, so a `switch` over a failure is exhaustive and the compiler finds
every site when a case is added.

**Put the `Failure` into `AsyncValue.error`, not `failure.message`.** Throwing the type
away at the ViewModel means the page cannot tell an expired session from a parse error.

**Widgets hold no networking and no persistence.** Data arrives through the constructor.
A widget that calls a repository is a bug, not a shortcut.

**Never trigger a request from `build()`.** Start work from a `ref.listen`, an explicit
user action, or a provider that fetches in its own `build`.

**Every async operation handles its failures explicitly.** There is no catch-all higher
up that will do it later.

**Guard against out-of-order responses.** Any ViewModel with a refresh, a filter or a
date range needs a request sequence number: increment on entry, compare before writing
state, drop the result if it does not match. A slow first request landing after a fast
second one is how a stale window ends up drawn under a new label, and it is invisible
without a test.

**`final` by default.** Prefer immutable data. Prefer composition over inheritance.
Constructor injection for dependencies.

**Avoid abstractions for trivial functionality.** An interface with one implementation
and no test seam is a cost with no benefit.

## State and its four branches

| State | Meaning |
|---|---|
| `null` | Idle, nothing asked for yet |
| `AsyncLoading()` | In flight |
| `AsyncData<T>()` | Succeeded |
| `AsyncError()` | Failed, carrying the `Failure` |

```dart
class MyPage extends ConsumerWidget {
  const MyPage({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final state = ref.watch(myViewModelProvider);

    return switch (state) {
      null => const MyIdleView(),
      AsyncLoading() => const MySkeleton(),
      AsyncError(:final error) => MyErrorView(failure: error as Failure),
      AsyncData(:final value) => MyContent(data: value),
    };
  }
}
```

A screen with nine panels has four states nine times over, and a real screen shows
several of them at once. Loading is a skeleton shaped like the content, never a centred
spinner. An error in one panel degrades that panel and never takes the page down.

## Code generation

Include the `part` directive before generating. Never edit a generated file. Not every
file needs a `.g.dart` companion, only ones with supported declarations. A commit never
carries a source change without its regenerated companion.

## Local persistence

Nothing is persisted locally until a feature needs it. The server is the source of truth
and Riverpod already caches in memory for the lifetime of a session. In a first version
the only things worth persisting are the session and a few preferences, which belong in
`flutter_secure_storage` and `shared_preferences`.

When a real offline or large-local-dataset requirement arrives, reach for `drift` rather
than ObjectBox: ObjectBox's Dart binding needs a native library and has no web support,
while drift runs on web through `sqlite3.wasm` plus a worker and on mobile through
`drift_flutter`, from one schema and one set of queries. Web needs `sqlite3.wasm` and
the worker served from `web/`; treat them as checked-in assets and verify a release
build actually loads them.

One database per app. A persistence layer owns it, repositories own remote access, and
ViewModels coordinate between them. Widgets never read or write it. Do not duplicate
state a provider already holds in memory.

## Secrets and tokens

- No secrets in the source tree. A base URL is configuration, not a secret, and arrives
  through `--dart-define`.
- Never log an access token, a refresh token or a password, and never log a response
  body from an authenticated endpoint.
- A failure message never carries a raw exception, a stack trace, or a URL with a token
  in it.
- `flutter_secure_storage` on web is encrypted `localStorage`, which is not real
  protection. The mitigation is a short-lived access token and an `HttpOnly` refresh
  cookie. Never move a refresh token into Dart-readable storage to make something more
  convenient.

## Lints

`strict-casts`, `strict-inference` and `strict-raw-types` on. `**/*.g.dart` excluded.
`missing_required_param` and `missing_return` raised to errors. Beyond
`flutter_lints`, these pull their weight: `always_declare_return_types`,
`avoid_dynamic_calls`, `avoid_print`, `cancel_subscriptions`, `close_sinks`,
`directives_ordering`, `omit_local_variable_types`, `prefer_const_constructors`,
`prefer_final_locals`, `prefer_relative_imports`, `prefer_single_quotes`,
`require_trailing_commas`, `unawaited_futures`, `use_super_parameters`.

`riverpod_lint` is an analysis-server plugin from 3.1 on, not a `custom_lint` plugin, so
it goes under `plugins:` in `analysis_options.yaml` and `flutter analyze` picks it up
directly.
