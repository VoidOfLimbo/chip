# Pages & Page Builder

## Overview
Pages are the primary content unit inside a Server. A Server can contain multiple Pages. Pages are built with a **block-based page builder** — a flexible system of composable content blocks arranged by the Server owner or moderator. A Page can be anything: a portfolio, showcase, wiki, landing page, blog, or any other custom layout.

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
  visibility        enum: public | link | authenticated | server | group | private
  group_id          FK → groups nullable   (when visibility = group)
  preview_visible   boolean default false  (show during time-limited Server preview)
  is_published      boolean default false
  order             integer
  created_by        FK → users
  deleted_at        timestamp nullable     (soft delete)
  created_at / updated_at
```

### `page_blocks`

```
page_blocks
  id                ulid
  page_id           FK → pages
  type              string                 (block type slug, e.g. "paragraph")
  content           json                   (block-specific content — shape varies by type)
  visibility        enum: public | link | authenticated | server | group | private
  group_id          FK → groups nullable
  order             integer
  created_by        FK → users
  deleted_at        timestamp nullable     (soft delete — preserved for access log integrity)
  created_at / updated_at
```

---

## Visibility Rule

A block's visibility **cannot be less restrictive than its parent Page's visibility**.

| Page visibility | Block can be set to |
|---|---|
| `private` | `private` only |
| `group` | `group`, `private` |
| `server` | `server`, `group`, `private` |
| `authenticated` | `authenticated`, `server`, `group`, `private` |
| `link` | `link`, `authenticated`, `server`, `group`, `private` |
| `public` | Any level |

---

## Block Types (v1)

Each block type defines its own `content` JSON shape. New block types are added via platform updates — no migration required (existing `content` JSON is unaffected by new type additions).

| Type slug | Description | Key `content` fields |
|---|---|---|
| `paragraph` | Titled section with rich text body and optional footer | `title` (opt), `body` (rich text), `footer` (opt) |
| `image_card` | Image with header, body text, and footer; image inline or as background | `header` (opt), `body` (opt), `footer` (opt), `image_path`, `image_as_background` (bool) |
| `hero` | Full-width banner with background image, headline, sub-text, and CTA button | `headline`, `subtext` (opt), `background_path` (opt), `cta_label` (opt), `cta_url` (opt) |
| `feature_list` | Vertically or horizontally listed items each with icon, title, description | `layout` (vertical\|horizontal), `items[]` (icon, title, description) |
| `cta_banner` | Highlighted call-to-action strip with text and button | `text`, `button_label`, `button_url` |
| `divider` | Visual separator between blocks | `style` (line\|space\|dotted) |
| `embed` | Embed external content via URL (video, map, tweet, etc.) | `url`, `caption` (opt) |
| `3d_container` | Fixed-height or full-page 3D background canvas (WebGL / CSS 3D) | `height_mode` (fixed\|fullpage), `fixed_height_px` (opt), `scene_config` (json) |
| `gallery` | Grid of images with optional captions | `columns` (int), `images[]` (path, caption) |
| `profile_card` | User or server profile snapshot: name, avatar, bio, links | `source` (user\|manual), `user_id` (opt), `name`, `avatar_path`, `bio`, `links[]` |
| `stats` | Row of highlighted numbers or metrics | `stats[]` (label, value, description opt) |
| `timeline` | Vertical timeline of events | `events[]` (date, title, description opt) |

---

## Page Builder Rules

- Server owner and moderators can create, edit, reorder, publish, and delete Pages and blocks.
- Block order is managed via the `order` integer. Reordering updates all affected `order` values in a single transaction.
- Draft Pages (`is_published = false`) are only visible to the Server owner and moderators.
- Deleting a block uses **soft delete** (`deleted_at`) — the record is retained to preserve access log integrity.
- The Server owner can mark individual Pages as `preview_visible = true` to expose them during time-limited visitor previews.

---

## Layouts (Starter Templates)

Server owners can choose a starter layout when creating a Server or a new Page. Layouts are pre-composed sets of blocks that can be customised freely after selection.

| Layout name | Description |
|---|---|
| Portfolio | Hero + profile card + feature list + gallery + CTA banner |
| Showcase | Hero + stats + timeline + gallery |
| Wiki | Paragraph blocks with dividers |
| Landing | Hero + feature list + CTA banner |
| Blank | Empty canvas |

Layouts are templates only — all blocks are fully editable after selection.

---

## Page Quotas

The number of Pages a Server can publish is determined by the Server's subscribed features (specific quota values configured by `super_owner` in platform config — TBD). Draft Pages do not count against the quota.

---

## Future: Discussion Pages

Pages with real-time chat, voice/video communication, and screen sharing are planned but out of scope for v1. When implemented, they will be an additional Page type.

---

## Open Questions
- What is the base Page quota for a new Server (before any feature subscriptions)?
- Is rich text implemented via Trix, ProseMirror, or markdown in v1?
- Can individual blocks be locked so moderators can view but not edit them?
- Should block soft-delete have a hard purge policy (e.g. purge after 90 days)?
