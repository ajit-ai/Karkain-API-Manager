# Karkain API Manager — Product Specification

**Document status:** Design COMPLETE. Research = COMPLETE. Repository creation = COMPLETE. Application implementation = NOT STARTED. Application compile/execute verification = NOT STARTED (toolchain path itself VERIFIED on Karkain v1.2.0).


## A. Product definition

**Product name:** Karkain API Manager.

**Purpose and problem:** Small teams and air-gapped environments need API lifecycle management and a simple HTTP routing/proxy layer without a database, a control plane, or per-request metering. Incumbent gateways and API platforms are powerful but browser-GUI, multi-component and metered. This product supplies a declarative, file-configured API catalogue and a development-oriented HTTP proxy built solely in Karkain v1.2.0.

**Vision:** A reviewable, Git-friendly API-management workspace where routes, versions, policies and rate limits are text, and where every request is recorded locally — within the explicit limits of the Karkain runtime.

**Intended users and segments:** APIOps and platform engineers; small development teams; integrators in regulated or air-gapped sites; developers wanting a local dry-run gateway before deploying elsewhere.

**Usage scenarios:** (1) Register an API with versions and publish it. (2) Define a route mapping an inbound path to an upstream endpoint. (3) Attach policies and a rate limit. (4) Dry-run the route table against a sample request. (5) Run the single-threaded gateway for local testing and inspect local analytics.

**Market positioning:** A lightweight, CLI-first complement for development, testing and constrained sites — not a replacement for a production gateway. It targets the "single-binary, file-configured proxy and catalogue" gap.

**Primary value proposition:** Declarative routing, local analytics and no metering or control plane, with the honest caveat that TLS, concurrency, WebSocket and UDP are BLOCKED and must be resolved at the language level before production gateway claims.

**Explicit non-goals and boundaries:** No server/application management, network device management or network monitoring. No TLS/HTTPS, WebSocket, UDP/QUIC, concurrent multi-client serving or native GUI. No language/compiler modification.

## B. Functional architecture

| Module | Responsibility |
|---|---|
| API Catalogue | Register, publish, deprecate and retire API definitions and semantic versions. |
| Route Table | Declarative path/method → upstream mapping; exact/prefix match; validate and dry-run. |
| Policy Engine | Ordered file-defined request/response rules (header rewrite, simple transforms, allow/deny). |
| Rate Limiter | In-process token-bucket enforcement per consumer/route. |
| Version Manager | Version aliases, deprecation windows and version-to-route binding. |
| Gateway Service | Sequential `http` building-block loop: listen, accept, read, apply, forward, respond. |
| Analytics | Append-only local access log and counters; text/JSON reports. |
| Config Loader | Reads declarative text config via `config`/`io`; validates before load. |
| CLI/TUI Frontend | Command parsing, dry-run, text/JSON output. |

**Core workflows:** Define API → publish version → define route → attach policy/rate limit → validate → dry-run → run gateway locally → inspect analytics.

**Admin workflows:** Publish/deprecate versions, edit policy order, set rate limits, rotate upstream references, export configuration.

**Error/recovery workflows:** Invalid config is rejected at load with field/line detail; an unreachable upstream produces a structured 502-style response and an analytics event; malformed requests are rejected without state change; a full rate bucket returns a throttled response; no crash leaves partially applied policies.

**Roles and permissions:** Local roles (read, edit, publish, admin) gate catalogue and policy changes. No remote authentication is claimed because TLS is BLOCKED.

**Configuration/customization:** File-based declarative rules; no database required; config validated before activation.

**Import/export:** Config and catalogue export as text/simple JSON; `json` is PARTIAL, so only simple field extraction is assumed and no full schema is claimed.

**Integration requirements:** Upstream calls via the `http` client building blocks; no service discovery or control-plane integration is assumed. Webhooks requiring TLS are BLOCKED.

## C. Competitor analysis

Researched competitors and sources are in `docs/competitor-analysis/COMPETITORS.md`, derived from portfolio research. Evidence labels: **[VF]** verified fact, **[RI]** research-based interpretation, **[PROP]** Karkain proposal, **[BLOCKED]** unattainable with v1.2.0, **[UA]** unverified assumption.

**Competitors:** Kong Gateway, Google Apigee, and Tyk.

