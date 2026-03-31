# Organisations & Teams

## Overview
There are two types of Organisations in Chip:

| Type | Who it belongs to | Purpose |
|---|---|---|
| **Owner Organisation** | The app owner (system-created) | The root admin space — the owner's private organisation; controls and configures the whole platform |
| **Public Organisation** | System-created | The default home for all Free users — read-only portfolio viewers |
| **Supporter Organisation** | Each Supporter (user-created) | The Supporter's own demo-scoped space, created at registration or on upgrade from Free |
| **Tenant Organisation** | Each Investor (user-created) | The Investor's dedicated private space — their own portfolio, tools, and teams |

Every user belongs to exactly one Organisation.

---

## Owner Organisation

- Created automatically at install/seed time. There is exactly one Owner Organisation.
- Owned and controlled by the **Owner** (the app owner, super-admin).
- Free, Supporter, and Investor users are **not** members of the Owner Organisation — they belong to their own organisations.
- The Owner Organisation's slug is the root of the app (e.g. the `/` and `/portfolio`, `/demo/*` routes belong here).

### Organisation Attributes
| Attribute | Description |
|---|---|
| `id` | ULID |
| `name` | Set at install (e.g. owner's name or brand) |
| `slug` | URL-safe slug |
| `owner_id` | FK → users (the Owner account) |
| `type` | `owner` |
| `created_at` / `updated_at` | Timestamps |

### Seed Data
| Field | Value |
|---|---|
| Organisation name | Limbo |
| Organisation slug | limbo |
| Organisation type | `owner` |
| Owner user name | Void |
| Owner user email | bipin.paneru.9@gmail.com |
| Owner user role | `owner` |

---

## Public Organisation

- Created automatically at install/seed time alongside the Owner Organisation. There is exactly one Public Organisation.
- It is the system-managed default home for all **Free** users only.
- When a user registers as Free, they are automatically assigned to the Public Organisation.
- Free users are read-only members — they have no administrative control over the org.
- The Public Organisation has no user owner — it is a system-managed organisation.
- When a Free user upgrades to Supporter or Investor, they leave the Public Organisation and a new personal organisation is created for them.

### Organisation Attributes
| Attribute | Description |
|---|---|
| `id` | ULID |
| `name` | Set at install |
| `slug` | URL-safe slug |
| `owner_id` | `null` — system-managed, no individual owner |
| `type` | `public` |
| `created_at` / `updated_at` | Timestamps |

### Seed Data
| Field | Value |
|---|---|
| Organisation name | Public |
| Organisation slug | public |
| Organisation type | `public` |
| Owner | None (system-managed) |

---

## Supporter Organisation

- Created when a user registers as Supporter, or when a Free user upgrades to Supporter.
- Each Supporter has exactly one Supporter Organisation.
- The Supporter Organisation is scoped for **demo access only** — no persistent real data, no team management.
- The Supporter who created it is the sole member; there is no invitation or member management.
- Name is chosen by the Supporter at registration/upgrade (must be unique across all organisations).

### Organisation Attributes
| Attribute | Description |
|---|---|
| `id` | ULID |
| `name` | User-chosen, unique |
| `slug` | URL-safe, derived from name |
| `owner_id` | FK → users (the Supporter) |
| `type` | `supporter` |
| `created_at` / `updated_at` | Timestamps |

---

## Tenant Organisation

- Created when a user registers as Investor, or when a Free/Supporter user upgrades to Investor.
- The Investor who created it is the **Tenant Owner**.
- Name is chosen by the Investor at registration/upgrade (must be unique across all organisations).
- The Tenant Organisation has its own scoped space within the app:
  - Its own public portfolio section (visited by the Investor's own Free-tier audience).
  - Its own demo section (accessed by Supporter-tier users of the Investor's tenant).
  - Its own full-feature workspace (Expense Planner, Life Planner, Smart Tools, etc.).
- Tenant routing is scoped by the organisation slug (e.g. `/t/{slug}/` prefix, or subdomain — TBD).

### Organisation Attributes
| Attribute | Description |
|---|---|
| `id` | ULID |
| `name` | Investor-chosen, unique |
| `slug` | URL-safe, derived from name |
| `owner_id` | FK → users (the Investor) |
| `type` | `tenant` |
| `created_at` / `updated_at` | Timestamps |

### Organisation Rules
- A user belongs to exactly one Organisation at any time.
- Owner → Owner Organisation (seeded at install, cannot change).
- Free → Public Organisation (auto-assigned on registration).
- Supporter → their own Supporter Organisation (created at registration or on upgrade from Free).
- Investor → their own Tenant Organisation (created at registration or on upgrade from Free/Supporter).
- On upgrade, the user's previous organisation membership is removed and the new organisation is created.
- Deleting a Supporter or Tenant Organisation is out of scope for now.

---

## Teams

### What is a Team?
A Team is a named sub-group within a **Tenant Organisation**. Teams allow an Investor to organise workspace members and scope access to shared plans, expenses, or other content.

### Who Can Manage Teams?
- Only the **Investor (Tenant Owner)** can create, edit, and disband Teams within their Organisation.
- Teams are not available in the Owner Organisation, Public Organisation, or Supporter Organisation.
- Free and Supporter users have no Team access.

### Team Membership
- Members of a Team must already belong to the same Tenant Organisation.
- The Investor assigns and removes members from Teams.
- A user can belong to multiple Teams within their Organisation.

### Team Attributes
| Attribute | Description |
|---|---|
| `id` | ULID |
| `organisation_id` | FK → organisations (must be type `tenant`) |
| `name` | Unique within the Organisation |
| `created_by` | FK → users (Investor who created it) |
| `created_at` / `updated_at` | Timestamps |

### Team Membership Table (`team_user`)
| Column | Description |
|---|---|
| `team_id` | FK → teams |
| `user_id` | FK → users |
| `role` | Optional team-level role (TBD) |

---

## Relationships Summary

```
Owner Organisation  (type: owner)
  └── Owner  (Void — bipin.paneru.9@gmail.com)

Public Organisation  (type: public)
  └── Free users (read-only portfolio viewers)

Supporter Organisation  (type: supporter, one per Supporter)
  └── Supporter (sole member, demo-scoped)

Tenant Organisation  (type: tenant, one per Investor)
  ├── Investor (Tenant Owner)
  ├── [future: other tenant members — invite TBD]
  └── Teams (Investor-managed)
       ├── Team A
       │    ├── member1
       │    └── member2
       └── Team B
            └── member3
```

---

## Open Questions
- Can an Investor invite other (non-Investor) users into their Tenant Organisation as members?
- If so, what tier/role do those invited members get within the tenant?
- Can a Tenant Organisation have co-owners (multiple Investors)?
- What happens to a Tenant Organisation when the Investor's subscription lapses — freeze it or delete?
- Is tenant routing via URL path prefix (`/t/{slug}/`) or subdomain (`{slug}.chip.app`)?
- Is Team membership invitation-based or admin-assigned by the Investor?

