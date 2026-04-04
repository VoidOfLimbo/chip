# Prerequisites & Project Setup Plan

## Overview
This document defines everything that must be in place before feature implementation begins. It is ordered — earlier phases must be complete before the next phase starts. Each prerequisite maps to specific files or configuration that will be committed to the repository.

---

## Phase 0 — Local Environment

These are one-time developer machine requirements. They do not produce committed code.

| # | Prerequisite | Tool / Notes |
|---|---|---|
| 0.1 | Docker Desktop (or Colima on macOS) installed and running | Required for Laravel Sail |
| 0.2 | Node.js ≥ 20 + npm installed on host | For `npm install` and Vite |
| 0.3 | Stripe CLI installed on host | For webhook forwarding (`stripe listen`) |
| 0.4 | Git configured with SSH or PAT for GitHub | Push access to `VoidOfLimbo/chip` |

---

## Phase 1 — Repository & Sail Baseline

Everything needed to run the app locally with all Sail services healthy.

| # | Prerequisite | Deliverable |
|---|---|---|
| 1.1 | Configure Sail with all 5 containers | `docker-compose.yml` — `laravel.test`, `pgsql` (17), `pgadmin`, `redis` (7), `mailpit` |
| 1.2 | `.env.example` covering all required variables | `.env.example` committed; `.env` gitignored |
| 1.3 | Verify `php artisan migrate` runs clean on a fresh database | Existing base migrations pass |
| 1.4 | Verify `npm run dev` and `npm run build` succeed | Vite config correct, no asset errors |
| 1.5 | Verify queue worker starts (`artisan queue:work`) | Redis connection works inside Sail |
| 1.6 | Verify scheduler starts (`artisan schedule:work`) | No errors on startup |
| 1.7 | Verify Mailpit captures outbound mail | Send a test notification from tinker |

### Required `.env` Variables (full list)

```
# App
APP_NAME=Chip
APP_ENV=local
APP_KEY=
APP_DEBUG=true
APP_URL=http://localhost

# Database
DB_CONNECTION=pgsql
DB_HOST=pgsql
DB_PORT=5432
DB_DATABASE=chip
DB_USERNAME=sail
DB_PASSWORD=password

# Redis
REDIS_HOST=redis
REDIS_PORT=6379
QUEUE_CONNECTION=redis
CACHE_STORE=redis
SESSION_DRIVER=redis

# Mail
MAIL_MAILER=smtp
MAIL_HOST=mailpit
MAIL_PORT=1025
MAIL_FROM_ADDRESS=noreply@chip.local
MAIL_FROM_NAME=Chip

# pgAdmin
PGADMIN_DEFAULT_EMAIL=admin@chip.local
PGADMIN_DEFAULT_PASSWORD=secret
PGADMIN_LISTEN_PORT=5050

# Stripe
STRIPE_KEY=pk_test_...
STRIPE_SECRET=sk_test_...
STRIPE_WEBHOOK_SECRET=whsec_...

# Platform Seed
SUPER_OWNER_NAME=Void
SUPER_OWNER_EMAIL=bipin.paneru.9@gmail.com
SUPER_OWNER_PASSWORD=         # set locally, never committed
```

---

## Phase 2 — Database Foundation

All tables that every other feature depends on. Must be migrated before any feature work.

| # | Migration | Depends on |
|---|---|---|
| 2.1 | `users` table (already exists — verify ULID + roles) | — |
| 2.2 | `servers` table | `users` |
| 2.3 | `server_members` table | `servers`, `users` |
| 2.4 | `groups` table | `servers`, `users` |
| 2.5 | `group_members` table | `groups`, `users` |
| 2.6 | `features` table (platform feature catalogue) | — |
| 2.7 | `server_features` table | `servers`, `features` |
| 2.8 | `server_feature_access` table | `servers`, `features`, `users`, `groups` |
| 2.9 | `server_credit_transactions` table | `servers`, `users` |
| 2.10 | `pages` table | `servers`, `groups`, `users` |
| 2.11 | `component_definitions` table | `servers` |
| 2.12 | `page_components` table | `pages`, `component_definitions`, `groups`, `users` |
| 2.13 | `share_tokens` table | `users` |
| 2.14 | `access_logs` table | `users`, `share_tokens` |
| 2.15 | `payments` table | `users`, `servers` |
| 2.16 | `donations` table | `users`, `servers` |
| 2.17 | `payment_consents` table | `users` |

### ULID Convention (all tables)
- Primary key: `$table->ulid('id')->primary()`
- Foreign ULID: `$table->foreignUlid('user_id')->constrained()`
- All Eloquent models use the `HasUlids` trait.

---

## Phase 3 — Stripe & Cashier Setup

Must be complete before any payment or subscription feature is built.

