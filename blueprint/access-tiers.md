# Access Tiers

## Overview
Chip is the owner's personal portfolio and productivity platform. Access tiers control what different visitors and paying users can see and do. The model is:

- **Free** — browse the owner's public profile and any tenant's public profile (CV/portfolio)
- **Supporter** — experience interactive demos of the app's special features
- **Investor** — become a full tenant with their own dedicated space

Every request passes through a gate that evaluates the user's tier and, for Investors, their tenancy context.

---

## Public (Unauthenticated)

| Attribute | Value |
|---|---|
| Route | `/` (landing page) |
| Auth required | No |
| Access | Landing page only |

- Any visitor can view the landing page.
- The landing page presents the owner's elevator pitch and the three account options with feature comparison.
- All other routes redirect unauthenticated visitors to the landing/login page.

---

## Free Account

| Attribute | Value |
|---|---|
| Sign-up cost | Free |
| Billing | None |
| Organisation | Public Organisation (read-only member) |
| Role | `free` |
| Access level | Read-only portfolio/CV view — owner and all tenant public spaces |

### What Free Users See
The owner's public profile and any Investor tenant's public profile — the same information each party chooses to make public:
- Bio, skills, experience, and education (CV-style)
- Showcase projects and work samples
- Published blog posts or articles (if enabled by the owner/tenant)
- Contact / social links

This is analogous to browsing LinkedIn profiles. Free users cannot interact with any tools.

### Restrictions
- No access to special features (Expense Planner, Life Planner, Smart Tools).
- No demo access.
- No persistent workspace of their own.
- Cannot name an Organisation or manage anything.

### Upgrade Path
- Upgrade to Supporter (one-off payment) for demo access.
- Upgrade directly to Investor (monthly subscription) to become a tenant.

---

## Supporter Account

| Attribute | Value |
|---|---|
| Sign-up cost | One-off payment (amount TBD) |
| Billing | None recurring |
| Organisation | Their own named **Supporter Organisation** (created at registration or on upgrade from Free) |
| Role | `supporter` |
| Access level | Free content + interactive demos of special features |

### What Supporters Can Do
- Everything a Free user can see.
- Access **interactive demos** of all special features:
  - **Expense Planner demo** — explore the feature using sandboxed sample data. Changes do not persist between sessions (or persist in a sandboxed demo workspace, TBD).
  - **Life Planner demo** — explore task and goal tracking with sample plans.
  - **Smart Productivity Tools demo** — upload a file and try OCR/image processing; results are ephemeral.
- The demo experience is intentionally limited — it showcases the value without granting a full tenant workspace.

### Restrictions
- No persistent tenant space — demo data is sandboxed and ephemeral.
- Supporter Organisation is demo-scoped only: no member invites, no team management.
- Cannot unlock Investor Abilities.

### Upgrade Path
- Upgrade to Investor to get a full tenant space and persistent data.

---

## Investor Account

| Attribute | Value |
|---|---|
| Sign-up cost | Monthly subscription (feature add-on driven pricing) |
| Billing | Monthly, based on chosen add-ons |
| Organisation | Their own named **Tenant Organisation** |
| Role | `investor` |
| Access level | Full tenant — own space, own public section, own demo section, full tools |

### What Investors Get
An Investor becomes a **tenant** of the platform. Their Tenant Organisation is a self-contained space that mirrors the structure of the owner's platform:

#### Their Own Public Section (mirrors Free tier)
- Investors can publish their own public profile within their tenant space.
- Visitors to the Investor's tenant space (their own URL/slug) can browse the Investor's profile for free, just as Free users browse the owner's profile.

#### Their Own Demo Section (mirrors Supporter tier)
- Investors can expose interactive demos of their tenant's tools to their own Supporter-tier visitors.
- This is effectively a white-label demo environment scoped to the Investor's tenant.

#### Full Special Feature Access
- **Expense Planner** — full access with persistent, real data scoped to their Tenant Organisation.
- **Life Planner** — full access; team-shared plans available as an add-on.
- **Smart Productivity Tools** — full access to OCR parser, image processor, etc. via add-on Abilities.

#### Team Management
- Investor can create and manage Teams within their Tenant Organisation.
- Teams scope access to plans, expenses, and shared content within the tenant.

### Premium Feature Add-ons (Abilities)
- Investor selects add-ons at sign-up or from billing settings.
- Each add-on unlocks one or more **Abilities** and increases the monthly cost.
- Removing an add-on revokes associated Abilities at the next billing cycle.
- See `blueprint/roles-permissions.md` for the full Ability model.

---

## Access Matrix Summary

| Feature | Public | Free | Supporter | Investor |
|---|---|---|---|---|
| Landing page | ✅ | ✅ | ✅ | ✅ |
| Owner's public portfolio/CV | ❌ | ✅ | ✅ | ✅ |
| Owner's published content | ❌ | ✅ | ✅ | ✅ |
| Tenant public portfolio/CV | ❌ | ✅ | ✅ | ✅ |
| Expense Planner (sandboxed demo) | ❌ | ❌ | ✅ | ✅ |
| Expense Planner (full — tenant) | ❌ | ❌ | ❌ | ✅ |
| Expense Planner (org-shared — tenant) | ❌ | ❌ | ❌ | ✅ (Ability) |
| Life Planner (sandboxed demo) | ❌ | ❌ | ✅ | ✅ |
| Life Planner (full — tenant) | ❌ | ❌ | ❌ | ✅ |
| Life Planner (team-shared / timeline) | ❌ | ❌ | ❌ | ✅ (Ability) |
| OCR File Parser (demo, ephemeral) | ❌ | ❌ | ✅ | ✅ |
| OCR File Parser (full — tenant) | ❌ | ❌ | ❌ | ✅ (Ability) |
| Image Processor (demo, ephemeral) | ❌ | ❌ | ✅ | ✅ |
| Image Processor (full — tenant) | ❌ | ❌ | ❌ | ✅ (Ability) |
| Own public portfolio section | ❌ | ❌ | ❌ | ✅ (tenant) |
| Own demo section for their visitors | ❌ | ❌ | ❌ | ✅ (tenant) |
| Named Tenant Organisation | ❌ | ❌ | ❌ | ✅ |
| Team management | ❌ | ❌ | ❌ | ✅ |
| Premium Abilities (add-ons) | ❌ | ❌ | ❌ | ✅ |

---

## Open Questions
- Does Supporter demo data persist between sessions (sandboxed workspace) or is it fully reset each session? No
- Can a Supporter upgrade to Investor and have their demo workspace migrated to a real tenant space? Yes
- What is the Supporter one-off payment amount? 6.9 GBP
- What is the pricing model for Investor subscriptions and add-ons? TBD, but likely a base fee + incremental cost per add-on.
- What happens when an Investor's subscription lapses — immediate downgrade to Supporter or a grace period? Investor retains access until the end of the billing cycle, then subscription becomes inactive. There will be multiple reminders and a clear downgrade path that may result in data loss or data will be preserved if it stays just inactive.
- Can an Investor's tenant space be accessed via a custom subdomain or URL slug (e.g. `chip.app/org-ulid/dashboard`)? Yes, likely a URL slug (e.g. `chip.app/org-ulid/dashboard`) to avoid DNS complications. 
- Do Investor tenants share the same app URL structure, or is routing per-tenant? Shared URL structure with tenant context determined by URL slug.
