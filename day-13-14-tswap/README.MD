## 🧠 Day 13–14 of Learning Smart Contract Auditing (Security Review) 🔍

Following @CyfrinUpdraft’s Smart Contract Auditing course 🚀

**Focus:** TSwap Protocol & Advanced Fuzz Testing.

### 🔹 Overview

Started auditing **TSwap**, a Uniswap-inspired decentralized exchange (DEX) built with AMM (Automated Market Maker) logic.
This phase introduced **advanced auditing and testing tools**, including:

* Fuzz testing (stateless & stateful)
* Invariant/property testing
* Handler-based fuzzing
* Understanding DeFi logic like **constant product formula** (`x * y = k`)
* Spotting complex protocol invariants

---

### 🔹 Key Learnings

#### 🧩 1. Understanding DEX & AMM

* **DEX (Decentralized Exchange):** Enables permissionless swaps between assets at algorithmic fair prices.
* **AMM Mechanism:** Uses liquidity pools instead of order books.

  * Formula: `x * y = k`
  * As users trade, pool ratios adjust → this defines token prices.
* **Liquidity Providers (LPs):**

  * Deposit assets → receive LP tokens as proof of share.
  * Earn transaction fees as reward → LP token value increases.

#### 🧠 2. Invariant Concept

* **Invariant:** A condition that must *always hold true* in a system.

  * Example: In an AMM, `x * y = k` must remain constant (except for small deviations due to fees).
* During audit/testing, breaking this invariant indicates a logical or financial flaw.

---

### 🔹 3. Fuzz Testing Techniques

#### 🧪 Stateless Fuzzing

* Sends random inputs to functions each run (resets state every time).
* Good for catching simple edge cases.
* **Command Example:**

  ```bash
  forge test --mt testFuzzCatchesStateless -vvvv
  ```
* ✅ Pros: Easy setup, quick detection of simple invariant breaks.
* ⚠️ Cons: Misses multi-call or state-dependent bugs.

#### ⚙️ Stateful Fuzzing

* Retains contract state between runs → catches state-dependent bugs.
* Example: A bug that only triggers after several sequential transactions.
* Configurable in `foundry.toml`:

  ```toml
  [invariant]
  runs = 64
  depth = 32
  fail_on_revert = true
  ```

#### 🔒 Handler-Based Stateful Fuzzing

* Introduces a **Handler contract** to guide fuzzing in logical directions.
* Avoids meaningless random inputs & unnecessary reverts.
* Steps:

  1. Build a `Handler` contract wrapping your target contract.
  2. Whitelist specific functions for fuzzing.
  3. Use `targetSelector()` & `targetContract()` in tests.
* Ensures realistic, efficient, and precise testing.

---

### 🧩 4. Example Invariant Test Flow

**Target Contract:** `HandlerStatefulFuzzCatches.sol`

**Invariant:**

> “Users must always be able to withdraw the exact balance amount.”

**Fuzz Test Workflow:**

1. Deploy mock tokens & protocol.
2. Deposit and withdraw tokens.
3. Assert balances before & after:

   ```solidity
   assert(userBalanceAfter == userBalanceBefore);
   ```
4. Catch invariant breaks → often leads to real vulnerabilities (e.g. Fee-on-Transfer “Weird ERC20” behavior).

---

### ⚔️ 5. Weird ERC20 (Fee-on-Transfer) Exploit

* Found through handler-based fuzzing.
* Certain ERC20s deduct fees on transfer → protocol’s math breaks if not handled.
* Key Lesson: Always **account for token behavior differences**.

---

### 🔍 6. Tools & Concepts Covered

| Tool                     | Use                             |
| ------------------------ | ------------------------------- |
| **Foundry Fuzzing**      | Automated invariant testing     |
| **Echidna**              | Smart, constraint-based fuzzing |
| **Invariant Testing**    | Ensures system consistency      |
| **Handler Contracts**    | Constrain fuzz input space      |
| **Constant Product AMM** | Core of DEX price mechanism     |

---

### 🧠 Key Takeaways

* **Information = Currency**: Thorough onboarding (docs, scope, commit hash) makes auditing smoother.
* **Context First**: Understand protocol logic deeply *before reading code*.
* **Fuzz Everything**: Use fuzz testing to simulate unpredictable real-world inputs.
* **Always Track Invariants**: They define protocol stability.
* **Beware Weird Tokens**: Fee, rebasing, or deflationary tokens often break invariants.

---

### 📘 Next Steps

* Apply fuzzing & invariants testing directly on TSwap contracts.
* Move toward manual review and deeper DeFi exploit discovery (like flash loans & reentrancy).

---