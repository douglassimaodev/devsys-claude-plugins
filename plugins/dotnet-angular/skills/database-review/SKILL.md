---
name: database-review
description: Use when reviewing database schema, migrations or EF Core mappings on MariaDB — ownership scoping, constraints, indexes, soft delete, audit, concurrency, financial integrity, status history and migration safety.
---

# Skill: database-review

## Review Areas
- Ownership scoping (owner ids per the rules in `project-description.md`)
- Relationships
- Constraints
- Indexes
- Nullability
- Soft delete strategy
- Audit fields
- Concurrency (e.g. double booking / overselling)
- Financial data integrity
- Status history
- Migration safety (Identity and business contexts migrate separately)
- Query performance

## Generic Database Rules
- Owner-scoped tables carry the owner id required by the project's ownership rules.
- Every business table inherits the base `Entity` columns (`CreatedBy/On`, `UpdatedBy/On`, `RemovedBy/On`); all dates UTC.
- Soft delete = `RemovedBy`/`RemovedOn` set; there is no `Active` column. The `BaseEntityMap` query filter hides removed rows. Flag any `Active`/`IsDeleted` column used for deletion.
- The global conventions (Restrict FKs, string defaults, `char(36)` Guids, table names) run after the mappings; no mapping sets `Cascade`.
- Money columns are `decimal(18,2)`; never floating point.
- Line items snapshot price/rates at transaction time — later changes must not alter past records.
- Separate status columns/tables for separate workflows (e.g. order vs payment); every transition is written to a status history table.
- Capacity/availability conflicts are guarded at the database level (unique constraint or concurrency token), not only in application checks.
- External gateway records store the provider's reference/event id with a unique index so webhooks are idempotent.
- Financial and audit records are never hard-deleted; all FKs `Restrict`.
- Enum columns use `HasConversion<string>()` by default.
- Add indexes for common filters.
- Use explicit constraints where useful.
- Identity (`AspNet*`) tables are stock — never altered.

## Project-Specific Rules
Apply every rule in the database section of `project-description.md`. Those take precedence.

## Output
1. Schema summary
2. Issues found
3. Missing constraints
4. Missing indexes
5. Migration risks
6. Recommended improvements
