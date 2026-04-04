# Roles, Permissions & Abilities

## Overview
Authorization has two distinct layers:

1. **Platform tiers** — permanent, one per user. Defined in [`blueprint/access-tiers.md`](access-tiers.md). Short form: `free` (registered user), `investor` (can create one Server), `super_owner` (webapp operator, bypasses all gates unconditionally).
2. **Server roles** — per-server, independent of platform tier. Defined in [`blueprint/servers-groups.md`](servers-groups.md). Short form: `server_owner` → `moderator` → `supporter` → `member`.

This file defines only the **permission strings**, **policies**, **middleware**, and the **authorization flow** — not the roles themselves.

---

## Permissions

Permissions are strings defined in code. They are not stored in the database. Convention: `{resource}.{action}`.

### `super_owner`
No permission checks apply. All Gates and Policies are bypassed unconditionally.

### `investor` (as `server_owner` on own Server)
Inherits all `free` permissions, plus:

**Server management (own Server only)**
- `server.manage` — configure Server settings, visibility, preview duration
- `server.pages.create` — create Pages
- `server.pages.update` — edit Pages and components
- `server.pages.delete` — soft-delete Pages and components
- `server.pages.publish` — publish / unpublish Pages
- `server.members.add.direct` — directly add any existing platform user as a member by email (**server_owner only — never delegated to moderators**)
- `server.members.invite` — generate invite links and manage the invite list (covers `link` and `invite_list` join modes)
- `server.members.suspend` — suspend or remove members
- `server.groups.manage` — create, edit, delete Groups; manage Group membership
- `server.moderators.manage` — appoint and remove moderators
- `server.features.manage` — subscribe / unsubscribe Features to their Server
- `server.credits.invest` — allocate Server credit pool
- `server.access.configure` — set per-page, per-component, and per-feature access rules for members
- `server.components.manage` — define and manage custom Server components (v1: server_owner only)

### `moderator` (Server role — granted by `server_owner`)
Moderators are responsible for **content moderation and community management** within a Server. They cannot make access-control decisions (join approvals, feature subscriptions, credit investment).

**What moderators can do:**
- `server.members.invite` — send email or link invites
- `server.members.suspend` — suspend or remove members who violate rules
- `server.members.preview_extend` — approve **or deny** weekly preview extension requests from users (denial permanently blocks the user from requesting on that Server)
- `server.groups.members.manage` — add or remove users from existing Groups (cannot create or delete Groups)
- `server.content.hide` — hide a page component that violates Server rules (`is_locked = true` on the instance)
- `server.pages.view.all` — view all Pages regardless of visibility level (for moderation purposes)

**What moderators cannot do (server_owner only):**
- Approve or reject public join requests
- Appoint or remove other moderators
- Create, configure, or delete Pages
- Subscribe or unsubscribe Features
- Invest or allocate Server credits
- Configure per-page or per-feature access rules
- Change Server settings

### `free`
- `server.preview.view` — view preview-accessible content during preview window
- `server.preview.extend` — request a weekly preview extension (self-service; blocked permanently after a denial — owner can still add the user directly)
- `server.page.view` — view Pages the Server owner has granted access to
- `server.feature.view` — access Features the Server owner has granted access to
- `server.feature.demo.request` — request demo access to a feature on a Server **only when the feature has `demo_accessible = false`** (self-service path; this request flow is skipped for auto-demo features; blocked permanently after a denial for that feature on that Server — owner can still grant access directly via `server.access.configure`)
- `server.boost` — make a boost payment to a Server
- `server.subscribe` — take out a supporter subscription to a Server (any term)
- `user.follow` — follow another user (bidirectional; followback creates mutual follow state) or a Server

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
  demo_accessible     boolean default true   (super_owner toggles at runtime; true = auto-grant during preview)
  demo_usage_limits   json nullable          (reduced limits during demo; null = use base usage_limits)
  is_active           boolean          (super_owner can disable a feature globally)
  created_at / updated_at
