# Karkain API Manager

A CLI/TUI-configured HTTP routing and API-lifecycle service built exclusively on Karkain Language v1.2.0.

## Status

| Item | Status |
|---|---|
| Research | COMPLETE |
| Product design | COMPLETE |
| Repository creation | COMPLETE |
| Implementation | NOT STARTED |
| Compile/execute verification of the application | NOT STARTED (toolchain itself VERIFIED on Karkain v1.2.0) |

## What it does

Karkain API Manager brings together API lifecycle management and a lightweight HTTP gateway: a declarative route table, versioned API definitions, policies, rate limits and local access analytics. It is configured from text files and driven by a CLI/TUI service, with stable text and JSON output. It is an API-management and proxy tool, not a server manager, not a network device manager and not a network monitor. Its gateway mode is explicitly a sequential building-block choreography over Karkain's `http` module — not a production multi-client proxy.

## Who it serves

It serves small development teams, APIOps engineers and air-gapped environments wanting a single-file-configured, database-free proxy and catalogue without a control plane or per-request metering. Developers who want declarative, reviewable routing rules in version control are the primary audience.

## Planned features

All items below are **(planned)** and are specification targets, not implemented behaviour.

- **API lifecycle (planned):** register, publish, deprecate and retire API definitions; semantic versions and aliases.
- **Routing (planned):** declarative path/method → upstream mapping; prefix and exact matching; route validation and dry-run.
- **Policies (planned):** file-defined request/response rules (header rewrite, simple transforms, allow/deny) applied in order.
- **Rate limiting (planned):** in-process token-bucket limits per consumer/route.
- **Versioning (planned):** version aliases, deprecation windows and route-to-version binding.
- **Analytics (planned):** append-only local access log and counters (status, latency, consumer) with text/JSON reports.
- **Gateway mode (planned):** single sequential `http_listen`/`accept`/`read_request`/`write_response` loop for development and testing.
- **CLI/TUI admin (planned):** declarative configuration, validation, dry-run and stable text/JSON output.

## Explicit non-goals

- Not a server/application manager (Karkain Server Manager).
- Not a network device manager (Karkain Network Manager).
- Not a network performance monitor (Karkain Network Monitor).
- No TLS/HTTPS termination or upstream TLS, no WebSocket, no UDP/QUIC, no concurrent multi-client gateway, no native GUI.
- No modification of the Karkain language, compiler or standard library.

## Karkain foundation

Exclusive foundation: Karkain Language v1.2.0. All application source will be Karkain. No other implementation language. No compiler modifications.

## Platform support status

- Windows x86-64 toolchain: **VERIFIED** (Karkain v1.2.0 compiles and runs a native executable on this host).
- Linux cross-target output: **PRESENT but execution NOT VERIFIED** (cross-compilation output exists; it was not executed on this host).
- macOS: **NOT VERIFIED**.

## Build and run status

No application binary exists yet. The intended generic commands, which become active when implementation starts, are:

```
karkain build src/main.kark -o <name>
karkain run src/main.kark
```

These are placeholders for the planned source layout, not verified commands for this application.

## Known blockers

- **TLS** — HTTPS termination and upstream TLS are BLOCKED; no secure gateway claim is credible until this exists.
- **Concurrent HTTP server framework** — only sequential HTTP building blocks exist; a production multi-client gateway is BLOCKED.
- **WebSocket** — real-time API features are BLOCKED.
- **UDP/QUIC** — datagram/QUIC transport is BLOCKED.
- **GUI** — no native GUI toolkit; GUI is a planned, blocked layer.
- **JSON** — only partial field extraction; no full JSON DOM or schema (PARTIAL).

## Repository

- Source: https://github.com/ajit-ai/Karkain-API-Manager
- Issues: https://github.com/ajit-ai/Karkain-API-Manager/issues
- License: Apache-2.0
