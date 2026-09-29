# Vybe Checkout — API reference

Base: `https://www.vybe.finance`

## Auth

```
Authorization: Bearer vybe_col_…
```

Dashboard session auth with `crossmintId` also works for first-party Vybe UI; **third-party merchants use the Collect key only.**

## Create session

`POST /api/checkout/sessions`

### Fixed

```json
{
  "mode": "payment",
  "line_items": [{ "price": "svc_…", "quantity": 1 }],
  "success_url": "https://…",
  "cancel_url": "https://…",
  "metadata": { "order_id": "…" }
}
```

Aliases: `line_items[0].offering_id` or top-level `offering_id`.

### Dynamic

```json
{
  "mode": "payment",
  "amount": "42.50",
  "currency": "usdc",
  "memo": "Order #1042",
  "metadata": { "order_id": "…" }
}
```

Or:

```json
{
  "mode": "payment",
  "line_items": [{
    "price_data": {
      "currency": "usdc",
      "unit_amount": "42.50",
      "product_data": { "name": "Invoice #1042", "description": "…" }
    },
    "quantity": 1
  }]
}
```

### Limits

- `line_items` max length **50** (catalog cart)
- `quantity` **1–99** per line
- `currency`: `usdc` | `usdt` (all catalog lines must share one token)
- `unit_amount` / `amount`: `^\d+(\.\d{1,6})?$`
- Optional: `shipping_amount`, `shipping_address` (required when any line `requiresShipping`)
- Optional: `variant_id` per line for size/color SKUs

### Multi-SKU cart example

```json
{
  "mode": "payment",
  "line_items": [
    { "offering_id": "svc_A", "quantity": 2 },
    { "offering_id": "svc_B", "variant_id": "var_…", "quantity": 1 }
  ],
  "shipping_amount": "5.00",
  "shipping_address": {
    "name": "Buyer",
    "line1": "1 Main St",
    "city": "Lagos",
    "country": "NG"
  },
  "metadata": { "order_id": "ord_123" }
}
```

Shopify: Business → Products → Shopify (Admin API token) syncs products; paid sessions can complete a draft order.

## Retrieve

`GET /api/checkout/sessions/{id}`  
Same Bearer auth.

## Agent path on hosted URL

```
GET /r/{id}
Accept: application/json
→ 402
```

Body includes `agent_pay_url`. Headers include x402 `PAYMENT-REQUIRED`. HTML also sets `Link: rel="payment"`.

## Webhooks (Collect Business API)

Register via Collect webhook routes (Bearer `vybe_col_…`):

| Method | Path | Purpose |
|--------|------|---------|
| `GET` / `PUT` | `/api/collect/v1/webhook` | Register HTTPS endpoint |

Events: `payment.paid`, `payment.expired`.

Signature header: `X-Vybe-Signature: sha256=…` (HMAC of raw body with webhook secret).

**Fulfill only on `payment.paid`.** Verify signature before trusting payload. Make handlers idempotent.

Exact payload fields: confirm against live OpenAPI at `/collect/openapi` or a test event from the dashboard.

## Env checklist

```bash
VYBE_COLLECT_API_KEY=vybe_col_…   # server only
VYBE_ORIGIN=https://www.vybe.finance
# optional: webhook secret from Collect webhook setup
```

## Related Collect routes (catalog, not Checkout)

- Catalog: `GET /api/agent/collect/{username}`
- Offerings admin: `GET|PUT /api/collect/v1/offerings`
- Credentials: `/api/collect/v1/credentials`

For 402-on-your-API, use the **vybe-collect** skill + `express-vybe-402`.
