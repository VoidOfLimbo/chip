# Development Setup

## Overview
Local development runs entirely inside Docker via **Laravel Sail**. The goal is a single-command startup that covers the application, database, queues, mail, payments, and scheduled tasks — everything needed to develop and test the full feature set without external dependencies.

---

## Stack

| Service | Tool |
|---|---|
| PHP / Laravel | Sail (PHP 8.4) |
| Database | PostgreSQL (via Sail) |
| Primary keys | ULID on all tables |
| Queue & Cache | Redis (via Sail) |
| Frontend | Vite dev server (`npm run dev`) |
| Payments | Stripe test mode + Stripe CLI webhook forwarding |
| Email | Mailpit (local SMTP capture, web UI) |
| Database GUI | pgAdmin (web UI, runs inside Sail) |
| Scheduler | Sail `artisan schedule:work` (long-running process) |
| Queue worker | Sail `artisan queue:work` (long-running process) |

---

## Sail Services (`docker-compose.yml`)

| Container | Image | Purpose | Exposed Port |
|---|---|---|---|
| `laravel.test` | PHP 8.4 (Sail) | Laravel application | 80 |
| `pgsql` | PostgreSQL 17 | Primary database | 5432 |
| `pgadmin` | pgAdmin 4 | Database GUI (browser-based) | 5050 |
| `redis` | Redis 7 | Queue driver + cache backend | 6379 |
| `mailpit` | Mailpit | Local SMTP capture + email web UI | 1025 (SMTP), 8025 (UI) |

All services start with `./vendor/bin/sail up -d`. No additional host tooling required except the Stripe CLI for webhook forwarding.

### pgAdmin Access
- URL: `http://localhost:5050`
- Default email: `admin@chip.local`
- Default password: set in `.env` as `PGADMIN_PASSWORD`
- Add a new server in pgAdmin pointing to host `pgsql`, port `5432`, credentials from `.env`.

### Mailpit Access
- URL: `http://localhost:8025`
- Captures all outbound mail from the app (SMTP `localhost:1025` inside the container).
- No configuration needed — Sail's `mailpit` service is pre-wired to Laravel's mail settings.

---

## Primary Keys — ULID

All tables use **ULID** as the primary key type instead of auto-incrementing integers.

- ULIDs are 26-character, lexicographically sortable, URL-safe unique identifiers.
- Use `$table->ulid('id')->primary()` in migrations.
- Use the `HasUlids` trait on all Eloquent models.
- Foreign keys referencing ULID primary keys use `$table->foreignUlid('user_id')`.

```php
// Migration example
$table->ulid('id')->primary();
$table->foreignUlid('server_id')->constrained();

// Model example
use Illuminate\Database\Eloquent\Concerns\HasUlids;

class User extends Authenticatable
{
    use HasUlids;
}
```

---

## First-Time Setup

```bash
# 1. Copy environment file
cp .env.example .env

# 2. Install PHP dependencies (using host PHP or Docker)
docker run --rm -v $(pwd):/app composer install

# 3. Start Sail
./vendor/bin/sail up -d

# 4. Generate app key
./vendor/bin/sail artisan key:generate

# 5. Run migrations + seeders
./vendor/bin/sail artisan migrate --seed

# 6. Install JS dependencies and build
./vendor/bin/sail npm install
./vendor/bin/sail npm run dev   # or npm run build for production mode

# 7. Configure Stripe
# Add to .env:
# STRIPE_KEY=pk_test_...
# STRIPE_SECRET=sk_test_...
# STRIPE_WEBHOOK_SECRET=whsec_...  (from `stripe listen` output)
```

---

## Daily Development

```bash
# Start all services
./vendor/bin/sail up -d

# Start Vite hot-reload
./vendor/bin/sail npm run dev

# Run queue worker (long-running — separate terminal)
./vendor/bin/sail artisan queue:work --tries=3

# Run scheduler (long-running — separate terminal)
./vendor/bin/sail artisan schedule:work

# Forward Stripe webhooks (separate terminal, requires Stripe CLI installed on host)
stripe listen --forward-to http://localhost/stripe/webhook
```

> The Stripe CLI runs on the host machine (not inside Sail) and forwards webhook events to the Sail-exposed HTTP port.

---

## Queue Management

- Driver: **Redis** (via `QUEUE_CONNECTION=redis` in `.env`).
- All queued jobs (emails, webhook processing, subscription state changes) run through the Redis queue.
- `queue:work --tries=3` — retries failed jobs up to 3 times before marking as failed.
- Failed jobs are stored in the `failed_jobs` table; inspect and retry via `artisan queue:failed` and `artisan queue:retry`.
- In development, Horizon is optional but can be added for a visual queue dashboard if needed.

### Jobs expected to run via queue

| Job | Trigger |
|---|---|
| Send verification email | User registration |
| Send password reset email | Forgot password request |
| Send MFA code | MFA challenge |
| Send alert email | System or admin-triggered alert |
| Send newsletter email | Owner-triggered broadcast |
| Process Stripe webhook | Stripe event received |
| Grant / revoke Feature access | Server feature subscription change |
| Process boost payment | Boost payment completed |

---

## Scheduled Tasks