**[VF]** Kong is a lightweight NGINX-based cloud-native reverse proxy with OSS, Enterprise and Konnect tiers and a plugin model (Lua, Go, Python, JavaScript); Apigee is a full-lifecycle API platform (REST/SOAP/GraphQL/gRPC) with policies, products, portals and analytics; Tyk is a batteries-included OSS gateway (Gateway/Pump OSS; dashboard/portal/operator licensed). **[RI]** Kong suits APIOps teams, Apigee governed enterprises, Tyk OSS-oriented teams.

**Feature comparison matrix (Karkain = proposed):**

| Capability | Karkain (proposed) | Kong Gateway | Apigee | Tyk |
|---|---|---|---|---|
| CLI/TUI + file config (GUI **[BLOCKED]**) | **[PROP]** yes | API, decK, GUI | UI, docs | REST API, Dashboard |
| Concurrent multi-client proxy **[BLOCKED]** | No | Yes | Yes | Yes |
| HTTPS/TLS termination **[BLOCKED]** | No | Yes | Yes | Yes |
| Plugin/PDK extensibility | Config/text rules only | Lua/Go/Python/JS | Policies | Go plugins |
| No usage metering **[PROP]** | Yes | Konnect metered | Per-call + env | Quote-based |
| Free OSS core | **[PROP]** Apache-2.0 product | Yes (licence UNVERIFIED) | No | Yes (MPL 2.0, third-party) |

**Platform/deployment comparison [VF]:** Kong supports on-prem, cloud, Kubernetes and serverless with DB, hybrid or DB-less declarative modes; Apigee offers a sandbox, pay-as-you-go, subscription and hybrid on GKE; Tyk is a Go binary plus Redis with self-managed/hybrid/cloud options. Karkain is planned as a single Karkain-built CLI/TUI service and sequential gateway on the VERIFIED Windows x86-64 toolchain, with other targets unverified.

**Licensing comparison:** **UNKNOWN** — official pages cited in `COMPETITORS.md` do not give a verified licence for Kong OSS; Tyk's MPL 2.0 is reported third-party, not verified on an official page; Apigee is a commercial subscription. No verified licence comparison can be made. **[VF]** Official pricing figures (Konnect Plus from $25/month plus usage, $200 per extra 1M/month; Apigee 60-day free sandbox, Standard $20 per 1M calls, Extensible $100 per 1M, environments from $365/month/region, deployments $0.04/hour/region; Tyk prices UNKNOWN) are reproduced in `COMPETITORS.md`. No prices are invented.

**Strengths/limitations [VF]/[RI]:** Kong is deployment-agnostic and plugin-rich but RBAC/SSO/analytics are behind Enterprise/Konnect; Apigee has deep governance but hybrid is GKE-centric; Tyk is batteries-included. Karkain's strength is text-native simplicity and no metering; its limitation is the BLOCKED primitive set, which prevents a production gateway claim.

**Market gaps [RI]:** Small teams want a single-binary, file-configured proxy with no database, control plane or per-request metering; air-gapped operation is underserved.

**Karkain differentiation [PROP]:** CLI/TUI-configured HTTP router with declarative rules, local logging, service mode and text output. **[BLOCKED]** no TLS, no concurrent server framework, no WebSocket, no UDP/QUIC — all must be cleared before any gateway claim is credible.

**Features deliberately not copied:** Plugin PDKs in four languages, developer portals and monetisation, service mesh/ingress controllers, AI gateway features, distributed control/data-plane splits and per-request billing.

## D. Technical architecture

**Application architecture:** A single process with a command dispatcher, catalogue/route/policy domain modules and a text/db2 persistence layer; a foreground "service mode" runs the gateway loop.

**Component boundaries:** Catalogue, Route Table, Policy Engine, Rate Limiter, Version Manager and Analytics are separate modules. All network I/O is isolated in the Gateway Service so BLOCKED transports remain confined.

**Process/service boundaries:** One process, one sequential request path. No daemon, worker pool or multi-process model is claimed.

**UI architecture:** The UI is CLI/TUI/service-first because Karkain v1.2.0 has no native GUI toolkit (BLOCKED). GUI is a planned layer contingent on ecosystem capability and is not in the MVP.

**Domain model:** Api, Version, Route, Policy, Consumer, RateLimit, AccessEvent.

