# TheCube CORE SDK Shared Docs

This folder contains documentation that applies to every SDK in this repository.

The goal is to define the common client contract for talking to TheCube CORE, regardless of implementation language.

## Documents

- `transports.md` describes how SDKs connect to CORE over HTTP and Unix domain sockets.
- `endpoints.md` describes endpoint discovery, path structure, verbs, and naming.
- `authentication.md` describes the current local auth model for private endpoints.
- `schemas.md` describes request and response schema conventions and discovery.
- `reliability.md` describes pagination status, retries, timeouts, and idempotency guidance.
- `errors.md` describes status codes, error categories, and SDK normalization guidance.

## Scope

These docs are based on the current implementation in `Companion-TheCube---Core`, especially the device-local API subsystem and the endpoint metadata type `HttpEndPointData_t`.

Where the platform does not yet expose a stable, uniform rule in code, these docs say so directly and provide SDK guidance that should be implemented consistently across all language SDKs.

## Source Of Truth

For a running CORE instance, SDKs should treat these discovery endpoints as authoritative:

- `GET /getEndpoints`
- `GET /openapi.json`
- `GET /getCubeSocketPath`

These docs are the stable cross-language interpretation of those runtime contracts.