```

**Demo access rule:** When a user has `preview` status on a Server (`server_members.status = preview`) AND that Server has the Feature subscribed AND `demo_accessible = true` → access is **automatically granted** at demo limits with no manual request required. When `demo_accessible = false`, the self-service request flow below applies instead.

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
  grantee_type    enum: member | supporter | moderator | group | user
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
  -> Is user super_owner?             Yes -> Allow unconditionally
  -> Resolve viewer's highest level   (1–9 per content-visibility.md evaluation order)
       Level 3 (pros) = user has at least one active supporter subscription platform-wide
       Note: one-off boosts do NOT grant pros status
  -> Is visibility = `private`?       Yes -> Check content_private_grants; Deny if no grant
  -> Is visibility = `link`?          Yes -> Check share_token + OTP; Deny if invalid
  -> Is viewer's level >= content level?  No -> Deny
  -> Is visibility = `members` + group scope? Yes -> Check Group membership; Deny if not in group
  -> Check feature access rule        Fail -> Deny
  -> Allow
```

## Enforcement Architecture (Middleware + Policy)

All authorization is enforced **server-side only**. Frontend UI guards (hidden buttons, disabled inputs) are purely UX — they are never relied upon for security.

### Layer 1 — Middleware (fast-fail before controller)

| Middleware | Applied to | What it checks |
|---|---|---|
| `EnsureServerMember` | Full-member-required routes: server dashboard, settings, write actions, and member-only content routes | User is an **active** member (`status = active`) of the route-bound Server. **Not applied to feature routes** — feature endpoints check membership status at the Policy level (step 0 in Feature Access Rule Resolution) so that users in `preview` status can still access auto-demo features (`demo_accessible = true`). Also **not applied to content-serving routes** — pages at `public`, `users`, `pros`, `followers`, `friends`, or `private` visibility must be accessible to non-members; those routes go directly to `PagePolicy`. |
| `EnsureServerRole` | Role-gated routes (e.g. moderator-only) | User's `server_role` on the route-bound Server is at least the required level |

- Middleware runs **before** the controller. Any request that fails the middleware check is aborted with `403 Forbidden` immediately — the controller never runs.
- This stops direct API requests and frontend exploits at the application boundary.
- `EnsureServerMember` is registered in `bootstrap/app.php` with an alias and applied to the `server` route group.
- `EnsureServerRole` is parametric: `->middleware('server.role:moderator')` restricts a route to moderator or above.

### Layer 2 — Policies (per-action fine-grained authorization in controllers)

| Policy | Resource | Key methods |
|---|---|---|
| `ServerPolicy` | `Server` | `view`, `manage`, `approveJoin`, `inviteMembers`, `suspendMember`, `manageFeatures`, `investCredits`, `configureAccess`, `manageComponents` |
| `PagePolicy` | `Page` | `view`, `create`, `update`, `publish`, `delete` |
| `PageComponentPolicy` | `PageComponent` | `view`, `update`, `delete`, `hide` |
| `ModerationPolicy` | `Server` | `moderate`, `extendPreview`, `manageGroupMembers` |

- Policies live in `app/Policies/`. Registered automatically by Laravel's convention.
- Every controller action calls `$this->authorize()` or `Gate::authorize()` — even if middleware already passed.
- The `super_owner` check is in `Gate::before()` in `AppServiceProvider` — it short-circuits all policy checks unconditionally.
- `ServerPolicy@approveJoin` checks `server_role === server_owner` explicitly — no moderator can pass this check.

### Why Both Layers?

- **Middleware** prevents unauthenticated / non-member requests from hitting business logic entirely.
- **Policies** enforce role-level granularity inside valid requests — e.g. a moderator hitting an `approveJoin` action still gets `403` even though they passed `EnsureServerMember`.
- Together they make it impossible to exploit the frontend (e.g. manually calling a route the UI hides) or to escalate permissions through crafted requests.

### Feature Access Rule Resolution (Most-Specific-Wins)
When evaluating whether a user can access a Feature on a Server, the system first checks for auto-demo access, then resolves explicit rules with **most-specific-wins**:

0. **Demo check first**: Is the user in `preview` status on this Server AND `features.demo_accessible = true` AND the Server has the Feature subscribed? → Grant demo access (limited by `demo_usage_limits`). Skip further rule resolution.
1. Look for a `user`-scoped rule for the requesting user on this Server + Feature. If found, that rule is authoritative — stop.
2. Look for a `group`-scoped rule covering any Group the user belongs to on this Server + Feature. If found, apply that rule.
3. Look for a `member`-scoped rule (applies to all members) or `supporter`-scoped rule (applies to all supporters).
4. If no rule matches, access is denied by default.

This means a user-level explicit grant always overrides a group-level denial, and vice versa. The Server owner's most targeted decision about a specific person is always authoritative.

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
- Should the `super_owner` be able to delegate a second super-admin without full `super_owner` seeding?
