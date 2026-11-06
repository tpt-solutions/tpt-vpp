# tpt-vpp — Project Todo

**Owner:** TPT Solutions · **License:** `MIT OR Apache-2.0` (dual, everything) · **Stack:** all-Rust (Cargo monorepo, GitHub)
**Dependency rule:** permissive only. Allowed: MIT, BSD, ISC, Unlicense, CC0, dual `MIT OR Apache-2.0`. Rejected: Apache-only, GPL/AGPL/LGPL/MPL.
**Deploy:** self-hosted + local first · **Region:** NZ implemented, AU/NEM design-ready · **Beta:** 10 self-hosting users

Key decisions (supersede `spec.txt`): all-Rust incl. UI · JSON over local HTTP/Unix socket (no gRPC) · SQLite default, Postgres optional · tpt-lang/tpt-swarm are inspiration only · generic adapters so any device can be supported.

Legend: `[ ]` todo · `[x]` done

---

## Phase 0 — Foundation & Legal (Week 0–1)
- [ ] Create Cargo workspace monorepo layout (`crates/`, `docs/`, `schemas/`, `ui/`)
- [ ] Create GitHub repo under TPT Solutions; set branch protection
- [ ] Add `LICENSE-MIT` and `LICENSE-APACHE` (copyright TPT Solutions)
- [ ] README license section ("MIT OR Apache-2.0, at your option") + contribution clause
- [ ] Add SPDX header (`SPDX-License-Identifier: MIT OR Apache-2.0`) convention + check script
- [ ] Set `license = "MIT OR Apache-2.0"` in every `Cargo.toml`
- [ ] Add `deny.toml` (`cargo-deny`): allow-list permissive + dual MIT/Apache; deny Apache-only, copyleft
- [ ] CI license gate: fail build on disallowed/Apache-only dependency (incl. transitive)
- [ ] Audit UI/JS/WASM toolchain deps (trunk/wasm-bindgen/npm if any) against license rule
- [ ] Generate third-party license report (`cargo-about` or equivalent, permissive-only)
- [ ] CONTRIBUTING.md (inbound = outbound licensing, DCO sign-off)
- [ ] CODE_OF_CONDUCT.md, SECURITY.md (vuln disclosure)
- [ ] GitHub Actions CI: fmt, clippy, test, deny, build matrix (Windows/Linux/macOS)
- [ ] ADR-001: all-Rust stack decision
- [ ] ADR-002: IPC = JSON over local HTTP/Unix socket + versioned JSON Schema
- [ ] ADR-003: storage trait, SQLite default, optional Postgres, no TimescaleDB
- [ ] ADR-004: licensing & dependency policy
- [ ] Trademark/name check for "tpt-vpp" / TPT Solutions
- [ ] Engage legal review: confirm "user-side optimization only" is not electricity retailing (NZ)
- [ ] Decide rust toolchain pinning (MSRV) and `rust-toolchain.toml`

## Phase 1 — Core Domain & Contracts (Week 1–2)
- [ ] `vpp-core` crate: `DeviceState`, `Vendor`, `FullyLoadedRate`, `Decision`, `Action`, `Policy` types
- [ ] Units + validation (kW/kWh/¢/kWh newtypes; SoC 0–100 bounds)
- [ ] Define JSON Schemas for DeviceState, FullyLoadedRate, Command, CommandResult, AuditEvent
- [ ] Schema versioning strategy + compatibility tests
- [ ] Storage trait (`vpp-store`): telemetry, devices, users, policies, audit
- [ ] SQLite backend + migrations (permissive driver only)
- [ ] Config system (file + env), secrets handling for device credentials
- [ ] Structured logging + metrics using permissive-licensed crates (verify no Apache-only)
- [ ] Shared test fixtures + deterministic clock abstraction
- [ ] Time handling: NZ timezone, trading periods (half-hour + 5-min), DST edge cases

## Phase 2 — Price Ingest, NZ (Weeks 1–4)
- [ ] Worker-pool framework (tpt-swarm-style, reimplemented): scheduling, backoff, jitter, cancellation
- [ ] `PriceSource` trait (region-agnostic, AU/NEM-ready)
- [ ] Electricity Authority EMI poller (wholesale spot, half-hourly + 5-min, 30s poll)
- [ ] EM6 fallback poller (forecast + historical, 5 min) — confirm API terms/licensing
- [ ] Failover logic EMI → EM6, staleness detection, "price unavailable" fail-safe state
- [ ] Lines-company tariff loader (daily) for ICPs with public tariffs
- [ ] Manual CSV tariff upload fallback + validation
- [ ] ICP lookup/validation
- [ ] `FullyLoadedRate` assembly per ICP every 5 min (wholesale + lines variable + amortized fixed + retailer margin)
- [ ] `IsForecast` handling (v1 reacts to current price; forecast hooks only)
- [ ] Persist rates + history in storage
- [ ] Tests with recorded EMI/EM6 payloads; schema-drift detection + alerts
- [ ] Price Ingest binary exposing JSON HTTP API (current rate per ICP)