**Data storage strategy:** Plain text config files (declarative, reviewable) plus embedded `db2` for catalogue counters and analytics events.

**API/interface architecture:** Internal Karkain module calls. Externally, a single sequential HTTP request loop using `http_listen/accept/read_request/write_response`; no routing framework exists in the stdlib — routing is Karkain-native code.

**Event/message architecture:** In-process access events appended to a local log; no broker assumed.

**Logging/diagnostics/observability:** Structured text/JSON access lines to stdout and an append-only analytics log; `karkain lint`/`check --format=json` and the `test` runner provide diagnostics.

**Configuration management:** Declarative text loaded via `config`/`io`, validated before activation; no external config service.

**Plugin/extension architecture:** Ordered text policies are the extension point; no dynamic plugin loading. Any FFI/OS integration must be a Karkain-compatible native interface, never a replacement language; none is required for MVP.

**Explicit blockers:** TLS, concurrent HTTP server framework, WebSocket, UDP/QUIC, GUI and OS process/metric APIs are BLOCKED and must not be claimed as solved.

## E. Data design

**Main entities:** Api, Version, Route, Policy, Consumer, RateLimit, AccessEvent.

**Relationships:** Api has many Versions; Version has many Routes; Route has ordered Policies and one RateLimit; AccessEvent references a Route and optional Consumer.

**Ownership:** Catalogue owns Api/Version; Route Table owns Route; Policy Engine owns Policy; Rate Limiter owns RateLimit; Analytics owns AccessEvent.

**Validation:** Route paths and methods validated at load; policies must reference known ids; rate values must be positive; version aliases must resolve; malformed JSON rejected given PARTIAL `json`.

**Lifecycle:** Api draft→published→deprecated→retired; Route active→disabled; Policy draft→active; AccessEvent immutable once written.

**Persistence:** Text config for declarative state; embedded `db2` for counters/events. `db_begin`/`db_commit`/`db_rollback` exist; crash-atomicity UNVERIFIED.

**Transactions:** A config activation is staged then swapped; atomic rename UNVERIFIED and recorded as a constraint. Analytics writes are append-only.

**Backup/restore:** Export config tree plus `db2` dump; restore into a fresh workspace and revalidate.

**Migration:** Schema version recorded from the start; no pre-v1.2.0 compatibility required.

**Import/export formats:** Text config and simple JSON for export; no full JSON schema dependency.

**Audit history:** Configuration changes append a structured event with actor/target/timestamp; file-permission protection is PARTIAL.

## F. Karkain implementation architecture

**Required capabilities:** Compiler v1.2.0 only; `http` (building blocks), `net` (client), `db`/`db2`, `io`, `config`, `json` (PARTIAL), `string`, `collections`, `crypto`, `encoding`, `log`, `async` (cooperative), `time` (PARTIAL).

**Compiler/runtime requirements:** Karkain v1.2.0; the VERIFIED Windows x86-64 C-free backend with the Go front-end engine is the only currently working compile/run path on this host.

**stdlib modules actually present (per capability register):** `http` PRESENT — 18 functions but explicitly a building-block choreography with no routing, middleware or concurrent server framework; `net` PRESENT (5 s read timeout); `db` PRESENT/PARTIAL (SQL subset + transactions); `crypto` PRESENT (sha256/sha512); `encoding` PRESENT; `async` PRESENT (cooperative, not preemptive); `json` PARTIAL; `system`/`env`/`process`/`time` PARTIAL shell wrappers.

**Required ecosystem components:** None external for MVP.

**Missing dependencies:** TLS library, concurrent HTTP server framework, WebSocket, UDP/QUIC, full JSON DOM/schema — all absent.

**Native implementation opportunities:** Karkain-native route matcher, policy chain and token-bucket limiter using `string`/`collections`/`db2`.

**FFI/OS integration needs:** None justified for MVP. Any future integration must be a Karkain-compatible native interface, never another language.

**Build/packaging:** Single Karkain binary plus text config tree; `karkain build`/`karkain run` with the C-free target.

**Explicit blockers (each a BLOCKER, not solved):** no TLS (no HTTPS termination/upstream TLS); no concurrent HTTP server framework (no production multi-client gateway); no WebSocket; no UDP/QUIC; no GUI; JSON PARTIAL; system/time PARTIAL. No workaround may change Karkain or introduce another language.