- Run via `artisan schedule:work` in development (polls every minute, no cron needed).
- In production this maps to a single cron entry: `* * * * * php artisan schedule:run`.

### Planned scheduled tasks

| Task | Frequency | Purpose |
|---|---|---|
| Revoke expired Feature access | Daily | Remove feature access past subscription period end |
| Subscription lapse check | Daily | Flag Investors whose subscription is past due |
| Failed job pruning | Weekly | Clear old failed job records |
| Apply credits to monthly bill | Monthly | Auto-offset Server feature subscription with credit balance |

---

## Email Testing (Mailpit)

All email sent by the application is captured by Mailpit during development. No real emails are delivered.

### Emails to verify in development

| Email type | Trigger | Scenario to test |
|---|---|---|
| Email verification | Registration | Link arrives, clicks through, account verified |
| Password reset | Forgot password | Link arrives, new password set, old session invalidated |
| MFA code | MFA challenge | Code arrives and is accepted within expiry window |
| Payment receipt | Boost payment / Server feature subscription | Correct amount, correct context shown |
| Subscription renewal | Investor billing cycle | Renewal confirmed, no downgrade |
| Subscription failed | Payment failure | User notified, grace period started |
| Subscription cancelled / lapsed | Cancellation or non-renewal | Downgrade warning, data retention notice |
| Alert | Admin-triggered | Contents and recipient correct |
| Newsletter | Owner broadcast | Unsubscribe link present, correct recipients |

Access all captured emails at `http://localhost:8025`.

---

## Payment Testing (Stripe Test Mode)

All payments use **Stripe test mode**. No real money is charged.

### Stripe Test Cards

| Card number | Behaviour |
|---|---|
| `4242 4242 4242 4242` | Payment succeeds |
| `4000 0000 0000 9995` | Payment fails (insufficient funds) |
| `4000 0025 0000 3155` | Requires 3D Secure authentication |
| `4000 0000 0000 0069` | Charge declined (expired card) |

Use any future expiry date, any 3-digit CVC, and any postal code.

### Investor One-Off Payment (Server Creation)
1. Register with the Investor option.
2. Complete the one-off payment with test card `4242 4242 4242 4242`.
3. Confirm `investor` platform role assigned and Server creation unlocked.
4. Verify payment receipt email arrives in Mailpit.

### Server Feature Subscription
1. As an Investor, subscribe a Feature to a Server.
2. Complete payment with test card `4242 4242 4242 4242`.
3. Confirm subscription created in Stripe test dashboard.
4. Confirm Feature access granted on the Server.
5. Test 3DS flow with card `4000 0025 0000 3155` — confirm the redirect/modal completes correctly.
6. Test feature removal and confirm access revoked at period end.
7. Test failed renewal with card `4000 0000 0000 9995` — confirm alert email received and grace period logic triggers.

### Boost Payment Test
1. As any registered user, boost a Server.
2. Complete the one-off boost payment with test card `4242 4242 4242 4242`.
3. Confirm `supporter` role granted on that Server.
4. Confirm credit balance updated on the Server.
5. Verify payment receipt email arrives in Mailpit.

### Webhook Testing
The Stripe CLI forwards events to the app.

```bash
stripe listen --forward-to http://localhost/stripe/webhook
```

Events to test manually via `stripe trigger`:

```bash
stripe trigger payment_intent.succeeded
stripe trigger invoice.payment_failed
stripe trigger customer.subscription.deleted
```

Check `storage/logs/laravel.log` for webhook receipt and processing outcome.

---

## Running Tests

```bash
# All tests
./vendor/bin/sail artisan test --compact

# Specific test
./vendor/bin/sail artisan test --compact --filter=PaymentTest
```

Pest feature tests for the payment flow should mock Stripe API responses (using `Http::fake()` or Cashier's test helpers) so tests do not hit the live Stripe test API in CI.

---

## Environment Variables Reference

| Key | Description |
|---|---|
| `APP_URL` | `http://localhost` |
| `DB_CONNECTION` | `pgsql` |
| `DB_HOST` | `pgsql` |
| `DB_PORT` | `5432` |
| `DB_DATABASE` | `chip` |
| `DB_USERNAME` | `sail` |
| `DB_PASSWORD` | `password` |
| `PGADMIN_PASSWORD` | pgAdmin login password (dev only) |
| `REDIS_HOST` | `redis` |
| `QUEUE_CONNECTION` | `redis` |
| `CACHE_STORE` | `redis` |
| `MAIL_MAILER` | `smtp` |
| `MAIL_HOST` | `mailpit` |
| `MAIL_PORT` | `1025` |
| `STRIPE_KEY` | Stripe publishable test key (`pk_test_...`) |
| `STRIPE_SECRET` | Stripe secret test key (`sk_test_...`) |
| `STRIPE_WEBHOOK_SECRET` | Stripe webhook signing secret (`whsec_...`) |
| `STRIPE_SECRET` | Stripe secret test key |
| `STRIPE_WEBHOOK_SECRET` | From `stripe listen` output |
| `CASHIER_CURRENCY` | `gbp` (or adjust) |

---

## Open Questions
- Should Redis be included from day one, or use `database` queue driver initially?
- Will the Stripe CLI be installed locally by each developer, or containerised?
- Do we need a `.env.testing` with separate Stripe mocks for CI?
