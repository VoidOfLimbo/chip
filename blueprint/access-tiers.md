# Access Tiers

## Overview
Chip has two platform account tiers. Tiers are permanent and platform-wide. Server-level roles (`server_owner`, `moderator`, `supporter`, `member`) are separate and described in `blueprint/servers-groups.md`.

| Tier | How gained | Platform role |
|---|---|---|
| Free | Registration (no cost) | `free` |
| Investor | One-off payment to platform | `investor` |

The Super Owner is **not a tier** — they are the operator of the Chip webapp itself. There is exactly one Super Owner and one Super Organisation (Limbo). See the [Super Owner (Webapp Operator)](#super-owner-webapp-operator) section below.

---

## Public (Unauthenticated)

| Attribute | Value |
|---|---|
| Auth required | No |
| Access | Public pages of any Server; landing page |

- Any visitor sees the landing page and any Server content with `public` visibility.
- Public pages are crawler-indexed.
- All other routes redirect unauthenticated visitors to the landing/login page.

---

## Free Account

| Attribute | Value |
|---|---|
| Sign-up cost | Free |
| Billing | None (unless they choose to boost a Server) |
| Platform role | `free` |

### What Free Users Can Do
- Browse and view public pages on any Server.
- **Join a Server** via invite link (OTP-verified), invite list auto-enrol, or direct add by the owner/moderator — subject to the Server's `join_mode`.
- Receive a **time-limited preview** of a Server upon first visit. During the preview they can see pages the Server owner marks as preview-accessible.
  - Before or after expiry: can request a **1-week extension once per week** — the Server owner or moderator approves or **denies** the request with an optional message. A denial permanently prevents the user from making further **self-service** extension requests on that Server. The server owner can still directly add the denied user as a full member at any time.
- **Request access to demo features** on a Server — the Server owner approves or **denies** with an optional message. A denial permanently prevents the user from placing further **self-service** demo access requests for that feature on that Server. The server owner can still grant access directly at any time.
- **Follow** other users (bidirectional with followback — see User Follows below) or Servers.
- Followers can see Servers owned by people they follow and access content at `followers` visibility or lower.
- **Boost a Server** (one-off payment) → gains `supporter` server role on that Server and access to whatever the Server owner grants supporters.
- **Donate** to a Server creator (any amount, anonymous or public in Hall of Fame).
- **Subscribe** to any Server for sustained supporter access (see Supporter Subscriptions below).
- **Profile**: set a **static** profile image (JPEG / PNG / WebP) and a **static** cover image.

### Free users cannot
- Create a Server.
- Publish Pages or manage any Server content.

### Supporter Subscriptions
Free users can take out a recurring supporter subscription to **any Server** (including Limbo). There is no cap on how many Servers a user can subscribe to simultaneously:
- Each subscription is billed independently.
- Available terms: monthly, 3-month, 6-month, or annual (terms offered are set per Server by the Server owner).
- At signup and on **every auto-renewal**, the user must actively waive their 14-day EU/UK statutory cancellation right.
- The platform sends warnings during the **final 7 days** before each renewal date and before expiry.
- This does **not** upgrade their platform role — they remain `free`.
- Supporter access on each Server is retained until the end of the paid period if they cancel.

### Upgrade Path
- Pay the **Investor one-off fee** → gains `investor` platform role → can create their own Server.

---

## Investor Account

| Attribute | Value |
|---|---|
| How gained | One-off payment (fixed amount set by `super_owner` in platform config) |
| Billing | Monthly: sum of Features subscribed to their Server. No features = no monthly cost. |
| Platform role | `investor` |

### What Investors Can Do
- Everything a Free user can do.
- **Create one Server** — names it, configures it, builds Pages.
- **Subscribe Features** to their Server (Expense Planner, Life Planner, Smart Tools, etc.) — each adds to the Server's monthly cost.
- **Subscribe Member Slot add-ons** to increase their Server's member cap.
- **Control all access** on their Server: which pages and features are visible to free members, supporters, specific Groups.
- **Appoint moderators** from their supporter base.
- **Set boost amounts** for each Context (the maximum a supporter can pay per feature).
- **Invest Server credits** toward features, member slots, or monthly bill offset.
- **Profile**: set a **dynamic** profile image (animated GIF / WebM / MP4) and a **dynamic** cover image, in addition to static formats.

### Server Monthly Cost
The Investor pays monthly for Features subscribed to their Server. If their credit pool has a balance, it offsets the bill automatically. If the bill cannot be covered: a **7-day grace period** applies — the Server's features are suspended after 7 days of non-payment, but the Server itself (and its Pages) remains visible.

### What Investors Cannot Do
- Create more than one Server.
- Cash out Server credit pool funds.

---

## Super Owner (Webapp Operator)

The Super Owner is **not a platform tier**. They are the operator of the Chip webapp itself — the person who owns and runs the platform. There is exactly **one** Super Owner account and exactly **one** Super Organisation (Limbo), both seeded at install. Neither can be created, assigned, transferred, or revoked through the UI.

| Attribute | Value |
|---|---|
| How gained | Seeded at install only — never via registration or payment |
| Billing | None |
| Platform role | `super_owner` |
| Uniqueness | Exactly one; never creatable or reassignable via the UI |
| Super Organisation | Limbo — seeded alongside the Super Owner at install |

### What the Super Owner Can Do
- Everything an Investor can do, with no limits.
- Configures platform-wide settings: Investor fee amount, feature prices, feature caps, default preview duration.
- Manages all globally available Features (create, price, activate/deactivate).
- No Server creation limit; no monthly cost for any Server they own.
- Bypasses all Gates and Policies unconditionally — no permission check ever applies.
- Unlimited Servers at no cost.
- Define platform-wide Feature prices, Member Slot prices, boost tier thresholds, and all platform config values.
- Suspend, modify, or manage any user, Server, or content on the platform.
- Bypass all permission gates — no policy or gate check applies.

### Seed Data
| Field | Value |
|---|---|
| User name | Void |
| User email | bipin.paneru.9@gmail.com |
| Platform role | `super_owner` |

---

## User Follows

### User-to-User Following

- Following between users is **bidirectional**: user A can follow user B, and B can independently follow A back (followback).
- When both users follow each other they become **mutual followers** (friends). This is automatically calculated — there is no separate "mutual follow" action; it is simply the state where both follow records exist.
- **Mutual followers ≠ connections.** Connections are a separate, explicit social graph concept (to be defined). Mutual follow is a passive state; a connection requires an active connection request and acceptance.
- Followers gain access to content set at `followers` visibility or lower on the person they follow.
- Mutual followers gain access to content set at `friends` visibility or lower on each other.
- Content requiring explicit individual access uses the **`private` grant mechanism** — not an ordered visibility level, but an explicit row in `content_private_grants`. See [`blueprint/content-visibility.md`](content-visibility.md).

### Server Following

- Any registered user can follow a **Server**.
- Server follows are unidirectional — there is no "followback" concept for Servers.
- Following a Server grants access to content the server owner publishes at `followers` visibility or lower on that Server.

### Follow Data Models

```
user_follows
  id            ulid
  follower_id   FK → users   (the person who follows)
  following_id  FK → users   (the person being followed)
  created_at
```

Unique constraint on `(follower_id, following_id)`. Mutual follow is derived: both `(A→B)` and `(B→A)` rows exist.

```
server_follows
  id            ulid
  follower_id   FK → users
  server_id     FK → servers
  created_at
```

Unique constraint on `(follower_id, server_id)`.

---

## User Profiles

All registered users have a public profile.

### Username / Handle

- Every user must choose a unique **username** (handle) at registration — e.g. `voidoflimbo`.
- Used in profile URLs (`/u/{username}`) and for @-mentions.
- Alphanumeric + underscores; case-insensitive uniqueness check; between 3 and 30 characters.
- **Not a GUID** — the ULID primary key is the internal unique ID; the username is the human-readable public identifier.
- Can be changed (subject to a cooldown period TBD), but the old username is reserved for a grace period to prevent squatting.

### Profile Image & Cover Image

| Tier | Allowed formats |
|---|---|
| `free` | Static only: JPEG, PNG, WebP |
| `investor` / `super_owner` | Static + dynamic: GIF, WebM, MP4 (looping) |

- Dynamic image support is enforced at upload time based on the uploader's platform role.
- File size limits and dimension constraints are defined in platform config (set by `super_owner`).
- Users who downgrade (not currently possible, but future-proof) revert to displaying their last valid static image.

### User Data Model (additions)

```
users
  ...
  username          string unique          (handle; 3–30 chars; alphanumeric + underscores)
  username_changed_at timestamp nullable   (tracks cooldown)
  profile_image     string nullable        (storage path)
  cover_image       string nullable        (storage path)
```

---

## Access Matrix Summary

| Capability | Public | Free | Investor | Super Owner |
|---|---|---|---|---|
| View public Server pages | ✅ | ✅ | ✅ | ✅ |
| Register / log in | — | ✅ | ✅ | ✅ |
| Join a Server | ❌ | ✅ | ✅ | ✅ |
| Time-limited Server preview | ❌ | ✅ | ✅ | ✅ |
| Request weekly preview extension | ❌ | ✅ | ✅ | ✅ |
| Request demo feature access | ❌ | ✅ | ✅ | ✅ |
| Boost a Server (one-off) | ❌ | ✅ | ✅ | ✅ |
| Donate to a Server creator | ❌ | ✅ | ✅ | ✅ |
| Supporter subscription (any Server, no cap) | ❌ | ✅ | ✅ | ✅ |
| Follow users / Servers | ❌ | ✅ | ✅ | ✅ |
| Create a Server | ❌ | ❌ | ✅ (one) | ✅ (unlimited) |
| Subscribe Features to Server | ❌ | ❌ | ✅ | ✅ |
| Build Pages | ❌ | ❌ | ✅ (own Server) | ✅ |
| Appoint moderators | ❌ | ❌ | ✅ (own Server) | ✅ |
| Platform-wide config | ❌ | ❌ | ❌ | ✅ |

---

## Open Questions
- What is the exact Investor one-off payment amount? (Configured by `super_owner` — value TBD)
- What is the default Server preview duration?
