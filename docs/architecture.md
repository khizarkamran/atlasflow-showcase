# Architecture

This document describes AtlasFlow's architecture at the pattern level — enough to demonstrate the design without exposing the production topology, internal service names, or endpoints.

## Layered flow

```
Frontend (React)
        |
        v
Application / API Layer
        |
        v
Authentication & Authorization
        |
        v
PostgreSQL (Supabase)
        |
        +---> Row-Level Security     (tenant + role isolation, enforced in-database)
        +---> Serverless Functions   (auth flows, scheduled/background processing)
        +---> Realtime Pub/Sub       (live occupancy & presence updates to all clients)
        +---> Scheduled Jobs         (retention purges, absence alerting)
```

*(See `assets/architecture-diagram.png` for the rendered version of this diagram.)*

## What each layer does

**Frontend (React).** The client application staff and kiosk devices interact with. Renders the live occupancy board, inspection forms, and role-scoped views.

**Application / API Layer.** Handles requests from the frontend and kiosk hardware, applies business logic (e.g., inspection scoring), and talks to the database layer below it.

**Authentication & Authorization.** Verifies identity (staff login, or signed kiosk badge authentication) before any request reaches data. Authorization is then re-checked at the database layer — this layer is not the only gate.

**PostgreSQL (Supabase).** The system of record. All persistent data — presence logs, occupancy state, inspections, case notes, audit trail — lives here, with the four capabilities below layered on top:
- **Row-Level Security** — every query is automatically scoped by tenant and role at the database engine level, so authorization can't be bypassed by an application-layer bug alone
- **Serverless functions** — stateless functions handle auth verification and other backend logic without a standing server to manage or patch
- **Realtime pub/sub** — database changes are pushed to connected clients instantly, so every department sees the same live state
- **Scheduled jobs** — recurring database-level jobs handle retention and time-based alerting without manual intervention

## What this diagram intentionally omits

- Exact service/product names beyond the public-facing stack (React, PostgreSQL/Supabase)
- Internal or production URLs and endpoints
- Exact table names, column names, or schema details
- The literal RLS policy definitions
- Any infrastructure identifiers (project IDs, region names, internal hostnames)

These omissions are deliberate — the goal of this document is to demonstrate architectural thinking, not to provide a blueprint of the production system.
