---
name: create-api
description: Use when creating or changing ASP.NET Core backend APIs — REST endpoints, DTOs, FluentValidation, authorization, ownership, server-side totals, idempotent webhooks and pagination.
---

# Skill: create-api

## API Rules
- Controller inherits `BaseController`; routes `api/v1/[controller]` (plus the culture-prefixed route in localized projects) with named verb segments (`GetX`, `GetById/{id:guid}`, `Create`, `Update/{id:guid}`, `Delete/{id:guid}`) — see `stack-examples/backend-application-api.md`.
- Separate `*Request` (input, client-settable fields only) and `*ViewModel` (output) classes; AutoMapper 14 profile in Application.
- Validate requests with FluentValidation.
- Use application services/use cases.
- Return clear validation errors.
- Enforce authorization.
- Enforce resource ownership: user-scoped endpoints are filtered by the authenticated user/owner (rules in `project-description.md`).
- Recalculate any price, fee or total on the server; never accept amounts from the request body as authoritative.
- Make webhook endpoints signature-verified and idempotent.
- Support pagination for list endpoints.
- Avoid exposing domain entities directly.
- OpenAPI documentation on every public endpoint.

## Do Not
- Do not use CQRS by default.
- Do not create command/query handlers unless explicitly requested.
- Do not put business logic inside controllers.
- Do not bypass validation.

## Required Output
1. Endpoint list
2. Request models
3. Response models
4. Validation rules
5. Authorization rules
6. Error responses
7. Tests
8. Documentation updates
