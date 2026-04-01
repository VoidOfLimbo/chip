# Pages & Page Builder

## Overview
Pages are the primary content unit inside a Server. A Server can contain multiple Pages. Pages are built with a **drag-and-drop component builder** — a GridStack-inspired canvas where reusable components are placed, resized, configured, and optionally bound to a live data source. A Page can be anything: a portfolio, showcase, wiki, landing page, event page, or any other custom layout.

This is **not** a document editor. The page is a responsive grid canvas. Each placed component occupies a region of the grid and can be independently configured, styled, and set to a visibility level. Text editors (Tiptap) are embedded only inside specific text-based component types.

---

## Core Concepts

### Component Library
The **Component Library** is the set of available component types. It is split into:
- **Platform components** — shipped by the platform, available to all Servers.
- **Server-custom components** — additional components defined by the Server owner for their own Server.

New components are added to the platform library via platform releases. Server owners can define custom components without platform involvement.

### Component Definitions
A `component_definition` is the **template** for a component type. It defines:
- What configuration options the component accepts (config schema).
- What data sources it can bind to (data source schema; optional).
- Grid size constraints (minimum/maximum width and height in grid units).
- Default dimensions when dropped onto the canvas.

### Component Instances (Page Components)
A `page_component` is a **placed instance** of a component definition on a specific Page. Each instance has:
- Grid position (`x`, `y`) and size (`w`, `h`).
- Its own `config` — the content and settings for that specific instance (e.g. the text in a text block, the image path in a gallery).
- An optional `data_source` binding — connects the instance to live data instead of (or in addition to) manual config.
- Its own visibility level.

### Grid
- The canvas is a **12-column grid**. Row height is configurable per Page.
- Components snap to grid cells on drag and resize.
- Components cannot overlap.
- Minimum size per component is defined in `component_definitions.min_w` / `min_h`.

---

## Data Model

### `pages`

```
pages
  id                ulid
  server_id         FK → servers
  title             string
  slug              string (unique within server)
  description       text nullable
  visibility        enum: public | users | pros | followers | friends | members | supporter | moderator | owner | link | private
  group_id          FK → groups nullable   (sub-scope when visibility = members)
  preview_visible   boolean default false  (expose during time-limited Server preview)
  is_published      boolean default false
  row_height_px     integer default 80     (pixel height of one grid row)
  order             integer
  created_by        FK → users
  deleted_at        timestamp nullable
  created_at / updated_at
```

### `component_definitions`

```
component_definitions
  id                ulid
  slug              string unique           (e.g. "text", "hero", "gallery", "hall_of_fame")
  name              string                  (display name in the component picker)
  description       text nullable
  category          enum: content | media | data | interactive | layout
  config_schema     json                    (JSON Schema describing the config object)
  datasource_schema json nullable           (JSON Schema for data source binding; null = no binding supported)
  min_w             integer default 1       (minimum width in grid columns)
  min_h             integer default 1       (minimum height in grid rows)
  max_w             integer nullable        (null = unrestricted)
  max_h             integer nullable        (null = unrestricted)
  default_w         integer                 (initial width when dropped onto canvas)
  default_h         integer                 (initial height when dropped onto canvas)
  is_platform       boolean default true    (false = server-custom component)
  server_id         FK → servers nullable   (null = platform-wide; set = server-custom only)
  is_active         boolean default true
  created_at / updated_at
```

### `page_components`

```
page_components
  id                ulid
  page_id           FK → pages
  component_def_id  FK → component_definitions
  x                 integer                 (column start, 0-based, 0–11)
  y                 integer                 (row start, 0-based)
  w                 integer                 (width in columns, >= min_w)
  h                 integer                 (height in rows, >= min_h)
  config            json                    (instance content and settings — shape defined by component_def config_schema)
  data_source       json nullable           (binding spec — shape defined by component_def datasource_schema)
  visibility        enum: public | users | pros | followers | friends | members | supporter | moderator | owner | link | private
  group_id          FK → groups nullable    (sub-scope when visibility = members)
  is_locked         boolean default false   (true = only server_owner can edit; moderators view only)
  created_by        FK → users
  deleted_at        timestamp nullable      (soft delete — retained for access log integrity)
  created_at / updated_at
```

---

## Visibility Rule

A component's visibility **cannot be less restrictive than its parent Page's visibility**. Visibility levels are fully ordered (1–9); a component may only be set to the same level or a more restrictive one.

`link` and `private` are out-of-band mechanisms. A `link`-shared component overlays any stack level. A `private`-grant component sits outside the ordered comparison entirely.

| Page visibility | Component can be set to |
|---|---|
| `owner` | `owner`, `private`, `link` only |
| `moderator` | `moderator` and more restrictive |
| `supporter` | `supporter` and more restrictive |
| `members` | `members` and more restrictive |
| `friends` | `friends` and more restrictive |
| `followers` | `followers` and more restrictive |
| `pros` | `pros` and more restrictive |
| `users` | `users` and more restrictive |
| `public` | Any level |
| `private` | `private` only |
| `link` | `link` — stack level is independent of the link mechanism |

