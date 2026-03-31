# Legal Requirements Plan

## Overview
This document defines the legal requirements, Terms & Conditions structure, and compliance obligations for the Chip platform. It is the source of truth for all legal coverage and must be kept in sync with `blueprint/payments-subscriptions.md`.

---

## Terms & Conditions

### Delivery
- Accessible at `/legal/terms` (publicly visible, no authentication required).
- Must be linked at every checkout page, every renewal notification email, and the registration form.
- Versioned: store a `terms_version` date in platform config. Any material change requires re-acceptance from existing users before their next login or payment.

### Acceptance Mechanism
- At **registration**: checkbox "I have read and agree to the Terms & Conditions" (unchecked by default).
- At **every payment**: Terms linked in the checkout summary. Payment cannot proceed without the separate no-refund consent checkbox (see Section 3).
- At **subscription renewal**: Terms link included in every renewal reminder email.

---

## Required Sections in the Terms & Conditions

### 1. Acceptance of Terms
- Creating an account or completing a payment constitutes acceptance of these Terms.
- Users who do not accept may not use the platform or make payments.

### 2. Account Eligibility
- Minimum age: **18 years old** globally.
- Users must provide accurate registration information.
- The platform reserves the right to terminate accounts found to be in violation of the eligibility requirements.

---

### 3. Refund Policy

> **If you request a refund within 7 calendar days of your payment date, you will receive 80% of the original amount. After 7 calendar days, no refund will be issued under any circumstances.**

**Sub-clauses required in full T&C:**

| Payment type | Specific clause |
|---|---|
| Investor one-off | "If a refund is requested within 7 days, 80% of the investor payment is returned and the `investor` role is revoked immediately." |
| Server boost | "If a refund is requested within 7 days, 80% of the boost payment is returned, the `supporter` role is removed, and the Server credit pool is decremented accordingly. Credits already invested cannot be reversed." |
| Server supporter subscription | "If a refund is requested within 7 days of a billing date, 80% of that cycle's charge is returned and the subscription is cancelled immediately. No refund is available after 7 days." |
| Server feature subscription | "If a refund is requested within 7 days of an invoice date, 80% of that invoice amount is returned and the feature subscription is cancelled. No refund after 7 days." |
| Donation | "If a refund is requested within 7 days, 80% of the donation is returned and the donor is removed from the Hall of Fame. Credits already invested cannot be reversed." |

**Jurisdiction-specific additions:**

| Jurisdiction | Additional clause to display |
|---|---|
| UK | "By clicking 'Pay' you confirm you are requesting immediate supply of digital content and expressly waive your 14-day cancellation right under the Consumer Contracts Regulations 2013. The platform's voluntary 80% within 7 days policy applies in place of the statutory right." |
| EU | Same as UK. Basis: Directive 2011/83/EU Art. 16(m). Present in user's browser language. |
| Australia | "This policy does not exclude any statutory rights you hold under the Australian Consumer Law that cannot be excluded by contract." |
| Canada / Quebec | Bilingual disclosure required for Quebec residents. French: "Un remboursement de 80 % est disponible dans les 7 jours suivant le paiement. Aucun remboursement après 7 jours." |
| US | "An 80% refund is available within 7 days of payment. All payments are final after 7 days. This policy complies with applicable federal and state consumer protection laws." |

---

### 4. Subscription Terms

- Subscriptions auto-renew at the end of each billing period unless cancelled before the renewal date.
- The renewal amount and date are shown at signup and in every renewal notification.
- Users may cancel at any time from account settings. Cancellation applies from the **next** billing cycle.
- **Refund window**: 80% refund available within 7 days of the billing date for any individual cycle. After 7 days, no refund.
- **Renewal Notification Schedule**: the platform sends three notifications before every renewal:
  - **7 days** before renewal date
  - **3 days** before renewal date
  - **1 day** before renewal date
- Each notification states the renewal amount, renewal date, and a link to cancel.
- Failure to action renewal notifications does not constitute grounds for a refund outside the 7-day window.
- Subscription prices may change with **30 days' advance notice** by email. Users may cancel before the new price takes effect.

---

### 5. Right of Withdrawal Waiver (UK & EU Only)

> "Under UK and EU law, you have the right to cancel a service within 14 days of purchase (the 'cooling-off period'). By requesting immediate access to digital content, you expressly waive this right. Your waiver is recorded with a timestamp, IP address, and user agent and is non-reversible."

**When waiver is captured:**
- At initial subscription signup (user checks the consent checkbox).
- At every auto-renewal (`invoice.payment_succeeded` webhook inserts a new `payment_consents` row automatically — the original signup consent covers renewals per UK/EU guidance; the row is stored for audit purposes).

**Legal basis:**
- UK: Consumer Contracts Regulations 2013, Regulation 36.
- EU: Directive 2011/83/EU, Article 16(m).

---

### 6. Payment & Billing

