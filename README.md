# Engineering Portfolio

Sanitized engineering case studies about reliable domain systems, modular platforms, local-first data, testing, security, and delivery.

> **Maintained documentation**
>
> This repository explains engineering decisions and verification practices. It is not a substitute for the current inspectable code linked from my GitHub profile.

## What this portfolio shows

My work is backend-focused and often sits where business rules, data, integrations, and operational constraints meet. The recurring challenge is not merely implementing a happy path. It is changing behavior without losing compatibility, separating authority from derived state, making side effects recoverable, and proving that a change is safe enough to ship.

The case studies cover PHP/Symfony domain systems, TypeScript platform boundaries, Python and TypeScript local-data tools, Java host integration, and responsible AI-assisted delivery. Technologies are included where they explain a decision; the extended map is in [Capabilities](capabilities.md).

These case studies describe real engineering work using sanitized architecture, synthetic examples, and original diagrams. They omit proprietary code, operational data, and identifying organization details.

## Flagship case studies

| Case study | Focus |
|---|---|
| [Evolving Domain Rules in a Long-Lived Backend](case-studies/evolving-domain-rules.md) | Changing intertwined rules through explicit boundaries, focused tests, migrations, staged delivery, rollback, and production diagnosis. |
| [A Modular TypeScript Platform](case-studies/modular-typescript-platform.md) | Separating domain logic, API contracts, web concerns, persistence, and operational safeguards as a system grows. |
| [Trustworthy Local Data Systems](case-studies/trustworthy-local-data-systems.md) | Provenance, append-only evidence, uncertainty, deterministic reconstruction, migrations, indexing, and rollback. |
| [Verified AI-Assisted Engineering](case-studies/verified-ai-assisted-engineering.md) | Keeping scope, decisions, validation, security, and acceptance under human control. |

## Supporting engineering notes

- [Failure-Closed Java Host Integration](notes/failure-closed-java-host-integration.md) describes lifecycle ownership, readiness gates, bounded recovery, packaging, and verification around a third-party desktop host.
- [Trading-System and Automation Evolution](notes/trading-system-evolution.md) follows a historical Electron and Node.js experiment from direct integration toward clearer boundaries, durable state, simulation, and operational safeguards.

## How to read the evidence

Each page separates context, system boundaries, decisions, failure modes, verification, and limitations. Public repositories are linked only when their source can be inspected. Non-public work is represented with newly written explanations, original diagrams, and synthetic examples rather than copied source.

Claims are deliberately qualitative unless a result is reproducible from public material. A test strategy or migration plan is evidence of engineering practice; it is not converted into an invented scale or business-impact number.

## Engineering themes

- **Domain change:** make rule ownership and ordering explicit before changing behavior.
- **Reliable delivery:** connect acceptance criteria to focused tests, full gates, rollout, and rollback.
- **Data trust:** preserve provenance and distinguish authoritative records from rebuildable projections.
- **Secure integration:** fail closed on missing authority, isolate credentials, and bound side effects.
- **Operational clarity:** make state, failure, recovery, and remaining uncertainty observable.
- **Responsible tooling:** use AI assistance inside a reviewable, human-owned engineering process.

## Status

The documentation is maintained as the public portfolio evolves. Historical systems remain historical; updating their explanation does not imply current compatibility or maintenance.

New material must continue to meet the same evidence, privacy, accessibility, and truthful-status standards before it joins the navigation.