## Phase 3 — Decision Engine Skeleton (Weeks 1–4)
- [ ] `vpp-engine` crate (pure, no I/O, deterministic)
- [ ] Constraint model: SoC, charge/discharge limits, price caps, reserve floor, plug-in/grid state
- [ ] Constraint solver / `optimal_rate(device, rate)`
- [ ] Hard-coded `ChargeWhenCheap` policy
- [ ] Fail-safe: any unproven precondition → no action
- [ ] Postconditions (`ensures`) checked before emitting commands
- [ ] Safety clamps independent of policy (never exceed device limits, never charge when `controllable=false`)
- [ ] Decision output with human-readable reason strings
- [ ] Determinism tests (same input → same output, bit-exact)
- [ ] Property-based tests + fuzzing (e.g. never charge above cap, never exceed 100% SoC)
- [ ] Replay harness (re-run decision from stored inputs)
- [ ] Engine exposed via JSON HTTP/Unix socket service + library API

## Phase 4 — Adapter Framework & Tesla Powerwall (Weeks 1–4)
- [ ] Adapter trait + process model (one isolated binary per adapter)
- [ ] Adapter JSON protocol: discover, poll state, set command, health, capabilities
- [ ] Supervisor: restart on crash, backoff, health checks, isolation guarantees
- [ ] Device simulator / mock battery for dev & CI
- [ ] Adapter contract-test suite (runs against every adapter)
- [ ] Tesla Powerwall adapter (local gateway API) — auth, state poll, charge/discharge/reserve control
- [ ] Normalize Powerwall data → `DeviceState`
- [ ] Graceful degradation → advisory mode when control unavailable (`controllable=false`)
- [ ] Alerting on vendor schema drift
- [ ] Verify any community-wrapper code reused is permissive-licensed (else clean-room)

## Phase 5 — Orchestrator & 5-Minute Cycle (Weeks 3–6)
- [ ] `vpp-orchestrator` crate/binary
- [ ] Device registry (add/remove/list, credentials stored securely)
- [ ] Session management
- [ ] Cycle scheduler implementing T+0:00 … T+0:16 timeline
- [ ] Assemble (rate, device_states) tuples; handle partial failures/timeouts
- [ ] Call engine, dispatch commands to adapters, await confirmation
- [ ] Retry/idempotency for commands; command-failure reporting
- [ ] Persist telemetry + append audit entry each cycle
- [ ] Price-spike detection → pause-all behaviour
- [ ] REST + WebSocket API for UI (state, prices, decisions, policies, alerts)
- [ ] Local auth (user accounts, sessions/tokens), TLS option
- [ ] Manual override / emergency stop (kill switch)
- [ ] End-to-end integration test: simulator + recorded prices → expected commands

## Phase 6 — Generic Adapters & Second Vendor (Weeks 5–8)
- [ ] Generic adapter design: capability/config-driven device mapping (register maps, topics, endpoints)
- [ ] Modbus TCP / SunSpec adapter (inverters, batteries)
- [ ] MQTT adapter (topic/payload mapping templates)
- [ ] OCPP adapter (EV chargers)
- [ ] Generic HTTP/REST + webhook adapter
- [ ] Home Assistant bridge adapter (entities → DeviceState)
- [ ] Device profile format (shareable config for community-contributed devices)
- [ ] Feasibility spike: Aotea AE-1 API access (documented/reverse-engineer) — time-boxed
- [ ] Feasibility spike: BYD Atto 2/3 cloud API — time-boxed
- [ ] Implement whichever spike succeeds as dedicated adapter; other → advisory-mode backlog
- [ ] Anker Solix adapter (stretch / via generic or dedicated)
- [ ] EV semantics: plug-in state, departure time/target SoC
- [ ] Contract tests for every adapter

