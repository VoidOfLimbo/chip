# Access Tiers

## Overview
Chip has three platform-level account tiers. Tiers are permanent and platform-wide. Server-level roles (`server_owner`, `moderator`, `supporter`, `member`) are separate and described in `blueprint/servers-groups.md`.

| Tier | How gained | Platform role |
|---|---|---|
| Free | Registration (no cost) | `free` |
| Investor | One-off payment to platform | `investor` |
| Super Owner | Seeded at install | `super_owner` |

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
- **Request to join** a Server (if the Server has `is_public = true`) or accept an invite (email or link with OTP).
- Receive a **time-limited preview** of a Server upon first visit. During the preview they can see pages the Server owner marks as preview-accessible.
  - Before or after expiry: can request a **1-week access extension once per week** — approved by Server owner or moderator (`server.members.preview_extend` permission).
- **Request access to demo features** on a Server — the Server owner approves per user at their discretion.
- **Follow** other users or Servers (unidirectional).
- **Boost a Server** (one-off payment) → gains `supporter` server role on that Server and access to whatever the Server owner grants supporters.
- **Donate** to a Server creator (any amount, anonymous or public in Hall of Fame).
- **Subscribe** to any Server for sustained supporter access (see Supporter Subscriptions below).

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

### Server Monthly Cost
The Investor pays monthly for Features subscribed to their Server. If their credit pool has a balance, it offsets the bill automatically. If the bill cannot be covered: a **7-day grace period** applies — the Server's features are suspended after 7 days of non-payment, but the Server itself (and its Pages) remains visible.

### What Investors Cannot Do
- Create more than one Server.
- Cash out Server credit pool funds.

---

## Super Owner

| Attribute | Value |
|---|---|
| How gained | Seeded at install |
| Billing | None |
| Platform role | `super_owner` |

### What the Super Owner Can Do
- Everything an Investor can do, with no limits.
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
| Server | Limbo (type: `owner`) |

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
