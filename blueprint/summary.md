# Chip - Product Summary

## Planning Source of Truth
This file is the high-level direction for the project. For each feature, there are more detailed documents in the `plan/` directory. Each of those documents should expand on the high-level features described here, and should be consistent with the direction set in this file.

When this file changes, review and update the rest of the planning docs to keep everything aligned with the end goal. This is a consistency check to ensure that all the details in the planning docs are still relevant and accurate based on the high-level direction set here.

## Update Rule
- Keep this document short and high-level.
- Treat changes here as product-direction changes.
- When making changes here, review the rest of the planning docs to ensure they are still consistent with the new direction. Update any sections in those docs that need updating based on the changes made here.
- Additionally maintain a progress tracker 
- After each change, run a quick consistency pass on all files in `plan/` and update any sections that needs updating.

## What We Are Building
Chip is a **community platform with a flexible page builder and personal productivity tools**, built with Laravel + Inertia (Vue frontend). Users create accounts, join Servers (community spaces similar to Discord), view and interact with Pages published inside those Servers, and optionally pay to unlock features. The Super Owner runs the platform and hosts the primary Server ("Limbo"). Investors pay a one-off fee to create their own Server and subscribe features to it monthly. Any member can boost a Server to contribute toward its costs and gain supporter status on that Server.

---

## Platform Roles

| Role | How gained | What it unlocks |
|---|---|---|
| `super_owner` | Seeded at install (Void) | Absolute platform access; unlimited Servers at no cost; sets platform feature prices and caps |
| `investor` | One-off payment to platform (fixed price, set by super_owner) | Creates one Server; subscribes features to it monthly |
| `free` | Registration (no cost) | Joins Servers via public request or invite; sees what Server owner permits |

Platform roles are permanent and independent. A user holds exactly one platform role.

---

## Server Roles (per-Server, independent of platform role)

| Role | How gained |
|---|---|
| `server_owner` | Created the Server |
| `moderator` | Appointed by server_owner; must be a supporter of that Server |
| `supporter` | Has boosted that Server with a payment |
| `member` | Has joined the Server |

A user can hold different Server roles across different Servers simultaneously.

---

## High-Level Features

### Landing Page & Registration
- Landing page (`/`) is public — shows public pages of the platform and sign-up options.
- Registering creates a `free` account. No payment required.

### Access Flow — Owner's Server (Limbo)
1. Free user registers → receives **time-limited preview** of pages the Owner marks as free-accessible.
2. Before or after expiry: can request a **1-week extension once per week**.
3. Can request access to specific **demo features** — Owner approves per user.
4. **Monthly subscription** (supporter path): user waives the 14-day EU/UK cancellation right at signup and on each auto-renewal → gains sustained access to features the Owner permits supporters → gains `supporter` role on Limbo.
5. **Separate one-off Investor payment**: gains `investor` platform role → can create their own Server.

### Servers
- Only `super_owner` and `investor`-role users can create Servers.
- Maximum **one Server per user** (except `super_owner` who has no limit).
- `super_owner`'s Server is seeded at install — no cost.
- Each Server has its own monthly cost = sum of features currently subscribed to it.
- Server capacity (member slots, groups) has a base value and can be increased.

### Joining a Server
- Free users join via a **public join request** (if the Server exposes this) or an **invite** (email or link with OTP verification).
- A **time-limited preview** lets a user browse a Server before formally joining.
- Server owner controls what joined members (free, supporter, any group) can see and access.

### Boosting & Server Credit Pool
- Any registered user can **boost** a Server with a one-off payment.
- The boost amount must be between £0 and the maximum the Server owner sets for that feature.
- Boosting grants `supporter` role on that Server.
- All boost payments flow into the **Server's credit pool** — no cash out.
- The Server owner can invest credits toward:
  - Subscribing or maintaining Features on the Server
  - Purchasing additional Member Slots
  - Offsetting the Server's monthly feature subscription bill
- Unspent credits automatically offset the next monthly bill.
- **Refund policy on boosts**: 80% refunded if requested within 7 days; no refund after. See `blueprint/payments-subscriptions.md`.

### Donations
- Any registered user can make a **one-off donation** of any amount to the Server creator.
- Donations are voluntary — no platform entitlements attached.
- Donation funds enter the Server's credit pool (same as a boost).
- Donors choose: **display their name in the Server's Hall of Fame** (a placeable page component), or **remain anonymous**.
- Donors can include an optional personal message shown in the Hall of Fame.

