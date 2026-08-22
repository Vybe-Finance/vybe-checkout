# Vybe Native Checkout — one session, humans and agents

**Agent Skill for [Vybe Native Checkout](https://www.vybe.finance/checkout)** — crypto get-paid with a hosted checkout that works for **people in a browser** and **AI agents over HTTP**. Same session. Same amount. One balance.

Most stacks treat “checkout” and “agent commerce” as two products. Vybe doesn’t. You create one **Checkout Session**, redirect humans to a hosted page, and agents settle the **same URL** with JSON and HTTP 402 — no second storefront, no duplicate SKUs.

Settlement is **USDC / USDT** (Base, Ethereum, BNB). Not card acquiring. Built for merchants who want stablecoin rails **and** a path agents can actually pay.

---

## Why this matters

| Old world | Vybe Native Checkout |
|-----------|----------------------|
| Human checkout page + separate “AI billing” project | One session API, one catalog |
| Agents can’t complete card forms or 3DS | Agents hit `/r/{id}` with `Accept: application/json` → 402 → settle |
| Crypto checkout = another wallet tab to reconcile | Money lands on your Vybe Collect balance |
| Cart total needs a new product SKU every time | **Dynamic amount** sessions lock the order total at create time |

**The innovation:** your ecommerce or SaaS backend mints a session; **whoever pays** — person or agent — unlocks the same fulfillment webhook.

---

## What you get

- **Hosted Native Checkout** at `/r/{id}` (order summary + pay panel)
- **Sessions API** — `POST /api/checkout/sessions` with `vybe_col_…`
- **Fixed SKU** (`line_items[].price` = Collect offering) or **dynamic cart/invoice** (`amount` / `price_data`)
- **Agent pay** on the same URL — x402 / `agent_pay_url`, no UI clicks
- **Webhooks** — `payment.paid` → fulfill once, idempotently
- **This skill** — teaches Cursor / Claude how to integrate it correctly

Human rails on the hosted page: Vybe Wallet, WalletConnect, direct transfer, agentic. Agents: JSON 402 handshake, then settle and poll until `paid`.

---

## Install this skill

### Cursor (this project)

```bash
mkdir -p .cursor/skills
git clone https://github.com/Vybe-Finance/vybe-checkout.git .cursor/skills/vybe-checkout
```

### Cursor (all projects)

```bash
mkdir -p ~/.cursor/skills
git clone https://github.com/Vybe-Finance/vybe-checkout.git ~/.cursor/skills/vybe-checkout
```

### Claude Code

```bash
mkdir -p ~/.claude/skills
git clone https://github.com/Vybe-Finance/vybe-checkout.git ~/.claude/skills/vybe-checkout
```

### Cursor remote rule

**Customize → Rules → Add Rule → Remote Rule (Github)** →  
`https://github.com/Vybe-Finance/vybe-checkout`

---

## Invoke

In Cursor Agent chat:

```
/vybe-checkout
```

Or: *“Use vybe-checkout to wire Native Checkout into my cart flow.”*

The agent loads `SKILL.md` + `reference.md` and follows the integration checklist (session create → redirect → webhook fulfill).

---

## 60-second integration sketch

```bash
# 1. Create session (server-side — never expose vybe_col_… in the browser)
curl https://www.vybe.finance/api/checkout/sessions \
  -H "Authorization: Bearer vybe_col_…" \
  -H "Content-Type: application/json" \
  -d '{"mode":"payment","amount":"42.50","currency":"usdc","metadata":{"order_id":"ord_123"}}'

# 2. Redirect buyer to response.url  (/r/{id})

# 3. Fulfill on payment.paid webhook (or GET session until paid)
```

**Agent path (same session):**

```bash
curl -i "https://www.vybe.finance/r/SESSION_ID" -H "Accept: application/json"
# → 402 + agent_pay_url → settle → poll paid
```

Full API detail: [`reference.md`](reference.md) · OpenAPI: [mobiledocs](https://www.vybe.finance/mobiledocs)

---

## Who this is for

- **Ecommerce** — cart total → dynamic session → redirect → ship on webhook  
- **SaaS** — plan SKU or invoice amount → hosted pay link  
- **API / AI products** — same Collect catalog; pair with [vybe-collect](https://github.com/Vybe-Finance/vybe-collect) for 402 on your own routes  
- **Builders using Cursor / Claude** — stop guessing the API; invoke `/vybe-checkout`

---

## Honesty

- Settlement is **stablecoins**, not Visa/Mastercard  
- One line item per session today; quantity max 1  
- Buyer-side agent spend (VAPT / Pay MCP) is separate — see [pay-mcp](https://www.vybe.finance/pay-mcp)  
- This skill is for **merchants getting paid**, not agents paying arbitrary URLs  

---

## Links

| | |
|--|--|
| Product | [vybe.finance/checkout](https://www.vybe.finance/checkout) |
| Blog | [One session for humans and agents](https://www.vybe.finance/blog/vybe-checkout-humans-and-agents) |
| Agent brief | [llms.txt](https://www.vybe.finance/llms.txt) |
| Collect / 402 skill | [vybe-collect](https://github.com/Vybe-Finance/vybe-collect) |

Built by [Vybe](https://www.vybe.finance) — Collect gets you paid. Wallet holds it. Agents can pay too.
