# Payments & Subscriptions

## Overview
Chip has five distinct payment types. There is no platform subscription — users only pay for specific Server-related actions.

| Payment type | Who pays | When | Recurrence |
|---|---|---|---|
| Investor one-off | Any `free` user | To gain `investor` role and create a Server | One-off |
| Server feature subscription | `investor` or `super_owner` | Monthly, per active Feature on their Server | Monthly |
| Server boost | Any registered user | To support a Server | One-off |
| Server supporter subscription | Any registered user | Sustained supporter access on a specific Server | Monthly, or multi-month |
| Donation | Any registered user | One-off contribution to the Server creator | One-off |

---

## Refund Policy & Legal Compliance

### Platform Refund Window

| Window | Refund |
|---|---|
| Within 7 calendar days of the original payment | **80%** of the original amount, returned to the original payment method |
| After 7 calendar days | **No refund** under any circumstances |

This applies to every payment type: one-off, recurring, and donation. The 20% non-refunded portion covers processing costs and administrative overhead.

**Rules:**
- Refund requests must be submitted by the user within 7 calendar days of the transaction date.
- For subscriptions: a refund within the 7-day window cancels the subscription immediately; access is removed at the same time.
- For boosts: a refund removes the `supporter` role and decrements the Server's credit pool by the refunded credit amount.
- For donations: a refund removes the donor from the Hall of Fame.
- Credits invested from the credit pool before the refund is requested cannot be reversed and are deducted from the credit pool balance accordingly.
- Failed payments (`status = failed`) are never charged — no refund process applies.

### Jurisdiction Compliance

| Jurisdiction | Relevant Law | How We Comply |
|---|---|---|
| UK | Consumer Rights Act 2015; Consumer Contracts Regulations 2013 | Explicit pre-payment consent checkbox waiving the 14-day statutory digital content cancellation right; platform voluntary policy of 80% within 7 days applies after waiver |
| EU | Directive 2011/83/EU (Consumer Rights Directive) | Same waiver mechanism; presented in user's browser language |
| US | FTC Negative Option Rule; state consumer protection laws | Clear disclosure of 7-day 80% / after-7-day no-refund policy at checkout; consent checkbox |
| Australia | Australian Consumer Law (ACL), Competition and Consumer Act 2010 | Clear upfront disclosure at point of sale; consent checkbox |
| Canada / Quebec | Competition Act; provincial consumer protection; Quebec French requirement | Bilingual disclosure (English + French in Quebec); prominent placement; consent checkbox |
| India / Rest of World | Generally permissive for digital services | Clear disclosure at checkout; consent checkbox |

> **Note for UK/EU**: The statutory 14-day cooling-off right grants users a 100% refund. By waiving this right via the pre-payment consent checkbox, users receive immediate access to digital content and the platform's voluntary 80% within 7 days policy applies instead. The waiver is valid under Consumer Contracts Regulations 2013 (UK) and Directive 2011/83/EU Art. 16(m) (EU).

### Pre-Payment Consent Requirement (All Transaction Types)
Every payment requires an explicit consent checkbox before the user can complete the transaction:

1. Checkbox is **unchecked by default** — Pay button disabled until ticked.
2. Consent statement clearly states the payment type, amount, recurrence (for subscriptions), and the refund policy.
3. For UK/EU users: statement includes the statutory right waiver.
4. For subscriptions: consent is re-recorded at every auto-renewal via the `invoice.payment_succeeded` webhook.
5. All consent records are **immutable and append-only** — never updated or deleted.
6. Consent records are retained for the lifetime of the user account and a minimum of 7 years after account deletion (financial compliance).

### `payment_consents` Table
Records explicit user consent before every transaction (one-off and each subscription renewal).

```
payment_consents
  id              ulid
  user_id         FK -> users
  payment_ref     string        (stripe_payment_id or stripe_invoice_id)
  payment_type    enum: investor_upgrade | server_feature_subscription | server_boost | server_supporter_subscription | donation
  amount_pence    integer
  currency        string        default GBP
  consented_at    timestamp
  ip_address      string        (last octet masked, e.g. 192.168.1.xxx)
  user_agent      string nullable
```

### Terms & Conditions
The platform Terms & Conditions (see `blueprint/legal.md`) must:
- State the 80% within 7 days / no refund after 7 days policy in plain language.
- Be linked at every checkout page before payment.
- Include the subscription renewal notification schedule (7 days, 3 days, 1 day before).
- Reference the 7-day grace period for failed subscription payments.
- State that consent records are retained indefinitely.
- Include the right-of-withdrawal waiver for UK/EU subscribers.

