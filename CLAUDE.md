# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

> **IMPORTANT:** Read [`AGENTS.md`](./AGENTS.md) for project conventions, commands, risk tiers, workflow rules, and anti-patterns. Those instructions apply to all agents working on ZeroClaw.

## Project Overview

ZeroClaw is a Rust-first autonomous agent runtime optimized for performance, efficiency, and security. It runs personal AI assistants on messaging platforms (WhatsApp, Telegram, Discord, etc.) with minimal resource overhead.

**Key stats:**
- Rust Edition 2024, single binary with no runtime dependencies
- Runs on $10 hardware with <5MB RAM (99% less than OpenClaw)
- Trait-driven modular architecture for extensibility

## Development Commands

### Essential Commands

```bash
# Format and lint
cargo fmt --all -- --check
cargo clippy --all-targets -- -D warnings

# Test (5-level taxonomy: unit → component → integration → system → live)
cargo test                              # All non-ignored tests
cargo test --lib                        # Unit tests only
cargo test --test component             # Component tests (one subsystem)
cargo test --test integration           # Integration tests (multi-component)
cargo test --test system                # System tests (full flow)
cargo test --test live -- --ignored     # Live tests (real APIs, requires creds)

# Build
cargo build --release --locked          # Release build (optimized for size)
cargo build --profile release-fast      # Parallel codegen (requires 16GB+ RAM)

# Full pre-PR validation
./dev/ci.sh all                         # Comprehensive CI suite
```

### CI Script (`./dev/ci.sh`)

Run local CI validation in Docker:

```bash
./dev/ci.sh lint          # rustfmt + clippy correctness
./dev/ci.sh lint-strict   # Full clippy warnings gate
./dev/ci.sh test          # All tests
./dev/ci.sh build         # Release build smoke check
./dev/ci.sh security      # cargo audit + cargo deny
./dev/ci.sh all           # Everything above + docker smoke test
```

## Architecture Overview

### Core Trait Extension Points

ZeroClaw extends capabilities through trait implementations + factory registration:

| Trait | Location | Purpose |
|-------|----------|---------|
| `Provider` | `src/providers/traits.rs` | LLM provider integration (OpenAI, Anthropic, etc.) |
| `Channel` | `src/channels/traits.rs` | Messaging platform integration |
| `Tool` | `src/tools/traits.rs` | Agent executable capabilities |
| `Memory` | `src/memory/traits.rs` | Memory backend (markdown, SQLite, vector) |
| `Observer` | `src/observability/traits.rs` | Telemetry and observability |
| `RuntimeAdapter` | `src/runtime/traits.rs` | Runtime execution environment |
| `Peripheral` | `src/peripherals/traits.rs` | Hardware boards (STM32, RPi GPIO, etc.) |

### Module Responsibilities

Keep dependencies flowing inward to contracts. Avoid cross-subsystem coupling.

- `src/main.rs` — CLI entrypoint and command routing
- `src/agent/` — Agent orchestration loop
- `src/gateway/` — Webhook/gateway server (Axum-based HTTP/WS)
- `src/security/` — Policy, pairing, secret store, sandboxing
- `src/memory/` — Memory backends + embeddings/vector merge
- `src/providers/` — Model providers + resilient wrapper
- `src/channels/` — Messaging platform implementations
- `src/tools/` — Tool execution surface (shell, file, memory, browser)
- `src/peripherals/` — Hardware peripheral integrations
- `src/runtime/` — Runtime adapters (native, future: WASM/containers)
- `src/config/` — Schema + config loading/merging (TOML)

### Adding New Extensions

**Providers:** Implement `Provider` trait in `src/providers/`, register in factory (`src/providers/mod.rs`). Add focused tests for wiring + error paths.

**Channels:** Implement `Channel` trait in `src/channels/`. Keep `send`, `listen`, `health_check`, typing semantics consistent. Cover auth/allowlist/health with tests.

**Tools:** Implement `Tool` trait in `src/tools/` with strict parameter schema. Validate/sanitize inputs, return structured `ToolResult`, avoid panics. Use `Arc<RwLock<T>>` for shared state.

**Peripherals:** Implement `Peripheral` trait in `src/peripherals/`. Expose tools via `tools()` method. See `docs/hardware/hardware-peripherals-design.md`.

Full extension examples: `docs/contributing/extension-examples.md`
Change playbooks: `docs/contributing/change-playbooks.md`

## Testing Structure

ZeroClaw uses a 5-level testing taxonomy:

