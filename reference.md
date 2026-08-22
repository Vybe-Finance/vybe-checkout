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

- `line_items` max length 1
- `quantity` max 1
- `currency`: `usdc` | `usdt`
- `unit_amount` / `amount`: `^\d+(\.\d{1,6})?$`

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

Exact payload fields: confirm against live OpenAPI at `/mobiledocs` or a test event from the dashboard.

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
