# Endpoints

This document defines the shared endpoint model used by TheCube CORE.

## Endpoint Metadata Source

CORE endpoint definitions are produced by interface implementations returning `HttpEndPointData_t`.

The tuple shape is:

```text
(endpoint flags, action, endpoint name, request schema, description)
```

Each SDK should understand this as the canonical model behind the runtime discovery documents.

## Path Structure

Interface endpoints follow this path pattern:

```text
/InterfaceName-endpointName
```

Examples:

- `/AudioManager-start`
- `/AudioManager-setSound`
- `/CubeAuth-authHeader`
- `/CubeDB-saveBlob`

The hyphen between interface name and endpoint name is part of the current contract.

## Endpoint Classes

There are three broad classes of endpoints:

1. Discovery endpoints
2. Interface endpoints
3. Static file endpoints

### Discovery Endpoints

These are added centrally by the API builder:

- `GET /getEndpoints`
- `GET /getCubeSocketPath`
- `GET /openapi.json`

### Interface Endpoints

These come from registered components implementing the API interface contract.

The available interface set depends on runtime composition and build flags.

### Static File Endpoints

Files under CORE's `http/` folder are exposed as public GET endpoints.

SDKs generally should not treat these as part of the programmatic SDK surface unless a specific UI integration requires them.

## HTTP Methods

Current endpoint metadata only distinguishes between:

- `GET`
- `POST`

There is no shared REST-wide use of `PUT`, `PATCH`, or `DELETE` for the local CORE API at this time.

SDK guidance:

- Preserve the declared method from discovery metadata
- Do not infer method semantics from the endpoint name
- Do not automatically remap `POST` actions into other verbs

## Naming Conventions

Current naming patterns in code are mixed but consistent enough to document:

- Interface names are PascalCase, for example `CubeAuth`, `AudioManager`, `CubeDB`
- Endpoint names are usually lowerCamelCase or short verbs, for example `authHeader`, `setSound`, `saveBlob`
- Full wire paths are case-sensitive and should be treated as exact strings

SDK guidance:

- Preserve exact server path casing on the wire
- Expose idiomatic language bindings in each SDK if desired
- Keep a stable mapping from idiomatic client method names back to exact wire names

## Discovery Contract

`GET /getEndpoints` currently returns JSON grouped by interface name. Each endpoint entry includes:

- `name`
- `params`
- `description`
- `public`
- `endpoint_type`

`GET /openapi.json` currently returns an OpenAPI-like document with:

- `openapi`
- `info`
- `paths`

The current OpenAPI output focuses on request schemas and summary text. It does not yet define a fully standardized response schema catalog.

## SDK Recommendations

Every SDK should provide:

- Raw discovery access to `getEndpoints`, `getCubeSocketPath`, and `openapi.json`
- A generated or normalized in-memory endpoint catalog
- A low-level request API that can call any discovered endpoint
- Higher-level typed helpers for stable interfaces once implemented
