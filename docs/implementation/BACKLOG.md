# Karkain API Manager — First Implementation Backlog

## First vertical slice

**Slice goal:** Load a declarative config defining one API, one version and one route, then dry-run one sample request and print the resolved upstream as deterministic text. This exercises real config parsing, validation, route matching and output without depending on any BLOCKED capability.

**Karkain modules/files expected:** `src/main.kark` (command dispatch), `src/config/loader.kark` (config read/validate), `src/catalogue/api.kark` (API/version records), `src/router/routes.kark` (route matching), `src/ui/text_output.kark`, `configs/default.kark` (default settings), `examples/routes.kark` (sample declarative config).

**Exact input:** `examples/routes.kark` declaring `api = "orders"`, `version = "v1"`, `route.path = "/v1/orders"`, `route.method = "GET"`, `route.upstream = "http://127.0.0.1:9000/orders"`. Command: `karkain run src/main.kark dryrun examples/routes.kark --path /v1/orders --method GET`.

**Exact expected observable output:** one line `route: /v1/orders -> http://127.0.0.1:9000/orders` followed by `status: matched` and `api: orders@v1`. A non-matching path prints `status: unmatched` and exits without error. An invalid config prints the offending field and line and exits non-zero.

**Verification command pattern (Karkain v1.2.0):** with the VERIFIED Go front-end engine and C-free Windows target selected, run `karkain lint src/main.kark`, then `karkain build src/main.kark -o api-mgr-slice` (or `karkain run src/main.kark ...`), execute the binary and compare stdout to the expected fixture. Commands become active only when implementation starts.

## Backlog items

| ID | Feature | Files/modules | Dependencies | Acceptance criteria | Tests | Complexity | Blocking Karkain capability | Definition of done |
|---|---|---|---|---|---|---|---|---|
| API-001 | Config loader + validation | `src/config/loader.kark` | config, io | Valid config loads; invalid field reported with line | Unit | M | None | Lint-clean; fixture passes |
| API-002 | API + version catalogue | `src/catalogue/api.kark`, `src/storage/store.kark` | db2, API-001 | Register/publish/deprecate; query by name@version | Unit, integration | M | None (db2 PRESENT) | Catalogue fixture passes |
| API-003 | Route table (exact/prefix) | `src/router/routes.kark` | API-001 | Match precedence correct; duplicate routes rejected | Unit | M | HTTP routing BLOCKED — Karkain-native matcher | Route fixture passes |
| API-004 | Dry-run engine | `src/router/dryrun.kark` | API-003 | Sample request resolves to exact upstream text | Unit, E2E | S | None | Dry-run fixture passes |
| API-005 | Text/JSON output modes | `src/ui/text_output.kark`, `src/ui/json_output.kark` | API-002, API-003 | Identical data in both modes; deterministic order | Unit | S | json PARTIAL for JSON mode | Output fixtures pass |
| API-006 | Policy engine (ordered rules) | `src/policies/engine.kark` | API-003 | Ordered header rewrite/allow-deny applies exactly; invalid policy refused | Unit, integration | L | None (string/collections PRESENT) | Policy fixture passes |
| API-007 | Rate limiter (token bucket) | `src/limits/bucket.kark` | API-003, db2 | Bucket refill and rejection deterministic | Unit | M | async cooperative only — single process | Limiter fixture passes |
| API-008 | Version aliases/deprecation | `src/catalogue/versions.kark` | API-002 | Alias resolves; deprecated version flagged in output | Unit | M | None | Version fixture passes |
| API-009 | Sequential gateway loop | `src/gateway/serve.kark` | API-003, API-006, API-007 | One request handled end-to-end via http building blocks | Integration, E2E | L | TLS BLOCKED; concurrent server framework BLOCKED; no multi-client claim | Gateway loop handles one request |
| API-010 | Analytics access log | `src/analytics/access.kark` | API-009 | Append-only event per request with status/latency | Integration | M | time PARTIAL — probe timing method | Analytics fixture passes |
| API-011 | Failure response handling | `src/gateway/failure.kark` | API-009 | Unreachable upstream yields structured failure + event | Failure/recovery | M | TLS BLOCKED | Failure path leaves config intact |
| API-012 | Workspace export/import | `src/config/workspace.kark` | API-001, API-002, API-006 | Export→import equals original config | E2E, compatibility | L | json PARTIAL | Round-trip equivalence |
| API-013 | Native slice verification harness | `tests/slice_verification.kark`, `scripts/verify.ps1` | API-004, API-005 | Build produces executable; stdout matches fixture; exit 0 | Actual-compiled-executable | M | Default kcc engine BLOCKED — use Go engine + C-free target | Harness passes end-to-end |

## Dependency order and gates

1. API-001 → gate: loader fixture passes. 2. API-002, API-003 → gate: catalogue/route fixtures pass. 3. API-004 → API-005. 4. API-006 → API-007. 5. API-008. 6. API-009 → API-010 → API-011 (gateway isolated; single-client only). 7. API-012 → API-013. No item proceeds until its gate passes and dependencies are lint-clean.

## Status

All items **NOT STARTED**. Implementation must not begin until all designs and repository initialization are complete. No tests have been run for this application; no application binary exists; all BLOCKED capabilities (TLS, concurrent server framework, WebSocket, UDP/QUIC, GUI) remain blockers and must not be claimed as solved.
