---
name: arc-x402
description: Access AIsa x402-paid /apis/v2/ endpoints using Arc testnet USDC and Circle Gateway. Use when setting up x402 payments, creating or funding an Arc wallet, depositing into Circle Gateway, picking the right AIsa endpoint for a task, estimating per-call cost, or making paid AIsa API calls without an API key.
---

# arc-x402

Use this skill for operational work with AIsa x402-paid endpoints.

## Files

- `scripts/check-env.sh` — verify local prerequisites, environment variables, and connectivity
- `scripts/save-mnemonic.mjs` — save a mnemonic into a local `.env` for workspace reuse
- `scripts/setup.mjs` — check balance, approve ERC-20 USDC, and deposit into Circle Gateway
- `scripts/x402_client.mjs` — make paid x402 API requests
- `references/setup.md` — environment and runtime guidance
- `references/troubleshooting.md` — common failures and fixes
- `references/endpoint-catalog.md` — **complete catalog of all 80 `/apis/v2/` endpoints with per-call USD prices**. Consult this before calling any endpoint so you know the exact cost and required parameters.

## Workflow

1. Run `scripts/check-env.sh` first.
2. If no wallet exists, create one and persist the mnemonic for workspace reuse:
   ```bash
   ows wallet create --name my-agent --show-mnemonic
   node scripts/save-mnemonic.mjs --mnemonic "<the mnemonic phrase from the output above>"
   ```
   Or export directly from an existing wallet:
   ```bash
   node scripts/save-mnemonic.mjs --wallet my-agent
   ```
3. If the wallet is not funded, use `scripts/setup.mjs` (approve + deposit).
4. Before making a paid request, look up the endpoint in `references/endpoint-catalog.md` to confirm the path, method, required params, and price.
5. If the user wants to make a paid request, use `scripts/x402_client.mjs`.
6. Read `references/troubleshooting.md` if payment or signature errors occur.

## Endpoint Selection & Pricing

The full priced catalog lives in `references/endpoint-catalog.md`. Always check it before calling — prices range from $0 (free) to $0.12 per call, a 100× spread. Category totals:

| Category | Count | Typical price |
|----------|------:|---------------|
| Twitter | 28 | $0.00044 – $0.036 |
| Search & Prediction Markets | 20 | $0.0096 – $0.010 |
| Financial | 23 | $0.012 – $0.120 |
| Scholar & Search | 4 | $0.0024 |
| Perplexity AI | 4 | $0.012 |
| YouTube | 1 | $0.0024 |

**Cost-control rules:**
- Before running expensive calls (≥ $0.036), confirm intent with the user. This includes `twitter/user/followers`, `twitter/user/followings`, `financial/financials`, `financial/analyst-estimates`, `financial/earnings/press-releases`, and all `financial/financials/{income-statements,balance-sheets,cash-flow-statements}`.
- The single most expensive endpoint is `/apis/v2/financial/financials` at $0.120 per call — prefer the individual statement endpoints at $0.048 each unless the user needs all three.
- There are no free endpoints in the current catalog.
- Do not loop calls without telling the user the total estimated cost.

## Error Handling

- If a request returns **403** with `"Pre-deduction failed"`, the agent's AIsa API quota is insufficient for the call. Follow these steps:
  1. Check current balance: `node scripts/setup.mjs balance`
  2. If the wallet's USDC balance on Arc testnet is below 5 USDC, prompt the user to claim more from the faucet: https://faucet.circle.com/
  3. Deposit USDC into the Gateway: `node scripts/setup.mjs deposit --amount 5`
  4. If the Gateway has not been approved yet: `node scripts/setup.mjs approve`
  5. Retry the failed request.

## Auto-Deposit

Before making any paid API request, check the Gateway balance with `node scripts/setup.mjs balance`. If the Gateway deposit balance falls below **0.5 USDC**, automatically deposit funds to top it up:

1. Verify the wallet has enough ERC-20 USDC on Arc testnet to cover the deposit.
2. If it does, run `node scripts/setup.mjs deposit --amount 5` (no user confirmation needed).
3. If the wallet's ERC-20 USDC balance is insufficient, prompt the user to claim more from the faucet: https://faucet.circle.com/

## Guardrails

- Do not confuse `/apis/v1/` API-key endpoints with `/apis/v2/` x402-paid endpoints.
- Do not tell users to transfer USDC directly to the Gateway contract; they must call `deposit()`.
- Do not deposit more USDC than the wallet's available ERC-20 USDC balance on Arc testnet.
- Do not run `/apis/v2/twitter/post_twitter` unless the user explicitly wants to publish a post.
- Always consult `references/endpoint-catalog.md` before quoting a price to the user. Do not guess prices from memory.
- Expect OpenClaw runtime environments to differ from an interactive shell. An env var exported in a terminal may not be visible to the running agent process.
- Prefer saving the mnemonic with `scripts/save-mnemonic.mjs` so commands can load it from the local `.env`.
- Prefer these mnemonic sources, in order: `OWS_MNEMONIC`, `X402_MNEMONIC`, local `.env`, `--mnemonic-env`, `--mnemonic`.

## Typical commands

```bash
bash scripts/check-env.sh
node scripts/save-mnemonic.mjs --wallet my-agent
node scripts/setup.mjs balance
node scripts/setup.mjs approve
node scripts/setup.mjs deposit --amount 5
node scripts/x402_client.mjs POST "https://api.aisa.one/apis/v2/scholar/search/scholar?query=bitcoin" --body '{}'
node scripts/x402_client.mjs GET "https://api.aisa.one/apis/v2/twitter/user/info?userName=jack"
node scripts/x402_client.mjs GET "https://api.aisa.one/apis/v2/twitter/user/last_tweets?userName=jack"
node scripts/x402_client.mjs GET "https://api.aisa.one/apis/v2/polymarket/markets?search=election&status=open"
node scripts/x402_client.mjs GET "https://api.aisa.one/apis/v2/kalshi/markets?search=election&status=open"
node scripts/x402_client.mjs POST "https://api.aisa.one/apis/v2/perplexity/sonar" --body '{"model":"sonar","messages":[{"role":"user","content":"What is Bitcoin? Keep it brief."}]}'
node scripts/x402_client.mjs GET "https://api.aisa.one/apis/v2/youtube/search?q=bitcoin&engine=youtube"
```

## Live-tested endpoint notes

- Twitter user endpoints require `userName`, not `screen_name`
- Polymarket and Kalshi search require `status=open|closed` when using `search`
- Perplexity endpoints require `model` in the JSON body
- YouTube search requires both `q` and `engine=youtube`
- `scholar/search/explain` is a follow-up endpoint and requires `search_id` in the request body
- `matching-markets/sports` requires `kalshi_ticker` or `polymarket_market_slug`
- If a `$0.000000` price ever appears (none currently in the catalog), still process the request through the normal x402 flow and treat `Invalid price: $0.000000` as an upstream pricing bug
