# The four layers

One feature, four directories, each with one job. The shapes below are the ones to
copy.

## Dependency injection

There is no service locator. Providers are the graph and `ProviderScope` overrides are
the only injection point.

**`lib/init_dependencies.dart`**

```dart
part 'init_dependencies.main.dart';

/// Overridden in [initDependencies]. Reading it without that override is a
/// programming error, which is what the throw says.
final secureStorageProvider = Provider<FlutterSecureStorage>(
  (ref) => throw UnimplementedError('initDependencies() was not awaited'),
);

Future<List<Override>> initDependencies() => _initDependencies();
```

**`lib/init_dependencies.main.dart`**

```dart
part of 'init_dependencies.dart';

Future<List<Override>> _initDependencies() async {
  final storage = await _initSecureStorage();

  return [
    secureStorageProvider.overrideWithValue(storage),
  ];
}
```

**`lib/main.dart`**

```dart
Future<void> main() async {
  WidgetsFlutterBinding.ensureInitialized();

  final overrides = await initDependencies();

  runApp(ProviderScope(overrides: overrides, child: const MyApp()));
}
```

The throwing default is deliberate: it turns a missing override into an immediate,
legible crash instead of a null somewhere later. Tests inject by overriding the same
providers, which is the whole point of this shape.

## Model

Strongly typed, serializable where needed, free of logic unrelated to its own data, and
independent of Flutter. Models more than one feature reads live in `core/models/`.

```dart
part 'my_model.g.dart';

@JsonSerializable()
class MyModel {
  @JsonKey(name: 'account_id')
  final String accountId;

  @JsonKey(name: 'legal_name')
  final String legalName;

  const MyModel({required this.accountId, required this.legalName});

  factory MyModel.fromJson(Map<String, dynamic> json) =>
      _$MyModelFromJson(json);

  Map<String, dynamic> toJson() => _$MyModelToJson(this);
}
```

- `final` everywhere.
- `@JsonKey` whenever the API's name differs from Dart naming, which against a
  `snake_case` API is most of the time.
- A default where a default is genuinely correct, `null` where "nobody has said" is a
  real state. A three-state nullable boolean must not collapse to `false`.
- Money arrives as a JSON number. Parse it into `Decimal` or an integer number of the
  smallest unit, never a `double`, anywhere it is summed or compared.
- Serialization stays in the model, never in a widget or a ViewModel.

## Repository

Repositories talk to external data sources. They expose capabilities through an abstract
contract, return `Future<Either<Failure, T>>` for anything that can fail, handle status
codes and network exceptions, and convert JSON into typed models.

They never leak `http.Response`, `http.Client` or `Map<String, dynamic>` through the
contract, and never touch `WidgetRef`, `BuildContext` or UI state.

**`repository/my_repository.dart`**

```dart
abstract interface class MyRepository {
  Future<Either<Failure, MyModel>> fetchData();

  Future<Either<Failure, MyModel>> updateData(MyModel data);
}
```

Declare only the operations the feature actually uses. A contract is not an inventory of
the API.

**`repository/my_remote_repository.dart`**

```dart
part 'my_remote_repository.g.dart';

@riverpod
MyRepository myRepository(Ref ref) =>
    MyRemoteRepository(ref.watch(authedClientProvider));

class MyRemoteRepository implements MyRepository {
  final http.Client _client;

  MyRemoteRepository(this._client);

  @override
  Future<Either<Failure, MyModel>> fetchData() async {
    try {
      final response = await _client
          .get(Uri.parse('${ServerConstants.baseUrl}/my/endpoint'))
          .timeout(ServerConstants.requestTimeout);

      if (response.statusCode == 401) {
        return const Left(AuthenticationFailure('Your session has expired.'));
      }

      if (response.statusCode != 200) {
        return Left(ServerFailure.fromResponse(response));
      }

      return Right(
        MyModel.fromJson(jsonDecode(response.body) as Map<String, dynamic>),
      );
    } on TimeoutException {
      return const Left(NetworkFailure('The server took too long to answer.'));
    } on SocketException {
      return const Left(NetworkFailure('No internet connection.'));
    } on FormatException {
      return const Left(ParsingFailure('The server sent something unexpected.'));
    } on http.ClientException catch (e) {
      debugPrint('HTTP error: $e');
      return const Left(NetworkFailure('The request could not be completed.'));
    } catch (e) {
      debugPrint('Unexpected error: $e');
      return const Left(Failure('Something went wrong.'));
    }
  }
}
```

Every one of those branches is a test. See `testing.md`.

## ViewModel

Holds feature logic and presentation state. Uses code generation, depends on the
repository interface rather than on `http`, never decodes JSON, never touches
`BuildContext`, and exposes typed state covering idle, loading, success and failure.

```dart
part 'my_viewmodel.g.dart';

@riverpod
class MyViewModel extends _$MyViewModel {
  late MyRepository _repository;

  @override
  AsyncValue<MyModel>? build() {
    _repository = ref.watch(myRepositoryProvider);
    return null;
  }

  Future<void> fetchData() async {
    state = const AsyncValue.loading();

    final result = await _repository.fetchData();

    if (!ref.mounted) return;

    switch (result) {
      case Left(value: final failure):
        state = AsyncValue.error(failure, StackTrace.current);
      case Right(value: final data):
        state = AsyncValue.data(data);
    }
  }
}
```

The `if (!ref.mounted) return` is not optional. A response arriving after the provider
is disposed otherwise writes to a dead notifier.

### Guarding out-of-order responses

```dart
@riverpod
class MyViewModel extends _$MyViewModel {
  late MyRepository _repository;
  int _request = 0;

  @override
  AsyncValue<MyModel>? build() {
    _repository = ref.watch(myRepositoryProvider);
    return null;
  }

  Future<void> fetchData() async {
    final request = ++_request;
    state = const AsyncValue.loading();

    final result = await _repository.fetchData();

    if (!ref.mounted || request != _request) return;

    switch (result) {
      case Left(value: final failure):
        state = AsyncValue.error(failure, StackTrace.current);
      case Right(value: final data):
        state = AsyncValue.data(data);
    }
  }
}
```

Needed for anything with a refresh, a filter or a date range. Without it the slower of
two overlapping requests wins, and the screen shows the old window under the new label.

## View

`ConsumerWidget` for presentation, `ConsumerStatefulWidget` only when a widget lifecycle
is genuinely needed, `StatelessWidget` for anything that needs no provider at all.

### Page rules

- Watch only the providers the page needs. A page watching one provider that holds
  everything rebuilds entirely when any part of it changes.
- No business logic in `build()`.
- Exhaustive state handling, every branch drawn.
- Never trigger a request from `build()`.
- Loading is a skeleton shaped like the content, not a centred spinner.
- An error in one panel degrades that panel, never the page.
- Delegate sections to widgets. A page file past a few hundred lines should have been
  three widgets.

### Widget rules

- Data in through the constructor.
- Never call a repository from a widget.
- Prefer `ListTile` over a hand-rolled `Row` inside a fixed-height `SizedBox`. The
  latter is how rows end up top-aligned instead of centred.
