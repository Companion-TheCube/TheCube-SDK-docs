# Reliability, Pagination, Retries, And Idempotency

This document defines the cross-SDK reliability model for the local CORE API.

## Pagination

There is no platform-wide pagination contract in the current device-local API.

Current state:

- No shared `limit`, `offset`, `cursor`, or `next_cursor` wire format is defined across endpoints
- Some internal components use local limits, but that is not yet a stable API-wide pagination standard

SDK guidance:

- Do not invent automatic pagination behavior for generic endpoint wrappers
- Treat pagination as endpoint-specific until the platform standardizes it
- If a future endpoint adds pagination, expose it explicitly in the typed wrapper for that endpoint

## Retries

The local CORE API does not publish a general retry contract today.

SDK guidance:

- Retry only transport-level failures by default
- Do not automatically retry non-idempotent requests
- Do not retry `4xx` responses except where the caller explicitly handles auth renewal or rate limiting

Recommended defaults:

- Retry connect failures and transient socket resets up to 2 times
- Use exponential backoff starting at 100 ms and capping near 1 second
- Add jitter for network HTTP calls
- Do not retry schema validation failures, auth failures, or semantic request failures

## Idempotency

The current local API does not advertise explicit idempotency keys or request IDs.

SDK guidance by method:

- `GET`: treat as safe to retry unless the endpoint is documented as stateful
- `POST`: treat as non-idempotent unless the endpoint wrapper explicitly marks it safe

Examples:

- Discovery calls such as `/getEndpoints` are safe to retry
- Control actions such as `/AudioManager-start` and `/AudioManager-toggleSound` should not be blindly retried without caller intent
- Storage writes such as `/CubeDB-saveBlob` should not be retried automatically unless the caller provides deduplication logic

## Timeouts

SDKs should expose timeouts at three levels:

- connect timeout
- request timeout
- total operation timeout including retries

Recommended defaults:

- Local Unix socket connect timeout: 1 to 2 seconds
- Loopback HTTP connect timeout: 2 to 3 seconds
- Default request timeout: 10 to 15 seconds

## Readiness And Discovery

The Core source includes a note about a future readiness endpoint, but there is no dedicated readiness contract in the current API.

SDK guidance:

- Treat a successful response from `/getEndpoints` or `/openapi.json` as practical readiness for now
- Surface discovery failures as connection or availability errors

## Rate Limiting

There is no API-wide HTTP rate-limit contract or shared response format today.

If future endpoints introduce rate limiting, SDKs should expose the raw headers and status code rather than attempting to generalize ahead of time.