## G. Non-functional requirements

| Target | Workload + measurement method |
|---|---|
| Performance/latency | Route 1,000 dry-run requests through policies; measure wall-clock via a Karkain-native timing harness (PARTIAL `time` — probe first). |
| Memory | Load 500 routes/policies; measure process RSS with an external OS tool. |
| CPU | Run the 1,000-request dry-run; measure elapsed CPU via `karkain prof` where applicable. |
| Scalability | 500 routes/1,000 versions; confirm validation and lookup stay bounded. |
| Reliability | Simulate unreachable upstream; confirm structured failure response and intact config. |
| Security | Attempt path traversal and oversized headers; confirm rejection. TLS is absent — transport security NOT provided. |
| Privacy | Consumer identifiers only; no credentials in exported analytics. |
| Portability | Build with C-free Windows target; cross-compile for Linux and record execution NOT VERIFIED. |
| Maintainability | Each module lint-clean with documented interfaces. |
| Accessibility | All output text/JSON; no GUI dependency. |
| Observability | Every config change and access event emits a structured line. |
| Recovery | Restore config + `db2` dump into empty workspace and revalidate to a clean state. |

Practical machine constraint: ~4 GB RAM; workloads sized accordingly.

## H. Compatibility and portability

**Supported OS:** Windows x86-64 toolchain VERIFIED; Linux cross-target output PRESENT but execution NOT VERIFIED; macOS NOT VERIFIED. **Architectures:** x86-64 for the VERIFIED target; others unverified. **Karkain v1.2.0 compatibility matrix:** all source compiles with v1.2.0 only. **Binary/package formats:** single native executable plus text config tree. **Install/uninstall:** copy/unpack binary and config; uninstall by removal. **Config migration:** schema version field planned; no pre-v1.2.0 compatibility required. **Data migration:** documented text/JSON export/import. **Backward compatibility:** none required before v1.2.0. **Upgrade/rollback:** keep prior binary/config; rollback by restore + revalidate. **Cross-platform differences:** `filesystem`/`system` shell-outs vary (PARTIAL/UNVERIFIED). **Unsupported:** TLS/HTTPS, WebSocket, UDP/QUIC, concurrent serving, GUI.

## I. Testing strategy

All tests are **PLANNED**; none have been run for this application.

- **Unit:** Route matcher, policy chain, token bucket; criterion — known inputs yield exact outputs, all named cases pass.
- **Integration:** Config load → dry-run → analytics event; criterion — one event per request, zero state corruption.
- **E2E:** Define API/route/policy, dry-run a sample request, verify text/JSON output fixtures.
- **Platform-specific:** C-free Windows build executes (exit 0); Linux/macOS execution NOT VERIFIED.
- **Security:** path traversal/oversized input rejected; criterion — no file written outside workspace.
- **Performance:** 1,000-request dry-run within G workload; criterion — bounded memory.
- **Regression:** prior route/policy fixes remain correct.
- **Compatibility:** lint/build on v1.2.0; criterion — zero errors.
- **Actual-compiled-executable:** build with `karkain build`, run the binary, check exit 0 and output.
- **Failure/recovery:** unreachable upstream and invalid config; criterion — clean failure, prior config intact.

## J. Implementation roadmap

**MVP:** Config loader, API catalogue, route table, dry-run, CLI/TUI; gate — first vertical slice compiles and runs.

**Core functional release:** Policy engine, rate limiter, versioning, analytics; gate — unit/integration/E2E criteria pass.

**Advanced release:** richer analytics, config validation tooling, approvals; gate — security/recovery criteria pass.

**Production-readiness release:** hardening and platform probes; gate — all G targets measured and BLOCKED capabilities either resolved at language level or still explicitly documented.

**Feature dependencies:** Routes depend on Catalogue; Policies/RateLimits depend on Routes; Analytics depends on Gateway Service; TLS/concurrency are language-level prerequisites for any production gateway claim.

**Milestones/gates:** each release proceeds only after its gate; no fictional dates or completion claims. **Risks/mitigations:** BLOCKED primitives kept as blockers (never faked); PARTIAL modules probed before use; single-threaded design documented as a constraint.
