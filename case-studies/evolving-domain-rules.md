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
