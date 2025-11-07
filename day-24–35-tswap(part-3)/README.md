# 🧠 Day 24–35 of Learning Smart Contract Auditing (Security Review) 🔍 [TSwap (Part 3)]

**Focus:** TSwap (Part 3) — Recap Full Module and Final Auditing Note .

**Project:** TSwap (Uniswap-like DEX)  
**Course:** [@CyfrinUpdraft Smart Contract Security Course](https://updraft.cyfrin.io/courses/security)  
**TSwap Repo:** [5-t-swap-audit](https://github.com/Cyfrin/5-t-swap-audit)

---

## 🔁 3-Day Full Recap (Before Starting the Audit)
In 3 days, I re-studied the **entire TSwap module** before beginning my manual audit —  
this was not new learning but a **full consolidation** of everything from DEX basics to fuzzing and invariant design.

**Focus during recap:**
- Deep review of **DEX/AMM concepts** and how `x * y = k` maintains balance.  
- Revisited **fuzzing approaches** — stateless, stateful, and handler-based testing.  
- Reviewed **invariant test design** using `StdInvariant` and `Handler.t.sol`.  
- Rechecked **constant product math**, `swapExactInput`, `swapExactOutput`, and liquidity functions.  
- Validated fuzz findings to prepare for manual review and real-world exploit thinking.

After the 3-day recap, I was ready for **manual auditing, bug isolation, and report writing**.

---

## 🔍 Day 27–35: Manual Review + Findings + Reporting

### 🎯 Focus
Manual code review of TSwap contracts (`PoolFactory`, `TSwapPool`) → detect logic flaws, confirm fuzz hints, write Foundry PoCs, and prepare final professional audit report.

---

## ⚡ Key Findings (Detection → PoC → Mitigation)

---

### 1️⃣ Critical — Unbounded Giveaway in `_swap()`
**Problem:** `_swap()` included a reward logic transferring tokens to `msg.sender` every few swaps.  
Attackers can trigger it repeatedly and **drain the pool**, violating the invariant `x * y = k`.

**Detection checklist:**
- Search `_swap()` for `safeTransfer(msg.sender` or `transfer(msg.sender`.
- Check for counters or loops awarding rewards.
- Simulate repetitive swaps.

**PoC:** `TSwap_GiveawayDrain`  
**Run:**  
```bash
forge test --match-test testGiveawayDrain -vv
````

**Mitigation:**

* Remove reward logic from `_swap`.
* If rewards are required, move to a separate incentive contract with capped payouts.
* Always keep invariant checks intact post-swap.

---

### 2️⃣ High — Fee Multiplier Bug (`getInputAmountBasedOnOutput`)

**Problem:** Wrong constant (`10000` instead of `1000`) inflated input cost by ~10x.
Users could overpay during swaps, breaking pricing fairness.

**Detection checklist:**

* Grep for `1000`, `997`, `10000` constants.
* Compare formula results manually for small values.

**PoC:** `TSwap_FeeOvercharge`
**Run:**

```bash
forge test --match-test testFeeOverchargeInputAmount -vv
```

**Mitigation:**

* Use named constants for clarity:

  ```solidity
  uint256 constant FEE_NUMERATOR = 997;
  uint256 constant FEE_DENOMINATOR = 1000;
  ```
* Correct multiplier and add `+1` rounding.
* Add boundary tests for small and large reserves.

---

### 3️⃣ High — Missing Slippage Protection (`swapExactOutput`)

**Problem:** Function lacks `maxInputAmount`, allowing users to overpay if price moves before confirmation.

**Detection checklist:**

* Inspect `swapExactOutput` signature → ensure `maxInputAmount` param exists.
* Simulate price movement during swaps.

**PoC:** `TSwap_SwapExactOutput_Slippage`
**Run:**

```bash
forge test --match-test testSwapExactOutputWithoutMaxInput -vv
```

**Mitigation:**

* Add a `maxInputAmount` parameter.
* Include check:

  ```solidity
  if (inputAmount > maxInputAmount) revert InputTooHigh();
  ```
* Update frontend logic accordingly.

---

### 4️⃣ High — Imbalanced Bootstrap (Unrestricted First Deposit)

**Problem:** First LP can deposit one-sided liquidity → skewed pricing, exploitable pool initialization.

**Detection checklist:**

* Look for `if (totalSupply == 0)` in `addLiquidity` or `deposit`.
* Attempt first deposit with only one token.

**PoC:** `TSwap_ImbalancedBootstrap`
**Run:**

```bash
forge test --match-test testImbalancedInitialDeposit -vv
```

**Mitigation:**

* Enforce ratio check on first deposit:

  ```solidity
  require(amountA * reserveB == amountB * reserveA, "Unbalanced deposit");
  ```
* Use Uniswap-style LP minting for fairness.

---

### 5️⃣ Medium — Wrong Function Used in `sellPoolTokens()`

**Problem:** Function incorrectly calls `swapExactOutput` instead of `swapExactInput`, reversing logic and corrupting reserves.

**Detection checklist:**

* Read `sellPoolTokens()` → trace internal calls.
* Verify event order and parameter mapping.

**PoC:** `TSwap_SellPoolTokens_Misuse`
**Run:**

```bash
forge test --match-test testSellPoolTokensWrongCall -vv
```

**Mitigation:**

* Call correct internal swap helper.
* Ensure parameters and events match expected order.

---

### 6️⃣ Medium — Factory Missing Zero Address Validation

**Problem:** Constructor doesn’t check if WETH address is zero.
Deploying with `address(0)` breaks protocol permanently.

**Detection checklist:**

* Review `constructor` in `PoolFactory`.
* Test with `address(0)` input.

**PoC:** `TSwap_FactoryConstructor_Zero`
**Run:**

```bash
forge test --match-test testFactoryConstructorRejectsZero -vv
```

**Mitigation:**

* Add:

  ```solidity
  require(address(weth) != address(0), "Invalid WETH");
  ```
* Optionally check `Address.isContract()` for safety.

---

### 7️⃣ Informational — Weird ERC20 / Fee-on-Transfer Tokens

**Problem:** Protocol assumes exact token amounts after transfer.
Fee-on-transfer or rebasing tokens break this assumption.

**Detection checklist:**

* Grep for `transferFrom` assumptions in accounting logic.
* Run swap tests with a mock fee-on-transfer token.

**PoC:** `TSwap_WeirdERC20`
**Run:**

```bash
forge test --match-test testFeeOnTransferBehavior -vv
```

**Mitigation:**

* Use `balanceOf` deltas instead of assumed transfer amounts.
* Document that only standard ERC20 tokens are supported.

---

## 🧩 Quick Commands Summary

| Finding                 | Test                             | Command                                                          |
| ----------------------- | -------------------------------- | ---------------------------------------------------------------- |
| Giveaway drain          | `TSwap_GiveawayDrain`            | `forge test --match-test testGiveawayDrain -vv`                  |
| Fee overcharge          | `TSwap_FeeOvercharge`            | `forge test --match-test testFeeOverchargeInputAmount -vv`       |
| Missing slippage        | `TSwap_SwapExactOutput_Slippage` | `forge test --match-test testSwapExactOutputWithoutMaxInput -vv` |
| Imbalanced bootstrap    | `TSwap_ImbalancedBootstrap`      | `forge test --match-test testImbalancedInitialDeposit -vv`       |
| Wrong function usage    | `TSwap_SellPoolTokens_Misuse`    | `forge test --match-test testSellPoolTokensWrongCall -vv`        |
| Zero address in factory | `TSwap_FactoryConstructor_Zero`  | `forge test --match-test testFactoryConstructorRejectsZero -vv`  |
| Weird ERC20 handling    | `TSwap_WeirdERC20`               | `forge test --match-test testFeeOnTransferBehavior -vv`          |

---

## 🧠 Audit Tips — How to Find & Confirm Bugs

* **Check math first:** Fee constants, rounding, and multipliers are high-risk.
* **Search for transfers inside core logic:** `safeTransfer` / `transfer` → potential reward or payout logic.
* **Bootstrap checks:** Look for `totalSupply == 0` → ensure fair pool creation.
* **Reentrancy:** External calls before state updates or within loops.
* **Static tools:** Run both `slither .` and `aderyn .` — combine results.
* **Compiler warnings:** Never ignore them — often hide unused or incorrect parameters.
* **Fuzz smartly:** Handler-based fuzzing covers state transitions more accurately than stateless tests.
* **Write one focused PoC per bug:** Keeps debugging simple and fast.

---

## ✅ Mini Audit Checklist

* [ ] Run `forge test -vv` for baseline and PoCs.
* [ ] Check invariants (`x * y = k`) with fuzz/invariant tests.
* [ ] Verify all math functions and constants.
* [ ] Ensure bootstrap deposit has ratio enforcement.
* [ ] Confirm slippage protection exists.
* [ ] Run static analysis + compiler warnings.
* [ ] Test with a fee-on-transfer mock token.
* [ ] Compare results to `TSwap_final_report.md` to confirm same issues.

---

## 📘 Final Reference

🔗 **Full detailed audit report (with PoCs & code fixes):**
[TSwap_final_report.md/pdf → Codehawks-Firstflights-Audits/audits/TSwap](https://github.com/0xrafiKaji/Codehawks-Firstflights-Audits/tree/main/audits/TSwap)

This final report contains:

* Complete technical write-ups for all findings
* Proof of Concepts (Foundry tests)
* Recommended code fixes and invariant verifications

---

### 🧩 Final Takeaway

This phase marks the completion of my **first full DeFi-style audit** from recon → fuzzing → manual review → report.
Through this, I learned not just how to **find** bugs but how to **prove and fix** them clearly.

> “An auditor’s goal isn’t just finding flaws — it’s proving their impact and helping builders prevent them forever.” 🚀