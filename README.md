
#STX-Zygelink – Cross-chain Atomic Swap Contract

*STX-Zygelink** is a secure, decentralized smart contract written in Clarity for executing **cross-chain atomic swaps** between fungible tokens. It facilitates non-custodial token exchanges between users on different blockchain networks using a hashed time-locked contract (HTLC) mechanism.

---

## ⚙️ Overview

This contract enables two parties to exchange tokens securely across blockchain networks (e.g., Stacks ↔ Bitcoin/Ethereum) by locking funds in a hash-based conditional escrow. If the participant can reveal the correct secret (hash preimage) within a specific time frame, they can redeem the locked tokens. Otherwise, the initiator can refund them after expiration.

---

## 🔐 Key Features

* **Atomic Swaps** with HTLC mechanism.
* **Cross-chain Compatibility** via hashed secrets.
* **Blockchain-Aware Validation** for supported chains (Bitcoin, Ethereum, Stacks).
* **Secure Token Handling** using Clarity’s `ft-trait` interface.
* **Time-Locked Refunds** if swaps aren't completed.

---

## 📄 Contract Summary

| Function                    | Description                                                                        |
| --------------------------- | ---------------------------------------------------------------------------------- |
| `initialize-atomic-swap`    | Starts a new atomic swap by locking tokens and providing a hash lock & expiration. |
| `register-swap-participant` | Allows a participant to register for a swap before redeeming.                      |
| `redeem-atomic-swap`        | Enables the participant to redeem locked tokens by revealing the secret preimage.  |
| `process-swap-refund`       | Allows the initiator to reclaim tokens if the swap expires without redemption.     |
| `get-atomic-swap-details`   | Read-only query to fetch details of an existing swap.                              |
| `verify-hash-preimage`      | Read-only utility to verify a hash preimage.                                       |

---

## 📦 Data Schema

### Swap Record (stored in `atomic-swaps` map)

| Field                        | Type                 | Description                                               |
| ---------------------------- | -------------------- | --------------------------------------------------------- |
| `atomic-swap-identifier`     | `(buff 32)`          | Unique ID generated per swap.                             |
| `swap-initiator`             | `principal`          | Address of the initiator locking tokens.                  |
| `swap-participant`           | `optional principal` | Address of the participant redeeming the swap.            |
| `token-contract-principal`   | `principal`          | Token contract implementing `ft-trait`.                   |
| `token-amount`               | `uint`               | Amount of tokens involved.                                |
| `atomic-hash-lock`           | `(buff 32)`          | SHA-256 hash of the secret (preimage).                    |
| `swap-expiration-height`     | `uint`               | Block height at which refund becomes possible.            |
| `swap-current-status`        | `(string-ascii 20)`  | Status: `active`, `participated`, `redeemed`, `refunded`. |
| `destination-blockchain`     | `(string-ascii 8)`   | Target chain name: `bitcoin`, `ethereum`, `stacks`.       |
| `destination-wallet-address` | `(string-ascii 42)`  | Receiver’s address on the destination chain.              |

---

## ✅ Validation Rules

* Token contract must implement `ft-trait`.
* Hash lock must be 32 bytes and not all-zero.
* Blockchain must be one of: `bitcoin`, `ethereum`, `stacks`.
* Wallet address must be valid, begin with recognized prefix (`0x`, `bc`, `SP`).
* Swap duration must be between 1 and 1440 blocks.
* Token amount must be non-zero and within bounds.
* Initiator must have sufficient balance to lock funds.

---

## 🚀 How it Works (Swap Lifecycle)

1. **Initiator creates a swap** using `initialize-atomic-swap`, locking tokens with a hash of a secret and expiration.
2. **Participant registers** via `register-swap-participant`, linking themselves to the swap.
3. **Participant redeems** tokens via `redeem-atomic-swap` by providing the correct secret preimage.
4. If the participant fails to redeem before expiration, **initiator refunds** via `process-swap-refund`.

---

## 🔄 Supported Chains

Currently supports swaps targeting these blockchains:

* **Stacks**
* **Bitcoin**
* **Ethereum**

> ⚠️ This contract does not actually perform cross-chain transfers. External relayers or oracles are expected to observe preimage revelations and complete the counterpart action on the destination chain.

---

## 🛡️ Security Considerations

* Only the **participant** can redeem the swap.
* Only the **initiator** can reclaim after expiry.
* The **secret preimage must match** the hash lock.
* All external interactions with token contracts must succeed or the transaction aborts.

---

## 📘 Example Usage

### Start a swap:

```clojure
(initialize-atomic-swap
  token-contract
  u1000
  0xABCDEF... ; 32-byte hash
  u100        ; blocks until expiry
  "bitcoin"
  "bc1qxyz..."
)
```

### Participate in a swap:

```clojure
(register-swap-participant 0x1234...)
```

### Redeem a swap:

```clojure
(redeem-atomic-swap 0x1234... 0xSECRET... token-contract)
```

### Refund a swap:

```clojure
(process-swap-refund 0x1234... token-contract)
```

---
