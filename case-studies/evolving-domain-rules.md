# Evolving Domain Rules in a Professional Backend

PHP/Symfony backend work on a claims platform whose business rules and configuration differ per insurer.

## Context

The platform runs claims workflows for several insurers, each with its own rules and configuration. A change to one workflow touches the API, application behaviour, persisted data, reporting, access control, tests, and delivery configuration together, so every change has to be traced across all of them.

## How the platform places a rule

The platform keeps each kind of variation in its own layer:

- **Business decisions** such as eligibility, routing, and refusal conditions live in a rules engine (BRMS).
- **Insurer-specific settings and overrides** cover configurable differences: thresholds, wording, templates, and workflow options.
- **The Symfony and API Platform backend** owns stable domain behaviour, persistence, APIs, and integrations.

Rule sources are authoritative, and generated rule artifacts are never edited by hand. A new variation is first classified as a business decision, an insurer preference, or new domain behaviour, and then goes into that layer instead of becoming an insurer-specific conditional in code.

## Decisions are codes, wording is presentation

I separated stored decisions from their wording. Rule outcomes travel as stable machine-readable codes through the rules engine, persistence, and the API, and those codes run in production. Insurer-specific wording, with insurer overrides and fallbacks, is resolved only at the presentation boundaries: the interface, PDFs, and exports. A wording change can therefore never change a stored decision.

I then converted the stored explanations that still held display text, with a data migration that maps each known text exactly to its code and leaves unknown or dynamic historical values unchanged, so nothing is discarded and a second run changes nothing.

## Tracing a wrong deductible

In one settlement mode a claim should carry no deductible, yet the back office showed a standard deductible and reduced the compensation. I traced the value through the parameter computation, the creation of the damage group, the persisted deductible, and its API and back-office representation. I fixed the computation and the damage-group creation, and added regression tests that run that mode against a standard settlement.

The first fix passed unit tests, static analysis, and CI, but QA still reproduced the discrepancy: my fix had not reached a later step in the path. The lesson I took from it is to test the persisted, rehydrated state through the API, not only the computation.

## Access control

Identity comes from Keycloak through OpenID Connect, and the API authenticates each request with a JWT. Symfony roles and application-level authorization then decide what the caller may do. The platform keeps those two questions apart: authenticating a caller does not by itself authorize a business operation.

I adapted the API authenticator so the API accepts Keycloak-issued OpenID Connect bearer tokens. It verifies signatures against cached JWKS keys, takes the issuer from OpenID Connect discovery, checks `exp` and `nbf`, and resolves the user from the email claim. It keeps the existing shared-secret path for service integrations, so both kinds of caller pass through one authentication boundary. This runs in production for several insurers.

## Migrations in release preparation

On the platform, each environment's migrations are checked through the deployment logs and the database's record of applied migrations, and a separate QA team does the functional testing.

Before a hotfix release, I ran the full pending sequence of 14 migrations against an isolated restore of a production backup, leaving production itself untouched. I compared schema and data before and after, and reviewed a destructive migration for its effect on existing records. Integrity checks reported no failures, and I classified the remaining identity and default differences as non-blocking drift rather than grounds for another migration. I also ran backend syntax checks, targeted controller tests, route checks, and the frontend lint, tests, and build.

## What I did

- I developed PHP/Symfony and API Platform services, REST APIs, and background processing for the multi-insurer claims platform, including BRMS rules and insurer-specific workflows.
- I maintained Doctrine ORM models and migrations, and the PostgreSQL queries used for reporting and data exports.
- I wrote PHPUnit, Jest, and functional regression tests, and investigated behaviour across code, configuration, and data.
- I adapted the API authenticator to accept Keycloak-issued OpenID Connect tokens alongside the existing service-integration path, within the platform's JWT authentication and role-based access control.
- I maintained Docker, CircleCI, Helm, and OpenShift delivery configuration, and verified migrations during release preparation and after deployment. I added a deployment readiness gate, so a deployment reports success only once the backend is ready, with a regression test for the deploy script.
- I contributed to Vue.js/Nuxt.js interfaces.

## How I work on this platform

- I keep each commit small and focused on one concern, and prefix its subject with the ticket it belongs to.
- I add a short context header to source files, stating their purpose, dependencies, invariants, and related files, so reviewers and agents can find their way quickly.
- I work across the backend, the rule sources, and the frontend, and I maintain the delivery configuration that ships them.
