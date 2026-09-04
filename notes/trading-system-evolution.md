# Trading-System and Automation Evolution

From a historical Electron client and Node.js service toward clearer API boundaries, persistent state, deterministic simulation, and guarded operations.

- Development period: Historical personal projects; exact years not asserted here
- GitHub publication: Historical repositories are public
- Context: Personal architecture study using sanitized examples
- Current status: Historical and unmaintained; public note maintained
- Last verification: 2026-09

## Summary

The earliest TradeMate experiment combined an Electron shell, React interface, client-side state, charts, and direct HTTP integration. A companion Node.js and Express service added authentication, MongoDB persistence, queued work, and a provider adapter. That separation improved where credentials and durable state could live, but it did not by itself make a safe automated system.

The useful engineering story is the progression from a visible desktop experiment toward explicit boundaries: market data as input, strategy-independent decisions, precision-aware calculations, durable intent, simulation before mutation, idempotent execution, and reconciliation against externally observed state.

This is an architecture note, not a current trading product. It makes no claim about profitability, performance, provider compatibility, or production readiness.

## Evolution of the boundary

### Desktop experiment

Electron and React made it quick to explore navigation, charts, and state presentation. Direct client integration also exposed a structural limitation: a packaged desktop or frontend application is not an appropriate secret boundary. UI state is useful for presentation and operator intent, but it should not own credentials or durable execution truth.

### Service boundary

A Node.js service created a place for authentication, provider adapters, persistence, validation, and queued work. The client could request an operation without needing direct provider credentials. MongoDB stored application state and the queue separated request time from execution time.

That architecture still needs a precise contract. A queue entry is not proof that an external action succeeded. A retry can be dangerous without a stable operation identity. Provider responses need normalization before they influence domain state.

### Guarded automation

A safer target model separates:

- external observations from internal decisions;
- proposed actions from approved actions;
- simulation from application;
- durable intent from provider delivery;
- provider acknowledgement from later reconciliation;
- decimal quantities from binary floating-point presentation;
- retryable transport failure from rejected business input.

The system records a normalized intent with a stable idempotency key, validates precision and bounds, and defaults to simulation or dry-run. Application requires an explicit authorization boundary. Reconciliation compares durable intent with observed provider state without inferring success from a submitted request alone.

## Failure modes and safeguards

| Failure mode | Safeguard |
|---|---|
| Credentials enter a packaged client | Environment-based server configuration and no production credentials in historical clients |
| Floating-point rounding changes an amount | Decimal representation and provider-specific precision validation |
| A retry duplicates an operation | Stable idempotency key and durable execution record |
| A timeout is treated as failure and retried blindly | Unknown outcome state followed by reconciliation |
| Historical output is read as performance evidence | Explicit historical status and no profitability claim |
| Provider state drifts from local state | Periodic read-only reconciliation before further mutation |
| A simulation path and live path diverge | Shared decision model with separate adapters for simulation and application |
| Logs expose sensitive request material | Structured redaction and no completed credential-bearing URLs or payload dumps |

## Verification

The historical repositories are not executed for this note. Repository evidence supports the Electron/React client, Node.js/Express service, MongoDB persistence, a queue, authentication, and external API adapter. Current compatibility is not inferred from old dependency manifests.

A current implementation of the safer target would require synthetic provider fixtures, decimal boundary tests, idempotency and retry tests, restart recovery, simulation-to-application parity, and reconciliation tests. Live-provider validation would be a separately authorized step and would never use production credentials merely to demonstrate the portfolio.

## Evidence basis and limitations

This note links architectural lessons to the public historical [TradeMate](https://github.com/wouthh/TradeMate) and [TradeMate-Server](https://github.com/wouthh/TradeMate-Server) repositories. It does not reproduce or discuss strategies, symbols, balances, signals, order parameters, credentials, provider endpoints, or historical outputs. Technologies not established by those repositories are not attributed to them.
