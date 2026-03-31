# Payments & Subscriptions

## Overview
Chip has two distinct payment models depending on account tier.

| Tier | Model | Recurrence |
|---|---|---|
| Supporter | One-off payment | None |
| Investor | Subscription | Monthly |

---

## Payment Provider
- **Stripe** is the assumed provider (industry standard, good Laravel/Cashier support).
- Use **Laravel Cashier (Stripe)** for subscription management.
- One-off Supporter payments are processed as a Stripe Payment Intent (not a subscription).

---

## Supporter — One-Off Payment

### Flow
1. User browses landing page and selects "Supporter" plan.
2. User fills in registration details and is directed to payment.
3. Stripe Payment Intent is created server-side; Stripe Elements renders the payment form.
4. On success, user account is created with `supporter` role and prompted to name their Organisation.
5. Stripe `payment_intent.succeeded` webhook confirms and activates the account.

### Data Model
```
payments
  id                ulid
  user_id           FK → users (nullable until account created)
  stripe_payment_id string
  amount            integer  (pence/cents)
  currency          string
  status            enum: pending | succeeded | failed | refunded
  created_at / updated_at
```

### Notes
- No recurring billing — once paid, access is permanent unless revoked by admin.
- Refund policy TBD.

---

## Investor — Monthly Subscription

### Pricing Model
The monthly cost is the **sum of chosen feature add-on prices**. There is a base subscription price, and each add-on adds to it.

```
Base monthly price: TBD (e.g. £X/month)
+ Feature Add-on A: £Y/month
+ Feature Add-on B: £Z/month
= Total monthly charge
```

### Feature Add-ons (Subscription Items)
Stripe Subscriptions with multiple items (one per active add-on) map cleanly to this model.

Each add-on:
- Has a Stripe `Price` ID.
- Maps to one or more **Abilities** (see `blueprint/roles-permissions.md`).
- Can be added/removed by the Investor from their subscription settings.

```
subscription_addons  (build-time seeded)
  id           ulid
  slug         string   (e.g. "premium_analytics")
  label        string
  description  text
  stripe_price_id  string
  price_pence  integer  (display only — Stripe is authoritative)
  abilities    json[]   (ability slugs this add-on unlocks)
```

### Subscription Data Model (via Cashier)
Cashier manages `subscriptions` and `subscription_items` tables automatically. The app stores:

```
users
  stripe_id             string nullable   (Stripe Customer ID — added by Cashier)
  pm_type               string nullable
  pm_last_four          string nullable
  trial_ends_at         timestamp nullable
```

### Investor Subscription Flow
1. User selects "Investor" on landing page.
2. User chooses desired feature add-ons; running total shown dynamically.
3. User fills registration details and is sent to Stripe Checkout (or embedded Stripe Elements).
4. On success, account is created with `investor` role, user names their Organisation.
5. Cashier webhook (`customer.subscription.created`) activates the subscription and grants Abilities.

### Subscription Management (Post Sign-up)
- Investor can view/change add-ons from their billing dashboard.
- Add-on additions take effect immediately (prorated).
- Add-on removals take effect at next billing cycle; Abilities are revoked then.
- Cancel subscription → account downgrades (Free or Supporter TBD) at period end.

### Webhook Events to Handle
| Event | Action |
|---|---|
| `payment_intent.succeeded` | Activate Supporter account |
| `customer.subscription.created` | Activate Investor account, grant Abilities |
| `customer.subscription.updated` | Sync add-ons, grant/revoke Abilities |
| `customer.subscription.deleted` | Downgrade account, revoke Abilities |
| `invoice.payment_failed` | Notify user, grace period TBD |

---

### Development & Testing

### Stripe Test Mode
- Use Stripe test mode keys in `.env` during development (`STRIPE_KEY`, `STRIPE_SECRET`).
- Use Stripe CLI to forward webhooks locally: `stripe listen --forward-to localhost/stripe/webhook`.
- In Sail, expose the webhook endpoint via Sail's configured port.
- Test cards: `4242 4242 4242 4242` (success), `4000 0000 0000 9995` (decline), `4000 0025 0000 3155` (3DS required), `4000 0000 0000 0069` (expired).
- See `blueprint/development-setup.md` for the full payment testing checklist.

### Cashier Setup Checklist
- [ ] `composer require laravel/cashier`
- [ ] Run Cashier migrations
- [ ] Add `Billable` trait to `User` model
- [ ] Set `STRIPE_KEY`, `STRIPE_SECRET`, `STRIPE_WEBHOOK_SECRET` in `.env`
- [ ] Register Cashier webhook route in `routes/web.php`
- [ ] Seed `subscription_addons` with test Stripe Price IDs

---

## Open Questions
- What is the base Supporter one-off price?
- What is the Investor base monthly price?
- What are the initial feature add-ons and their prices?
- What happens when an Investor cancels — downgrade to Free or Supporter?
- Is there a trial period for Investors?
- Should Supporters get a future upgrade credit toward Investor subscription?
