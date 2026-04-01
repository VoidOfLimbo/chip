# Content Visibility & Access Logging

## Overview
All content in Chip — Pages, Page Components, and future content types — carries a `visibility` setting that controls who can see it. Link-based sharing requires OTP verification to identify the viewer. All access events are logged for security and accountability.

---

## Visibility Levels

All content carries a `visibility` level. Levels form a **fully ordered stack** — each level N is a strict superset of all levels below it (N supersedes N-1 through 1).

| Level | Slug | Who can access | Crawler indexed? |
|---|---|---|---|
| 1 | `public` | Anyone, no login required | Yes |
| 2 | `users` | Any registered user | No |
| 3 | `pros` | Any registered user who is an active supporter of **at least one** Server platform-wide | No |
| 4 | `followers` | Users who follow the content owner | No |
| 5 | `friends` | Mutual followers of the content owner (supersedes `followers` — mutual follow implies one-way follow) | No |
| 6 | `members` | Active members of **this** Server | No |
| 7 | `supporter` | Active supporters of **this** Server (server role ≥ `supporter`: includes `supporter`, `moderator`, `server_owner`) | No |
| 8 | `moderator` | Moderators of **this** Server (server role = `moderator` or `server_owner`) | No |
| 9 | `owner` | Content owner only | No |

### Access Evaluation Rule

To determine whether a viewer can see a piece of content, resolve their **highest qualifying level** using the following ordered checks (first match wins):

1. Is the viewer the content owner? → level **9** (`owner`)
2. Is the viewer a `moderator` of this Server? → level **8**
3. Is the viewer a `supporter` of this Server? → level **7**
4. Is the viewer an active `member` of this Server? → level **6**
5. Is the viewer a mutual follower of the content owner? → level **5** (`friends`)
6. Does the viewer follow the content owner? → level **4** (`followers`)
7. Is the viewer a `pros` user (supporter of any server, platform-wide)? → level **3**
8. Is the viewer any registered user? → level **2** (`users`)
9. Unauthenticated visitor → level **1** (`public`)

If the viewer’s highest qualifying level ≥ the content’s visibility level, access is **granted**. Otherwise, access is **denied**.

**Breaking point at `members` (6):** Levels 1–5 are determined by platform/social standing and may span any server or user relationship. Levels 6–9 are determined by this Server’s membership hierarchy. Because N supersedes N-1, a server `member` automatically qualifies for follower-level and friends-level content on that Server even if they do not actually follow the content owner — server membership is the higher privilege.

**Moderator → supporter dependency:** A user can only be a `moderator` while they maintain `supporter` status. If supporter status is lost (e.g. boost refunded), the user is automatically demoted to `member` — not `supporter`. Moderator level access and `moderator`-visibility content are revoked as part of that demotion.

### Out-of-Band Mechanisms

Two mechanisms sit **outside** the ordered stack. They do not appear in the ordered level table and cannot be compared with a level number:

