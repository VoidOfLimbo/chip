# Progress Tracker

## How to Use
- Update `Status` as work moves forward.
- Keep `Notes` concise — link to relevant files or PRs.
- Statuses: `Planning` | `Ready` | `In Progress` | `Done` | `Blocked`

---

## Phase 0 — Local Environment Setup

| # | Task | Status | Notes |
|---|---|---|---|
| 0.1 | Docker installed and running | Planning | — |
| 0.2 | Node.js ≥ 20 + npm on host | Planning | — |
| 0.3 | Stripe CLI installed on host | Planning | — |
| 0.4 | Git / GitHub SSH access configured | Planning | — |

---

## Phase 1 — Repository & Sail Baseline

| # | Task | Status | Notes |
|---|---|---|---|
| 1.1 | Sail configured with all 5 containers (pgsql, pgadmin, redis, mailpit) | Planning | See `blueprint/development-setup.md` |
| 1.2 | `.env.example` with all required variables | Planning | See `blueprint/prerequisites.md` |
| 1.3 | `php artisan migrate` runs clean | Planning | — |
| 1.4 | `npm run dev` and `npm run build` succeed | Planning | — |
| 1.5 | Queue worker verified (`artisan queue:work`) | Planning | — |
| 1.6 | Scheduler verified (`artisan schedule:work`) | Planning | — |
| 1.7 | Mailpit captures outbound mail | Planning | — |

---

## Phase 2 — Database Migrations

| # | Table | Status | Notes |
|---|---|---|---|
| 2.1 | `users` (verify ULID + `platform_role` column) | Planning | Existing migration needs ULID + role column |
| 2.2 | `servers` | Planning | — |
| 2.3 | `server_members` | Planning | — |
| 2.4 | `groups` | Planning | — |
| 2.5 | `group_members` | Planning | — |
| 2.6 | `features` | Planning | — |
| 2.7 | `server_features` | Planning | — |
| 2.8 | `server_feature_access` | Planning | — |
| 2.9 | `server_credit_transactions` | Planning | — |
| 2.10 | `pages` | Planning | — |
| 2.11 | `component_definitions` | Planning | — |
| 2.12 | `page_components` | Planning | — |
| 2.13 | `share_tokens` | Planning | — |
| 2.14 | `access_logs` | Planning | — |
| 2.15 | `payments` | Planning | — |
| 2.16 | `donations` | Planning | — |
| 2.17 | `payment_consents` | Planning | Replaces `subscription_waivers`; records consent for all payment types |

---

## Phase 3 — Stripe & Cashier

| # | Task | Status | Notes |
|---|---|---|---|
| 3.1 | Install Laravel Cashier | Planning | — |
| 3.2 | Run Cashier migrations | Planning | — |
| 3.3 | Cashier columns on `users` | Planning | — |
| 3.4 | `Billable` trait on `User` model | Planning | — |
| 3.5 | Stripe webhook route registered | Planning | — |
| 3.6 | Stripe webhook secret in `.env` | Planning | — |
| 3.7 | Webhook forwarding smoke test | Planning | — |

---

## Phase 4 — Authentication & Platform Roles

| # | Task | Status | Notes |
|---|---|---|---|
| 4.1 | Auth scaffolding (register, login, reset, verify) | Planning | — |
| 4.2 | `platform_role` enum on `users` | Planning | — |
| 4.3 | `free` role assigned on registration | Planning | — |
| 4.4 | `super_owner` Gate::before bypass | Planning | — |
| 4.5 | Platform role gates defined | Planning | — |
| 4.6 | Database seeder: Void + Limbo + feature catalogue + component definitions | Planning | — |

---

## Phase 5 — Server Roles, Policies & Feature Access

| # | Task | Status | Notes |
|---|---|---|---|
| 5.1 | `ServerMember` query service | Planning | — |
| 5.2 | `EnsureServerMember` middleware | Planning | — |
| 5.3 | `EnsureServerRole` middleware (parametric) | Planning | `server.role:moderator` etc. |
| 5.4 | `ServerPolicy` | Planning | `approveJoin` server_owner only |
| 5.5 | `PagePolicy` (with visibility checks) | Planning | — |
| 5.6 | `PageComponentPolicy` | Planning | — |
| 5.7 | `ModerationPolicy` | Planning | Moderator-scoped actions |
| 5.8 | Feature access resolver (most-specific-wins) | Planning | — |

---

## Phase 6 — Frontend Foundation

| # | Task | Status | Notes |
|---|---|---|---|
| 6.1 | Inertia v3 wired up | Planning | — |
| 6.2 | Vue component directory structure | Planning | — |
| 6.3 | Wayfinder type generation | Planning | — |
| 6.4 | Tailwind CSS configured | Planning | — |
| 6.5 | Base layout components (auth + guest) | Planning | — |
| 6.6 | Toast / flash notification system | Planning | — |
| 6.7 | GridStack.js for Vue integrated | Planning | — |

---

## Phase 7 — Seed Data & Smoke Test

