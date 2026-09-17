# Security Overview

This document explains AtlasFlow's security design at the concept level. It does not, and will never, include actual secrets, keys, tokens, internal endpoints, or logic that could be used to bypass a control.

## Design principles

**Authorization lives in the database, not just the UI.** Row-Level Security policies enforce tenant and role scoping directly in PostgreSQL. Even if an application-layer check were ever missed, the database itself still refuses to return or modify data outside a user's authorized scope. This is a materially stronger model than checking permissions only in application code.

**Role-based access, tiered by responsibility.** Five permission tiers (super admin, org admin, director, case manager, ops staff) each map to a defined, minimal set of allowed operations — staff only have the access their role actually requires.

**Device authentication is signed, not trusted by identity alone.** Kiosk check-in/out uses signed-token authentication rather than assuming a device or network location is trustworthy by default.

**Every access-affecting action is audited.** Corrections, approvals, and status changes are logged, supporting both compliance review and dispute resolution.

**Environments are isolated.** Development and production use separate projects and separate credentials, so testing and iteration never touch live operational or client data.

## Credential rotation

After a hosting-provider security incident, all credentials were rotated and RLS policies and authentication configuration were re-verified.

## What is intentionally not in this repository

- `.env` files, API keys, database URLs, or any credential, current or historical
- Actual RLS policy SQL or exact table/column names
- Internal or production service endpoints
- Any logic that, if known, would make it easier to bypass a control
- Any real user, client, resident, or employee data — including in screenshots

## Illustrative pattern (not the real implementation)

To demonstrate the *kind* of pattern used without publishing the actual policies, here is a generic, textbook example of tenant-isolation RLS — not AtlasFlow's real table or column names:

```sql
-- Illustrative only — generic pattern, not an actual AtlasFlow policy
CREATE POLICY tenant_isolation ON example_table
  USING (organization_id = current_setting('app.current_org')::uuid);
```

This shows the *shape* of the approach (scoping every row to the caller's organization at the database level) without revealing which tables it applies to, the exact tiering logic, or any detail that would help someone reason about the real system's attack surface.
