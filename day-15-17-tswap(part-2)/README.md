# 🧠 Day 15–17 of Learning Smart Contract Auditing (Security Review) 🔍

Following @CyfrinUpdraft’s Smart Contract Auditing course 🚀

### 🎯 Focus(TSwap Protocol)

Advanced auditing techniques — **stateful fuzzing**, **invariant testing**, and **manual review** of the `TSwap` protocol.

---

### 🧩 Topics Covered

#### 🧪 Invariant Testing

* Built **stateful fuzz suite** using `StdInvariant` and `Handler` pattern.
* Designed to validate `x * y = k` constant product formula (core DEX invariant).
* Created mock ERC20 tokens and liquidity pools using `PoolFactory` + `TSwapPool`.
* Verified pool behaviors via `deposit()` and `swapExactOutput()` fuzz tests.
* Assertions compared `expectedDeltaX/Y` vs `actualDeltaX/Y`.

#### ⚙️ Handler Contract (test/invariant/Handler.t.sol)

* Tracked pool states (`startingX/Y`, `expectedDeltaX/Y`, `actualDeltaX/Y`).
* Used functions:

  * `deposit(uint256 wethAmount)`
  * `swapPoolTokenForWethBasedOnOutputWeth(uint256 outputWeth)`
* Calculated deltas based on token reserves and protocol math.
* Detected **Invariant Violation:** protocol gives extra tokens every 10 swaps — breaks invariant.

#### 🧩 Key Learning: Invariants in AMMs

* Core invariant:

  ```
  Δx = (β / (1 - β)) * x
  ```

  where `β = Δy / y`
* Any deviation from this equation signals a **protocol logic bug**.
* Built tests for both sides (`statefulFuzz_constantProductFormulaStaysTheSameX/Y`).

---

### 🧠 Manual Review Notes

#### 🛠️ Slither Analysis

* Detected **informational findings**:

  * Unused error (`PoolFactory__PoolDoesNotExist`)
  * Missing zero-address check for constructor input.
  * Reentrancy flagged in `_swap()` (needs manual validation).
  * Solidity version inconsistencies.

#### 🧠 Aderyn Analysis

* Low-severity issues:

  * Public functions not used internally → mark `external`.
  * Use constants instead of literals.
  * Event parameters not indexed.

#### 🔍 Compiler Warnings as Audit Tool

* Found `unused variable` & `unused parameter` warnings.
  Example:

  ```solidity
  //@Audit-High - unused deadline parameter in deposit()
  //@Audit-Informational - unused variable poolTokenReserves
  ```
* Important reminder: **compiler warnings can reveal security bugs**.

---

### 🪲 Manual Findings Summary

| Severity          | Finding                                                           | Description                                                     |
| ----------------- | ----------------------------------------------------------------- | --------------------------------------------------------------- |
| **High**          | Unused `deadline` param in `deposit()`                            | May mislead users — expected deadline enforcement doesn’t exist |
| **Medium**        | Incorrect event parameter order in `_addLiquidityMintAndTransfer` | Emits swapped values, breaks event-based indexing               |
| **Low**           | Missing zero-address checks                                       | Risk of deploying with invalid token address                    |
| **Informational** | Unused custom error, literal values                               | Can be cleaned for gas & clarity                                |

---

### 🧰 Tools Used

* Foundry (`forge test`, `forge build`, `forge coverage`)
* **Slither**, **Aderyn**
* **Static Compiler Warnings**
* Manual Line-by-Line Review

---

### 🚀 Takeaways

* **Invariant fuzzing** uncovers deep logical bugs (state-based rather than input-based).
* **Handler pattern** = foundation for robust fuzz tests.
* **Static + Manual combo** gives best coverage.
* Always document findings inline with `//@Audit` tags for easy report generation.

---

