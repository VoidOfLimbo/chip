# Portfolio / CV

## Overview
The Portfolio is the owner's public-facing personal profile — the primary content Free users come to see. It functions like a rich, interactive CV or LinkedIn profile: bio, skills, experience, projects, and any content the owner chooses to publish.

Investors also get their own Portfolio section within their Tenant space.

---

## Who Sees What

| Visitor | What They See |
|---|---|
| Public (unauthenticated) | Landing page teaser only |
| Free user | Full owner portfolio (read-only) |
| Supporter | Full owner portfolio + demo navigation |
| Investor | Full owner portfolio + their own tenant portfolio |

---

## Owner Portfolio Sections

### 1. Profile / Bio
- Name, avatar/photo, headline, tagline.
- About / summary (rich text).
- Location, contact links (email, LinkedIn, GitHub, etc.).

### 2. Skills
- List of skills grouped by category (e.g. Languages, Frameworks, Tools).
- Optional proficiency level indicator.

### 3. Experience
- Work history entries: company, role, start/end dates, description.
- Ordered chronologically (most recent first).

### 4. Education
- Qualifications: institution, degree/qualification, dates.

### 5. Projects / Portfolio Pieces
- Showcase items: title, description, technologies used, link (live/repo), cover image.
- Can be marked as featured (shown prominently) or listed.

### 6. Published Content (optional)
- Blog posts or articles the owner writes.
- Each post has a title, published date, slug, body (rich text), and optional tags.
- Owner controls visibility (published / draft).

### 7. Contact / Social Links
- Configurable set of links (GitHub, LinkedIn, Twitter/X, personal site, email, etc.).

---

## Data Model

### `profile` (one row per organisation — owner + each investor tenant)
```
profiles
  id                uuid
  organisation_id   FK → organisations
  name              string
  headline          string         (e.g. "Full Stack Developer")
  tagline           string nullable
  bio               text           (rich text / markdown)
  avatar_path       string nullable
  location          string nullable
  is_published      boolean        (controls public visibility)
  created_at / updated_at
```

### `skills`
```
skills
  id              uuid
  profile_id      FK → profiles
  name            string
  category        string nullable  (e.g. "Languages", "Tools")
  level           enum nullable: beginner | intermediate | advanced | expert
  order           integer
```

### `experiences`
```
experiences
  id              uuid
  profile_id      FK → profiles
  company         string
  role            string
  description     text nullable
  started_at      date
  ended_at        date nullable    (null = current)
  order           integer
```

### `educations`
```
educations
  id              uuid
  profile_id      FK → profiles
  institution     string
  qualification   string
  description     text nullable
  started_at      date
  ended_at        date nullable
  order           integer
```

### `projects`
```
projects
  id              uuid
  profile_id      FK → profiles
  title           string
  description     text
  technologies    json            (array of strings)
  url             string nullable
  repo_url        string nullable
  cover_image_path string nullable
  is_featured     boolean
  is_published    boolean
  order           integer
  created_at / updated_at
```

### `posts` (published content / blog)
```
posts
  id              uuid
  profile_id      FK → profiles
  title           string
  slug            string           (unique per profile)
  body            text             (rich text / markdown)
  excerpt         text nullable
  published_at    timestamp nullable  (null = draft)
  tags            json             (array of strings)
  created_at / updated_at
```

---

## Investor Tenant Portfolio

Each Investor gets their own Portfolio scoped to their Tenant Organisation. The data model is identical (same tables), differentiated by `organisation_id`.

Tenant portfolio is accessible at the Investor's tenant URL (e.g. `/t/{slug}/portfolio` or similar — routing TBD).

Investors manage their portfolio from their tenant dashboard (same UI as the owner's portfolio management, scoped to their tenant).

---

## Owner CMS / Management

The owner manages all portfolio content via an authenticated admin/dashboard area:
- Edit profile, skills, experience, education.
- Create / edit / publish / unpublish projects and posts.
- Preview how the portfolio looks to a Free visitor.

---

## Permissions

| Permission | `owner` | `free` | `supporter` | `investor` |
|---|---|---|---|---|
| `portfolio.read` (owner's) | ✅ | ✅ | ✅ | ✅ |
| `portfolio.manage` (owner's) | ✅ | ❌ | ❌ | ❌ |
| `portfolio.read` (own tenant) | — | — | — | ✅ |
| `portfolio.manage` (own tenant) | — | — | — | ✅ |
| `posts.read` (published) | ✅ | ✅ | ✅ | ✅ |
| `posts.manage` (own profile) | ✅ | ❌ | ❌ | ✅ (own tenant) |

---

## Open Questions
- Should the owner's portfolio be fully public (no login required beyond landing page), or restricted to logged-in Free users?
- Does the blog/posts feature apply to both owner and investors, or only the owner in v1?
- Should project cover images use the same upload/storage system as Smart Productivity Tools, or a separate simpler upload?
- Is rich text (e.g. Trix, Quill, or markdown) needed for bio/descriptions in v1, or plain text first?
- Should the portfolio have a public URL that doesn't require login (e.g. `chip.app/bipin`) that anyone can browse?
