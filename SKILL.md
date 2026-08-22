---
name: vybe-checkout
description: >-
  Integrate Vybe Native Checkout (hosted crypto checkout sessions) into an app
  or ecommerce store. Creates hosted /r/{id} sessions for humans and agents,
  fixed Collect SKUs or dynamic cart amounts, webhooks, and fulfillment. Use
  when the user mentions Vybe Checkout, Native Checkout, checkout.sessions,
  vybe_col_, hosted crypto checkout, /r/ payment links, merchant Collect API,
  or cart total USDC checkout.
metadata:
  version: 1.1.0
---

# Vybe Native Checkout (merchant integration)

You are integrating **Vybe Native Checkout** — Collect’s merchant door. Not a fourth product. Settlement is **USDC/USDT**, not card acquiring.

**Canonical origin:** `https://www.vybe.finance` (prefer **www** for TLS).

## When to use this vs Collect

| Goal | Skill |
|------|--------|
| Cart / Buy Now → redirect → pay → fulfill | **This skill** (`vybe-checkout`) |
| Catalog offerings, HTTP 402 on your API, express-vybe-402 | `vybe-collect` |
| Buyer agent spend key (VAPT / Pay MCP) | Not this — see `/pay-mcp` / `/llms.txt` |

## Preconditions (ask if missing)

1. Seller has a Vybe account + `@username`
2. Collect API key `vybe_col_…` (Business → Developers; shown once)
3. Optional: fixed products as Collect offerings (`svc_…` ids)
4. Env: `VYBE_COLLECT_API_KEY` (server-only — never `NEXT_PUBLIC_`)

## Integration workflow

Copy and track:

```
Checkout Progress:
- [ ] Choose fixed SKU vs dynamic amount
- [ ] Create session from server with Bearer vybe_col_…
- [ ] Persist session id ↔ order id
- [ ] Redirect human to session.url
- [ ] Fulfill on payment.paid (or poll paid) — idempotent
- [ ] Smoke: human pay + optional agent JSON 402 on same URL
```

### Step 1 — Pick amount mode

**Fixed SKU** — product already in Collect:

```http
POST https://www.vybe.finance/api/checkout/sessions
Authorization: Bearer vybe_col_…
Content-Type: application/json

{
  "mode": "payment",
  "line_items": [{ "price": "svc_OFFERING_ID", "quantity": 1 }],
  "success_url": "https://merchant.example/orders/thanks",
  "cancel_url": "https://merchant.example/cart",
  "metadata": { "order_id": "ord_123" }
}
```

**Dynamic amount** — cart / invoice / quote (no offering required):

```http
POST https://www.vybe.finance/api/checkout/sessions
Authorization: Bearer vybe_col_…
Content-Type: application/json

{
  "mode": "payment",
  "amount": "42.50",
  "currency": "usdc",
  "memo": "Order #1042",
  "metadata": { "order_id": "ord_123" }
}
```

Or `line_items[0].price_data` with `unit_amount` + `product_data.name`.

**Constraints (do not invent):**

- One line item; `quantity` max **1** today
- Currencies: `usdc` | `usdt`
- Amounts: decimal strings up to 6 fractional digits

### Step 2 — Handle create response

Use fields:

| Field | Use |
|-------|-----|
| `id` | Store with merchant order |
| `url` | Redirect humans (`/r/{id}`) |
| `agent_pay_url` | Agent settle endpoint for **this** session |
| `amount_mode` | `fixed` \| `dynamic` |
| `payment_status` | Poll until `paid` |

`success_url` / `cancel_url` are client hints only — **fulfillment truth is webhook or GET session**.

### Step 3 — Human checkout

Redirect `302` / client navigate to `session.url`. Hosted page is Native Checkout (summary + pay panel). Rails: Vybe Wallet, WalletConnect, direct transfer, agentic.

### Step 4 — Agent checkout (same URL)

Agents do **not** need a second product:

```bash
curl -i "https://www.vybe.finance/r/SESSION_ID" \
  -H "Accept: application/json"
# → 402 + agent_pay_url + PAYMENT-REQUIRED
```

Settle on `agent_pay_url` (signature or supported `txHash` flow). Poll until `payment_status: "paid"`.

### Step 5 — Fulfill

1. Register Collect webhook (`payment.paid`) via Collect Business API — see [reference.md](reference.md)
2. On paid: match `metadata.order_id` / session id → fulfill once (idempotent)
3. Fallback: `GET /api/checkout/sessions/{id}` with same Bearer key

## Code patterns (default)

Prefer **server-side** session create. Never expose `vybe_col_…` to the browser.

**Node fetch sketch:**

```js
async function createVybeCheckout({ amount, memo, orderId }) {
  const res = await fetch("https://www.vybe.finance/api/checkout/sessions", {
    method: "POST",
    headers: {
      Authorization: `Bearer ${process.env.VYBE_COLLECT_API_KEY}`,
      "Content-Type": "application/json",
    },
    body: JSON.stringify({
      mode: "payment",
      amount,
      currency: "usdc",
      memo,
      metadata: { order_id: orderId },
    }),
  });
  if (!res.ok) throw new Error(await res.text());
  return res.json(); // { id, url, agent_pay_url, amount_mode, … }
}
```

Match the host app’s language/framework; keep the same HTTP contract.

## Honesty — refuse wrong claims

- Not Visa/Mastercard acquiring
- Not “pay any URL on the internet”
- Not silent auto-capture without buyer action on hosted/agent settle
- Native Checkout ≠ Pay MCP (buyer VAPT). Seller uses `vybe_col_…`
- Do **not** brand or describe this as another company’s checkout product

## Concept → Vybe cheat sheet

| Concept | Vybe |
|---------|------|
| Secret API key | `vybe_col_…` |
| Create session | `POST /api/checkout/sessions` |
| Hosted URL | `url` → `/r/{id}` |
| Catalog prices | Collect offering ids |
| Webhooks | Collect `payment.paid` |

## Done criteria

- [ ] Server creates session with secret key
- [ ] Order linked to session id
- [ ] Human can pay on `url`
- [ ] Fulfillment runs only when paid (idempotent)
- [ ] Docs/comments say stablecoins + optional agent 402 on same URL
- [ ] No third-party checkout brand names in merchant-facing copy

## More detail

- API fields, webhook signature, retrieve — [reference.md](reference.md)
- Product docs: https://www.vybe.finance/checkout
- Blog: https://www.vybe.finance/blog/vybe-checkout-humans-and-agents
- OpenAPI: https://www.vybe.finance/mobiledocs
- Agent brief: https://www.vybe.finance/llms.txt
