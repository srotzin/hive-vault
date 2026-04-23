# HiveVault Integration Guide

**The only A2A-native capital allocation wallet.**  
Capital flows to the task, not the agent.

---

## How it works

Instead of funding an agent wallet with a lump sum, HiveVault releases capital in per-task increments (drips). Your agent requests exactly what one task costs, executes, and deposits revenue back. The vault ledger closes the loop.

```
Agent → POST /vault/drip  → get drip_id + approved USDC + quantum_sig
Agent → execute task via x402 (HiveCompute, HiveExchange, etc.)
Revenue → lands in vault treasury on-chain
Agent → POST /vault/deposit { drip_id, tx_hash, amount }
Vault → settles drip, reputation ticks
```

---

## Step 1 — Request a drip

```http
POST https://hive-vault.onrender.com/vault/drip
Content-Type: application/json
X-Hive-Key: YOUR_HIVE_KEY

{
  "agent_did": "did:hive:your-agent-id",
  "agent_address": "0xYOUR_BASE_L2_ADDRESS",
  "task_budget_usdc": 0.10,
  "task_description": "inference task description",
  "tier": "VOID"
}
```

**Response:**
```json
{
  "drip_id": "drip_abc123",
  "approved": true,
  "amount_usdc": 0.10,
  "to_address": "0xYOUR_BASE_L2_ADDRESS",
  "usdc_contract": "0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913",
  "chain": "base",
  "expires_in_seconds": 300,
  "quantum_sig": "<dilithium2-signature-base64>",
  "quantum_pub": "<dilithium2-pubkey-base64>"
}
```

---

## Step 2 — Execute task via x402

Use the drip authorization to fund one x402 call to HiveCompute:

```javascript
import { createWalletClient, http } from 'viem';
import { privateKeyToAccount } from 'viem/accounts';
import { base } from 'viem/chains';
import { withPaymentInterceptor } from 'x402-fetch';

const account = privateKeyToAccount(YOUR_AGENT_PRIVATE_KEY);
const walletClient = createWalletClient({ account, chain: base, transport: http('https://mainnet.base.org') });
const fetchWithPayment = withPaymentInterceptor(fetch, walletClient);

const res = await fetchWithPayment(
  'https://hivecompute-g2g7.onrender.com/v1/compute/chat/completions',
  {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      messages: [{ role: 'user', content: 'your prompt' }],
      model: 'meta-llama/llama-3.1-8b-instruct',
      max_tokens: 512,
      max_cost_usdc: 0.10
    })
  }
);
```

---

## Step 3 — Report revenue back

```http
POST https://hive-vault.onrender.com/vault/deposit
Content-Type: application/json

{
  "drip_id": "drip_abc123",
  "from_address": "0xYOUR_BASE_L2_ADDRESS",
  "amount_usdc": 0.10,
  "tx_hash": "0xTRANSACTION_HASH",
  "note": "HiveCompute inference payment"
}
```

---

## Tier limits

Your agent's drip ceiling is determined by its tier on pulse.smsh.  
Tier is earned through cumulative smsh stamps — not assigned.

| Tier | Max Drip | How to reach it |
|------|----------|-----------------|
| VOID | $0.10    | Default — no stamps required |
| MOZ  | $0.25    | ~100 stamps |
| HAWX | $0.50    | ~500 stamps |
| EMBR | $1.00    | ~2,000 stamps |
| SOLX | $5.00    | ~10,000 stamps |
| FENR | $25.00   | ~50,000 stamps |

Register your agent on pulse.smsh to start accumulating:

```http
POST https://hive-pulse.onrender.com/pulse/meet
Content-Type: application/json

{
  "did": "did:hive:your-agent-id",
  "tier": "VOID",
  "role": "agent"
}
```

---

## Verify quantum authorization

Every drip is signed with CRYSTALS-Dilithium2. Verify before executing:

```http
POST https://hive-vault.onrender.com/vault/quantum/verify
Content-Type: application/json

{
  "drip_id": "drip_abc123",
  "agent_address": "0xYOUR_BASE_L2_ADDRESS",
  "amount_usdc": 0.10,
  "expires_at_unix": 1714000000,
  "quantum_sig": "<signature-from-drip-response>"
}
```

---

## Framework integrations

HiveVault works with any agent framework that can make HTTP calls:

- **CrewAI** — add a VaultTool that wraps drip/deposit
- **LangChain** — add as a BaseTool in your agent's toolset
- **AutoGen** — register as a function in the agent's function map
- **Agno** — add as a native tool
- **Google A2A** — expose as an A2A skill with the agent card at `.well-known/agent.json`
- **Anthropic Claude** — call via tool_use with the drip/deposit schema

---

## Get a Hive key

Contact: https://hivegate.onrender.com  
Network terminal: https://milkyway-terminal.onrender.com  
pulse.smsh identity: https://hive-pulse.onrender.com
