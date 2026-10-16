# Evolving Domain Rules in a Professional Backend

PHP/Symfony backend work on a claims platform whose business rules and configuration differ per insurer.

## Context

The platform runs claims workflows for several insurers, each with its own rules and configuration. A change to one workflow touches the API, application behaviour, persisted data, reporting, access control, tests, and delivery configuration together, so every change has to be traced across all of them.

## What I did

- I developed PHP/Symfony and API Platform services, REST APIs, and background processing for the multi-insurer claims platform, including BRMS rules and insurer-specific workflows.
- I maintained Doctrine ORM models and migrations, and the PostgreSQL queries used for reporting and data exports.
- I wrote PHPUnit, Jest, and functional regression tests, and investigated behaviour across code, configuration, and data.
- I secured application and API access with Keycloak, OAuth 2.0, OpenID Connect, JWT, and role-based access control.
- I maintained Docker, CircleCI, Helm, and OpenShift delivery configuration, and verified migrations, rollback procedures, and post-deployment smoke tests.
- I contributed to Vue.js/Nuxt.js interfaces.

## Structuring insurer-specific rules

<!-- QUESTION FOR WOUT: How were insurer-specific rules and configuration structured? For example: where the BRMS rules lived relative to the Symfony services, whether insurer differences were expressed as configuration, rule sets, or code (strategy classes, tagged services), and what decision you made about where a new insurer variant belongs. One or two concrete decisions you made or argued for. -->

## Testing configuration variants

<!-- QUESTION FOR WOUT: How did you test configuration variants? For example: whether functional regression tests ran per insurer configuration, how fixtures for a variant were built, and how you chose which variants a change had to cover. A concrete example of tracing a workflow discrepancy through API, configuration, and data to a regression test would fit here, described without client details. -->

## Verifying migrations and rollback

<!-- QUESTION FOR WOUT: How did you verify a Doctrine migration and its rollback before and after release? For example: running down migrations against a copy, checking reporting queries after a schema change, what the post-deployment smoke tests covered, and how this fitted into the CircleCI, Helm, and OpenShift pipeline. -->

## Layering access control

<!-- QUESTION FOR WOUT: How was access control layered? For example: Keycloak as the identity provider, OIDC login for the interface, JWT validation at the API, and how RBAC roles mapped to API Platform security expressions or Symfony voters. Which part of this did you design or change? -->
