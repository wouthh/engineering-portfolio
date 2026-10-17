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

## Access control

Identity comes from Keycloak through OpenID Connect, and the API authenticates each request with a JWT. Symfony roles and application-level authorization then decide what the caller may do. The platform keeps those two questions apart: authenticating a caller does not by itself authorize a business operation.

## What I did

- I developed PHP/Symfony and API Platform services, REST APIs, and background processing for the multi-insurer claims platform, including BRMS rules and insurer-specific workflows.
- I maintained Doctrine ORM models and migrations, and the PostgreSQL queries used for reporting and data exports.
- I wrote PHPUnit, Jest, and functional regression tests, and investigated behaviour across code, configuration, and data.
- I secured application and API access with Keycloak, OAuth 2.0, OpenID Connect, JWT, and role-based access control.
- I maintained Docker, CircleCI, Helm, and OpenShift delivery configuration, and verified migrations, rollback procedures, and post-deployment smoke tests.
- I contributed to Vue.js/Nuxt.js interfaces.
