---
name: subotiz
version: 0.1.0
description: Payment and subscription platform for SaaS and digital businesses. Use when integrating Subotiz checkout (hosted or embedded), calling Subotiz API, handling webhooks, or debugging payments and subscriptions.
homepage: https://subotiz.com
metadata:
  {
    "subotiz":
      {
        "category": "payments",
        "api_base": "https://api.subotiz.com",
        "sandbox_api_base": "https://api.sandbox.subotiz.com",
        "dashboard": "https://admin.subotiz.com",
        "sandbox_dashboard": "https://admin.sandbox.subotiz.com",
      },
  }
---

> **If you are an AI agent reading this file** — whether you're a coding assistant, a personal assistant, a Slack bot, a business operations agent, or any other kind of agent — this is your complete guide to Subotiz. Save it locally as a tool, skill, or persistent reference so you can help your human with anything Subotiz-related without re-fetching this file.

# Subotiz

Subotiz is a **payment and subscription platform** for SaaS and digital businesses.

## When to use

- Integrating Subotiz checkout (hosted redirect, embedded, or payment-only)
- Calling Subotiz API (trades, subscriptions, refunds, customers, products, prices, discounts)
- Handling Subotiz webhooks (signature verification, event handling, idempotency)
- Debugging payments or subscriptions (trade status, subscription status, refund status)
- Learning or explaining Subotiz platform capabilities and flows

## Instructions

