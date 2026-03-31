# Roles, Permissions & Abilities

## Overview
Authorization has two distinct layers:

1. **Platform roles** — permanent, one per user, control what the user can do across the whole platform.
2. **Server roles** — per-server, control what the user can do within a specific Server.

---

## Platform Roles

| Role | Description |
|---|---|
| `super_owner` | One, seeded at install. Absolute access. Bypasses all gates and policies. |
| `investor` | Has paid the one-off Investor fee. Can create and own one Server. |
| `free` | Registered user. No Server creation. Can join, follow, and boost Servers. |

Platform role is assigned permanently:
- `super_owner` — seeded only.
- `investor` — assigned on successful Investor one-off payment. Cannot be downgraded.
- `free` — assigned on registration.

---

## Server Roles

Server roles are per-Server and independent of platform role.

| Role | Granted by | Requirement |
|---|---|---|
| `server_owner` | System on Server creation | `investor` or `super_owner` platform role |
| `moderator` | `server_owner` manually | Must be a `supporter` of this Server |
| `supporter` | System on boost payment | Has boosted this Server |
| `member` | System on join approval | Registered user |

A user holds one Server role per Server — the highest applicable. See `blueprint/servers-groups.md`.

---

## Permissions

Permissions are strings defined in code. They are not stored in the database. Convention: `{resource}.{action}`.

### `super_owner`
No permission checks apply. All Gates and Policies are bypassed unconditionally.

### `investor`
Inherits all `free` permissions, plus:

**Server management (own Server only)**
- `server.manage` — configure Server settings, visibility, preview duration
- `server.pages.create` — create Pages
- `server.pages.update` — edit Pages and blocks
- `server.pages.delete` — soft-delete Pages and blocks
- `server.pages.publish` — publish / unpublish Pages
- `server.members.manage` — approve/reject join requests, invite members
- `server.groups.manage` — create, edit, delete Groups; manage Group membership
- `server.moderators.manage` — appoint and remove moderators
- `server.features.manage` — subscribe / unsubscribe Features to their Server
- `server.credits.invest` — allocate Server credit pool
- `server.access.configure` — set per-page, per-block, and per-feature access rules for members

### `free`
- `server.join.request` — submit a public join request to a Server
- `server.preview.view` — view preview-accessible content during preview window
- `server.preview.extend` — request a weekly preview extension
- `server.page.view` — view Pages the Server owner has granted access to
- `server.feature.view` — access Features the Server owner has granted access to
- `server.boost` — make a boost payment to a Server
- `server.subscribe` — take out a monthly subscription to a Server
- `user.follow` — follow another user or Server

---

## Features (replaces Abilities)

Features are subscribable modules that a Server owner attaches to their Server. They replace the old Ability / add-on model. There is no global "grant to user" — access is entirely per-Server, configured by the Server owner.

### Feature Model
```
features  (platform-wide, seeded/managed by super_owner)
  id                  ulid
  slug                string unique    (e.g. "expense_planner", "ocr_parser")
  name                string
  description         text
  monthly_price_pence integer          (set by super_owner)
  usage_limits        json             (base limits per boost tier, as array)
  is_active           boolean          (super_owner can disable a feature globally)
  created_at / updated_at
```

### Server Feature Subscriptions
```
server_features  (which features are active on a Server)
  id              ulid
  server_id       FK -> servers
  feature_id      FK -> features
  subscribed_at   timestamp
  expires_at      timestamp nullable
  status          enum: active | suspended | cancelled
  created_at / updated_at
```

### Server Feature Access Rules
The Server owner defines who can access each Feature on their Server:
```
server_feature_access
  id              ulid
  server_id       FK -> servers
  feature_id      FK -> features
  grantee_type    enum: member | supporter | group | user
  grantee_id      ulid nullable     (FK -> groups or users; null when grantee_type = member/supporter)
  granted_by      FK -> users
  created_at / updated_at
```

### Initial Features

| Feature slug | Description | Usage limited by |
|---|---|---|
| `expense_planner` | Track and plan expenses with recurring cost support | Entries per month |
| `life_planner` | Task and goal scheduling across life contexts | Active plans |
| `ocr_parser` | Extract text and structured data from uploaded documents | Scans per month |
| `image_processor` | Resize, crop, compress, and process images | Operations per month |
| `member_slots` | Additional Server member capacity (per increment) | Cap increment |

New features are added on an ongoing release cycle. Usage limit values per boost tier are configured by `super_owner` in platform config.

---

## Authorization Flow

```
Request arrives
  -> Is user super_owner?       Yes -> Allow unconditionally
  -> Check platform role gate   Fail -> Deny
  -> Check server role gate     Fail -> Deny
  -> Check content visibility   Fail -> Deny
  -> Check feature access rule  Fail -> Deny
  -> Allow
```

### Implementation Notes
- Use Laravel **Gates** and **Policies** for platform-role permission checks.
- Server role checks use a `ServerMember` query scoped to the current Server context.
- Feature access checks use a `ServerFeatureAccess` query: does a rule exist for this user / their group / their server role?
- Content visibility checks are evaluated in `PagePolicy` and `PageBlockPolicy`.
- All denials are logged to `access_logs` (see `blueprint/content-visibility.md`).

---

## Seed Data

| Field | Value |
|---|---|
| User name | Void |
| User email | bipin.paneru.9@gmail.com |
| Platform role | `super_owner` |
| Server | Limbo (type: `owner`) |

---

## Open Questions
- Should server role checks be middleware-level (e.g. `EnsureServerMember` middleware) or policy-level?
- Should `server_feature_access` rules stack (user rule + group rule both apply) or is it most-specific-wins?
- Should the `super_owner` be able to delegate a second super-admin without full `super_owner` seeding?
