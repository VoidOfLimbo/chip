# Expense Planner

## Overview
The Expense Planner lets users track, categorise, and plan their personal or organisational expenses. A key focus is first-class support for **repeating costs** — bills, rent, subscriptions, and any other recurring outgoing — so users can see what is coming up and plan accordingly.

---

## Access by Tier

| Tier | Access |
|---|---|
| Public | None |
| Free | None (Expense Planner is a special feature, not part of the portfolio view) |
| Supporter | **Interactive demo** — explore using sandboxed sample data; changes are ephemeral (not persisted long-term) |
| Investor | **Full access** within their Tenant Organisation — real, persistent expense data scoped to their tenant |

The Expense Planner is a **special feature**. Supporter users can experience it as a demo; Investor users use it as a genuine productivity tool in their own tenant space.

---

## Core Concepts

### Expense
A single recorded cost. Can be one-off or recurring.

| Field | Type | Description |
|---|---|---|
| `id` | uuid | |
| `user_id` | FK → users | Owner |
| `organisation_id` | FK → organisations | Scoped to org (for shared expenses) |
| `title` | string | e.g. "Monthly Rent" |
| `amount` | integer | In smallest currency unit (pence/cents) |
| `currency` | string | ISO 4217, default from user settings |
| `category_id` | FK → expense_categories | nullable |
| `is_recurring` | boolean | |
| `recurrence_rule` | json | See Recurrence below |
| `next_due_at` | date | Computed next occurrence date |
| `notes` | text | nullable |
| `created_at` / `updated_at` | timestamps | |

### Expense Category
User-defined or system-default categories (e.g. Housing, Utilities, Transport, Food, Subscriptions).

| Field | Type | Description |
|---|---|---|
| `id` | uuid | |
| `user_id` | FK → users (nullable for system defaults) | |
| `name` | string | |
| `colour` | string | Hex colour for UI |
| `icon` | string | nullable, icon identifier |

---

## Recurrence

Recurring expenses use a simple recurrence rule stored as JSON to describe the repeat pattern.

### Recurrence Rule Shape
```json
{
  "frequency": "monthly",   // daily | weekly | fortnightly | monthly | yearly | custom
  "interval": 1,            // every N frequency units (e.g. every 2 months)
  "day_of_month": 1,        // for monthly: which day (1–31, or "last")
  "day_of_week": null,      // for weekly: 0=Sun ... 6=Sat
  "start_date": "2026-04-01",
  "end_date": null           // null = indefinite
}
```

### Occurrence Generation
- A scheduled job (daily) calculates upcoming occurrences for all active recurring expenses.
- Generates `expense_occurrences` records for the next N days (rolling window, e.g. 90 days ahead).
- Completed/paid occurrences are marked as such by the user.

### Expense Occurrences Table
```
expense_occurrences
  id             uuid
  expense_id     FK → expenses
  due_date       date
  amount         integer     (snapshot; can differ if user overrides)
  status         enum: upcoming | paid | skipped
  paid_at        timestamp nullable
  notes          text nullable
```

---

## Features

### Dashboard / Overview
- Summary card: total upcoming this month, total this year.
- List of upcoming occurrences (next 30 days), sorted by due date.
- Quick-add expense button.
- Filter by category, date range, status.

### Expense List
- Paginated list of all expenses (one-off and recurring).
- Inline status for recurring: next due date, frequency.
- Actions: edit, delete, mark occurrence as paid/skipped.

### Recurring Expense Management
- Create with recurrence rule wizard (frequency picker).
- Edit recurrence: changes apply from next occurrence forward (or from start — user's choice).
- Pause / resume a recurring expense.
- End a recurrence (set end date).

### Calendar View
- Monthly/weekly calendar showing when expenses are due.
- Colour-coded by category.

### Reports (Supporter / Investor)
- Monthly spending breakdown by category (bar/pie chart).
- Year-over-year comparison.
- Export to CSV.

---

## Permissions

| Permission | `owner` | `free` | `supporter` | `investor` |
|---|---|---|---|---|
| `expenses.read` | ✅ | ❌ | ✅ (demo/sandboxed) | ✅ (tenant) |
| `expenses.create` | ✅ | ❌ | ✅ (demo/sandboxed) | ✅ (tenant) |
| `expenses.update` | ✅ | ❌ | ✅ (demo/sandboxed) | ✅ (tenant) |
| `expenses.delete` | ✅ | ❌ | ✅ (demo/sandboxed) | ✅ (tenant) |
| `expenses.org_shared` | ✅ | ❌ | ❌ | ✅ (Ability) |
| `expenses.export` | ✅ | ❌ | ❌ | ✅ (tenant) |

`expenses.org_shared` — allows creating and viewing expenses shared across an Organisation. Gated behind an Investor Ability/add-on.

---

## Data Considerations
- Amounts stored as integers (smallest currency unit) to avoid floating-point errors.
- Currency stored per-expense; user can have a default currency in their settings.
- Soft deletes on `expenses` so history is preserved.

---

## Open Questions
- Should Free users see real (anonymised) community expense data, or a purely static demo?
- Multi-currency: should the overview convert to a base currency, or show per-currency totals?
- Should team members within an Investor org be able to contribute to shared expenses, or only the owner?
- Is CSV export sufficient for MVP, or is PDF/Excel needed?
- Notification / reminder for upcoming due expenses — email, in-app, or both?
