# HiveVault

**The only A2A-native wallet on the planet.**

Capital flows to the task, not the agent. No agent ever holds more than one task's worth of USDC. The seed never leaves the vault.

## What it does

Every other wallet hands an agent a key and hopes for the best. HiveVault drips capital per-task, enforced by tier:

| Tier | Max Drip |
|------|----------|
| VOID | $0.10 |
| MOZ  | $0.25 |
| HAWX | $0.50 |
| EMBR | $1.00 |
| SOLX | $5.00 |
| FENR | $25.00 |

## Endpoints

| Method | Path | Description |
|--------|------|-------------|
| GET | `/health` | Service status |
| GET | `/vault/balance` | Live treasury USDC balance |
| POST | `/vault/drip` | Request capital for one task |
| POST | `/vault/deposit` | Record revenue received |
| GET | `/vault/ledger` | Full drip/deposit history |
| GET | `/vault/stats` | Lifetime stats |

## Drip flow

```
Agent → POST /vault/drip { did, address, budget, tier }
Vault → approves min(budget, tier_limit), returns drip_id
Agent → executes task, x402 payment lands in treasury
Agent → POST /vault/deposit { drip_id, tx_hash, amount }
Vault → settles drip, ledger updated
```

## Auth

Pass `X-Hive-Key` or `X-Vault-Key` header on drip requests.

## The pitch

Every A2A payment system today hands agents a wallet and hopes.  
HiveVault is the first wallet that thinks like a formation.  
Capital flows to the task. Not the agent.