| Level | Tests | Directory | Command |
|-------|-------|-----------|---------|
| **Unit** | Single function/struct, everything mocked | `src/**/*.rs` (#[cfg(test)]) | `cargo test --lib` |
| **Component** | One subsystem, external boundaries mocked | `tests/component/` | `cargo test --test component` |
| **Integration** | Multiple components, external APIs mocked | `tests/integration/` | `cargo test --test integration` |
| **System** | Full request→response, only external APIs mocked | `tests/system/` | `cargo test --test system` |
| **Live** | Full stack with real external services (#[ignore]) | `tests/live/` | `cargo test --test live -- --ignored` |

**Shared test infrastructure:** `tests/support/` provides `MockProvider`, `TraceLlmProvider`, `TestChannel`, test helpers, JSON trace fixtures.

**Manual tests:** `tests/manual/` contains human-driven scripts (shell, Python) for non-automated scenarios.

Full testing guide: `docs/contributing/testing.md`

## Feature Flags

Enable optional capabilities with `--features`:

```bash
# Channel platforms
cargo build --features channel-matrix       # Matrix E2EE
cargo build --features channel-lark         # Lark/Feishu (alias: channel-feishu)
cargo build --features channel-nostr        # Nostr protocol
cargo build --features whatsapp-web         # WhatsApp Web client

# Hardware
cargo build --features hardware             # USB device enumeration (nusb + tokio-serial)
cargo build --features peripheral-rpi       # Raspberry Pi GPIO (Linux only)

# Observability
cargo build --features observability-prometheus  # Prometheus metrics (default)
cargo build --features observability-otel        # OpenTelemetry OTLP export

# Capabilities
cargo build --features browser-native       # Native browser automation (fantoccini)
cargo build --features plugins-wasm         # WASM plugin system (extism)
cargo build --features probe                # probe-rs for STM32 debugging
cargo build --features rag-pdf              # PDF ingestion for RAG
cargo build --features voice-wake           # Voice wake word detection (cpal)

# Sandboxing (Linux only)
cargo build --features sandbox-landlock     # Landlock LSM isolation
cargo build --features sandbox-bubblewrap   # Bubblewrap container isolation

# CI meta-feature (all except system-dependent)
cargo build --features ci-all
```

## Build Profiles

| Profile | Use Case | Optimization | LTO | Codegen Units |
|---------|----------|--------------|-----|---------------|
| `dev` | Development | 0 | No | Default |
| `release` | Production (size) | z (size) | fat | 1 |
| `release-fast` | Fast builds on powerful machines | z | fat | 8 |
| `ci` | CI validation | z | thin | 16 |
| `dist` | Distribution releases | z | fat | 1 |

**Note:** `release` profile uses `codegen-units = 1` for minimal memory during compilation (suitable for Raspberry Pi 3 with 1GB RAM). Use `release-fast` for local development on machines with 16GB+ RAM.

## Security and Risk Tiers

ZeroClaw connects to real messaging platforms. Treat inbound DMs as untrusted input.

**Risk tiers:**
- **Low:** docs/chore/tests-only changes
- **Medium:** most `src/**` behavior changes without boundary/security impact
- **High:** `src/security/**`, `src/runtime/**`, `src/gateway/**`, `src/tools/**`, `.github/workflows/**`, access-control boundaries

When changing high-risk code, include threat/risk notes, rollback strategy, and validation evidence in PR.

**Default security:**
- DM pairing required (unknown senders get pairing code, message blocked)
- Supervised autonomy (agent requires approval for medium/high risk operations)
- Workspace isolation, path traversal blocking, command allowlisting
- Forbidden paths: `/etc`, `/root`, `~/.ssh`
- Rate limiting: max actions/hour, cost/day caps

See `SECURITY.md` and `docs/security/` for full security model.

## Documentation Structure

- `docs/setup-guides/` — Installation, onboarding, migration guides
- `docs/reference/` — API references (commands, config, providers, tools)
- `docs/ops/` — Operations (troubleshooting, monitoring, deployment)
- `docs/security/` — Security model, policies, threat model
- `docs/hardware/` — Hardware peripheral guides
- `docs/contributing/` — Contribution guides, playbooks, CI map
- `docs/maintainers/` — Maintainer-specific processes
- `docs/i18n/` — Localized documentation (30+ languages)

**i18n contract:** When changing user-facing wording or navigation, maintain locale parity for all supported languages. See `docs/contributing/docs-contract.md`.

## Common Development Patterns

**1. Read before write:** Inspect existing module, factory wiring, and tests before editing.

**2. Minimal patch:** No speculative abstractions, no config keys without concrete use case.

**3. One concern per PR:** Avoid mixed feature+refactor+infra patches.

**4. Tool shared state:** Use `Arc<RwLock<T>>` handles, accept at construction, namespace by `ClientId`. See `docs/architecture/adr-004-tool-shared-state-ownership.md`.

**5. Error handling:** Return structured errors, avoid panics in runtime paths. Use `anyhow` for application errors, `thiserror` for library errors.

**6. Config changes:** Treat config keys as public contract. Document defaults, migration path, rollback strategy.

**7. Test coverage:** Add tests matching risk tier. High-risk changes require system-level or integration tests.

## PR Workflow

Follow `.github/pull_request_template.md` exactly. Required sections:

- Summary (problem, impact, changes, scope boundary)
- Label snapshot (risk, size, scope, module)
- Validation evidence (test output, commands run)
- Security impact assessment
- Privacy/data hygiene status
- Rollback plan

Work from non-`master` branch. PR to `master`. Use conventional commit titles. Prefer small PRs (`size: XS/S/M`).

See `docs/contributing/pr-workflow.md` for full process.

## Hooks and Slash Commands

_No custom hooks or slash commands defined yet. Configure in Claude Code settings if needed._
