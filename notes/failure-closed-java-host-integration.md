# Failure-Closed Java/Swing Host Integration

General design guidance for lifecycle ownership, readiness checks, bounded recovery, packaging, and verification around a third-party desktop host.

- GitHub publication: This note is public; the underlying source is not
- Context: Personal Java/Swing integration; host and protocol details are omitted
- Documentation status: Guidance maintained; implementation and current project status are not independently verified here

## Contribution and evidence scope

My documented personal-project experience includes work on a Java/Swing desktop extension, its interface, and related tests. The underlying source is not public, so readers cannot independently authenticate project behavior from this note. The technical sections below are general guidance, not a verified feature list or a claim that I personally designed or reviewed every detail. This personal integration is separate from my professional Java/Kotlin/Spring assignments and the public [G-Earth Trade Assistant](https://github.com/wouthh/g-earth-trade-assistant).

## Summary

A plugin beside a third-party desktop host depends on state it does not own: connection lifecycle, runtime arguments, document or room context, protocol compatibility, and host shutdown. A robust design should avoid acting merely because a UI control was used when required context is unknown.

An integration can use explicit readiness facts and fail closed. Configuration expresses intent, but does not prove that a host is connected or that the current context is authorized. Each operation should pass through a fresh gate, and lifecycle events should cancel delayed work before it outlives its authority.

## Boundary and lifecycle

A design can separate four responsibilities:

1. **UI and configuration** collect persistent user intent.
2. **Application services** own lifecycle and scheduling.
3. **Protocol adapters** translate project-owned commands into host-specific structures.
4. **Host boundary** owns the live connection and observed context.

Persistent configuration should exclude credentials, runtime authentication arguments, live connection state, transient context, and raw payloads. A missing configuration can load safe defaults. Malformed data should be preserved for recovery before defaults load. An unsupported future schema should block activation rather than be partially rewritten.

A safe lifecycle can begin stopped after launch and configuration reload. Starting should require valid configuration, a connected host, known matching context, resolved operation bindings, and no conflicting transition. A successful adapter return proves only that a command was submitted to the host, not that a remote system accepted it.

## Safety choices

- **Fresh gate at execution:** recheck connection and context instead of trusting state captured when work was scheduled.
- **Cancellation ownership:** stop, disconnect, context loss, configuration replacement, and shutdown should cancel pending schedules without one final delayed action.
- **No automatic activation:** saving an enabled profile should not start it.
- **Transient overrides:** a manual context override should be conspicuous, never persisted, and cleared at lifecycle boundaries.
- **Atomic configuration writes:** save through a same-directory temporary file, retain one valid backup, then move atomically.
- **Bounded diagnostics:** normal summaries should omit credentials, authentication arguments, and raw protocol bodies. Any raw view should be separately opt-in and treated as sensitive.
- **Recovery without invention:** an ambiguous state should block the operation and explain the unmet gate; it should not guess a context or protocol layout.

## Verification

An appropriate local gate for this kind of project can compile the application, run unit tests, and verify the packaged JAR with isolated test homes. UI previews should not open a host connection. Protocol fixtures should use synthetic, project-owned records. Packaging checks can inspect the main class, dependency layout, and artifact structure.

Live acceptance is separate from fixture testing. A manual sequence can verify connection, deliberate context selection, start, cancellation on context loss, recovery, and disconnect cleanup. A submitted command is not proof that a remote system accepted it. This note does not claim that these checks were run for the private integration.

## Lessons

The broad lesson is that integrations need an authority model even when they are local plugins. Connection presence is not authorization, configuration is not live context, and a timer is not a reason to act after the state that created it has disappeared.

A second lesson is to keep host-native types at the adapter boundary. Application modules should consume project-owned records and ports. That limits compatibility changes and lets most tests run without starting the third-party application.

## Evidence basis and limitations

The supported personal-project scope is limited to Java/Swing extension, interface, and testing work. The underlying source is private and is not reproduced here. Detailed lifecycle, protocol, configuration, test, packaging, and failure behavior above is general guidance, not a verified description of the implementation or its current host compatibility. The private evidence cannot be independently authenticated from this page.
