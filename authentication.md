# Authentication

This document describes the current authentication model for private CORE endpoints.

## Security Split

CORE distinguishes between public and private endpoints.

Current behavior:

- Public endpoints are exposed over both TCP HTTP and Unix socket
- Private endpoints require an `Authorization` header on the TCP HTTP server
- The Unix socket server exposes endpoints without the HTTP-side authorization gate

This means SDKs should treat the Unix socket as the preferred trusted local transport, and authenticated HTTP as the preferred network transport.

## Header Format

CORE accepts an `Authorization` header and strips a `Bearer ` prefix when present.

SDK guidance:

- Always send `Authorization: Bearer <token>` when using authenticated HTTP
- Be tolerant of legacy code paths that may have stored the raw token value without the prefix
- Normalize to Bearer format in all SDK examples and public APIs

## Token Acquisition Flow

The current client-oriented auth flow is implemented by `CubeAuth` endpoints.

### 1. Obtain Or Register A Client Identity

The auth subsystem stores client records in the local auth database.

### 2. Request An Initial Code

CORE exposes an endpoint that issues an initial code tied to a client identity.

### 3. Exchange For A Token

`GET /CubeAuth-authHeader` accepts:

- `client_id`
- `initial_code`

On success it returns JSON including:

- `success`
- `message`
- `auth_code`

The returned `auth_code` is the token that should be sent in the `Authorization` header for private HTTP endpoints.

### 4. Rotation And Revocation

CORE also defines token rotation and revocation endpoints.

## Token Lifetime

Current implementation issues client tokens with a 24 hour expiry and records:

- issue time
- expiry time
- last used time
- revoked status

SDK guidance:

- Treat tokens as short-lived
- Support token refresh or rotation in client abstractions
- Surface `403` errors as authentication failures, not generic transport failures

## Auth Failure Semantics

Current HTTP behavior for private endpoints:

- Missing `Authorization` header: `403`
- Empty `Authorization` header: `403`
- Invalid token: `403`

SDK guidance:

- Raise a dedicated authentication or authorization error type
- Include whether the failure was missing credentials or rejected credentials when the response body makes that distinction visible

## Recommended SDK Surface

Every SDK should provide:

- A token type or auth context wrapper
- Automatic Bearer header injection for private HTTP calls
- A transport selector that avoids auth when using the trusted local Unix socket
- Token rotation and revocation helpers once those endpoints are wrapped
