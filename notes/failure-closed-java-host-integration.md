# Failure-Closed Java Host Integration

Lifecycle ownership, readiness gates, bounded recovery, packaging, and verification around a third-party desktop host.

- Development period: Not precisely documented in this public note
- GitHub publication: Underlying source is not public
- Context: Personal Java integration; protocol and operational details are omitted
- Current status: Active implementation; public note maintained
- Last verification: 2026-09

## Summary

A plugin running beside a third-party desktop host inherits state it does not control: connection lifecycle, runtime arguments, room or document context, protocol compatibility, and host shutdown. Sending an operation merely because a UI button was pressed is unsafe when any of that context is unknown.

This Java 21 and Swing integration uses explicit readiness facts and fails closed. Configuration can express intent, but intent never proves that the host is connected or that the current context is authorized. Every operation passes through a fresh gate, and lifecycle events cancel delayed work before it can outlive its authority.

## Boundary and lifecycle

The integration separates four layers:

1. **UI and configuration** collect persistent user intent.
2. **Application services** own lifecycle and scheduling.
3. **Protocol adapters** translate project-owned commands into host-specific structures.
4. **Host boundary** owns the live connection and observed context.

Persistent configuration excludes credentials, runtime authentication arguments, live connection state, transient context, and raw payloads. A missing configuration loads safe defaults. Malformed data is preserved for recovery before defaults load. An unsupported future schema blocks activation rather than being partially rewritten.

The global runtime begins stopped after launch and configuration reload. Starting requires valid configuration, a connected host, known matching context, resolved operation bindings, and no conflicting transition. A successful adapter return means only that the command was submitted to the host; it is not described as remote acceptance.

## Safety choices

- **Fresh gate at execution:** delayed work rechecks connection and context instead of trusting state captured when it was scheduled.
- **Cancellation ownership:** stop, disconnect, context loss, configuration replacement, and shutdown cancel pending schedules without one final delayed action.
- **No automatic activation:** saving an enabled profile does not start it.
- **Transient overrides:** a manual context override is conspicuous, never persisted, and cleared at lifecycle boundaries.
- **Atomic configuration writes:** save through a same-directory temporary file, retain one valid backup, then move atomically.
- **Bounded diagnostics:** normal summaries omit credentials, authentication arguments, and raw protocol bodies. Any raw view is separately opt-in and treated as sensitive.
- **Recovery without invention:** an ambiguous state blocks the operation and explains the unmet gate; it does not guess a context or protocol layout.

## Verification

The authoritative local gate compiles, runs unit tests, and verifies the packaged JAR with isolated test homes. UI preview does not open a host connection. Protocol tests use fixtures and project-owned records. Packaging checks the main class, dependency layout, and deterministic artifact structure.

Live acceptance remains separate from fixture testing. A manual sequence verifies connection, deliberate context selection, start, cancellation on context loss, recovery, and disconnect cleanup. It does not treat a submitted command as proof that a remote system accepted it.

## Lessons

The broad lesson is that integrations need an authority model even when they are local plugins. Connection presence is not authorization, configuration is not live context, and a timer is not a reason to act after the state that created it has disappeared.

A second lesson is to keep host-native types at the adapter boundary. Application modules should consume project-owned records and ports. That limits compatibility changes and lets most tests run without starting the third-party application.

## Evidence basis and limitations

The note is based on a verified Java 21 desktop integration with lifecycle callbacks, configuration versioning, a packaged-JAR gate, fixture-backed adapters, bounded diagnostics, and fail-closed context checks. It intentionally omits the host name, protocol identifiers, packet structures, live room or account information, commands, hashes, and deployment paths.

Fixture and packaging evidence do not establish current compatibility with every host release. No such compatibility claim is made here.
