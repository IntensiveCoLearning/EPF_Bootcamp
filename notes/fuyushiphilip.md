---
timezone: UTC+8
---

# FYS

**GitHub ID:** fuyushiphilip

**Telegram:** 

## Self-introduction

EPF 实习计划

## Notes

<!-- Content_START -->
# 2026-04-22
<!-- DAILY_CHECKIN_2026-04-22_START -->
![image.png](https://raw.githubusercontent.com/IntensiveCoLearning/EPF_Bootcamp/main/assets/fuyushiphilip/images/2026-04-21-1776784711565-image.png)
<!-- DAILY_CHECKIN_2026-04-22_END -->

# 2026-04-21
<!-- DAILY_CHECKIN_2026-04-21_START -->

![image.png](https://raw.githubusercontent.com/IntensiveCoLearning/EPF_Bootcamp/main/assets/fuyushiphilip/images/2026-04-21-1776784711565-image.png)
<!-- DAILY_CHECKIN_2026-04-21_END -->

# 2026-04-20
<!-- DAILY_CHECKIN_2026-04-20_START -->


![image.png](https://raw.githubusercontent.com/IntensiveCoLearning/EPF_Bootcamp/main/assets/fuyushiphilip/images/2026-04-20-1776694165764-image.png)
<!-- DAILY_CHECKIN_2026-04-20_END -->

# 2026-04-19
<!-- DAILY_CHECKIN_2026-04-19_START -->



![image.png](https://raw.githubusercontent.com/IntensiveCoLearning/EPF_Bootcamp/main/assets/fuyushiphilip/images/2026-04-19-1776609679524-image.png)
<!-- DAILY_CHECKIN_2026-04-19_END -->

# 2026-04-18
<!-- DAILY_CHECKIN_2026-04-18_START -->




![image.png](https://raw.githubusercontent.com/IntensiveCoLearning/EPF_Bootcamp/main/assets/fuyushiphilip/images/2026-04-18-1776526638743-image.png)![image.png](https://raw.githubusercontent.com/IntensiveCoLearning/EPF_Bootcamp/main/assets/fuyushiphilip/images/2026-04-18-1776526729835-image.png)
<!-- DAILY_CHECKIN_2026-04-18_END -->

# 2026-04-17
<!-- DAILY_CHECKIN_2026-04-17_START -->





![image.png](https://raw.githubusercontent.com/IntensiveCoLearning/EPF_Bootcamp/main/assets/fuyushiphilip/images/2026-04-17-1776435695155-image.png)

  
  

![image.png](https://raw.githubusercontent.com/IntensiveCoLearning/EPF_Bootcamp/main/assets/fuyushiphilip/images/2026-04-17-1776435627551-image.png)
<!-- DAILY_CHECKIN_2026-04-17_END -->

# 2026-04-16
<!-- DAILY_CHECKIN_2026-04-16_START -->






![image.png](https://raw.githubusercontent.com/IntensiveCoLearning/EPF_Bootcamp/main/assets/fuyushiphilip/images/2026-04-16-1776343930155-image.png)
<!-- DAILY_CHECKIN_2026-04-16_END -->

# 2026-04-15
<!-- DAILY_CHECKIN_2026-04-15_START -->







![image.png](https://raw.githubusercontent.com/IntensiveCoLearning/EPF_Bootcamp/main/assets/fuyushiphilip/images/2026-04-15-1776263515539-image.png)
<!-- DAILY_CHECKIN_2026-04-15_END -->

# 2026-04-14
<!-- DAILY_CHECKIN_2026-04-14_START -->








## **L2 → L1 結算路徑圖**  
  

![image.png](https://raw.githubusercontent.com/IntensiveCoLearning/EPF_Bootcamp/main/assets/fuyushiphilip/images/2026-04-14-1776180476953-image.png)
<!-- DAILY_CHECKIN_2026-04-14_END -->

# 2026-04-13
<!-- DAILY_CHECKIN_2026-04-13_START -->









![image.png](https://raw.githubusercontent.com/IntensiveCoLearning/EPF_Bootcamp/main/assets/fuyushiphilip/images/2026-04-13-1776089625195-image.png)
<!-- DAILY_CHECKIN_2026-04-13_END -->

# 2026-04-12
<!-- DAILY_CHECKIN_2026-04-12_START -->










![image.png](https://raw.githubusercontent.com/IntensiveCoLearning/EPF_Bootcamp/main/assets/fuyushiphilip/images/2026-04-12-1776008346204-image.png)![image.png](https://raw.githubusercontent.com/IntensiveCoLearning/EPF_Bootcamp/main/assets/fuyushiphilip/images/2026-04-12-1776008465033-image.png)

  

![image.png](https://raw.githubusercontent.com/IntensiveCoLearning/EPF_Bootcamp/main/assets/fuyushiphilip/images/2026-04-12-1776008483108-image.png)![image.png](https://raw.githubusercontent.com/IntensiveCoLearning/EPF_Bootcamp/main/assets/fuyushiphilip/images/2026-04-12-1776008534683-image.png)![image.png](https://raw.githubusercontent.com/IntensiveCoLearning/EPF_Bootcamp/main/assets/fuyushiphilip/images/2026-04-12-1776008607530-image.png)
<!-- DAILY_CHECKIN_2026-04-12_END -->

# 2026-04-11
<!-- DAILY_CHECKIN_2026-04-11_START -->











\---

\## Transaction Lifecycle: From Creation to Finality

\`\`\`

Wallet Signed Tx → Txpool → Inclusion (in a block) → Confirmation → Finality

\`\`\`

\---

\## 1. Signed Transaction

A **Signed Transaction** is a raw transaction cryptographically signed by the sender's private key.

\### Structure

| Field | Description |

|-------|-------------|

| `nonce` | Sender's transaction count (prevents replay attacks) |

| `to` | Recipient address (or `null` for contract creation) |

| `value` | ETH amount to transfer (in wei) |

| `gas` | Maximum gas willing to pay |

| `gasPrice` | Price per gas unit (or `maxFeePerGas` / `maxPriorityFeePerGas` for EIP-1559) |

| `data` | Calldata (function selector + arguments) |

| `v, r, s` | Signature components (from private key) |

\### Why Signing Matters

\- **Authenticity** — Proves the sender authorized the transaction

\- **Integrity** — Cannot be altered without invalidating signature

\- **Non-repudiation** — Sender cannot deny sending it

\- **Determinism** — Same signed tx → same hash on all nodes

\`\`\`javascript

// Simplified example

const tx = {

nonce: 5,

to: "0x...",

value: "1.0 ETH",

gas: 21000,

gasPrice: "50 gwei",

data: "0x"

};

const signedTx = await wallet.signTransaction(tx);

// → 0xf86a... (RLP-encoded signed transaction)

\`\`\`

\---

\## 2. Txpool (Transaction Pool)

The **Txpool** (also called **mempool**) is a pending transaction queue on each Ethereum node.

\### Key Concepts

| Term | Description |

|------|-------------|

| **Pending** | Transactions ready for inclusion (nonce = current account nonce) |

| **Queued** | Transactions waiting for earlier nonces to be processed |

| **Local Tx** | Added by node's own RPC (higher priority) |

| **Remote Tx** | Received from peer nodes via gossip |

\### Txpool Organization

\`\`\`

Account A: nonce 3 (pending), nonce 4 (queued), nonce 5 (queued)

Account B: nonce 1 (pending)

Account C: nonce 7 (pending), nonce 8 (queued)

\`\`\`

\### Txpool Rules

\- Validates signature, balance, nonce, gas limit

\- Removes transactions after block inclusion

\- Drops transactions that expire (age or low gas price)

\- Maximum size limit (configurable per node)

\### Common Txpool RPC Methods

| Method | Purpose |

|--------|---------|

| `txpool_status` | Number of pending + queued transactions |

| `txpool_content` | All pending/queued transactions (detailed) |

| `txpool_inspect` | Summary view (lightweight) |

\---

\## 3. Inclusion

**Inclusion** means a transaction has been selected by a validator/miner and added to a candidate block.

\### How Inclusion Happens

1\. Validator selects transactions from txpool (prioritizing highest gas price / priority fee)

2\. Transactions are executed in order (by nonce, within block)

3\. Valid block is proposed to the network

\### Inclusion Criteria

| Factor | Impact |

|--------|--------|

| **Gas price / Priority fee** | Higher = faster inclusion |

| **Nonce** | Must match sender's current nonce |

| **Account balance** | Must cover gas cost + value |

| **Block gas limit** | ~30M gas per block → limited slots |

\### Time to Inclusion

\- Fast (high gas): 1-2 blocks (~12-24 seconds)

\- Normal (medium gas): 3-5 blocks

\- Slow (low gas): Maybe never (gets dropped from txpool)

\---

\## 4. Confirmation

A **Confirmation** occurs when a block containing your transaction is **built upon by subsequent blocks**.

\### Confirmation Count

\`\`\`

Block 100: Your transaction included ← 1 confirmation

Block 101: Built on Block 100 ← 2 confirmations

Block 102: Built on Block 101 ← 3 confirmations

...

\`\`\`

\### Confirmation vs. Inclusion

| State | Meaning | Safety Level |

|-------|---------|--------------|

| **Included** | Transaction is in a block | Low (could be reorged) |

| **1 confirmation** | 1 block built on top | Low |

| **6 confirmations** | 6 blocks built on top | Medium (standard for exchanges) |

| **12+ confirmations** | Many blocks built on top | Very high |

\### Why Confirmations Matter

\- **Reorg protection** — Longer chain = harder to reverse

\- **Finality approaching** — Probability of reversal decreases exponentially

\- **Industry standard** — Most exchanges wait 12-20 confirmations

\### Probability of Reversal

| Confirmations | Risk of reversal (estimate) |

|---------------|----------------------------|

| 0 | ~10% (uncle/orphan risk) |

| 1 | ~1% |

| 6 | ~0.01% |

| 12 | ~0.0001% |

\---

\## 5. Finality

**Finality** means a transaction **cannot be reverted** under normal network conditions.

\### Types of Finality

| Type | Description | Ethereum |

|------|-------------|----------|

| **Probabilistic** | Gets more certain over time (PoW) | Legacy (discontinued) |

| **Economic** | Too expensive to revert (PoS) | ✅ Current |

| **Absolute** | Guaranteed irreversible | Not in Ethereum |

\### Ethereum PoS Finality (Since Merge 2022)

\- **Checkpoints** occur every epoch (32 blocks, ~6.4 minutes)

\- **Two rounds of voting** by validators

\- **Justified** → **Finalized** after 2 epochs (~12.8 minutes)

\- Finalized transactions **cannot be reverted** without burning >33% of all staked ETH (~$10B+)

\### Finality Comparison

| Network | Finality Time | Mechanism |

|---------|---------------|-----------|

| Bitcoin | ~1 hour (6 blocks) | Probabilistic |

| Ethereum (PoS) | ~12-15 minutes | Economic (Casper FFG) |

| Solana | ~1-2 seconds | Proof of History |

| Traditional finance | Instant (centralized) | Legal/trust |

\---

\## 6. Replacement Transaction

A **Replacement Transaction** is a new transaction that **replaces an existing pending transaction** in the txpool.

\### Replacement Rules (EIP-1559 & Legacy)

The new transaction must:

\- Have the **same nonce** as the pending one

\- Have a **higher gas price** (legacy) or **higher priority fee** (EIP-1559)

\- Have the **same "from" address**

\### Why Replace a Transaction?

| Scenario | How |

|----------|-----|

| **Speed up (gas bump)** | Increase gas price, keep same nonce |

| **Cancel** | Send 0 ETH to self, same nonce, higher gas |

| **Fix parameters** | Change `to`, `value`, or `data`, higher gas |

\### Example: Canceling a Stuck Transaction

\`\`\`javascript

// Stuck transaction (nonce 10, gas price 10 gwei)

// Too slow → stuck in txpool

// Cancel transaction (nonce 10, higher gas price)

const cancelTx = {

nonce: 10, // Same nonce

to: myAddress, // Send to self

value: 0, // 0 ETH

gas: 21000,

maxPriorityFeePerGas: "2 gwei", // Higher than original

maxFeePerGas: "100 gwei"

};

await wallet.sendTransaction(cancelTx);

// Original transaction is replaced

\`\`\`

\### Replacement Caveats

| Condition | Result |

|-----------|--------|

| Same nonce, lower gas | **Ignored** (does nothing) |

| Same nonce, higher gas | **Replaces** the transaction |

| Same nonce, already included | **Invalid** (nonce already used) |

| Different nonce | **New independent transaction** |

\---

\## Complete Lifecycle Diagram

\`\`\`

┌─────────────┐

│ 1. SIGNED │ ← Private key signing

│ TX │

└──────┬──────┘

↓

┌─────────────┐

│ 2. TXPOOL │ ← Pending + Queued

│ (mempool) │ (gossip to peers)

└──────┬──────┘

↓ (selected by validator)

┌─────────────┐

│ 3. INCLUSION│ ← In a block (mined/proposed)

└──────┬──────┘

↓ (more blocks added)

┌─────────────┐

│ 4. CONFIRM- │ ← 1, 6, 12+ blocks on top

│ ATION │

└──────┬──────┘

↓ (2 epochs pass)

┌─────────────┐

│ 5. FINALITY │ ← Cannot revert (PoS finalized)

└─────────────┘

↑ (same nonce, higher gas)

REPLACEMENT TX → Bypasses txpool rules

\`\`\`

\---

\## Key Summary Table

| Concept | Definition | Timeframe | Reversible? |

|---------|------------|-----------|-------------|

| **Signed Tx** | Cryptographically authenticated transaction | Before submission | N/A |

| **Txpool** | Pending transaction queue on node | Seconds to hours | Yes (replace) |

| **Inclusion** | Transaction added to a block | ~12 seconds | Yes (reorg) |

| **Confirmation** | Blocks built on top | Minutes | Decreasing probability |

| **Finality** | Economically irreversible | ~12-15 min | No (except 33+% attack) |

| **Replacement Tx** | Same nonce, higher gas | Instant (replace) | Replaces pending |
<!-- DAILY_CHECKIN_2026-04-11_END -->

# 2026-04-10
<!-- DAILY_CHECKIN_2026-04-10_START -->












Opcode · Stack · Memory · Storage · Calldata · Rever  
  
**1\. EVM**

-   **Deterministic** — same input → same output on all nodes.
    
-   **256-bit stack machine** (max 1024 elements).
    
-   **State root** reflects final state after each transaction.
    
-   **Atomic transactions** — all or nothing.
    

### **2\. Memory Model & Data Locations**

-   **Stack** — temporary, very low cost.
    
-   **Memory** — per-call temporary, low cost.
    
-   **Storage** — permanent, high cost (20k gas).
    
-   **Calldata** — read-only external input, extremely cheap.
    
-   Storage writes are expensive because of permanent global consensus; Memory/Stack are discarded after execution.
    

### **3\. Errors & Rollback**

-   `revert` — rolls back state, returns remaining gas (not all gas).
    
-   `assert` **/** `panic` — internal errors, consumes **all** remaining gas.
    
-   `require` **/** `revert` — for input/condition checks (preferred).
    
-   **Events** — for off-chain indexing, do **not** change state root.
    

  
Opcode (short for Operation Code) is the low-level instruction that the Ethereum Virtual Machine (EVM) directly understands and executes.

Solidity = high-level language (human readable)

Bytecode = compiled contract (machine readable)

Opcode = individual human-readable names for each byte in the bytecode  
  

| Concept | Takeaway |
| --- | --- |
| Opcode | Basic instruction the EVM executes (1 byte each) |
| Mnemonic | Human-readable name (e.g., ADD, SSTORE) |
| Gas | Each opcode has a fixed gas cost |
| Memory model | Different opcodes target Stack, Memory, Storage, or Calldata |
| Determinism | Same sequence of opcodes → same result on all nodes |
<!-- DAILY_CHECKIN_2026-04-10_END -->

# 2026-04-09
<!-- DAILY_CHECKIN_2026-04-09_START -->













**Fork Choice · Finality · Slot · Epoch · Checkpoint · Slashing**

-   **Fork Choice**: Determines which chain is the correct one when temporary disagreements occur.
    
-   **Finality**: Provides an irreversible guarantee that a block will never be reverted or reorganized.
    
-   **Slot**: A 12-second time unit in which a single validator is assigned to propose a block.
    
-   **Epoch**: A group of 32 slots (6.4 minutes) used as the fundamental period for validator rewards and finality checks.
    
-   **Checkpoint**: The block at the start of each epoch, which serves as the target for finality votes.
    
-   **Slashing**: Penalizes malicious or dishonest validators by burning a portion of their staked ETH.  
      
    Time is divided into **slots** (12 seconds) and **epochs** (32 slots). The boundary blocks of each epoch are **checkpoints**. Validators vote on checkpoints, and when two consecutive checkpoints receive over 2/3 agreement, **finality** is achieved. If the chain temporarily diverges, the **fork choice** rule (LMD-GHOST) is used to select the correct chain. Malicious votes are penalized through **slashing**.  
      
    

![deepseek_mermaid_20260409_50c4c4.png](https://raw.githubusercontent.com/IntensiveCoLearning/EPF_Bootcamp/main/assets/fuyushiphilip/images/2026-04-09-1775742008351-deepseek_mermaid_20260409_50c4c4.png)
<!-- DAILY_CHECKIN_2026-04-09_END -->

# 2026-04-08
<!-- DAILY_CHECKIN_2026-04-08_START -->














**EIP-1559**  
  
Before EIP-1559, Ethereum used a First-Price Auction mechanism. Users would set their own gasPrice, and miners (later validators) would prioritize including transactions with the highest bids. This led to several serious issues:

-   Extreme fee volatility, with users easily overbidding or guessing wrong during peak times, resulting in transaction failures.
    
-   Poor user experience and difficulty estimating gas.
    
-   Miners had an incentive to manipulate block size to drive up fees (although this didn't happen often in practice, it was theoretically possible).
    

EIP-1559 introduced Type 2 transactions (EIP-1559 transactions), splitting fees into three components:

-   **Base Fee**: Automatically calculated by the protocol, adjusted per block based on the gas usage of the previous block. The target block size is approximately 15 million gas (with a maximum limit of 30 million gas). If the previous block exceeds the target, the base fee increases by up to 12.5%; if it falls below the target, the base fee decreases by up to 12.5%. This portion of the fee is completely burned, permanently removed from the system, and given to no one.
    
-   **Priority Fee (Tip)**: Set by the user (maxPriorityFeePerGas). This portion goes directly to the validator as an incentive for including the transaction.
    
-   **Max Fee Per Gas**: The total upper limit set by the user. The actual payment = Base Fee + Priority Fee (any surplus is refunded).  
      
      
    new\_base\_fee = old\_base\_fee × (1 + 0.125 × (gas\_used - target\_gas) / target\_gas)
<!-- DAILY_CHECKIN_2026-04-08_END -->

# 2026-04-07
<!-- DAILY_CHECKIN_2026-04-07_START -->















![Screenshot 2026-04-07 at 7.07.19 PM.png](https://raw.githubusercontent.com/IntensiveCoLearning/EPF_Bootcamp/main/assets/fuyushiphilip/images/2026-04-07-1775560105106-Screenshot_2026-04-07_at_7.07.19_PM.png)![Screenshot 2026-04-07 at 7.15.48 PM.png](https://raw.githubusercontent.com/IntensiveCoLearning/EPF_Bootcamp/main/assets/fuyushiphilip/images/2026-04-07-1775560560940-Screenshot_2026-04-07_at_7.15.48_PM.png)![Screenshot 2026-04-07 at 7.15.11 PM.png](https://raw.githubusercontent.com/IntensiveCoLearning/EPF_Bootcamp/main/assets/fuyushiphilip/images/2026-04-07-1775560527376-Screenshot_2026-04-07_at_7.15.11_PM.png)
<!-- DAILY_CHECKIN_2026-04-07_END -->
<!-- Content_END -->
