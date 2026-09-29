# PokéAPI Kotlin SDK

All the Pokémon data you'll ever need in one place, easily accessible through a modern free open-source RESTful API.

## What is this?

This is a full RESTful API linked to an extensive database detailing everything about the Pokémon main game series.

We've covered everything from Pokémon to Berry Flavors.

## Where do I start?

We have awesome [documentation](https://pokeapi.co/docs/v2) on how to use this API. It takes minutes to get started.

This API will always be publicly available and will never require any extensive setup process to consume.

Created by [**Paul Hallett**](https://github.com/phalt) and other [**PokéAPI contributors**](https://github.com/PokeAPI/pokeapi#contributing) around the world. Pokémon and Pokémon character names are trademarks of Nintendo.

> Package `dev.octri.demo:pokeapiUnofficialSdk` · Version `2.10.1` · 102 operations

## Installation

Install the delivered checkout into your local Maven repository:

```sh
gradle publishToMavenLocal
```

Then add the dependency to your own `build.gradle.kts`:

```kotlin
repositories {
    mavenLocal()
}

dependencies {
    implementation("dev.octri.demo:pokeapiUnofficialSdk:2.10.1")
}
```

## Quickstart

The example calls `meta_retrieve` (GET `/api/v2/meta/`), a low-friction operation that requires no request arguments.

```kotlin
import dev.octri.demo.pokeapiUnofficialSdk.*

fun main() {
    val config = ClientConfig(
        baseUrl = "https://pokeapi.co",
    )
    val client = PokAPI(config)
    val result = client.utility.metaRetrieve()
    println(result)
}
```

## Authentication

This build does not declare an authentication scheme. If the service requires a custom credential, set it through `ClientAuthConfig.headers`; those headers are merged into every request.

## Client behavior

- Base URL: `https://pokeapi.co`.
- Transport: OkHttp.
- Timeout: 30,000 ms per attempt.
- Retries: up to 3 attempts for status codes `408`, `425`, `429`, `500`, `502`, `503`, `504`, with 500–8,000 ms backoff.
- Idempotency: enabled for `POST`, `PATCH` using `Idempotency-Key`.
- Error telemetry: off. This build has no reporting endpoint and sends no error reports.

High-level operation methods return the typed response body directly. The low-level request layer returns an `SdkResponse<T>` envelope containing data, status, headers, request ID, latency, and attempt count.

## Errors and response metadata

All failure paths use a small, predictable hierarchy:

| Error | Meaning |
| --- | --- |
| `SdkValidationError` | A request argument failed an OpenAPI constraint before network I/O. |
| `SdkHttpError` | The server returned a non-2xx response. |
| `SdkNetworkError` | DNS, connection, TLS, or socket failure. |
| `SdkTimeoutError` | The configured per-attempt timeout elapsed. |

HTTP errors expose `statusCode`, the response body and headers, plus `requestId` when the server supplies one. Preserve the request ID in support logs; it is the fastest way to correlate a failed SDK call with server-side traces.

Common statuses are reported as a subclass of `SdkHttpError`, so a handler can catch only the one it handles: `SdkBadRequestError` (400), `SdkUnauthorizedError` (401), `SdkPermissionDeniedError` (403), `SdkNotFoundError` (404), `SdkConflictError` (409), `SdkUnprocessableEntityError` (422), `SdkRateLimitError` (429), `SdkInternalServerError` (any 5xx). Any other status is reported as `SdkHttpError` itself.

When the API declares a model for an error response, `error.decodeBody<ErrorModel>()` decodes the body into it, where `ErrorModel` is that model. The raw body stays available on the error.

## Pagination

50 operations expose generated pagination helpers. Each operation has a `Paginated` companion whose callback can stop iteration early.

Pagination follows the cursor, offset, page-number, or next-URL contract declared by the OpenAPI operation. It stops when the API signals completion and does not prefetch the entire collection.

## Project layout and API discovery

- Operation implementations are grouped under `src/main/kotlin/dev/octri/demo/pokeapiUnofficialSdk/methods/`.
- 278 component models are split by API domain under `src/main/kotlin/dev/octri/demo/pokeapiUnofficialSdk/models/Models<Domain>.kt` or `models/<tag path>/Models.kt`; real declarations follow those model packages and `dev.octri.demo.pokeapiUnofficialSdk.Models.kt` provides compatibility aliases.
- Component schemas can choose a nested model folder with `x-octri-sdk-tags: ["Billing/Invoices"]`; the first tag owns the model and `/` creates nesting.
- [`sdk-manifest.json`](sdk-manifest.json) is the language-neutral public API index: operations, request/response modes, model properties, enum values, and generation settings.
- Public barrel/module exports are the compatibility boundary. Import public model names from those exports; internal domain filenames may evolve without changing model names.

## Links

- [API documentation](https://pokeapi.co/docs/v2)

<!-- sdk-studio-mock-tests -->
## Local mock-server tests

Generated SDK includes schema-derived, zero-dependency mock server and network
contract suite. Node.js 20+ required. Contract probes use authored response
examples only; schema-synthesized routes remain available to the local server.

`./scripts/mock --port 4010` starts server. `./scripts/test` runs the mock contract suite, then native SDK tests. A zero-authored-example contract run succeeds with an explicit zero-test
summary; mismatches in authored examples still fail.