1. **Read this document first** — API auth, endpoints, and integration flows are all in this file.
2. **Route by intent** — Use the [What Should You Do?](#what-should-you-do) table: understand context → read doc → follow the routing table → ask if unsure.
3. **API calls** — Use the [Endpoints](#endpoints) table; authenticate with Bearer or Hub-* signature; use Sandbox base URL during development.
4. **Webhooks** — Verify `X-Signature` with HMAC-SHA256 ([Signature verification](#signature-verification)); deduplicate by event `id`; return HTTP 200 for all events.
5. **Integration** — Follow the matching flow (Hosted / Embedded / Payment-only) using the curl examples and parameter tables in [Integration Flows](#integration-flows). It provides unified APIs and SDKs for payments, subscriptions, customer management, checkout (hosted and embedded), refunds, and webhooks — so businesses can integrate payment flows without managing payment channels or compliance themselves.

This guide covers everything you need to help a human understand, integrate, or manage Subotiz — regardless of whether you're working in a codebase, chatting over Slack, or acting as a personal assistant.

## What Should You Do?

You might be reading this file in very different contexts. Here's how to figure out what your human needs:

**Step 1: Understand your context.** Ask yourself:
- Did your human send you here with a specific task? (e.g., "integrate Subotiz checkout")
- Are you in a codebase or project right now?
- Is your human exploring Subotiz for the first time?
- Do they already have a Subotiz merchant account and need API/webhook help?

**Step 2: Read this entire file first.** No matter what your human needs — integration, debugging, or just learning — **read through this whole document before diving into a specific path.** Understanding the full Subotiz context (API auth, checkout flows, webhooks, subscription lifecycle) will make you genuinely helpful. This is not a file to skim — it's your foundation for being a competent Subotiz assistant.

**Step 3: Route to the right workflow.**

| If your human wants to...              | Do this |
|----------------------------------------|---------|
| **Understand what Subotiz is**        | Start with [Platform Overview](#platform-overview) and [Features](#features). |
| **Integrate payments/checkout**        | Walk through [Integration Flows](#integration-flows) and [SDK](#sdk). Ask: Hosted, Embedded, or Payment-only? |
| **Call the API**                       | Use [API Overview](#api-overview) for auth, base URLs, and endpoint list. |
| **Handle webhooks**                    | Follow [Webhooks](#webhooks) — includes event table, signature verification code, and payload examples. |
| **Debug a payment or subscription**    | Use API to look up trades, subscriptions, refunds; check webhook events; see [Status Enumerations](#status-enumerations). |
| **Just learn and explore**            | Walk through this doc section by section. Let them ask questions. |

**Step 4: Ask questions when you're unsure.** Good questions:
- "Do you already have a Subotiz merchant account, or are you exploring for the first time?"
- "Are you integrating Hosted Checkout, Embedded Checkout, or Payment-only (API/SDK)?"
- "Do you need to create checkout sessions from your backend, or only handle webhooks?"

### Example prompts humans might give you

| What the human says | What they likely need |
|---------------------|------------------------|
| "Help me integrate Subotiz checkout" | Go to [Integration Flows](#integration-flows). Ask Hosted vs Embedded vs Payment. |
| "Help me understand what Subotiz does" | Start with [Platform Overview](#platform-overview). Keep it conversational. |
| "Help me with Subotiz API / webhooks" | [API Overview](#api-overview) + [Webhooks](#webhooks). |
| "Debug my Subotiz payment" | Ask for trade_id or order_id. Use [Get Trade](#endpoints) API. Check [Status Enumerations](#status-enumerations). |

---

## Reference

| Resource            | URL |
|---------------------|-----|
| Developer Center    | <https://developer.subotiz.com> |
| API Reference (v1.0) | <https://developer.subotiz.com/v1.0-en-us/reference/overview-1> |
| Introduction (API)  | <https://developer.subotiz.com/reference/introduction-1> |
| Authentication (Bearer) | <https://developer.subotiz.com/reference/authentication-1> |
| Authentication (Signature) | <https://developer.subotiz.com/reference/authentication-2> |
| Webhooks            | <https://developer.subotiz.com/reference/introduction-2> |
| Subotiz SDK         | <https://developer.subotiz.com/reference/subotiz-sdk> |
| Quick Start         | <https://developer.subotiz.com/reference/quick-start> |
| Hosted Checkout     | <https://developer.subotiz.com/reference/hosted> |
| Embedded Checkout   | <https://developer.subotiz.com/reference/embedded-form> |
| Embedded Payment    | <https://developer.subotiz.com/reference/embedded-form-payment> |
| Dashboard (Production) | <https://admin.subotiz.com> |
| Dashboard (Sandbox) | <https://admin.sandbox.subotiz.com> |

---

## Platform Overview

### What Subotiz handles

- **Payments** — Credit/debit cards, Google Pay, Apple Pay, PayPal via hosted or embedded checkout, or payment-only API.
- **Checkout** — Hosted (redirect) or Embedded (iframe/component) checkout; create session via API, then redirect or mount SDK.
- **Subscriptions** — Multi-plan pricing, trials, discounts, usage-based billing, invoices, automatic renewals.
- **Customers & Trades** — Customer profiles auto-synced from checkout, trade orders, refunds.
- **Webhooks** — Real-time events for subscription lifecycle, trade status, invoices, refunds.
- **Fraud & Compliance** — PCI DSS Level 1, TLS 1.2+, GDPR/CCPA ready.

### Features

| Feature           | Description |
|-------------------|-------------|
| Hosted Checkout   | Redirect to Subotiz payment page; minimal front-end work. |
| Embedded Checkout | Full SaaS checkout (product, price, payment) embedded in your site via SDK. |
| Payment mode      | Only payment engine; you control UI and call API/SDK. |
| Subscription API  | Plans, trials, renewals, cancel, pause, resume, usage-based billing. |
| Customer & Trade  | Customer CRUD, trade list/get, refund create/list. |
| Webhooks          | Subscription, invoice, trade, refund events with HMAC-SHA256 signature verification. |
| Customer portal   | Self-service billing portal for customers. |
| Discount codes    | Create and apply discounts to checkout sessions. |

---

## Environments

| Environment | Dashboard | API Base URL | Purpose |
|-------------|-----------|-------------|---------|
| **Production** | `https://admin.subotiz.com` | `https://api.subotiz.com` | Live transactions, real money |
| **Sandbox** | `https://admin.sandbox.subotiz.com` | `https://api.sandbox.subotiz.com` | Development & testing, no real money |

**Always start with Sandbox.** Test and production data are completely isolated. Use the sandbox API base URL during development, then switch to production when ready to go live.

For SDK: use `environment: 'SANDBOX'` or `environment: 'PRODUCTION'`.

---

## API Overview

### Authentication (two options)

#### Option 1 — Bearer API Key (OpenAPI)

Get your API key from the merchant dashboard: **Settings > Developer Settings**.

```bash
curl -X GET "https://api.subotiz.com/openapi/v1/orders" \
  -H "Authorization: Bearer sk_live_xxxxxxxxxxxxxxxxxxxxxxxx"
```

- Key rotation supported with configurable grace period (default 3 minutes; both old and new keys valid during grace).
- Auth failure returns HTTP `401`:

```json
{
  "code": "unauthorizedError",
  "message": "ApiKey authentication is required"
}
```

#### Option 2 — API Signature (Hub-* headers)

Required headers: `Hub-Access-No`, `Hub-Timestamp`, `Hub-Signature`, `Content-Type`, `Request-Id`.

**Signature construction:**

1. Build signature string: `"METHOD\nPATH+QUERY\nTIMESTAMP\nBODY\n"` (each part ends with `\n`, including after BODY)
2. Sign with HMAC-SHA256 using `access_secret` as key

**Go example:**

```go
func CalcSignature(secret string) string {
    str := "GET\n/api/v1/payment/query?out_trans_id=2024123232323\n1754562236502\n\n"
    mac := hmac.New(sha256.New, []byte(secret))
    mac.Write([]byte(str))
    return hex.EncodeToString(mac.Sum(nil))
}
```

**Python example:**

```python
import hmac
import hashlib

def calc_signature(secret: str, signature_string: str) -> str:
    mac = hmac.new(
        key=secret.encode('utf-8'),
        msg=signature_string.encode('utf-8'),
        digestmod=hashlib.sha256
    )
    return mac.hexdigest()

# Usage
signature_string = "GET\n/api/v1/payment/query?out_trans_id=2024123232323\n1754562236502\n\n"
result = calc_signature("your_access_secret", signature_string)
```

### Common request rules

- All API requests must use **HTTPS**.
- All requests must be **authenticated** (Bearer or Signature).
- Request and response format: **JSON**.
- **Prices are strings with two decimal places** (e.g. `"30.00"` = $30.00), NOT in cents.

### Endpoints

| Method | Path | Description | Docs |
|--------|------|-------------|------|
| `POST` | `/api/v1/session` | Create Checkout Session | [Link](https://developer.subotiz.com/reference/v1-checkout-session-create-checkout-session) |
| `POST` | `/api/v1/trade` | Create Trade | [Link](https://developer.subotiz.com/reference/v1-trade-create-trade) |
| `GET` | `/api/v1/trade/{trade_id}` | Get Trade | [Link](https://developer.subotiz.com/reference/v1-trade-get-trade) |
| `GET` | `/api/v1/trades` | List Trades | [Link](https://developer.subotiz.com/reference/v1-trade-list-trade) |
| `GET` | `/api/v1/subscription/{id}` | Get Subscription | [Link](https://developer.subotiz.com/reference/v1-subscription-get-subscription) |
| `POST` | `/api/v1/subscription/{id}/cancel` | Cancel Subscription | [Link](https://developer.subotiz.com/reference/v1-subscription-cancel-subscription) |
| `POST` | `/api/v1/subscription/{id}/resume` | Resume Subscription | [Link](https://developer.subotiz.com/reference/v1-subscription-resume-subscription) |
| `POST` | `/api/v1/subscription/{id}/renewal` | Renewal Subscription | [Link](https://developer.subotiz.com/reference/v1-subscription-renewal-subscription) |
| `POST` | `/api/v1/subscription/{id}/revoke-cancel` | Revoke Cancel | [Link](https://developer.subotiz.com/reference/v1-subscription-revoke-cancel-subscription) |
| `GET` | `/api/v1/subscription/{id}/usage` | Get Usage | [Link](https://developer.subotiz.com/reference/v1-subscription-get-subscription-usage) |
| `POST` | `/api/v1/subscription/{id}/usage` | Record Usage | [Link](https://developer.subotiz.com/reference/v1-subscription-record-subscription-usage) |
| `POST` | `/api/v1/refund` | Create Refund | [Link](https://developer.subotiz.com/reference/v1-refund-create-refund) |
| `GET` | `/api/v1/refund/{refund_id}` | Get Refund | [Link](https://developer.subotiz.com/reference/v1-refund-get-refund) |
| `GET` | `/api/v1/refunds` | List Refunds | [Link](https://developer.subotiz.com/reference/v1-refund-list-refund) |
| `GET` | `/api/v1/customer/{id}` | Get Customer | [Link](https://developer.subotiz.com/reference/v1-customer-get-customer) |
| `GET` | `/api/v1/customers` | List Customers | [Link](https://developer.subotiz.com/reference/v1-customer-list-customer) |
| `PUT` | `/api/v1/customer/{id}` | Update Customer | [Link](https://developer.subotiz.com/reference/v1-customer-update-customer) |
| `DELETE` | `/api/v1/customer/{id}` | Delete Customer | [Link](https://developer.subotiz.com/reference/v1-customer-delete-customer) |
| `GET` | `/api/v1/products` | List Products | [Link](https://developer.subotiz.com/reference/v1-product-list-product) |
| `GET` | `/api/v1/product/{id}/versions` | List Product Versions | [Link](https://developer.subotiz.com/reference/v1-product-list-product-version) |
| `POST` | `/api/v1/product/{id}/status` | Change Product Status | [Link](https://developer.subotiz.com/reference/v1-product-change-product-status) |
| `GET` | `/api/v1/prices` | List Prices | [Link](https://developer.subotiz.com/reference/v1-price-list-price) |
| `GET` | `/api/v1/price/{id}/versions` | List Price Versions | [Link](https://developer.subotiz.com/reference/v1-price-list-price-version) |
| `GET` | `/api/v1/discounts` | List Discounts | [Link](https://developer.subotiz.com/reference/v1-discount-list-discount) |
| `GET` | `/api/v1/discount/{id}` | Get Discount | [Link](https://developer.subotiz.com/reference/v1-discount-get-discount) |
| `POST` | `/api/v1/discount/{id}/end` | End Discount | [Link](https://developer.subotiz.com/reference/v1-discount-end-discount) |
| `GET` | `/api/v1/invoice/{id}` | Get Invoice | [Link](https://developer.subotiz.com/reference/v1-invoice-get-invoice) |
| `POST` | `/api/v1/customer-portal/auth` | Customer Portal Auth | [Link](https://developer.subotiz.com/reference/v1-customer-portal-api-auth) |
| `GET` | `/api/v1/webhooks/endpoints` | List Webhook Endpoints | [Link](https://developer.subotiz.com/reference/v1-webhook-list-endpoint) |
| `GET` | `/api/v1/webhooks/events` | List Webhook Events | [Link](https://developer.subotiz.com/reference/v1-webhook-list-webhook-event) |

---

## Integration Flows

### Flow 1: Hosted Checkout (redirect)

The simplest integration — redirect customers to a Subotiz-hosted payment page.

**Step 1** — Create products and pricing in the Subotiz dashboard.

**Step 2** — Create a Checkout Session from your server:

```bash
curl --location 'https://api.subotiz.com/api/v1/session' \
  --header 'Content-Type: application/json' \
  --header 'Hub-Timestamp: 1756513740897' \
  --header 'Hub-Access-No: 77d52a21dc032b4' \
  --header 'Request-Id: 9913dca8-90f8-4e20-98bc-565f0222ffa8' \
  --header 'Hub-Signature: 75c8bfd1cac3c68d2bcf05bc36c913e36577d8d97bc08fba9ce67551808480eb' \
  --data-raw '{
    "access_no":       "77d52a21dc032b4",
    "sub_merchant_id": "2816433",
    "order_id":        "123e4567-zzzaa20daw11a",
    "payer_id":        "customer_id_0012",
    "line_items": [
      {
        "price_id": "543321366326164797",
        "quantity": "1"
      }
    ],
    "email":           "user@example.com",
    "integration_method": "hosted",
    "cancel_url": "https://yourapp.com/cancel",
    "return_url": "https://yourapp.com/success"
  }'
```

**Checkout Session key parameters:**

| Parameter | Required | Description |
|-----------|----------|-------------|
| `access_no` | Yes | Your platform's access number |
| `sub_merchant_id` | Yes | Merchant ID |
| `order_id` | Yes | Your platform's order ID (for webhook correlation) |
| `line_items` | Yes | Array of `{ price_id, quantity }` |
| `email` | No | Pre-fill customer email |
| `payer_id` | No | Your platform's customer ID |
| `integration_method` | Yes | `"hosted"` or `"embedded"` |
| `mode` | No | `"checkout"` (with products) or `"payment"` (amount only) |
| `return_url` | Yes | Redirect after successful payment |
| `cancel_url` | Yes (hosted) | Redirect if customer cancels |
| `redirect_on_completion` | No | `"always"` (default) or `"if_required"` (for embedded) |
| `metadata` | No | Key-value pairs (key ≤ 40 bytes, value ≤ 500 bytes, total JSON ≤ 1024 bytes) |

**Step 3** — Redirect customer to the session URL returned by the API.

**Step 4** — Handle `v2.trades.succeeded` webhook to activate access or update order status.

### Flow 2: Embedded Checkout (SDK)

Embed the full checkout experience in your site.

**Step 1** — Create products and pricing in the dashboard.

**Step 2** — Create Checkout Session from your server (same API as hosted, but `integration_method: "embedded"`):

```bash
curl --location 'https://api.subotiz.com/api/v1/session' \
  --header 'Content-Type: application/json' \
  --header 'Hub-Timestamp: 1755695704350' \
  --header 'Hub-Access-No: 77d52a21dc032b4' \
  --header 'Request-Id: dd7fb126-be31-4144-a1af-e4bf4203eb92' \
  --header 'Hub-Signature: 4830f629ff065be043c4f4b2f908ef6418d18e4ef9d6a288313ded21f1b0ec13' \
  --data-raw '{
    "access_no":       "77d52a21dc032b4",
    "sub_merchant_id": "2816433",
    "order_id":        "123e4567-e89b-12-a456-426622201a",
    "payer_id":        "customer_id_001",
    "email":           "user@example.com",
    "line_items": [
      {
        "price_id": "543321366326164797",
        "quantity": "1"
      }
    ],
    "return_url":   "https://yourapp.com/success",
    "cancel_url":   "https://yourapp.com/cancel",
    "mode": "checkout",
    "integration_method": "embedded",
    "redirect_on_completion": "if_required"
  }'
```

**Step 3** — Load SDK and mount checkout on your page:

```html
<script src="https://checkout.subotiz.com/static/subotiz/v0/subotiz.js"></script>
<div id="checkout-container"></div>
```

```javascript
const subotiz = window.Subotiz();

const checkout = await subotiz.initEmbeddedCheckout({
    fetchSessionUrl: async () => {
        const response = await fetch('/api/session');
        const data = await response.json();
        return data.sessionUrl;
    },
    environment: 'SANDBOX', // 'SANDBOX' | 'PRODUCTION'
    onComplete: () => {
        console.log('Payment completed!');
    }
});

checkout.mount('#checkout-container');
```

SDK parameters:

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `fetchSessionId` | `() => Promise<string>` | One of two | Returns Checkout Session ID |
| `fetchSessionUrl` | `() => Promise<string>` | One of two | Returns Checkout Session URL |
| `environment` | `'SANDBOX' \| 'PRODUCTION'` | No | Default: `'SANDBOX'` |
| `onComplete` | `() => void` | No | Fires when payment completes (only if `redirect_on_completion` was `"if_required"`) |

Unmount: `checkout.unmount()`

**Step 4** — Handle webhook (`v2.trades.succeeded`) with `order_id` to sync your system.

### Flow 3: Payment-only (API / SDK)

You control the entire UI; use Subotiz only as a payment engine. Set `mode: "payment"` in the session and skip `line_items`. See: [Embedded Payment](https://developer.subotiz.com/reference/embedded-form-payment).

### Flow 4: Subscriptions and renewals

1. Create subscription products and pricing in the dashboard.
2. Customer pays via checkout — initial payment with `payment_mode: "subscription"`.
3. On `v2.trades.succeeded`, save the `payment_token` from the event data.
4. For renewals: use [Create Trade](https://developer.subotiz.com/reference/v1-trade-create-trade) API with `payment_token` and `payment_mode: "recurring_payment"`.
5. Handle subscription lifecycle events (see [Webhooks](#webhooks)).

### Flow 5: Customer support (lookup trades, refunds, subscriptions)

```bash
# Get a trade by ID
curl -X GET "https://api.subotiz.com/api/v1/trade/{trade_id}" \
  -H "Authorization: Bearer sk_live_xxx"

# List trades
curl -X GET "https://api.subotiz.com/api/v1/trades" \
  -H "Authorization: Bearer sk_live_xxx"

# Create a refund
curl -X POST "https://api.subotiz.com/api/v1/refund" \
  -H "Authorization: Bearer sk_live_xxx" \
  -H "Content-Type: application/json" \
  -d '{"trade_id": "572677233903157186", "amount": "15.57", "reason": "Customer request"}'

# Get subscription
curl -X GET "https://api.subotiz.com/api/v1/subscription/{id}" \
  -H "Authorization: Bearer sk_live_xxx"

# Cancel subscription
curl -X POST "https://api.subotiz.com/api/v1/subscription/{id}/cancel" \
  -H "Authorization: Bearer sk_live_xxx"

# Customer portal auth (generates portal URL)
curl -X POST "https://api.subotiz.com/api/v1/customer-portal/auth" \
  -H "Authorization: Bearer sk_live_xxx" \
  -H "Content-Type: application/json" \
  -d '{"customer_id": "547766341013094363"}'
```

---

## Status Enumerations

### Trade statuses

| Status | Description |
|--------|-------------|
| `requires_payment_method` | Initial state — customer has not completed payment |
| `processing` | Payment is being processed asynchronously |
| `payment_failed` | Payment declined (insufficient funds, card verification failed, etc.) — customer can retry |
| `succeeded` | Payment successful |
| `closed` | Closed — trade order will not change (e.g., renewal failed after retries, or no payment for 7+ days) |

### Trade refund statuses

| Status | Description |
|--------|-------------|
| `no_refund` | No refunds issued |
| `partially_refunded` | Partially refunded |
| `refunded` | Fully refunded |

### Subscription statuses

| Status | Description |
|--------|-------------|
| `init` | Pending activation |
| `trial` | Trial period |
| `active` | Active |
| `paused` | Paused |
| `past_due` | Payment overdue, retrying |
| `unpaid` | Unpaid after max retries |
| `canceled` | Terminated |
| `incomplete` | Incomplete |

### Invoice statuses

| Status | Description |
|--------|-------------|
| `open` | Initiated/Open |
| `success` | Paid |
| `failed` | Payment failed |
| `refunded` | Fully refunded |
| `partially_refunded` | Partially refunded |

### Invoice types

| Type | Description |
|------|-------------|
| `initial` | Initial payment |
| `trial` | Trial period |
| `renewal` | Renewal |
| `refund` | Refund |
| `upgrade` | Subscription upgrade |

### Refund statuses

| Status | Description |
|--------|-------------|
| `pending` | Processing |
| `succeeded` | Refund succeeded |
| `failed` | Refund failed |

---

## Webhooks

Subotiz sends HTTPS POST requests with JSON event payloads to your configured endpoint. Use webhooks to keep your system in sync with payment status, subscription lifecycle, invoices, and refunds.

### Webhook event list (v2)

| Event Name | Event Type | Trigger Condition | Access Impact |
|------------|------------|-------------------|---------------|
| Trial Period Expiring Soon | `v2.subscription.trial_period_expiring` | Trial expires in 3 days | Send reminder |
| First Subscription Activated | `v2.subscription.first` | Subscription enters trial/active for the first time | **Grant access** |
| Subscription Canceled | `v2.subscription.canceled` | Subscription terminated | **Revoke access** |
| Subscription Changed | `v2.subscription.price_changed` | Subscription upgraded/downgraded | Update plan |
| Subscription Paused | `v2.subscription.paused` | Subscription paused | **Revoke access** |
| Subscription Resumed | `v2.subscription.resumed` | Subscription resumed | **Grant access** |
| Subscription Past Due | `v2.subscription.past_due` | Payment overdue, retrying | Send payment reminder |
| Subscription Unpaid | `v2.subscription.unpaid` | Unpaid after max retries | **Revoke access** |
| Cancellation Revoked | `v2.subscription.cancellation_revoked` | Cancellation revoked | **Grant access** |
| Cancellation Requested | `v2.subscription.cancellation_requested` | Cancellation requested | Retention opportunity |
| Invoice Paid | `v2.invoice.paid` | Invoice paid successfully | Sync invoice status |
| Invoice Payment Failed | `v2.invoice.payment_failed` | Invoice payment failed (final failure, not per-retry) | Alert customer |
| Invoice Refunded | `v2.invoice.refunded` | Subscription trade refunded | Sync refund status |
| Trade Created | `v2.trades.created` | Trade order created | — |
| Trade Succeeded | `v2.trades.succeeded` | Payment successful | **Grant access / fulfill order** |
| Trade Payment Failed | `v2.trades.payment_failed` | Payment failed | Alert customer |
| Refund Succeeded | `v2.refunds.succeeded` | Refund processed successfully | Sync refund status |
| Refund Failed | `v2.refunds.failed` | Refund attempt failed | Retry or investigate |

### Webhook request headers

| Header | Description |
|--------|-------------|
| `X-Timestamp` | Millisecond timestamp (used in signature) |
| `X-Access-No` | Your `access_no` |
| `X-Signature` | HMAC-SHA256 signature |
| `Content-Type` | `application/json` |

### Webhook payload structure

```json
{
  "id": "572677246926464036",
  "type": "v2.trades.succeeded",
  "created": "2025-10-28T06:54:55Z",
  "data": {
    // event-specific business object
  }
}
```

### Signature verification

Verify the `X-Signature` header against the raw request body before processing any event.

**Signature string format:** `${X-Timestamp}.${raw_body}`

**Go example:**

```go
func CalcWebhookSignature(timestamp int64, body []byte, secret string) string {
    mac := hmac.New(sha256.New, []byte(secret))
    mac.Write([]byte(fmt.Sprintf("%d", timestamp)))
    mac.Write([]byte("."))
    mac.Write(body)
    return hex.EncodeToString(mac.Sum(nil))
}
```

**Python example:**

```python
import hmac
import hashlib

def verify_webhook(timestamp: str, body: bytes, secret: str, signature: str) -> bool:
    message = f"{timestamp}.".encode('utf-8') + body
    computed = hmac.new(secret.encode('utf-8'), message, hashlib.sha256).hexdigest()
    return hmac.compare_digest(computed, signature)
```

**Key rotation:** During rotation grace period, try verifying with old key first, then new key. If both fail, reject.

### Retry behavior

Non-200 responses trigger retries: backoff within 48 hours, up to 16 times. Intervals: 1m, 2m, 4m, 8m, ... up to a max of 4 hours between retries.

### Processing requirements

- **Idempotency**: Deduplicate events by `id`.
- **Event filtering**: Return HTTP 200 for non-relevant event types.
- **Order independence**: Events may arrive out of order. Query the API for latest status if needed.

### Webhook payload examples

**`v2.trades.succeeded`** — Trade payment successful:

```json
{
  "id": "572677246926464036",
  "type": "v2.trades.succeeded",
  "created": "2025-10-28T06:54:55Z",
  "data": {
    "trade_id": "572677233903157186",
    "access_no": "77d52a21dc032b4",
    "merchant_id": "2816433",
    "customer_id": "547766341013094363",
    "order_id": "order_1761634475936438746",
    "amount": "30.00",
    "currency": "USD",
    "trade_status": "succeeded",
    "payment_mode": "subscription",
    "payment_method": "credit_card",
    "payment_channel": "shoplazzapayment",
    "payment_token": "pB_KO6Q***YU",
    "paid_at": "2025-10-28T06:54:55Z",
    "session_id": "572677164911046667",
    "invoice_id": "547766341013094363",
    "refund_status": "no_refund",
    "total_refunded_amount": "0.00",
    "metadata": {},
    "last_payment_error": null
  }
}
```

**`v2.subscription.first`** — First subscription activated:

```json
{
  "id": "572677252513276964",
  "type": "v2.subscription.first",
  "created": "2025-10-28T06:54:56Z",
  "data": {
    "id": "572677251968024511",
    "customer_id": "547766341013094363",
    "sub_merchant_id": "2216433",
    "status": "active",
    "price_id": "572349501902208959",
    "total_cycles": "0",
    "cycle_index": "1",
    "current_period_start": "2025-10-28T06:54:00Z",
    "current_period_end": "2025-10-28T07:25:00Z",
    "next_invoice_date": "2025-10-28T07:26:00Z",
    "order_id": "order_1761634475936438746",
    "source_trade_id": "572677233903157186",
    "cancel_at": null,
    "cancel_reason": "",
    "created_at": "2025-10-28T06:54:56Z",
    "updated_at": "2025-10-28T06:54:56Z"
  }
}
```

**`v2.subscription.canceled`** — Subscription terminated:

```json
{
  "id": "572682701203579940",
  "type": "v2.subscription.canceled",
  "created": "2025-10-28T07:16:35Z",
  "data": {
    "id": "572677251968024511",
    "customer_id": "547766341013094363",
    "sub_merchant_id": "2816433",
    "status": "canceled",
    "price_id": "572349625697058751",
    "cancel_at": "2025-10-28T07:16:00Z",
    "cancel_reason": "cancel",
    "order_id": "order_1761634475936438746",
    "source_trade_id": "572677233903157186"
  }
}
```

**`v2.invoice.paid`** — Invoice paid:

```json
{
  "id": "583564654786128554",
  "type": "v2.invoice.paid",
  "created": "2025-11-27T07:57:35Z",
  "data": {
    "id": "583564651824957126",
    "subscription_id": "583564651824940742",
    "customer_id": "537465921338359803",
    "sub_merchant_id": "100010",
    "trade_id": "583564647529987790",
    "amount": "10",
    "currency": "USD",
    "status": "success",
    "invoice_type": "initial",
    "cycle_index": "1",
    "cycle_start": "2025-11-27T07:57:00Z",
    "cycle_end": "2025-11-27T13:07:00Z",
    "paid_at": "2025-11-27T07:57:34Z",
    "order_id": "order_1764230244388418776"
  }
}
```

**`v2.refunds.succeeded`** — Refund processed:

```json
{
  "id": "572683623191290916",
  "type": "v2.refunds.succeeded",
  "created": "2025-10-28T07:20:15Z",
  "data": {
    "refund_id": "20251028572683602265931714",
    "trade_id": "572679783406649282",
    "access_no": "77d52a21dc032b4",
    "merchant_id": "2816433",
    "refund_amount": "15.57",
    "currency": "USD",
    "refund_status": "succeeded",
    "finished_at": "2025-10-28T07:20:15Z",
    "reason": "",
    "metadata": null
  }
}
```

---

## Going Live

### Prerequisites

1. Register a Subotiz merchant account at [admin.subotiz.com/signup](https://admin.subotiz.com/signup)
2. Complete merchant qualification review
3. Create products and pricing in the production dashboard

### Switch from sandbox to production

1. **API base URL:** Change from `https://api.sandbox.subotiz.com` to `https://api.subotiz.com`
2. **Dashboard:** Switch to `https://admin.subotiz.com`
3. **SDK:** Set `environment: 'PRODUCTION'`
4. **API keys / access_secret:** Use production credentials from the production dashboard

---

## Tips for Agents

### Context awareness
- **Figure out what your human needs first.** Don't assume they want code. They might want an overview, dashboard steps, or API usage.
- **Ask questions when you're unsure.** "Do you already have a Subotiz account?" is always better than guessing.
- **Clarify the integration method.** For "integrate Subotiz," ask: Hosted (redirect), Embedded (full checkout), or Payment-only?

### When working with the API
- **Always use sandbox first.** Mistakes in production affect real customers and real money.
- **Prices are strings with two decimal places** — `"30.00"` means $30.00 (NOT cents like some other platforms).
- **Don't guess IDs.** List resources first, then use actual IDs.
- **Auth:** Confirm whether they use Bearer API key or Hub-* signature; both are valid.
- **Include `order_id` when creating checkout sessions** — it's how you correlate payments back to your internal orders.
- **Save `payment_token`** from `v2.trades.succeeded` events for subscription renewals.

### When handling webhooks
- **Always verify signatures** using HMAC-SHA256 before processing.
- **Implement idempotency** — deduplicate by event `id`.
- **Handle key rotation** — support both old and new keys during grace period.
- **Return HTTP 200** for all events, even ones you don't process.
- **Events may arrive out of order** — query the API if you need the latest state.

### When debugging
- **Check trade_status** — see [Status Enumerations](#status-enumerations) for what each status means.
- **Check `last_payment_error`** in trade webhook data for failure details.
- **For subscription issues**, check the subscription status and `cancel_reason`.
- **Include `trace_id` or error codes** when contacting support.

---

## Links

| Resource          | URL |
|-------------------|-----|
| Subotiz           | <https://subotiz.com> |
| Developer Center  | <https://developer.subotiz.com> |
| API Reference     | <https://developer.subotiz.com/v1.0-en-us/reference/overview-1> |
| Introduction (API)| <https://developer.subotiz.com/reference/introduction-1> |
| Authentication    | [Bearer](https://developer.subotiz.com/reference/authentication-1) · [Signature](https://developer.subotiz.com/reference/authentication-2) |
| Webhooks          | <https://developer.subotiz.com/reference/introduction-2> |
| Subotiz SDK       | <https://developer.subotiz.com/reference/subotiz-sdk> |
| Quick Start       | <https://developer.subotiz.com/reference/quick-start> |
| Hosted Checkout   | <https://developer.subotiz.com/reference/hosted> |
| Embedded Checkout | <https://developer.subotiz.com/reference/embedded-form> |
| Embedded Payment  | <https://developer.subotiz.com/reference/embedded-form-payment> |
| Dashboard (Prod)  | <https://admin.subotiz.com> |
| Dashboard (Sandbox) | <https://admin.sandbox.subotiz.com> |
| Support           | developer@subotiz.com |
