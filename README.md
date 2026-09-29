# Vybe Native Checkout — one session, humans and agents

[![skills.sh](https://skills.sh/b/Vybe-Finance/vybe-checkout)](https://skills.sh/Vybe-Finance/vybe-checkout)

**Agent Skill for [Vybe Native Checkout](https://www.vybe.finance/checkout)** — crypto get-paid with a hosted checkout that works for **people in a browser** and **AI agents over HTTP**. Same session. Same amount. One balance.

Most stacks treat “checkout” and “agent commerce” as two products. Vybe doesn’t. You create one **Checkout Session**, redirect humans to a hosted page, and agents settle the **same URL** with JSON and HTTP 402 — no second storefront, no duplicate SKUs.

Settlement is **USDC / USDT** (Base, Ethereum, BNB). Not card acquiring.

For HTTP 402 on **your** API / buildathons, use [vybe-collect](https://github.com/Vybe-Finance/vybe-collect) and the [hackathon path](https://www.vybe.finance/collect/hackathon).

## Install via skills.sh (recommended)

```bash
npx skills add Vybe-Finance/vybe-checkout -g -y
```

Installs to Cursor, Claude Code, Codex, Copilot, and [70+ agents](https://skills.sh/docs).

## Why this matters

| Old world | Vybe Native Checkout |
|-----------|----------------------|
| Human checkout page + separate “AI billing” project | One session API, one catalog |
| Agents can’t complete card forms or 3DS | Agents hit `/r/{id}` with `Accept: application/json` → 402 → settle |
| Crypto checkout = another wallet tab to reconcile | Money lands on your Vybe Collect balance |
| Cart total needs a new product SKU every time | **Dynamic amount** sessions lock the order total at create time |

## What you get

- **Hosted Native Checkout** at `/r/{id}`
- **Sessions API** — `POST /api/checkout/sessions` with `vybe_col_…`
- **Fixed SKU**, **multi-SKU cart**, or **dynamic** amount
- **Agent pay** on the same URL — x402 / `agent_pay_url`
- **Webhooks** — `payment.paid` → fulfill once, idempotently
- **This skill** — teaches Cursor / Claude how to integrate it correctly

## Install (manual)

```bash
mkdir -p .cursor/skills
git clone https://github.com/Vybe-Finance/vybe-checkout.git .cursor/skills/vybe-checkout
```

## Invoke

```
/vybe-checkout
```

Or: *“Use vybe-checkout to wire Native Checkout into my cart flow.”*

## 60-second sketch

```bash
curl https://www.vybe.finance/api/checkout/sessions \
  -H "Authorization: Bearer vybe_col_…" \
  -H "Content-Type: application/json" \
  -d '{"mode":"payment","amount":"42.50","currency":"usdc","metadata":{"order_id":"ord_123"}}'

# Redirect buyer to response.url  (/r/{id})
# Fulfill on payment.paid webhook (or GET session until paid)
```

**Agent path (same session):**

```bash
curl -i "https://www.vybe.finance/r/SESSION_ID" -H "Accept: application/json"
```

Full API: [`reference.md`](reference.md) · OpenAPI: [collect/openapi](https://www.vybe.finance/collect/openapi)

## Honesty

- Settlement is **stablecoins**, not Visa/Mastercard
- Up to **50** line items; quantity **1–99** per line (same token)
- Buyer-side agent spend (VAPT / Pay MCP) is separate — see [pay-mcp](https://www.vybe.finance/pay-mcp)
- This skill is for **merchants getting paid**, not agents paying arbitrary URLs

## Links

| | |
|--|--|
| Product | [vybe.finance/checkout](https://www.vybe.finance/checkout) |
| Collect / 402 | [vybe-collect](https://github.com/Vybe-Finance/vybe-collect) |
| Hackathon path | [collect/hackathon](https://www.vybe.finance/collect/hackathon) |
| Agent brief | [llms.txt](https://www.vybe.finance/llms.txt) |

Built by [Vybe](https://www.vybe.finance) — Collect gets you paid. Wallet holds it. Agents can pay too.
