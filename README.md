# vybe-checkout — Cursor / Claude Agent Skill

Integrate **Vybe Native Checkout** (hosted crypto checkout sessions) into your app or store.

## Install

### Cursor (project)

```bash
mkdir -p .cursor/skills
git clone https://github.com/Vybe-Finance/vybe-checkout.git .cursor/skills/vybe-checkout
```

### Cursor (global)

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

**Customize → Rules → Add Rule → Remote Rule (Github)** → `https://github.com/Vybe-Finance/vybe-checkout`

## Invoke

In Cursor Agent chat: **`/vybe-checkout`**

Or ask: “Use vybe-checkout to add Native Checkout to my cart.”

## What it covers

- `POST /api/checkout/sessions` (fixed SKU or dynamic amount)
- Hosted `/r/{id}` for humans
- Agent JSON / 402 on the same URL
- Webhooks + fulfillment checklist

## Docs

- https://www.vybe.finance/checkout
- https://www.vybe.finance/mobiledocs
- https://www.vybe.finance/llms.txt

## Related

- Catalog / HTTP 402: [vybe-collect](https://github.com/Vybe-Finance/vybe-collect) (when published)
- Buyer agent spend: https://www.vybe.finance/pay-mcp