---

## Payment Provider
- **Stripe** is the payment provider (Laravel Cashier for subscriptions).
- One-off payments use Stripe Payment Intents.
- Recurring subscriptions use Stripe Subscriptions (via Cashier).

---

## 1. Investor One-Off Payment

### Purpose
Unlocks `investor` platform role, enabling the user to create one Server.

### Amount
Fixed, set by `super_owner` in platform config. Value TBD (runtime-configurable).

### Flow
1. `free` user selects "Become an Investor" from their account.
2. Stripe Payment Intent created server-side.
3. Pre-payment consent displayed: "I confirm I understand the refund policy (80% within 7 days, nothing after)."
4. User checks the consent checkbox → Pay button activates.
5. User completes payment via Stripe Elements.
6. Stripe `payment_intent.succeeded` webhook fires.
7. `payment_consents` row inserted (type: `investor_upgrade`).
8. User platform role updated to `investor`.
9. User is prompted to create and name their Server.

### Data Model
```
payments
  id                  ulid
  user_id             FK -> users
  type                enum: investor_upgrade | server_boost | donation
  stripe_payment_id   string
  amount_pence        integer
  currency            string  default GBP
  status              enum: pending | succeeded | failed | refunded
  refund_amount_pence integer nullable   (80% of amount_pence when refunded)
  refunded_at         timestamp nullable
  server_id           FK -> servers nullable   (for boost payments and donations)
  created_at / updated_at
```

---

## 2. Server Feature Subscription

### Purpose
The Investor pays monthly for each Feature active on their Server. Monthly cost = sum of each active Feature's `monthly_price_pence` (as configured by `super_owner`).

### Credit Pool Offset
If the Server's `credit_balance` has funds, they are applied to the bill automatically before charging the Investor's payment method. Only the remainder is charged.

### Flow
1. Investor subscribes a Feature to their Server from the Server billing dashboard.
2. Stripe Subscription Item created (or updated) via Cashier.
3. Monthly invoice generated by Stripe.
4. Credit pool offset applied before charge.
5. `invoice.payment_succeeded` webhook updates subscription status; `payment_consents` row inserted (type: `server_feature_subscription`).
6. `invoice.payment_failed` webhook: notify Investor; 7-day grace period begins; Server features suspended on day 8.

### Refund
- If the Investor requests a refund within 7 days of the invoice date, 80% of the billed amount is returned and the feature subscription is cancelled immediately.
- After 7 days, no refund.

### Data Model (via Cashier)
Cashier manages `subscriptions` and `subscription_items` tables. The app adds:
```
users
  stripe_id             string nullable   (Stripe Customer ID)
  pm_type               string nullable
  pm_last_four          string nullable
  trial_ends_at         timestamp nullable
```

Server-level subscription state is tracked in `server_features` (see `blueprint/roles-permissions.md`).

---

## 3. Server Boost (One-Off)

### Purpose
Any registered user pays a one-off amount to support a specific Server. Boosts the Server's credit pool and grants `supporter` server role to the payer on that Server.

### Amount Rules
- Minimum: £0 (symbolic boost — grants supporter role, no credit).
- Maximum: set per-feature context by the Server owner, up to the platform-wide max set by `super_owner`.
- The Server owner sets the boost amounts visible to supporters.

### Flow
1. User chooses a boost amount from options the Server owner has configured.
2. Pre-payment consent displayed: "I confirm I understand the refund policy (80% within 7 days, nothing after)."
3. User checks the consent checkbox → Pay button activates.
4. Stripe Payment Intent created.
5. On `payment_intent.succeeded`:
   - `payment_consents` row inserted (type: `server_boost`).
   - `credit_balance` on the Server incremented.
   - `server_credit_transactions` row inserted (type: `boost_in`).
   - User's `server_members` role updated to `supporter` (if not already higher).
   - Access rules evaluated — Server owner's configured grants for supporters are applied.

### Refund
- If requested within 7 days: 80% returned; `supporter` role removed; Server credit pool decremented by the credited amount (if credits have not yet been invested, the full credited amount is removed; if some credits were invested, only the remaining uninvested portion is removed).
- After 7 days: no refund.

---

## 4. Server Supporter Subscription

