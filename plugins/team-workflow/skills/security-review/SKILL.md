---
name: security-review
description: Use when reviewing security — authentication, authorization, resource ownership, input validation, injection/XSS, secrets, uploads, payments/webhooks, financial workflows and personal data protection.
---

# Skill: security-review

## Focus Areas
- Authentication (framework identity system, external logins, email/phone confirmation, password reset)
- Authorization and role permissions (roles defined in `project-description.md`)
- Resource ownership (each audience only sees and changes its own data)
- Input validation
- SQL injection prevention
- XSS prevention (any user-generated content)
- CSRF considerations
- Secrets management (API keys, webhook secrets)
- Audit logging
- File upload security
- Payment webhook verification and idempotency
- Financial workflow protection (prices, fees, refunds, payouts)
- Personal data protection (the data-protection law named in `project-description.md`)

## Generic Critical Rules
- Users never see or change data that belongs to another user/organisation unless the project rules explicitly allow it; ownership is enforced in the query, not after loading.
- Prices, fees, discounts and totals are recalculated on the server; any amount sent by the client is ignored or validated against the server value.
- Payment/confirmation states are never set from a client-side redirect — only from a signature-verified, idempotent server notification.
- Privileged operations (refunds, payouts, approvals, blocking) require the admin role and are audited.
- Personal and sensitive data is collected only when needed, protected, and never logged.
- Important changes are audited.

## Project-Specific Rules
Apply every rule in the security section of `project-description.md`. Those rules take precedence.

## Output
1. Security summary
2. Vulnerabilities found
3. Severity
4. Exploit scenario
5. Recommended fix
6. Files affected
7. Verification steps