| # | Check | Status | Notes |
|---|---|---|---|
| 7.1 | `migrate:fresh --seed` completes | Planning | — |
| 7.2 | Void user exists (`super_owner`) | Planning | — |
| 7.3 | Limbo server seeded | Planning | — |
| 7.4 | Feature catalogue seeded | Planning | — |
| 7.5 | Component definitions seeded | Planning | — |
| 7.6 | Login as Void succeeds | Planning | — |
| 7.7 | Stripe webhook received and logged | Planning | — |
| 7.8 | Queue processes a test job | Planning | — |
| 7.9 | Email captured in Mailpit | Planning | — |

---

## Feature Development

### Core Platform

| Feature | Status | Blueprint | Notes |
|---|---|---|---|
| Landing page (public) | Planning | `pages-builder.md` | — |
| User registration + email verification | Planning | `access-tiers.md` | — |
| Login / logout / password reset | Planning | `access-tiers.md` | — |
| Platform role upgrade (Investor one-off payment) | Planning | `payments-subscriptions.md` | — |
| Server creation (Investor + Super Owner) | Planning | `servers-groups.md` | — |
| Server join (public request / email invite / link+OTP) | Planning | `servers-groups.md` | — |
| Time-limited Server preview + extension request | Planning | `servers-groups.md` | — |
| Groups management | Planning | `servers-groups.md` | — |
| Follow (unidirectional) | Planning | `summary.md` | — |

### Page Builder

| Feature | Status | Blueprint | Notes |
|---|---|---|---|
| Page CRUD (create, publish, delete) | Planning | `pages-builder.md` | — |
| Drag-and-drop component canvas (GridStack) | Planning | `pages-builder.md` | — |
| Component Library UI (picker panel) | Planning | `pages-builder.md` | — |
| Component config settings panel | Planning | `pages-builder.md` | — |
| Data source binding on components | Planning | `pages-builder.md` | — |
| Starter layout templates | Planning | `pages-builder.md` | — |
| Component visibility per instance | Planning | `pages-builder.md` | — |
| Component locking (`is_locked`) | Planning | `pages-builder.md` | — |
| Server-custom component definitions | Planning | `pages-builder.md` | — |
| All 13 platform component types implemented | Planning | `pages-builder.md` | — |

### Content Visibility & Access

| Feature | Status | Blueprint | Notes |
|---|---|---|---|
| Visibility enforcement on pages + components | Planning | `content-visibility.md` | — |
| Share token generation + OTP verification flow | Planning | `content-visibility.md` | — |
| Access logging (all event types) | Planning | `content-visibility.md` | — |

### Payments & Subscriptions

| Feature | Status | Blueprint | Notes |
|---|---|---|---|
| Investor one-off payment (Stripe Payment Intent) | Planning | `payments-subscriptions.md` | — |
| Server feature subscription (Stripe Subscription) | Planning | `payments-subscriptions.md` | — |
| Credit pool offset on Server monthly bill | Planning | `payments-subscriptions.md` | — |
| Server boost (one-off) + credit pool + supporter role | Planning | `payments-subscriptions.md` | — |
| Supporter subscription (multi-term, multi-Server) | Planning | `payments-subscriptions.md` | — |
| 14-day cancellation right waiver (signup + renewal) | Planning | `payments-subscriptions.md` | — |
| Renewal + expiry warning notifications (7d + 1d) | Planning | `payments-subscriptions.md` | — |
| Donation + Hall of Fame opt-in | Planning | `payments-subscriptions.md` | — |
| 7-day grace period on failed Investor Server bill | Planning | `payments-subscriptions.md` | — |
| Pre-payment consent (all checkout flows) | Planning | `payments-subscriptions.md` | — |
| 80% refund within 7 days flow | Planning | `payments-subscriptions.md` | — |
| Stripe webhook handler (all events) | Planning | `payments-subscriptions.md` | — |

### Features (Server Modules)

| Feature | Status | Blueprint | Notes |
|---|---|---|---|
| Expense Planner | Planning | `expense-planner.md` | — |
| Life Planner | Planning | `life-planner.md` | — |
| OCR File Parser | Planning | `smart-productivity-tools.md` | — |
| Image Processor | Planning | `smart-productivity-tools.md` | — |

---

## Blueprint Documents

| Document | Status | Last updated |
|---|---|---|
| `summary.md` | ✅ Complete | 2026-03-31 |
| `prerequisites.md` | ✅ Complete | 2026-03-31 |
| `access-tiers.md` | ✅ Complete | 2026-03-31 |
| `servers-groups.md` | ✅ Complete | 2026-03-31 |
| `pages-builder.md` | ✅ Complete | 2026-03-31 |
| `content-visibility.md` | ✅ Complete | 2026-03-31 |
| `roles-permissions.md` | ✅ Complete | 2026-03-31 |
| `payments-subscriptions.md` | ✅ Complete | 2026-03-31 |
| `development-setup.md` | ✅ Complete | 2026-03-31 |
| `expense-planner.md` | ✅ Complete | 2026-03-31 |
| `life-planner.md` | ✅ Complete | 2026-03-31 |
| `legal.md` | ✅ Complete | 2026-03-31 |
