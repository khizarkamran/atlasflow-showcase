# Database & Access Control Design

## Overview

AtlasFlow's data model supports a multi-tenant shelter operations workflow: client presence and occupancy, compliance inspections, case documentation, and a full audit trail — all scoped so that each organization's staff can only ever see their own organization's data.

- **20-table schema** covering the operational domain (presence/occupancy, inspections, case documentation, audit logging, and supporting reference data)
- **63 Row-Level Security policies** enforcing tenant and role scoping at the database layer
- **5 permission tiers**: super admin, org admin, director, case manager, ops staff — each with a defined, minimal set of allowed operations

Exact table names, column definitions, and policy SQL are not published here — this document explains the *approach*, not the literal schema.

## Tenant isolation, conceptually

Every tenant-scoped table carries an organization reference. Row-Level Security policies use that reference (combined with the requesting user's session context) to automatically restrict every query — reads and writes — to rows belonging to the user's own organization. This means isolation is enforced by the database engine itself for every query path, not only by the application code that happens to remember to filter correctly.

```
Tenant isolation (conceptual)

User request
    |
    v
Session carries: user id, organization id, role
    |
    v
Query reaches PostgreSQL
    |
    v
RLS policy checks: does this row's organization match
                    the session's organization?  ---> if no, row is invisible
    |
    v
Role-based policy checks: does this role permit
                    this operation on this table? ---> if no, operation is denied
    |
    v
Result: only authorized rows, only for authorized operations
```

## Role tiers (conceptual mapping)

| Tier | General scope |
|---|---|
| Super admin | Cross-organization administrative access (platform level) |
| Org admin | Full access within their own organization |
| Director | Approval/oversight actions within their organization (e.g., inspection sign-off) |
| Case manager | Day-to-day case and client-facing operations within their organization |
| Ops staff | Operational tasks (e.g., presence/occupancy logging) within their organization |

This is a conceptual mapping to explain the design, not a literal listing of the actual role permissions or policy logic.

## Audit trail

Every action that changes access-relevant state (a correction, an approval, a status change) is recorded with enough context to answer "who changed what, and when" — supporting both DHS compliance requirements and internal dispute resolution. The audit log itself is subject to the same RLS-based tenant isolation as the rest of the schema.