| # | Prerequisite | Notes |
|---|---|---|
| 3.1 | Install Laravel Cashier for Stripe | `composer require laravel/cashier` |
| 3.2 | Run Cashier migrations | Adds `subscriptions`, `subscription_items` tables |
| 3.3 | Add Cashier columns to `users` | `stripe_id`, `pm_type`, `pm_last_four`, `trial_ends_at` |
| 3.4 | Billable trait on `User` model | `use Billable` |
| 3.5 | Stripe webhook route registered | `POST /stripe/webhook` handled by `CashierController` |
| 3.6 | Stripe webhook secret configured in `.env` | `STRIPE_WEBHOOK_SECRET` |
| 3.7 | Test: `stripe listen --forward-to http://localhost/stripe/webhook` works | Verify webhook events received in logs |

---

## Phase 4 — Authentication & Platform Roles

| # | Prerequisite | Notes |
|---|---|---|
| 4.1 | Authentication scaffolding | Registration, login, logout, password reset, email verification |
| 4.2 | `platform_role` enum column on `users` | `super_owner` \| `investor` \| `free` |
| 4.3 | `free` role assigned on registration | Registration controller sets default |
| 4.4 | `super_owner` check bypasses all gates | Implement in `AppServiceProvider` or `AuthServiceProvider` — check once at Gate::before |
| 4.5 | Platform role gates defined | `Gate::define('investor', ...)` etc. via a `PlatformRoleServiceProvider` |
| 4.6 | Database seeder: Void user + Limbo server | `DatabaseSeeder` creates `super_owner` user + `owner`-type Server |

---

## Phase 5 — Server Role Middleware & Policies

| # | Prerequisite | Notes |
|---|---|---|
| 5.1 | `ServerMember` query service | Resolves a user's role on a given Server from `server_members` |
| 5.2 | `EnsureServerMember` middleware | Aborts 403 if user is not an active member (`status = active`) of the route-bound Server. **Not applied to feature routes** (to allow preview users with auto-demo access) or content-serving routes (handled by `PagePolicy`) |
| 5.3 | `EnsureServerRole` middleware | Parametric: `->middleware('server.role:moderator')` — aborts 403 if user's server_role is below required level |
| 5.4 | `ServerPolicy` | `view`, `manage`, `inviteMembers`, `suspendMember`, `manageFeatures`, `investCredits`, `configureAccess`, `manageComponents` |
| 5.5 | `PagePolicy` | `view`, `create`, `update`, `publish`, `delete` — visibility checks included |
| 5.6 | `PageComponentPolicy` | `view`, `update`, `delete`, `hide` — respects `is_locked` |
| 5.7 | `ModerationPolicy` | `moderate`, `extendPreview`, `manageGroupMembers` |
| 5.8 | Feature access resolver | Most-specific-wins lookup in `server_feature_access` |

---

## Phase 6 — Frontend Foundation

| # | Prerequisite | Notes |
|---|---|---|
| 6.1 | Inertia v3 wired up and working | `app.blade.php`, `HandleInertiaRequests` middleware, `app.ts` bootstrapped |
| 6.2 | Vue component structure established | `resources/js/pages/`, `resources/js/components/` layout decided |
| 6.3 | Wayfinder type generation working | `php artisan wayfinder:generate` produces typed route helpers |
| 6.4 | Tailwind CSS (or chosen CSS framework) configured | `vite.config.ts` and `app.css` set up |
| 6.5 | Base layout component | Authenticated shell layout (nav, sidebar) and public/guest layout |
| 6.6 | Toast / notification system | For success/error flash messages from Inertia responses |
| 6.7 | GridStack.js for Vue integrated | Install `gridstack` npm package; wrap in a Vue composable |

---

## Phase 7 — Seed Data & Local Smoke Test

Run after all migrations and auth are in place. Confirms the foundation is solid before feature work begins.

| # | Check | Pass condition |
|---|---|---|
| 7.1 | `php artisan migrate:fresh --seed` completes without error | — |
| 7.2 | Void user exists with `super_owner` role | Verify via pgAdmin or tinker |
| 7.3 | Limbo server seeded and owned by Void | `servers` table has one row |
| 7.4 | Platform feature catalogue seeded | `features` table has rows for all v1 features |
| 7.5 | `component_definitions` seeded | All 13 platform components present |
| 7.6 | Login as Void works | Session created, role detected |
| 7.7 | Stripe webhook received and logged | `stripe trigger payment_intent.succeeded` → logged |
| 7.8 | Queue worker processes a test job | Dispatch a simple test job; confirm processed |
| 7.9 | Email captured in Mailpit | Trigger a password reset; confirm in Mailpit UI |

---

## Dependency Map (simplified)

```
Phase 0 (local env)
  └── Phase 1 (Sail baseline)
        ├── Phase 2 (database)
        │     ├── Phase 3 (Stripe/Cashier)
        │     └── Phase 4 (auth + roles)
        │           └── Phase 5 (server roles)
        └── Phase 6 (frontend)
              └── Phase 7 (smoke test — runs after 1–6 all done)
```

---

## Open Questions
- Will the Stripe CLI be installed locally by each developer, or containerised inside Sail?
- Do we need a `.env.testing` with separate Stripe mocks for CI?