## Phase 7 — Web UI, Rust (Weeks 5–9)
- [ ] Spike: Leptos vs Dioxus (license audit, bundle size, WASM build, ecosystem) → ADR-005
- [ ] UI crate scaffold + build pipeline (permissive tooling only)
- [ ] Onboarding flow: add device (credentials/OAuth where applicable) → ICP lookup → tariff selection → policy config
- [ ] Dashboard: real-time device state
- [ ] Dashboard: spot-price graph (current + history)
- [ ] Dashboard: savings vs "dumb charging"/time-of-use baseline
- [ ] Policy editor: sliders for `max_price_cents`, `target_soc_percent`, `reserve_floor_percent`
- [ ] Alerts UI ("Price spike — devices paused", "Car charged at 6¢/kWh, saved $4.20")
- [ ] Decision history view with plain-English explanations ("Why did it charge at 3:14 AM?")
- [ ] Manual CSV tariff upload UI
- [ ] Responsive/mobile layout, accessibility pass
- [ ] Embedded static serving from orchestrator binary

## Phase 8 — Policy DSL v1 & Audit Log (Weeks 9–12)
- [ ] DSL grammar spec (`policy { requires / ensures / action }`)
- [ ] Parser + AST (permissive parser crate or hand-written)
- [ ] Type checker / static validation (units, vendor enums, field access)
- [ ] Evaluator integrated with solver; preconditions proven before action
- [ ] Policy library: ChargeWhenCheap, DischargeWhenExpensive, ReserveFloor, PauseOnSpike, EV-ready-by-time
- [ ] Policy versioning, validation on save, safe rollout/rollback
- [ ] Hash-chained audit log: `{ts, device_id, rate_snapshot, policy_id, preconditions_proven, action_taken}`
- [ ] Chain verification tool (detect tampering) + export
- [ ] Replay command/UI: re-run any decision from audit record
- [ ] Plain-English explanation generator
- [ ] DSL docs + examples
- [ ] Fuzz DSL parser/evaluator

## Phase 9 — Packaging & Self-Hosting (Weeks 10–12)
- [ ] Single-binary build + embedded UI
- [ ] Docker image + compose example
- [ ] Windows, Linux (x86_64/arm64), Raspberry Pi builds
- [ ] Installer / service setup (systemd, Windows service)
- [ ] Config & first-run wizard
- [ ] Backup/restore of SQLite store + audit log
- [ ] DB migrations/upgrade path
- [ ] Optional PostgreSQL backend (hosted multi-tenant readiness)
- [ ] Optional Supabase adapter (stretch)
- [ ] Self-hosting + adapter-development documentation
- [ ] Release automation (tagged builds, checksums, SBOM)

## Phase 10 — Closed Beta, 10 Users (Weeks 9–12+)
- [ ] Beta recruitment (10 users with supported hardware + NZ ICPs)
- [ ] Privacy/consent terms for telemetry; data minimisation
- [ ] Beta install & onboarding guide, support channel
- [ ] Define time-of-use baseline method for savings calculation
- [ ] Savings reporting (per user, per device) + audit export
- [ ] Monitoring/health reporting from beta installs (opt-in)
- [ ] 30-day beta run
- [ ] Success metric: ≥15% cost reduction vs TOU baseline across beta users
- [ ] Bug triage + hotfix cadence
- [ ] User-trust review ("why isn't my car charging?") using audit explanations
- [ ] Beta retrospective + go/no-go for public release

## Phase 11 — Open-Source Release & Post-Beta
- [ ] Pre-release security review (command safety, credentials, API auth, supply chain)
- [ ] Final dependency/license audit; confirm no Apache-only deps
- [ ] Legal sign-off on regulatory position
- [ ] Public repo launch, docs site, issue/PR templates
- [ ] Community guide: writing adapters & device profiles
- [ ] Price forecasting (v2)
- [ ] AU/NEM `PriceSource` implementation (AEMO)
- [ ] Hosted multi-tenant option (auth, tenancy, ops)
- [ ] Peer review of engine formalism (formal specification/proofs of key invariants)

---

## Cross-Cutting (ongoing)
- [ ] Test coverage targets + CI gating
- [ ] Security: secrets at rest, least-privilege device creds, rate-limited commands
- [ ] Observability: logs, metrics, health endpoints
- [ ] Documentation kept current with ADRs
- [ ] Dependency updates + monthly license re-audit

## Risk Watchlist (from spec §6)
- [ ] Vendor API breakage → adapter isolation + contract tests + drift alerts
- [ ] Opaque lines tariffs → public-tariff ICPs first + CSV fallback
- [ ] Forecasting difficulty → deferred to v2
- [ ] User trust → audit log + plain-English explanations
- [ ] Regulatory ("retailing") → legal review before launch
- [ ] Closed Aotea/BYD APIs → community RE + advisory mode