### Purpose
A registered user takes out a recurring subscription to a specific Server for sustained `supporter` access. Users can hold subscriptions to **multiple Servers simultaneously with no cap** — each Server subscription is independent. The Server owner sets the subscription price for their Server.

### Subscription Term Options
Subscriptions are not limited to one month. Available terms (each with a separate Stripe price):

| Term | Description |
|---|---|
| Monthly | Renews every calendar month |
| 3-month | Renews every 3 months |
| 6-month | Renews every 6 months |
| Annual | Renews every 12 months |

The Server owner chooses which terms to offer. The `super_owner` may configure whether discounting longer terms is allowed.

### Multi-Server Subscriptions
- A user may hold active subscriptions to multiple Servers at the same time.
- Each subscription is billed independently.

### Renewal Notification Schedule
Every renewal triggers three advance notifications. Expiry warnings (non-auto-renewing) follow the same schedule.

| Notification | When | Content |
|---|---|---|
| Renewal reminder | 7 days before renewal date | "Your subscription to **{Server name}** renews on {date} for {amount}." |
| Renewal reminder | 3 days before renewal date | Same message; increased urgency. |
| Renewal reminder | 1 day before renewal date | Final reminder before charge. |
| Expiry warning | 7 days before subscription end (non-auto-renewing) | "Your subscription to **{Server name}** expires on {date}. Renew to keep your supporter status." |
| Expiry warning | 3 days before subscription end | Same message. |
| Expiry warning | 1 day before expiry | Final warning. |
| Payment succeeded | Immediately after successful renewal | Receipt with amount and next renewal date. |
| Payment failed | Immediately on failed renewal | Notify user; 7-day grace period starts. |

**Rationale**: Three pre-renewal notifications (7 days, 3 days, 1 day) satisfy UK/EU good-faith renewal obligations, the US FTC Negative Option Rule, and Australian ACL renewal transparency expectations.

### Pre-Payment Consent & Right of Withdrawal Waiver (UK/EU)

At subscription signup, the user must confirm:
> "I confirm I understand the refund policy: 80% refund if requested within 7 days of payment, no refund after. For EU/UK users: by proceeding I confirm I want immediate access to digital content and waive my 14-day right of withdrawal under Consumer Contracts Regulations 2013 / Directive 2011/83/EU Art. 16(m)."

The user checks this box manually at signup. On every auto-renewal, a new `payment_consents` row is inserted automatically at `invoice.payment_succeeded` — the original consent at signup covers renewals (per UK/EU guidance), and the row is stored as an auditable compliance record.

### Refund
- If requested within 7 days of the billing date for a given billing cycle: 80% of that cycle's charge is returned and the subscription is cancelled; `supporter` role removed immediately.
- After 7 days: no refund for the current or any past billing cycle.