### Supporter Subscriptions
- Any registered user can take out a **recurring supporter subscription** to any Server.
- No cap — a user can hold simultaneous subscriptions to multiple Servers; each is billed independently.
- Available terms: monthly, 3-month, 6-month, or annual (Server owner chooses which to offer).
- The platform sends renewal reminders at **7 days**, **3 days**, and **1 day** before each renewal and the same schedule before expiry.
- **Refund policy on subscriptions**: 80% within 7 days of a billing date, no refund after; cancellation also terminates immediately within the window. EU/UK 14-day cancellation right is waived at signup via explicit consent checkbox.

### Features
- Features are subscribable modules: Expense Planner, Life Planner, OCR File Parser, Image Processor, and more to come.
- The `super_owner` defines available features and their monthly prices on the platform.
- A Server owner subscribes features to their Server — each adds to the Server's monthly cost.
- The Server owner fully controls which members (free, supporter, specific group) can access each feature, and at what cost (£0 up to the platform-defined maximum).
- Feature usage has limits per Server tier. Higher boost tier = higher usage limits.
- New features are released on an ongoing cycle; existing features receive continuous bug fixes and improvements.

### Pages & Page Builder
- The primary content unit in a Server is a **Page**.
- Pages are built with a **drag-and-drop component builder** (GridStack-inspired): reusable components are placed on a 12-column canvas, resized, configured, and optionally bound to a live data source.
- The page is a grid canvas — not a document editor. Text editors (Tiptap) are embedded only inside specific text-based components.
- Each component type has a defined config schema and optional data source bindings (server stats, member list, expense data, Hall of Fame donors, etc.).
- The Component Library holds platform-shipped components and Server-custom components defined by the Server owner.
- Each Page has a visibility setting; each placed component also has its own visibility.
- **Visibility levels**: `public`, `link` (requires OTP to view), `authenticated`, `server` (any Server member), `group` (specific Group), `private`.
- Link-based sharing requires OTP verification (email or phone) to identify the viewer — no anonymous access.
- All content access events, share token usage, and OTP attempts are logged for security and accountability.
- Public pages are indexable by crawlers; all other visibility levels are excluded via `robots.txt`.
- **Future**: discussion-type Pages with real-time chat, voice, video, and screen sharing.

### Groups
- Groups replace Teams. A Group is a named sub-group within a Server.
- Server owners use Groups to organise members and scope page/feature access.
- Any registered user can be in a Group; the Server owner manages Group membership.

### Follow
- Any authenticated user can follow another user or a Server.
- Unidirectional (Twitter/LinkedIn-style). Mutual connections deferred to a future version.

### Payments
- **Investor one-off**: fixed amount (set by `super_owner` in platform config) to unlock Server creation.
- **Server feature subscriptions**: monthly, per-feature price set by `super_owner`; billed to the Server owner.
- **Member slot add-on**: separately subscribable, priced per increment.
- **Supporter boost**: one-off payment to a specific Server; amount between £0 and the Server owner's set maximum.
- **Supporter subscription**: recurring subscription to any Server; monthly / 3-month / 6-month / annual terms; no cap on simultaneous Servers; 14-day cancellation right waived at signup and every renewal; 7-day pre-renewal and pre-expiry warnings.
- **Donation**: one-off any-amount gift to the Server creator; enters the Server credit pool; donor chooses public (Hall of Fame) or anonymous.
- All Server-received payments enter the Server credit pool — no cash out.
- **Refund policy**: 80% of original payment if requested within 7 calendar days; no refund after 7 days. See `blueprint/legal.md` for jurisdiction compliance and T&C requirements.

### Development Environment
- Local development via **Laravel Sail** (Docker) with PostgreSQL, Redis, pgAdmin, and Mailpit.
- All tables use **ULID** primary keys.
- Payment integration tested end-to-end in the Sail environment using Stripe test mode.
- Queues, scheduled tasks, and email notifications all run and are testable within the Sail stack.

---

## Progress Tracker

See `blueprint/progress.md` for the full phase-by-phase progress tracker.

| Area | Status | Notes |
|---|---|---|
| Prerequisites & Setup Plan | Planning | See `blueprint/prerequisites.md` |
| Account tiers & access | Planning | See `blueprint/access-tiers.md` |
| Servers & Groups | Planning | See `blueprint/servers-groups.md` |
| Pages & Page Builder | Planning | See `blueprint/pages-builder.md` |
| Content Visibility & Access Logging | Planning | See `blueprint/content-visibility.md` |
| Roles, Permissions & Features | Planning | See `blueprint/roles-permissions.md` |
| Payments & Subscriptions | Planning | See `blueprint/payments-subscriptions.md` |
| Development Setup | Planning | See `blueprint/development-setup.md` |
| Expense Planner | Planning | See `blueprint/expense-planner.md` |
| Life Planner | Planning | See `blueprint/life-planner.md` |
| Smart Productivity Tools | Planning | See `blueprint/smart-productivity-tools.md` |

