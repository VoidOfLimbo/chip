# Chip - Product Summary

## Planning Source of Truth
This file is the high-level direction for the project. For each feature, there are more detailed documents in the `blueprint/` directory. Each of those documents should expand on the high-level features described here, and should be consistent with the direction set in this file.

When this file changes, review and update the rest of the planning docs in `blueprint/` to keep everything aligned with the end goal. This is a consistency check to ensure that all the details in the planning docs are still relevant and accurate based on the high-level direction set here. While doing this ignore `blueprint/old/` directory as it contains old versions of docs that are only there for reference and should not be updated.

## Update Rule
- Keep this document short and high-level.
- Treat changes here as product-direction changes.
- When making changes here, review the rest of the planning docs to ensure they are still consistent with the new direction. Track any sections in those docs that need updating based on the changes made here and create or update the todo list.
- After each change, run a quick consistency pass on all files in `blueprint/` and update any sections that needs updating.

## What We Are Building
Chip is a **community platform with a flexible page builder and personal productivity tools**, built with Laravel + Inertia (Vue frontend). Users create free accounts and join the main server ("Limbo") where they will have access to the pages created by the server owner ("VoidOfLimbo"). Users can request for demo access to certain pages or features marked as demoable for a demo period. The Super Owner ("VoidOfLimbo") runs the platform and hosts the main Server ("Limbo"). Users can choose to pay one off fee to have their own Server, where they can create their own page, subscribe to features, and build their own community.

## Tech Stack
- Backend: Laravel 13
- Frontend: Inertia.js with Vue 3
- Database: PostgreSQL
- Caching: Redis
- Payments: Stripe
- Local development: Laravel Sail (Docker) with PostgreSQL, Redis, pgAdmin, and Mailpit

## Reference files
- `blueprint/principles.md` — key principles guiding the product development and design.
- `blueprint/access.md` — visibility and access control rules.

### User Profiles
- Every user has a public profile with a profile image and cover image.
- Each user chooses a unique **username** (handle) at registration — e.g. `voidoflimbo`. Human-readable; not a GUID (the internal ULID primary key serves as the unique ID).
- **Free users** may upload static images only (JPEG, PNG, WebP) for both profile and cover.
- **Premium / Loyalist** users may also upload dynamic images (animated GIF, WebM, MP4). They will also have the option to set a static fallback image for platforms that don't support dynamic formats. The platform will automatically serve the appropriate format based on the viewer's device capabilities and connection speed, prioritizing performance while still providing an enhanced experience for those who can take advantage of it.
- **Loyalist** users have access to special effects and filters for their profile and cover images, such as dynamic overlays, particle effects, and custom animations. These features allow Loyalist users to further personalize their profiles and express their creativity, while still adhering to the platform's guidelines for content and performance.

- Image format restrictions are enforced at upload time based on platform role.

### Landing Page & Registration
- Landing page (`/`) is public — shows public pages of the platform and sign-up options.
- Registering creates a `free` account. No payment required.

### Servers & Communities
- A **Server** is a branded community space with its own pages, members, subscriptions, and access configuration.
- The default server is **"Limbo"**, hosted by the Super Owner ("VoidOfLimbo") — all registered users are placed here on signup.
- Any user can pay a one-time fee to create their own Server.
- Server owners invite users as members, set visibility rules per-page, and configure server subscription tiers.
- Content visibility within a server follows the access stack defined in `blueprint/access.md`.
- Servers are isolated environments — members of one server do not automatically have access to another.

### Page Builder
- Server owners compose **Pages** using a block-based editor (text, media, embeds, forms, callouts, etc.).
- Pages are fully customizable and can be nested or linked within a server's navigation.
- Each page carries its own visibility level using the access stack in `blueprint/access.md`.
- Pages can be marked as **demoable** — non-members may request a limited-time preview, subject to server owner approval.
- The Limbo server hosts public-facing pages visible to unauthenticated visitors and all platform users.

### Productivity Tools
- Chip ships with built-in **personal productivity tools** available to all registered users.
- Tools include: **Expense Planner**, **Life Planner**, and other smart utility modules.
- All productivity data is private to the user by default — not exposed to the community.
- Server owners may optionally surface shared or server-scoped productivity features within their Server.

### Payments & Subscriptions
- Payments are processed via **Stripe**.
- **Server creation** is a one-time purchase (fee set by the Super Owner).
- **Loyalist** tier is a recurring platform-level subscription granting enhanced profile features and platform-wide benefits.
- **Server owners** configure their own subscription tiers, pricing, and feature gating for their server members.
- Demo access to demoable pages or features is time-limited and fully configured by the server owner.