---

## Data Source Bindings

Components with `datasource_schema` set can bind their content to a named data source instead of (or as a fallback to) manual `config` values.

| Data source key | Description | Example use |
|---|---|---|
| `manual` | Inline data from `config` JSON — default for all components | Any component |
| `server_members` | Live pull from the Server's member list | Member directory component |
| `server_stats` | Aggregate Server stats (member count, boost tier, credit balance) | Stats component |
| `hall_of_fame` | Donation records where donor opted in to public display | Hall of Fame component |
| `expense_summary` | Requesting user's own Expense Planner data (requires feature access) | Expense summary widget |
| `plan_summary` | Requesting user's own Life Planner data (requires feature access) | Plan summary widget |
| `external_api` | URL-polled JSON endpoint (future) | Custom data widgets |

Data source bindings are resolved server-side on page render and are permission-gated. If the viewing user lacks access to the bound data source, the component renders its `config` fallback (or is hidden if no fallback is configured).

---

## Platform Component Library (v1)

Each component defines its own `config_schema`. Below is a summary of platform-shipped components:

| Slug | Category | Description | Tiptap editor |
|---|---|---|---|
| `text` | content | Titled section with rich body and optional footer | ✅ body |
| `image_card` | media | Image with header, body, and footer; image inline or as background | ✅ body |
| `hero` | media | Full-width banner: background image, headline, sub-text, CTA button | ❌ |
| `feature_list` | content | Items with icon, title, and description in vertical or horizontal layout | ❌ |
| `cta_banner` | content | Highlighted CTA strip with text and button | ❌ |
| `divider` | layout | Visual separator (line / space / dotted) | ❌ |
| `embed` | media | External content via URL (YouTube, maps, social posts, etc.) | ❌ |
| `3d_scene` | media | WebGL / CSS 3D canvas (fixed height or fullpage) | ❌ |
| `gallery` | media | Image grid with optional captions; configurable column count | ❌ |
| `profile_card` | content | Name, avatar, bio, and links — sourced from a user ID or entered manually | ❌ |
| `stats` | data | Row of highlighted metrics with labels and descriptions | ❌ |
| `timeline` | content | Vertical timeline of dated events | ❌ |
| `hall_of_fame` | data | List of donors who opted in to public display; bound to `hall_of_fame` data source | ❌ |

### Adding New Components
New platform components are added via a migration that inserts a row into `component_definitions`. No schema migration is needed for existing page component instances — the `config` and `data_source` JSON columns are unaffected. Frontend Vue components are registered in the component renderer map by slug.

---

## Server-Custom Components

Server owners can extend the library with their own components:
- A custom component definition is inserted with `is_platform = false` and `server_id` set to their Server.
- Custom components are **only available** on the Server where they are defined.
- The Server owner defines the `config_schema`, grid constraints, and the Vue component (uploaded or declared in their Server config).
- Custom component management is controlled by `server.components.manage` permission (Server owner only in v1).

---

## Page Builder Rules

- Server owner and moderators can open the builder for any Page on their Server.
- The builder provides a drag-and-drop canvas: components are picked from the Component Library panel, dragged onto the grid, and positioned/resized freely within constraints.
- Each placed component opens a settings panel where the Server owner fills in the `config` fields and optionally sets a data source binding.
- Components marked `is_locked = true` can only be edited by the `server_owner`; moderators see them but cannot change them.
- Reordering / moving a component updates `x`, `y`, `w`, `h` in a single PATCH request.
- Multiple components can be moved in a single batch update (positions saved as an array).
- Draft Pages (`is_published = false`) are only visible to the Server owner and moderators.
- Deleting a component uses **soft delete** (`deleted_at`) — retained for access log integrity.

---

## Starter Layout Templates

When creating a new Page, the Server owner can choose a starter template. Templates are pre-composed sets of component instances that can be modified freely after selection.

| Template | Components pre-placed |
|---|---|
| Portfolio | Hero + profile card + feature list + gallery + CTA banner |
| Showcase | Hero + stats + timeline + gallery |
| Wiki | Text + divider chain |
| Landing | Hero + feature list + CTA banner |
| Blank | Empty 12-column canvas |

Templates are applied once at Page creation — they create `page_components` rows using the component definitions. There is no ongoing link to the template after creation.

---

## Page Quotas

The number of published Pages a Server can have is determined by platform config (values set by `super_owner`). Draft Pages do not count against the quota.

---

## Future: Discussion Pages

Pages with real-time chat, voice/video, and screen sharing are planned but out of scope for v1.

---

## Open Questions
- Should block soft-delete have a hard purge policy (e.g. purge after 90 days)?
- Should the external_api data source support polling intervals and caching in v1 or be deferred?