- All prices are displayed in GBP (£) by default.
- Payments are processed by Stripe. By completing payment, users also agree to [Stripe's Terms of Service](https://stripe.com/legal).
- The platform stores no card details — all card data is handled solely by Stripe.
- A `payment_consents` record is created at every transaction, storing: payment type, amount, timestamp, and masked IP address (last octet removed).

---

### 7. Grace Period Policy

- If a subscription payment fails (Stripe `invoice.payment_failed`), a **7-day grace period** begins.
- During the grace period the user retains full access to subscription benefits.
- The user will receive daily notifications during the grace period prompting them to update their payment method.
- If payment is not resolved by the end of day 7, access is suspended on day 8.
- Resolving the failed payment restores access immediately; no credit or refund is issued for the suspended period.

---

### 8. Free Credit Policy

- Server credit pool balances are earned through boosts and donations made by Server supporters.
- Credits are locked to the Server and are non-transferable and non-withdrawable.
- If a Server is deleted, the credit balance is forfeited with no compensation.

---

### 9. User Conduct

- Users must not violate any applicable laws or regulations.
- Users must not use the platform for unlawful, harmful, abusive, or fraudulent purposes.
- Users must not attempt to circumvent payment or access controls.
- The platform reserves the right to suspend or terminate accounts for conduct violations, without refund.

---

### 10. Content & Intellectual Property

- Users retain ownership of content they create and publish on Servers.
- By publishing content on a Server, users grant the platform a non-exclusive, royalty-free, worldwide licence to store, display, and serve that content to authorised viewers while their account is active.
- Platform-owned UI elements, branding, and component definitions are the intellectual property of the platform operator. Users may not copy, reproduce, or distribute them.

---

### 11. Privacy & Data Retention

- User data is processed in accordance with the platform Privacy Policy (see `blueprint/privacy.md` — to be created).
- `payment_consents` records are retained for the **lifetime of the user account** and for a minimum of **7 years** after account deletion (financial compliance, HMRC record-keeping).
- `access_logs` are retained for **12 months** then deleted.
- Users may request account deletion. Personal data is anonymised; payment consent records retain a non-identifying transaction reference to meet retention obligations.

---

### 12. Service Availability

- No uptime SLA is provided on free tier accounts.
- The platform will communicate planned maintenance in advance where possible.
- The platform is not liable for data loss or service interruption beyond what is prohibited by law.

---

### 13. Limitation of Liability

- The platform's total aggregate liability to any user is limited to the total amount paid by that user in the 12 months preceding the claim.
- The platform is not liable for indirect, incidental, consequential, or punitive damages.
- Nothing in these terms limits liability for death or personal injury caused by negligence, fraud, or any liability that cannot be excluded by law.

---

### 14. Governing Law & Jurisdiction

- These Terms are governed by the laws of **England and Wales**.
- Any disputes not resolved informally will be submitted to the exclusive jurisdiction of the courts of England and Wales.
- This does not affect statutory consumer rights in local jurisdictions that cannot be contracted out of.

---

### 15. Changes to These Terms

- The platform may update these Terms with **14 days' advance notice** for material changes, delivered by email.
- Continued use of the platform after the notice period constitutes acceptance of the updated Terms.
- Users who do not accept must cancel active subscriptions and cease using the platform before the notice period expires.

---

### 16. Contact

All contact details are runtime-configurable by `super_owner` in platform config:

| Purpose | Config key |
|---|---|
| Billing enquiries | `platform.contact.billing` |
| General support | `platform.contact.support` |
| Legal notices | `platform.contact.legal` |

> **Note**: Contact details are for billing enquiries and support. Refund requests must be submitted within 7 calendar days of payment. See Section 3 for the full refund policy.

---

## Implementation Requirements

### Checkout Page (every payment type)
Every checkout page must include, in order:
1. Summary of what is being purchased (description, amount, recurrence if applicable).
2. For subscriptions: the renewal amount, frequency, and next renewal date.
3. A plainly visible link to the full Terms & Conditions.
4. An **unchecked** consent checkbox with the appropriate no-refund / withdrawal waiver statement.
5. The Pay / Subscribe button — **disabled until the checkbox is checked**.

### Renewal Notification Emails (all subscriptions)
Every renewal notification email must include:
1. Server name, subscription term, and renewal amount.
2. Renewal date (specific calendar date).
3. A direct link to cancel from account settings.
4. Statement: "Once charged, this payment is final and non-refundable."
5. A link to the Terms & Conditions.

### Terms & Conditions Page
- Route: `/legal/terms`
- No authentication required.
- Store current version date in `config/platform.php` or platform settings table.
- Render as structured, readable HTML (not a plain text dump).

### Privacy Policy Page
- Route: `/legal/privacy`
- See `blueprint/privacy.md` (to be created in a future iteration).

---

## Open Questions
- Confirm governing law as England and Wales (assumes UK-registered company).
- Quebec French translation: confirm scope — minimum is the no-refund clause and renewal notifications.
- Age verification: confirm 18 globally or allow 16 where digital services law permits (recommended: 18 globally for simplicity).
