# x402 Payment Skill

Pay-per-call API access to AIsa endpoints using the [x402](https://www.x402.org/) HTTP payment protocol. No API key needed — pay with USDC on [Arc testnet](https://testnet.arcscan.app/) via [Circle Gateway](https://www.circle.com/gateway).

## How It Works

```
Agent ──► AIsa API (HTTP 402) ──► Agent signs EIP-712 payment ──► API returns data
                                         │
                               Circle Gateway (batched USDC settlement)
```

1. Agent requests a paid `/apis/v2/` endpoint
2. Server responds HTTP 402 with payment requirements
3. Agent signs an EIP-712 `TransferWithAuthorization` for USDC
4. Agent re-sends with signed payment headers
5. Server verifies, settles via Circle Gateway, returns data

## Quick Start

```bash
npm install -g @open-wallet-standard/core   # OWS CLI
npm install                                  # skill dependencies
ows wallet create --name my-agent            # create wallet
```

Fund with testnet USDC at **https://faucet.circle.com/** (select Arc Testnet), then deposit into the Gateway:

```bash
export OWS_MNEMONIC="your twelve word mnemonic phrase here"
node scripts/setup.mjs all       # approve + deposit 10 USDC
node scripts/setup.mjs balance   # check balances
```

Make a request:

```bash
node scripts/x402_client.mjs GET "https://api.aisa.one/apis/v2/twitter/user/info?userName=jack"
```

## Key Details

| Item | Value |
|------|-------|
| Chain | Arc Testnet (chain ID `5042002`) |
| RPC | `https://rpc.testnet.arc.network` |
| USDC Token | `0x3600000000000000000000000000000000000000` (6 decimals) |
| Gateway Contract | `0x0077777d7eba4688bdef3e311b846f25870a19b9` |
| API Base URL | `https://api.aisa.one/apis/v2/` |
| Faucet | https://faucet.circle.com/ |

## Documentation

- **[SKILL.md](./SKILL.md)** — Full agent instructions, setup workflow, request examples, error handling
- **[references/endpoint-catalog.md](./references/endpoint-catalog.md)** — All 80 endpoints with prices

## Resources

- [x402 Protocol](https://www.x402.org/) — HTTP payment standard
- [Open Wallet Standard](https://openwallet.sh/) — Local wallet management for agents
- [Circle Gateway](https://developers.circle.com/gateway/concepts/technical-guide) — Batched USDC settlement
- [Arc Testnet](https://docs.arc.network/) — Circle's EVM L1 with native USDC
- [AIsa API Docs](https://docs.aisa.one) — Full endpoint documentation
