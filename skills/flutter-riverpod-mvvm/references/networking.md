# Networking and failures

One client, built once, wrapped once. One place that turns a bad response into a typed
failure.

## The authenticated client

`core/network/authed_client.dart` holds an `http.BaseClient` subclass that:

- Adds `Authorization: Bearer <access token>` to every request.
- On a 401, refreshes once, then retries the original request once. A second 401 ends
  the session, and the router's redirect is what sends the viewer to login.
- Serialises concurrent refreshes behind a single in-flight future.
- Sets `Content-Type: application/json` unless the caller set its own.

```dart
final class AuthedClient extends http.BaseClient {
  final http.Client _inner;
  final SessionTokens _session;

  /// The in-flight refresh, shared by every request that hit a 401 while it
  /// was running. Nine panels loading at once must not trigger nine
  /// refreshes: the server rotates the refresh cookie on each call, so the
  /// second request would be refreshing with a token the first already spent.
  Future<String?>? _refresh;

  AuthedClient(this._inner, this._session);

  @override
  Future<http.StreamedResponse> send(http.BaseRequest request) async {
    // Snapshotted before the send, because sending finalizes the request and
    // a finalized one cannot go out twice. `null` for a streamed or multipart
    // body, which cannot be replayed, so those are not retried.
    final replay = _replayable(request);

    final response = await _inner.send(_authorize(request));
    if (response.statusCode != 401 ||
        replay == null ||
        _isRefreshRequest(request)) {
      return response;
    }

    // Hold the body: if the refresh fails this is still the answer the caller
    // has to see, and the stream can only be read once.
    final body = await response.stream.toBytes();

    final token = await _refreshOnce();
    if (token == null) {
      await _session.onSessionExpired();
      return _rebuild(response, body);
    }

    return _inner.send(_authorize(replay, token));
  }

  /// `close` is deliberately not overridden. The transport is owned by its own
  /// provider, which closes it; closing it here would give it two owners.
}
```

Four details that are easy to get wrong and expensive to debug:

1. **Snapshot the replay before sending.** Sending finalizes a request, and a finalized
   request cannot go out again. A streamed or multipart body cannot be replayed at all,
   so those requests are never retried.
2. **Buffer the 401 body before refreshing.** A response stream reads once. If the
   refresh fails, that body is still the answer the caller must see.
3. **Single-flight the refresh.** A server that rotates the refresh token on every call
   will reject the second concurrent refresh, because the first already spent the token.
4. **Do not override `close`.** The transport belongs to its provider. A repository
   handed a client never closes it. A feature that owns its own client disposes it in
   the provider's `onDispose`.

## Rules

- Base URL, timeouts and paths come from `core/constants/server_constants.dart`. No URL
  literals in a repository beyond its own path segment.
- `http` defaults a string body to `text/plain`, which a strict server rejects before a
  router sees it. Fix that once in the client, not at every call site.
- Never log a token, a password, or a response body from an authenticated endpoint.

## The sealed failure hierarchy

**`core/errors/failure.dart`**

```dart
sealed class Failure {
  final String message;

  const Failure(this.message);

  @override
  String toString() => message;
}

final class NetworkFailure extends Failure {
  const NetworkFailure(super.message);
}

final class AuthenticationFailure extends Failure {
  const AuthenticationFailure(super.message);
}

final class ParsingFailure extends Failure {
  const ParsingFailure(super.message);
}

/// One field the server rejected, out of a validation body.
final class FieldFailure {
  final String field;
  final String message;

  const FieldFailure({required this.field, required this.message});
}

final class ServerFailure extends Failure {
  final int statusCode;

  /// The machine-readable half of a `{code, message}` body, for the cases the
  /// UI has to branch on rather than merely display.
  final String? code;

  /// The per-field half of a validation body, empty for every other shape. A
  /// form reads this to mark its own fields.
  final List<FieldFailure> fieldFailures;

  const ServerFailure(
    super.message,
    this.statusCode, {
    this.code,
    this.fieldFailures = const [],
  });
}
```

`sealed`, so a `switch` over a failure is exhaustive and the compiler finds every site
when a case is added.

Messages are written for whoever reads them. An admin console can say what broke
precisely; a customer-facing app says less. Neither carries a raw exception, a stack
trace, a URL with a token in it, or another tenant's data.

## One place reads the server's error shapes

A FastAPI backend answers with `detail`, and `detail` has three shapes in practice:

- a plain string
- `{code, message}`, for a case the client must branch on
- `{message, errors: [{field, message}]}`, a rewritten 422

`ServerFailure.fromResponse` is the single place that untangles them. Every repository
reading an error response goes through it, so a shape the server adds is handled once.

```dart
factory ServerFailure.fromResponse(http.Response response) {
  final status = response.statusCode;
  final detail = _detailOf(response.body);

  return switch (detail) {
    final String text when text.trim().isNotEmpty => ServerFailure(
      text.trim(),
      status,
    ),
    final Map<String, dynamic> map when map['errors'] is List => ServerFailure(
      _stringOr(map['message'], _messageForStatus(status)),
      status,
      fieldFailures: _fieldFailures(map['errors'] as List<dynamic>),
    ),
    final Map<String, dynamic> map when map['code'] is String => ServerFailure(
      _stringOr(map['message'], _messageForStatus(status)),
      status,
      code: map['code'] as String,
    ),
    final Map<String, dynamic> map when map['message'] is String =>
      ServerFailure(
        _stringOr(map['message'], _messageForStatus(status)),
        status,
      ),
    _ => ServerFailure(_messageForStatus(status), status),
  };
}

static String _messageForStatus(int status) => switch (status) {
  400 => 'The server rejected that request.',
  401 => 'Your session has expired.',
  403 => 'That needs a different account.',
  404 => 'That is not on the server.',
  409 => 'That conflicts with what the server already holds.',
  422 => 'The server could not accept those values.',
  429 => 'Too many requests. Wait a moment and try again.',
  >= 500 => 'The server failed while answering (HTTP $status).',
  _ => 'The server answered with HTTP $status.',
};
```

The fallback matters as much as the parsing. A body that is not JSON at all, which is
what a proxy returning HTML on a 502 looks like, must become a message derived from the
status code and never surface as a parse error.

This is the highest-value test in the repository. Getting it wrong turns every server
error into "something went wrong".
