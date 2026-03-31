# Roles, Permissions & Abilities

## Overview
Authorization is layered into three distinct concepts:

| Concept | Defined When | Scope | Purpose |
|---|---|---|---|
| **Permission** | Build time | Action gate | Describes a single allowed action (e.g. `posts.create`) |
| **Role** | Build time | User level | Named bundle of permissions (e.g. `supporter`) |
| **Ability** | Build time + runtime | Feature flag | Unlocked by role OR by subscription add-on selection |

---

## Permissions

Permissions are strings defined in code (not stored in the database). They follow a `resource.action` convention.

### Convention
```
{resource}.{action}
```
Examples: `posts.read`, `posts.create`, `teams.manage`, `features.premium_analytics`

### Permission Sets by Role

#### `owner`
- While no permission is assigned there will be gate bypass, policy check bypass and so on for absolute access (unrestricted). The Owner bypasses any gated permission check.

#### `free`
- `portfolio.read` — view the owner's public portfolio (profile, skills, experience, projects, posts)
- `tenant.portfolio.read` — view any Investor tenant's public portfolio/profile

#### `supporter`
- Everything in `free`, plus:
- `expenses.demo` — read/write in the sandboxed demo expense workspace
- `plans.demo` — read/write in the sandboxed demo life planner workspace
- `tools.ocr.demo` — run OCR on an uploaded file (result is ephemeral)
- `tools.image_processor.demo` — run image processing (result is ephemeral)
- `uploads.demo` — upload a file for ephemeral demo tool use

#### `investor`
- `portfolio.read` — view the owner's public portfolio
- `portfolio.manage` — manage their own tenant portfolio
- `expenses.read` / `expenses.create` / `expenses.update` / `expenses.delete` (scoped to tenant)
- `expenses.export` (tenant)
- `plans.read` / `plans.create` / `plans.update` / `plans.delete` (scoped to tenant)
- `plans.tags.manage` (tenant)
- `teams.create` / `teams.update` / `teams.delete` / `teams.members.manage` (tenant)
- `abilities.manage` — choose/activate subscription feature add-ons
- `uploads.create` / `uploads.read` / `uploads.delete` (tenant)

> Exact permission strings will grow as features are built. The above represents the initial set.

---

## Roles

Roles are predefined and assigned to a user based on their account tier. A user has exactly one role.

| Role | Account Tier | Description |
|---|---|---|
| `owner` | App owner | Full super-admin; bypass all permission gates |
| `free` | Free account | Portfolio viewer only |
| `supporter` | Supporter account | Demo user — ephemeral access to special features |
| `investor` | Investor account | Full tenant — own space, persistent data, team management |

### Role Assignment
- Assigned at registration or on tier upgrade.
- Changing role is handled through the upgrade/subscription flow, not manually.
- `owner` is assigned only at install time (seeded); there is exactly one Owner.

### Seed Data
| Field | Value |
|---|---|
| User name | Void |
| User email | bipin.paneru.9@gmail.com |
| Role | `owner` |
| Organisation | Limbo (type: `owner`) |

---

## Abilities

Abilities are named feature flags that can be granted in two ways:

### 1. Role Grants (Build Time)
Certain Abilities are automatically granted to a role. For example, the `investor` role always receives the `team_management` Ability.

### 2. Subscription Add-on Grants (Runtime)
Investor users can activate premium feature add-ons through their subscription. Each add-on maps to one or more Abilities. When active, those Abilities are granted to the user. When the add-on is removed, the Abilities are revoked at the next billing cycle.

### Ability Model
```
abilities
  id          ulid
  name        string  (unique slug, e.g. "team_management", "premium_analytics")
  label       string  (human-readable)
  description text
  is_premium  boolean  (true = only unlockable via subscription add-on)
```

### Ability → User Grant Table
```
user_abilities
  user_id      FK → users
  ability_id   FK → abilities
  granted_by   enum: role | subscription
  expires_at   nullable timestamp (for subscription-linked grants)
```

### Initial Abilities

| Ability Slug | Granted By | Premium? | Description |
|---|---|---|---|
| `team_management` | `investor` role | No | Create and manage Teams |
| `subscription_management` | `investor` role | No | Manage subscription add-ons |
| `expenses_org_shared` | Add-on | Yes | Shared org-level expenses in Expense Planner |
| `plans_shared` | Add-on | Yes | Team-shared plans in Life Planner |
| `plans_timeline_view` | Add-on | Yes | Gantt/timeline view in Life Planner |
| `ocr_parser` | Add-on | Yes | OCR File Parser tool |
| `image_processor` | Add-on | Yes | Image Processor tool |
| `productivity_suite` | Add-on (bundle) | Yes | Bundles `ocr_parser` + `image_processor` |
| `premium_analytics` | Add-on | Yes | Advanced usage analytics |

---

## Authorization Flow

```
Request → Check Permission (role-based)
       → If permission missing, deny
       → If permission requires an Ability, check Ability grant
       → If Ability missing or expired, deny
       → Allow
```

### Implementation Notes
- Use Laravel's **Gates** and **Policies** for permissions.
- Use a custom `HasAbilities` trait on the `User` model to check Ability grants.
- Gate checks should delegate to policy classes per resource.
- Ability checks: `$user->hasAbility('premium_analytics')`.

---

## Open Questions
- Should Abilities have a hard expiry or only expire when a subscription lapses?
- Can an Admin manually grant an Ability to a specific user outside of subscription?
- Are there organisation-level Abilities (shared across all users in an org), or only user-level?
- Should the `free` role have any write Abilities in the future (e.g. a limited post draft)?
