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
Chip is a **personal portfolio and multi-tenant productivity platform** built with Laravel + Inertia (Vue frontend). The application owner uses it as their own productive workspace and personal showcase. Other users interact with the app at different levels — browsing the owner's public profile, trying interactive demos, or becoming full tenants with their own dedicated space.

## Roles at a Glance

| Who | What they are |
|---|---|
| **Owner** | The person who runs the app; has full super-admin control (`owner` role) |
| **Free user** | A visitor who can browse the owner's and any tenant's public portfolio/CV |
| **Supporter** | A paying visitor who can experience interactive demos of the app's features |
| **Investor** | A paying tenant who gets their own space — with their own public section, demo section, and full access to all productivity tools |

## High-Level Features

### Account Tiers
- **Landing page** (`/`) is public — accessible to everyone, no login required.
- **Free** — requires login — read-only access to the owner's public portfolio and any tenant's public portfolio (CV/profile, projects, public content). Similar to viewing a LinkedIn profile. Assigned to the Public Organisation.
- **Supporter** — everything Free can see, plus interactive demos of all special features (sandboxed, limited); one-off payment. Creates their own named Supporter Organisation.
- **Investor** — becomes a **tenant**: gets their own dedicated space that mirrors the owner's structure. Can expose their own public section (like Free) and their own demo section (like Supporter) to others. Full access to all special features. Monthly subscription, feature-driven pricing. Creates their own named Tenant Organisation.

### Organisations & Teams
- The **Owner Organisation** (seeded as "Limbo", type `owner`) is the root admin organisation, created at install. The Owner ("Void") is its sole member.
- The **Public Organisation** (seeded as "Public", type `public`) is also created at install. Free users are automatically assigned to it (read-only).
- **Supporter** users create their own named **Supporter Organisation** (type `supporter`) at registration or on upgrade from Free — demo-scoped, no teams.
- Each **Investor** creates a named **Tenant Organisation** (type `tenant`) at registration or on upgrade — their private space within the platform.
- Tenant Organisations can contain Teams; only the Investor (tenant owner) can manage Teams.

### Roles, Permissions & Abilities
- **Permissions** are predefined at build time and are granular action gates.
- **Roles** map to account tiers: `owner`, `free`, `supporter`, `investor`.
- **Abilities** are feature-flags granted by role or unlocked via Investor subscription add-ons.
- Investor subscription cost is the sum of chosen feature add-ons.

### Payments & Subscriptions
- Supporter pays a **one-off** fee for permanent demo access.
- Investor pays a **monthly subscription** whose cost is the sum of chosen feature add-ons.
- Payment flow built and tested in development (Sail + Stripe test mode).

### Development Environment
- Local development via **Laravel Sail** (Docker) with PostgreSQL, Redis, pgAdmin, and Mailpit.
- All tables use **ULID** primary keys.
- Payment integration tested end-to-end in the Sail environment using Stripe test mode.
- Queues, scheduled tasks, and email notifications all run and are testable within the Sail stack.

### Application Features

#### Portfolio / CV
- The owner's public profile: bio, skills, experience, projects, and any content they choose to publish.
- Fully visible to Free users (and everyone). This is the primary draw for Free sign-ups.
- Investors get their own portfolio section within their Tenant space.

#### Expense Planner
- Track and plan expenses with full recurring-cost support (bills, rent, subscriptions, etc.).
- **Supporter**: interactive demo with sandboxed sample data — can explore the feature without persistent storage.
- **Investor**: full access within their Tenant space — their own real, persistent expense data.

#### Life Planner
- Schedule and track tasks across life contexts: Work, Hobby, Life Goals, and custom tags.
- **Supporter**: interactive demo with sandboxed sample data.
- **Investor**: full access within their Tenant space, including team-shared plans (add-on).

#### Smart Productivity Tools
- OCR File Parser, Image Processor, and future tools.
- **Supporter**: limited demo — try a tool with a sample/uploaded file (no persistent results).
- **Investor**: full access via Ability/subscription add-ons within their Tenant space.

---

## Progress Tracker

| Area | Status | Notes |
|---|---|---|
| Account tiers & access | Planning | See `blueprint/access-tiers.md` |
| Organisations & Teams | Planning | See `blueprint/organisations-teams.md` |
| Roles, Permissions & Abilities | Planning | See `blueprint/roles-permissions.md` |
| Payments & Subscriptions | Planning | See `blueprint/payments-subscriptions.md` |
| Development Setup | Planning | See `blueprint/development-setup.md` |
| Portfolio / CV | Planning | See `blueprint/portfolio.md` |
| Expense Planner | Planning | See `blueprint/expense-planner.md` |
| Life Planner | Planning | See `blueprint/life-planner.md` |
| Smart Productivity Tools | Planning | See `blueprint/smart-productivity-tools.md` |