### Data Model (subscriptions)
Cashier manages `subscriptions` and `subscription_items`. The app adds a `server_subscriptions` view:
```
server_subscriptions  (scoped view of user subscriptions per Server)
  user_id                   FK -> users
  server_id                 FK -> servers
  stripe_subscription_id    string
  term                      enum: monthly | three_month | six_month | annual
  current_period_start      timestamp
  current_period_end        timestamp
  status                    enum: active | past_due | cancelled | expired
```
(Implemented as a query over Cashier's `subscriptions` table filtered by server metadata, not a separate migration.)

### Flow
1. User selects a subscription term on the Server.
2. Pre-payment consent shown (see above text).
3. User checks consent checkbox → Subscribe button activates.
4. Stripe Subscription created.
5. `payment_consents` row inserted (type: `server_supporter_subscription`).
6. Renewal reminders scheduled: queued jobs for 7 days, 3 days, and 1 day before the next billing date.
7. On each `invoice.payment_succeeded`: new `payment_consents` row inserted for the renewal cycle.
8. On cancellation: `supporter` server role retained until period end; access removed after.
9. On `invoice.payment_failed`: notify user; 7-day grace period begins; access suspended on day 8.

---

## 5. Donation

### Purpose
Any registered user can make a one-off donation of any amount to the creator (Server owner) of a specific Server. Donations are purely voluntary and carry no platform entitlements.

### Amount
Any amount of the user's choice, within a platform-configured minimum (e.g. £1) and maximum. Donation funds enter the Server's credit pool.

### Hall of Fame
- When completing a donation, the user chooses: **display my name publicly** or **remain anonymous**.
- Public donors appear in the **Hall of Fame** on the Server — a platform component (`hall_of_fame`) that the Server owner places on any Page.
- Anonymous donors are recorded internally but appear as "Anonymous" in any public display.

### Data Model
```
donations
  id                  ulid
  user_id             FK -> users
  server_id           FK -> servers
  stripe_payment_id   string
  amount_pence        integer
  currency            string  default GBP
  is_anonymous        boolean default false
  display_name        string nullable
  message             text nullable
  status              enum: pending | succeeded | failed | refunded
  refund_amount_pence integer nullable
  refunded_at         timestamp nullable
  created_at / updated_at
```

### Flow
1. User visits a Server and selects "Donate".
2. User enters an amount and an optional message.
3. User chooses: "Show my name in Hall of Fame" or "Donate anonymously".
4. Pre-payment consent displayed: "I confirm I understand the refund policy (80% within 7 days, nothing after)."
5. User checks consent checkbox → Donate button activates.
6. Stripe Payment Intent created.
7. On `payment_intent.succeeded`:
   - `payment_consents` row inserted (type: `donation`).
   - `donations` row inserted.
   - Server's `credit_balance` incremented.
   - `server_credit_transactions` row inserted (type: `donation_in`).
   - Hall of Fame data source updated (if `is_anonymous = false`).
8. Confirmation email sent to donor.

### Refund
- If requested within 7 days: 80% returned; donor removed from Hall of Fame; Server credit pool decremented by the credited amount (less any portion already invested).
- After 7 days: no refund.

---

## Credit Pool Investment

Server owner invests `credit_balance` from the Server dashboard:

| Investment type | Effect |
|---|---|
| Subscribe/renew a Feature | Deducts from `credit_balance`; activates `server_features` row |
| Purchase Member Slots | Deducts from `credit_balance`; increments `servers.member_cap` |
| Bill offset | Deducted automatically from next Stripe invoice |

All investments are recorded in `server_credit_transactions`.

Rules:
- Credits are locked to the Server — no cash out.
- Credits cannot be transferred between Servers.
- If a Server is deleted, the credit balance is forfeited.

---

## Webhook Events to Handle

| Stripe event | Action |
|---|---|
| `payment_intent.succeeded` | Investor upgrade OR boost OR donation processed; insert `payment_consents` row |
| `customer.subscription.created` | Server feature subscription or Server supporter subscription active |
| `customer.subscription.updated` | Feature added/removed; sync `server_features` |
| `customer.subscription.deleted` | Subscription cancelled; schedule access revocation; remove `supporter` server role from user on that Server; trigger moderator demotion check (demote to `member` if user was a `moderator`) |
| `invoice.payment_succeeded` | Record renewal; insert `payment_consents` row; schedule renewal reminders (7d, 3d, 1d) |
| `invoice.payment_failed` | Notify user; start 7-day grace period; schedule daily reminders |
| `invoice.upcoming` | Trigger 7-day renewal warning notification |
| `charge.refunded` | Record refund; update `payments.status = refunded`, `payments.refund_amount_pence`; revoke role/access where applicable |

> **Note**: The 3-day and 1-day notifications are dispatched as queued jobs scheduled from the `invoice.payment_succeeded` event — calculated as `renewal_date - 3 days` and `renewal_date - 1 day`.

---

## Development & Testing

See `blueprint/development-setup.md` for the full test checklist, Stripe test cards, and webhook forwarding setup.

### Key scenarios to test
- Investor upgrade: consent ticked → payment → role change → Server creation prompt.
- Server boost: consent ticked → payment → credit pool incremented → supporter role granted.
- Supporter subscription: consent ticked → signup → auto-renewal → consent re-recorded → 7-day, 3-day, 1-day renewal reminders dispatched.
- Multi-Server subscription: user subscribes to two Servers → two independent billing cycles → separate reminders for each.
- Multi-month subscription: user subscribes for 3 months → no renewal until term end → correct expiry warnings.
- Failed renewal (Server feature): notification received; 7-day grace period; features suspended on day 8.
- Refund within 7 days: 80% returned via Stripe; role/access revoked; credit pool adjusted.
- Refund after 7 days: request rejected; no refund issued.
- Credit offset: Server bill reduced by available credits before charge.
- Donation: consent ticked → payment → credit pool incremented → Hall of Fame updated (public or anonymous).

---

## Open Questions
- What is the Investor one-off payment amount? (TBD — runtime-configurable by `super_owner`)
- Can credits be inherited if an Investor transfers Server ownership? (Out of scope for v1 — credits forfeited on Server deletion)