| Mechanism | Slug | How it works |
|---|---|---|
| Share link | `link` | Generates a share token + OTP. Grants temporary access to a specific piece of content regardless of the viewer’s stack level. See [Link Sharing & Share Tokens](#link-sharing--share-tokens). |
| Private grant | `private` | Owner explicitly grants access to specific individual users via `content_private_grants`. Permanent until revoked. Not role-based — any registered user can be granted private access. See [Private Grants](#private-grants). |

### Group Restrictions

Group restrictions are a **sub-scope of `members`** (6): a page at `members` visibility can be further restricted to specific Group(s). Only active members who belong to the specified Group(s) can see it. Group-scoped content is still level 6 for all other evaluation purposes.

---

## Private Grants

Content set to `private` visibility is only accessible to users the content owner has explicitly granted access to. This is not an ordered level — it is an individual grant that bypasses the stack entirely.

- Any registered user can be granted private access, regardless of their server role or follow status.
- The owner manages grants via a UI panel; grants are revocable at any time.
- A `private`-visibility page is otherwise invisible to everyone, including server moderators and supporters, unless they are explicitly in the grant list or are the server_owner.

```
content_private_grants
  id              ulid
  grantable_type  string           (e.g. "page", "page_component")
  grantable_id    ulid
  granted_to      FK → users
  granted_by      FK → users
  created_at
```

Unique constraint on `(grantable_type, grantable_id, granted_to)`.

---

## Link Sharing & Share Tokens

Sharing content via the `link` mechanism generates a **share token**. All share tokens require OTP verification — there is no anonymous link access.

### OTP Verification Flow

1. Viewer opens the share link.
2. App shows an identity verification prompt.
3. Viewer enters their email address or phone number.
4. A short-lived numeric OTP is sent to the provided contact.
5. Viewer enters the OTP → identity is recorded → an `access_log` entry is written → content renders.
6. The OTP contact detail (email/phone) is **hashed** before storage — plain-text contact is never persisted in logs.

Each OTP attempt (success or failure) is individually logged. Expired or exhausted share tokens return a generic "link invalid or expired" message — no detail about why is given to the viewer.

### Share Token Data Model

```
share_tokens
  id              ulid
  shareable_type  string            (e.g. "page", "page_component")
  shareable_id    ulid
  type            enum: link | otp
  token           string unique
  created_by      FK → users
  expires_at      timestamp nullable
  max_uses        integer nullable   (null = unlimited)
  use_count       integer default 0
  created_at
```

---

## Access Logging

All access events are recorded regardless of outcome — allowed, denied, expired, or failed. Logs are **append-only**.

### `access_logs`

```
access_logs
  id                ulid
  user_id           FK → users nullable     (null for unauthenticated or pre-OTP attempts)
  contact_hint      string nullable         (hashed email/phone from OTP flow — never plain text)
  event_type        enum (see table below)
  target_type       string nullable         (e.g. "page", "page_component", "server")
  target_id         ulid nullable
  share_token_id    FK → share_tokens nullable
  ip_address        string                  (masked — see Privacy section)
  user_agent        string nullable
  metadata          json nullable           (additional event context)
  created_at
```

### Event Types

| Event | Trigger |
|---|---|
| `page_view` | User successfully views a Page |
| `block_view` | User views a Block with narrower visibility than its Page |
| `otp_requested` | Viewer submits email or phone to receive OTP |
| `otp_verified` | Viewer enters correct OTP |
| `otp_failed` | Viewer enters incorrect OTP |
| `token_expired` | Share token accessed after expiry |
| `token_exhausted` | Share token accessed after `max_uses` reached |
| `access_denied` | Content access refused (insufficient visibility level) |
| `join_requested` | User submits a join request to a Server |
| `join_approved` | Join request approved by Server owner |
| `invite_accepted` | User accepts an invite |
| `preview_started` | User begins a time-limited Server preview |
| `preview_expired` | User's preview window expires |
| `preview_extension_requested` | User requests a preview extension |
| `preview_extension_approved` | Server owner or moderator approves an extension request |
| `preview_extension_denied` | Server owner or moderator denies an extension request (blocks future self-service requests) |
| `demo_access_requested` | User requests access to a demo feature |
| `demo_access_approved` | Server owner approves a demo access request |
| `demo_access_denied` | Server owner denies a demo access request (blocks future self-service requests) |
| `server_boosted` | User makes a boost payment to a Server |
| `member_suspended` | Owner or moderator suspends a member |
| `member_suspension_lifted` | Suspension ended (manually lifted or `suspended_until` expired) |
| `member_kicked` | Owner or moderator removes a member (row deleted; can rejoin) |
| `member_banned` | Owner permanently bans a member |
| `member_ban_lifted` | Owner lifts a ban |
| `member_left` | Member voluntarily left the Server (row deleted) |
| `moderator_demoted` | Moderator automatically demoted to `member` due to loss of `supporter` status |
| `feature_access_granted` | Server owner grants a user or group access to a Feature |
| `feature_access_revoked` | Server owner revokes feature access |
| `user_followed` | User A follows user B |
| `user_unfollowed` | User A unfollows user B |
| `mutual_follow_formed` | User A and user B are now mutual followers |
| `mutual_follow_broken` | One side of a mutual follow is removed |

---

## Privacy Considerations

- **OTP contact**: email/phone is hashed (bcrypt or SHA-256) before being stored in `contact_hint`. Plain-text is never persisted.
- **IP masking**: IPv4 — last octet zeroed (e.g. `192.168.1.x`). IPv6 — last 80 bits zeroed. Stored masked only.
- **GDPR / data erasure**: A scheduled job can anonymise all `access_logs` rows for a given `user_id` (null out `user_id`, clear `contact_hint`, clear `ip_address`) on a verified erasure request.
- Logs are never deleted outright — only anonymised on erasure requests.

---

## Robots.txt Strategy

```
User-agent: *
Disallow: /dashboard
Disallow: /server/
Disallow: /auth/
Allow: /           (landing page)
Allow: /s/{slug}/public/   (public pages of a Server)
```

Exact paths TBD based on final routing structure (see `blueprint/access-tiers.md`).

---

## Open Questions
- Should OTP attempts have a rate limit and lockout policy (e.g. 5 failed attempts → 15 min lockout)?
- What is the OTP expiry duration (e.g. 10 minutes)?
- Should access logs have a retention policy (e.g. archived after 12 months, raw logs purged after 24 months)?
- Should `access_logs` be in a separate database schema or table partition for performance at scale?
