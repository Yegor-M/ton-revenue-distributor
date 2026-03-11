# Royalty Splitter Merkle (TON)

A minimal, gas-efficient **pull-based royalty distribution contract** for the TON blockchain.
Funds are streamed into the contract (e.g., marketplace fees, protocol revenue) and split between:

- **Creator** — receives a fixed percentage (e.g., 50%) on epoch finalization
- **Holders** — claim their share using **Merkle proofs** per epoch

This design enables scalable distribution to hundreds or thousands of holders **without iterating on-chain**.

---

## ✦ Key Features

| Feature | Description |
|---|---|
| **Pull-based distribution** | Holders claim individually via Merkle proofs — no loops, no gas spikes |
| **Epoch-based payouts** | Each epoch defines a `merkleRoot`, `perShare`, and a clean claimed dictionary |
| **Configurable economics** | Creator split is set via `CREATOR_BPS` (default: `5000` = 50%) |
| **Merkle-verified claiming** | Only addresses in the epoch's Merkle tree can claim |
| **Gas-efficient & safe** | Minimized storage, no external dependencies, double-claim protection |

---

## ✦ How It Works

### 1. Funding
Anyone (typically a treasury wallet or platform) can send TON to the contract.
Funds accumulate until the owner initiates a new epoch.

### 2. Finalizing an Epoch
Only the owner can call:
```
set_epoch(epochId, totalHolders, merkleRoot)
```

The contract then:
1. Calculates `pool = balance – keepAlive`
2. Splits pool using `CREATOR_BPS`
3. Sends the creator's share immediately
4. Computes `perShare` for holders
5. Resets the claimed dictionary

### 3. Claiming
Holders call:
```
claim(index, proofCell)
```

- Merkle proof is validated on-chain
- Holder receives `perShare`
- Claim is permanently recorded in a compact bit dictionary

---

## ✦ Security Model

- All proof verification is deterministic and stateless
- Double-claiming is prevented via a persistent bit dictionary
- Contract maintains a `keepAlive` reserve to remain deployable long-term
- All state mutation is scoped within epoch boundaries

---

## ✦ Repository Structure
```
contracts/
  royalty-splitter-merkle.fc       # FunC smart contract

wrappers/
  RoyaltySplitterMerkle.ts         # TypeScript contract wrapper

tests/
  royaltySplitterMerkle.spec.ts    # End-to-end multi-epoch test suite

utils/
  merkle.ts                        # Merkle tree builder (off-chain)
```

---

## ✦ Development
```bash
# Install dependencies
npm install

# Run all tests
npm test

# Rebuild contract
npx blueprint build
```

---

## ✦ Testing Coverage

| Scenario | Description |
|---|---|
| Single epoch | Basic distribution and claiming |
| Multi-epoch simulation | 100 epochs × 100 holders |
| Merkle proof verification | Valid and invalid proof handling |
| Gas accounting | Economic summary per epoch |
| Double-claim protection | Replay attack prevention |

Tests model realistic marketplace revenue flows using an external treasury wallet.

---

## ✦ License

MIT — free to use in commercial and open-source projects.
