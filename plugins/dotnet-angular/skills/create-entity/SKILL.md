---
name: create-entity
description: Use to create or improve .NET domain entities and value objects — invariants, factory methods, state-changing methods, status enums with allowed transitions, ownership ids, money and audit fields.
---

# Skill: create-entity

## Entity Rules
- Put business rules inside the domain model.
- Inherit the base `Entity` (id, audit columns, soft delete) — see `stack-examples/backend-domain.md`.
- Plain properties, except state with rules: status workflows use `private set` plus transition methods.
- Use factory methods when construction has rules.
- Use methods to change state instead of exposing unrestricted setters.
- Use value objects for concepts with validation and equality.
- Protect invariants.
- Audit fields come from the base `Entity` and are stamped by the repository — never set them by hand.
- Add the `<Entity>Map : BaseEntityMap<Entity>` in `EntityMapping/<Module>/` and the repository pair in `Interfaces/<Module>/` + `Repositories/<Module>/`, registered in `AddInfra()`.
- Include the owner id(s) required by the ownership rules in `project-description.md`.
- Model status fields as enums with explicit allowed transitions enforced by entity methods.
- Use `decimal` for money; snapshot prices and rates on line items when later changes must not alter past records.
- Ids come from the base `Entity` (`Guid.CreateVersion7()`).

## Do Not
- Do not create anemic entities when business rules exist.
- Do not put persistence-specific concerns in the domain.
- Do not let controllers modify entity state directly.
- Do not add CQRS patterns unless requested.

## Output
1. Entity purpose
2. Properties
3. Invariants
4. Methods
5. Value objects
6. Domain events if useful
7. Persistence considerations
8. Tests required
