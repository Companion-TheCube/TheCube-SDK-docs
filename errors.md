# Errors

This document defines the shared error interpretation model for all SDKs.

## Core Error Categories

The local API implementation currently uses these endpoint error categories internally:

- `ENDPOINT_NO_ERROR`
- `ENDPOINT_INVALID_REQUEST`
- `ENDPOINT_INVALID_PARAMS`
- `ENDPOINT_INTERNAL_ERROR`
- `ENDPOINT_NOT_IMPLEMENTED`
- `ENDPOINT_NOT_AUTHORIZED`
- `ENDPOINT_NOT_FOUND`

These are mapped into HTTP responses by the API layer.

## Current HTTP Status Mapping

Current code maps errors as follows:

- `ENDPOINT_INVALID_REQUEST` -> `400 Bad Request`
- `ENDPOINT_INVALID_PARAMS` -> `400 Bad Request`
- `ENDPOINT_INTERNAL_ERROR` -> `500 Internal Server Error`
- `ENDPOINT_NOT_IMPLEMENTED` -> `501 Not Implemented`
- `ENDPOINT_NOT_AUTHORIZED` -> `403 Forbidden`

Observed special cases:

- Missing or invalid auth header on private HTTP endpoints returns `403 Forbidden`
- Schema validation failures return `400 Bad Request`
- Some handlers return `ENDPOINT_NOT_FOUND`, but the central switch does not yet consistently map that category to `404`
- Static file lookup failures in the builder can surface as not found conditions

Because `ENDPOINT_NOT_FOUND` is not yet normalized everywhere, SDKs should classify it carefully.

## Response Body Behavior

Current failure bodies are not fully standardized.

They may be:

- Plain text such as `Authorization header not present`
- Plain text such as `Schema validation failed: ...`
- JSON such as `{ "success": false, "message": "..." }`

SDK guidance:

- Determine parse strategy from `Content-Type` when present
- Preserve the raw response body on every error object
- Normalize a best-effort machine category separately from the raw payload

## Recommended Cross-SDK Error Model

Each SDK should normalize errors into these categories:

- `transport_error`
- `authentication_error`
- `authorization_error`
- `validation_error`
- `not_found_error`
- `not_implemented_error`
- `server_error`
- `unknown_api_error`

Suggested mapping:

- Connection refused, timeout, socket failure -> `transport_error`
- `401` or token exchange rejection semantics -> `authentication_error`
- `403` -> `authorization_error`
- `400` or schema validation failure -> `validation_error`
- Explicit not found semantics -> `not_found_error`
- `501` -> `not_implemented_error`
- `5xx` -> `server_error`
- Anything else -> `unknown_api_error`

## SDK Error Payload Shape

Every SDK should expose structured error details including:

- category
- HTTP status if present
- endpoint path
- method
- response content type if present
- raw response body
- parsed JSON body when valid JSON is available
- original transport exception where applicable

## Practical Rule

SDKs should never discard raw error data even when they expose typed exceptions. The API is still evolving, and preserving original wire data is necessary for debugging and future compatibility.
