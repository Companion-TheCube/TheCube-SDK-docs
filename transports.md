# Transports

This document defines how SDKs should connect to TheCube CORE.

## Supported Transports

The current CORE implementation exposes the same device-local API over two transports:

1. TCP HTTP
2. HTTP over a Unix domain socket

The protocol is still HTTP in both cases. The difference is only the underlying transport.

## Default Bindings

Current code defaults are:

- HTTP address: `0.0.0.0`
- HTTP port: `55280`
- IPC socket path: `cube.sock`

These values can be overridden by environment variables loaded from the CORE process working directory:

- `HTTP_ADDRESS`
- `HTTP_PORT`
- `IPC_SOCKET_PATH`

## Client Addressing Rules

SDKs should apply this connection order:

1. If a socket path is explicitly configured by the caller, use HTTP over Unix socket.
2. Otherwise, if the SDK is running on the same machine as CORE, resolve the socket path from `GET /getCubeSocketPath` and prefer the Unix socket for local privileged access.
3. Otherwise, use TCP HTTP against `127.0.0.1:55280` for same-host loopback access or the configured host and port when the user intentionally exposes CORE on the network.

`TheCube.local` can be supported as a convenience hostname, but SDK defaults should prefer explicit loopback or explicit host configuration rather than relying on multicast DNS.

## Why SDKs Should Prefer The Unix Socket Locally

Current CORE behavior exposes all endpoints on the Unix socket without the HTTP-side public/private split. That makes the Unix socket the intended local transport for trusted device-local apps and SDK consumers.

In practice:

- HTTP server: public endpoints are open, private endpoints require auth
- IPC socket server: all endpoints are reachable over the local socket

SDKs should still support authenticated HTTP for private endpoints, but local-first SDK behavior should prefer the Unix socket when possible.

## TLS Behavior

CORE creates an HTTPS-capable server if `ssl/server.crt` and `ssl/server.key` exist in the process working directory. Otherwise it starts a plain HTTP server.

SDK guidance:

- Default to plain HTTP for local device access
- Allow callers to opt into HTTPS when the deployment provides certificates
- Do not assume HTTPS is always available

## Timeouts

Current server-side settings are:

- Read timeout: 10 seconds
- Write timeout: 10 seconds
- Keep-alive timeout: 10 seconds

SDK guidance:

- Default connect timeout: 3 to 5 seconds
- Default request timeout: 10 to 15 seconds
- Allow per-request overrides
- Treat socket connect failures as transport errors, not API errors

## Discovery Example

TCP HTTP:

```text
GET http://127.0.0.1:55280/getEndpoints
GET http://127.0.0.1:55280/getCubeSocketPath
```

Unix socket:

```text
GET /getEndpoints over HTTP on the resolved IPC socket
GET /openapi.json over HTTP on the resolved IPC socket
```
