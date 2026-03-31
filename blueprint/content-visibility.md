# Content Visibility & Access Logging

## Overview
All content in Chip — Pages, Blocks, and future content types — carries a `visibility` setting that controls who can see it. Link-based sharing requires OTP verification to identify the viewer. All access events are logged for security and accountability.

---

## Visibility Levels

| Level | Who can access | Crawler indexed? |
|---|---|---|
| `public` | Anyone, no login required | Yes — allowed via `robots.txt` |
| `link` | Anyone with the link — **after OTP verification** | No |
| `authenticated` | Any logged-in user | No |
| `server` | Any active member of this Server | No |
| `group` | Members of a specific Group within the Server | No |
| `private` | Server owner and moderators only | No |

All routes except `public` content are excluded from crawler indexing via `robots.txt`. A Block's visibility cannot be less restrictive than its parent Page's visibility (see `blueprint/pages-builder.md`).

---

## Link Sharing & Share Tokens

Sharing content at the `link` visibility level generates a **share token**. All share tokens require OTP verification — there is no anonymous link access.

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
  shareable_type  string            (e.g. "page", "page_block")
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
  target_type       string nullable         (e.g. "page", "page_block", "server")
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
| `access_denied` | Content access refused (insufficient role or visibility) |
| `join_requested` | User submits a public join request to a Server |
| `join_approved` | Join request approved by Server owner or moderator |
| `invite_accepted` | User accepts an email or link invite |
| `preview_started` | User begins a time-limited Server preview |
| `preview_expired` | User's preview window expires |
| `preview_extended` | Server owner or moderator grants a preview extension |
| `server_boosted` | User makes a boost payment to a Server |
| `feature_access_granted` | Server owner grants a user or group access to a Feature |
| `feature_access_revoked` | Server owner revokes feature access |

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
