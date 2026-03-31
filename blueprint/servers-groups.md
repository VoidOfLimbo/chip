# Servers & Groups

## Overview
Servers are the primary community spaces in Chip — analogous to Discord servers. They are created and owned by `super_owner` or `investor`-role users. Each Server hosts Pages, Groups, Features, and its own member community.

---

## Server Types

| Type | Created by | Limit | Monthly cost |
|---|---|---|---|
| `owner` | `super_owner` (seeded at install) | Unlimited | None |
| `standard` | `investor` | One per user | Sum of Features subscribed to that Server |

---

## Server Data Model

```
servers
  id                ulid
  name              string (unique)
  slug              string (unique, URL-safe)
  description       text nullable
  type              enum: owner | standard
  owner_id          FK → users
  is_public         boolean  (whether public join requests are accepted)
  member_cap        integer  (base from Investor payment; increases with Member Slot add-on)
  group_cap         integer  (base from boost tier)
  boost_tier        integer  default 0
  credit_balance    integer  (pence; accumulated boost payments not yet invested)
  preview_duration  integer  (hours; how long a new visitor's preview window lasts)
  created_at / updated_at
```

---

## Seed Data

| Field | Value |
|---|---|
| Server name | Limbo |
| Server slug | limbo |
| Server type | `owner` |
| Owner user name | Void |
| Owner user email | bipin.paneru.9@gmail.com |
| Owner user role | `super_owner` |

---

## Membership

### Joining a Server — Three Paths

| Path | How it works |
|---|---|
| **Public join request** | `is_public = true` on the Server; any registered user submits a request; **Server owner approves or rejects only** |
| **Email invite** | Server owner or moderator sends an email invite; recipient registers or logs in and accepts |
| **Link invite** | Server owner or moderator generates an invite link; recipient verifies via OTP and joins |

All invite links require OTP verification before the join is recorded — no anonymous joins. See `blueprint/content-visibility.md`.

### Time-Limited Preview

- When a registered user first visits a Server (before joining), they receive a **time-limited preview window**.
- Duration is set by the Server owner via `preview_duration`. Default TBD, configured by `super_owner`.
- During the preview, the user can see Pages the Server owner marks as preview-accessible (`preview_visible = true` on the page).
- After expiry, all Server content is hidden until they formally join.
- A user can request a **1-week extension once per week** — Server owner or moderator approves (`server.members.preview_extend` permission).

### Member Data Model

```
server_members
  id                  ulid
  server_id           FK → servers
  user_id             FK → users
  server_role         enum: server_owner | moderator | supporter | member
  joined_at           timestamp nullable    (null while in preview or pending)
  status              enum: preview | pending | active | suspended
  preview_expires_at  timestamp nullable
  preview_extended_at timestamp nullable    (last extension granted)
  created_at / updated_at
```

Unique constraint on `(server_id, user_id)`.

### Server Roles

| Role | Granted by | Requirement |
|---|---|---|
| `server_owner` | System on Server creation | Must hold `investor` or `super_owner` platform role |
| `moderator` | `server_owner` manually | Must be a `supporter` of this Server |
| `supporter` | System on boost payment | Has boosted this Server at least once |
| `member` | System on join approval | Registered user |

A user holds exactly one Server role per Server — the highest applicable role. A `supporter` who is also appointed moderator has role `moderator`.

---

## Boost & Credit Pool

### How Boosting Works

1. Any registered user visits a Server and chooses to boost it.
2. They pay a one-off amount — between £0 and the maximum the Server owner has set for that feature context.
3. Payment enters the Server's `credit_balance`.
4. The user gains `supporter` server role on that Server (if not already moderator or owner).
5. Server owner grants feature/page access to supporters at their own discretion.

### Boost Tiers

The Server's `boost_tier` rises as the cumulative all-time boost income crosses thresholds defined by the `super_owner` in platform config. Higher tiers unlock increased limits.

| Tier | Example unlocks (values set by super_owner) |
|---|---|
| 0 | Base member cap, base group cap, base feature usage limits |
| 1 | +N member cap, higher feature usage limits |
| 2 | More groups, higher limits, additional privileges |
| … | Defined in platform config |

### Investing Credits

Server owner allocates credits from `credit_balance` toward:
- **Features** — subscribe a new Feature or renew an existing one on the Server
- **Member Slots** — purchase `member_cap` increments
- **Monthly bill offset** — applied as credit against the next monthly feature subscription invoice

Unspent credits automatically offset the next monthly bill.

### Credit Pool Rules

- Credits are locked to the Server — no cash out.
- **Refund policy on boosts and donations**: 80% of the original payment refunded if requested within 7 calendar days; no refund after 7 days. Credits already invested from the pool before a refund is requested cannot be reversed. See `blueprint/payments-subscriptions.md` for full details.
- Credit allocation is recorded in `server_credit_transactions`.

```
server_credit_transactions
  id              ulid
  server_id       FK → servers
  type            enum: boost_in | donation_in | feature_invest | slot_invest | bill_offset
  amount_pence    integer
  description     string nullable
  created_by      FK → users nullable
  created_at
```

---

## Member Slots (N)

- A base value `member_cap` is set when the Investor creates their Server (included in the Investor one-off payment, value TBD).
- Additional slots are a separately subscribable add-on (priced per increment, e.g. per 10 extra members).
- Any Server member can boost the Server to contribute to the credit pool; the Server owner can invest those credits in additional slots.

---

## Groups

### What is a Group?
A Group is a named sub-group within a Server. Groups allow the Server owner to organise members and scope Page visibility and Feature access.

### Group Management
- Created and managed by the Server owner or a moderator (with permission granted by Server owner).
- Any active Server member can be added to a Group.
- A member can belong to multiple Groups in the same Server.

### Group Data Model

```
groups
  id              ulid
  server_id       FK → servers
  name            string (unique within server)
  description     text nullable
  created_by      FK → users
  created_at / updated_at
```

### Group Membership Table (`group_members`)

```
group_members
  id              ulid
  group_id        FK → groups
  user_id         FK → users
  added_by        FK → users
  created_at
```

Unique constraint on `(group_id, user_id)`.

---

## Relationships Summary

```
Server (type: owner)
  └── super_owner  (Void — bipin.paneru.9@gmail.com)
       └── Members (free users, supporters, moderators)
            └── Groups

Server (type: standard, one per investor)
  └── server_owner  (Investor)
       ├── Moderators (supporters appointed by owner)
       ├── Supporters (boosted the server)
       ├── Members (free users granted access)
       └── Groups
```

---

## Open Questions
- What are the exact boost tier thresholds and what each tier unlocks? (Configured by `super_owner` — values TBD)
- What is the base member cap for a new standard Server?
- What is the default preview duration, and can the Server owner customise it?
