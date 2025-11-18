# Day 36–42 — Section 6: Thunder Loan (Cyfrin Updraft)  
**Course (ref):** https://github.com/Cyfrin/security-and-auditing-full-course-s23  
**My repo:** https://github.com/0xrafiKaji/smart-contract-auditing-note-best-practice-  
**Status:** Videos watched (Section 6 complete). _I have not performed hands-on auditing yet — starting audits from today._

---

## 1) Quick summary — what I finished (Section 6)
- Watched all videos for **Thunder Loan** (lending/loan-flash patterns, upgradeability, storage layout refresher, attacker surfaces and defensive techniques).
- Learned core roles & flows (liquidity provider, borrower, oracle, liquidator, pauser/admin).
- Revisited important upgrade & storage-collision topics (how proxy upgrades can break state layout).
- Understood typical exploit classes relevant to loans: flash-loan abuse, oracle manipulation, reentrancy, interest/logic bugs, storage collisions when upgrading.
- Key take: many Thunder Loan bugs are **business-logic + economic** problems on top of code-level vulnerabilities — so both code & economic reasoning required.

---

## 2) What you (or I) should *expect* when auditing Thunder Loan
1. **Complex flows**: borrow → use funds → repay in same tx (flash loans) or over time (margin loans).  
2. **External dependencies**: oracles, price feeds, cross-contract calls — watch trust boundaries.  
3. **Upgradeability traps**: storage layout mismatches across implementations.  
4. **Economic invariants**: collateral ratios, interest calculations, rounding — small math bugs can be catastrophic.  
5. **Stateful edge-cases**: liquidations, paused states, reentrancy during callback flows.

---

## 3) How to learn this section well (step-by-step — for you & others)
1. **Watch videos actively**: pause and take inline notes (`// @Audit`, `// Q`) for every function that moves money or calls external contracts.  
2. **Read the contracts**: start from entry points (borrow, repay, liquidate, setPrice, upgrade). Map the callgraph.  
3. **Draw the flows**: sequence diagrams (borrower → loan pool → oracle → etc). Visuals help for economic bugs.  
4. **Re-run the code locally**: clone the ThunderLoan repo (if provided) or sample contracts and `forge build` / `forge test`.  
5. **Stage PoCs early**: add small Foundry tests that assert intended invariants (e.g., collateral ratio never falls below X after function).  
6. **Discuss**: use course GitHub discussions — often instructors highlight tricky edge cases.

---

## 4) Vulnerability classes likely in Thunder Loan & **how to detect them (audit-time checklist)**

> Below each vulnerability I show: **(a)** Why it's dangerous, **(b)** How to detect it during an audit, **(c)** Minimal PoC idea / Foundry test, **(d)** Mitigation suggestions.

---

### A. Flash-loan / Business-logic Exploits (Atomic economic manipulation)
**Why dangerous:** An attacker can take a flash loan, manipulate on-chain price or state, perform arbitrage or exploit logic, then repay — profit extracted in same tx.  
**Detect:**  
- Search functions that allow borrowing without long-term checks (flashLoan hooks, callbacks).  
- Identify points where external price/oracle values are read and immediately used for accounting without delay/validation.  
- Look for functions that change global state then call external protocols in same transaction.  
**PoC idea:** simulate flash loan in Foundry by calling `hook`-style functions with manipulated oracle (stub), assert attacker profit > 0.  
```solidity
// pseudo: set oracle price -> flashLoan -> manipulate -> repay
vm.prank(attacker);
vm.deal(attacker, 0);
attackerContract.executeFlashLoan(...);
assert(attackerContract.balance > 0);
```

**Mitigation:** require TWAP/oracle aggregation, oracle sanity checks, delay windows, or on-chain guardrails (max-slippage, circuit breaker).

---

### B. Exploit: Business Logic Edge Case (collateral math, rounding)

**Why dangerous:** Incorrect collateral calculations or rounding errors can allow borrowers to withdraw more than allowed or avoid liquidation.
**Detect:**

* Inspect all math (interest, collateralFactor, liquidationThreshold).
* Check use of `uint256` vs smaller ints, `unchecked` blocks, and divisions (rounding direction).
* Simulate extreme values in tests (very large collateral, tiny collateral).
  **PoC idea:** fuzz collateralFactor and amounts to find a scenario where `withdraw` is allowed despite under-collateralization.
  **Mitigation:** clear math invariants, use safe arithmetic (Solidity >=0.8), add explicit require checks after operations.

---

### C. Exploit: Reentrancy (callbacks during loan flows)

**Why dangerous:** Loans often involve transfer/callback flows; if external call happens before state update, attacker can re-enter and drain funds.
**Detect:**

* Search for `call{value:...}()` or external transfers before updating balances/flags.
* Run `slither` — it flags reentrancy patterns.
* Identify `external` hooks that call back into core contract.
  **PoC snippet (Foundry):** deploy attacker contract that executes `borrow` then reenters on callback.
  **Mitigation:** CEI pattern (update state then external calls), `nonReentrant` guards, minimal external calls.

---

### D. Exploit: Weak/Manipulable Randomness (winner selection etc)

**Why dangerous:** If any random selection (e.g., in incentive distributions) uses `block.timestamp` or `blockhash` alone, miners/callers can bias results.
**Detect:** search for `keccak256(abi.encodePacked(...))` using block variables; review selection logic.
**Mitigation:** use Chainlink VRF or commit-reveal; ensure randomness is not used for high-value decisions unless verifiable.

---

### E. Exploit: Oracle Manipulation & Price Feed Attacks

**Why dangerous:** Oracles control valuations; a manipulated price can allow underpriced collateral withdraws or easy liquidations.
**Detect:**

* Trace all places where a price is fetched. Which oracle? single source? is aggregator used?
* Check if contract trusts `msg.sender` for price inputs or uses LP reserves directly without slippage guard.
  **PoC:** mock oracle feed in a test to a manipulated price and show profitable exploit.
  **Mitigation:** use aggregated TWAPs, peg checks, circuit-breakers, min/max price bounds.

---

### F. Exploit: Storage Collision / Upgradeability Bugs

**Why dangerous:** In proxy patterns, changing storage layout in implementations can overwrite critical variables — causing admin loss, logic breaks, or ability for attacker to set balances.
**Detect:**

* Check for proxies (UUPS/Transparent) and compare `impl` storage layout across versions.
* Look for storage gaps/gap variables; check any new variables inserted in wrong slots.
* Use `solidity-storage-layout` tooling (e.g. `forge inspect storage-layout`) or manual mapping.
  **PoC idea:** show that after upgrade a variable becomes overwritten (requires multi-step test with upgrade).
  **Mitigation:** follow storage-gap patterns, use unstructured storage carefully, write upgrade tests.

---

### G. Exploit: Integer Overflow / Unsafe Casting

**Why dangerous:** casting big `uint256` into `uint64` or unchecked arithmetic can cause truncation and wrong balances.
**Detect:** search for explicit casts (`uint64(...)`), `unchecked` blocks, and divisions. Use `chisel` or static checks.
**PoC idea:** create scenario where accumulated amounts exceed smaller type limit and show misbehaviour.
**Mitigation:** prefer `uint256` for money, assert before downcasts, avoid `unchecked` unless proven safe.

---

### H. Exploit: Access Control & Admin Key Misuse

**Why dangerous:** improper `onlyOwner` or missing checks on critical functions (pause, upgrade) allow unauthorized changes.
**Detect:** grep for `owner`, `onlyOwner`, `admin`, and ensure appropriate authority checks; search for public `initialize()` or missing `initializer` guards.
**Mitigation:** add `onlyOwner`/RBAC, use timelocks for upgrades, multi-sig for privileged ops.

---

## 5) Practical audit steps — start auditing Thunder Loan (what I will do next)

1. **Clone & run**:

   ```bash
   git clone https://github.com/Cyfrin/security-and-auditing-full-course-s23
   cd security-and-auditing-full-course-s23/Section6-ThunderLoan  # adjust path
   forge build
   forge test
   ```
2. **Quick static scan**:

   ```bash
   slither .
   ```

   Save `slither-report.txt`.
3. **Storage & upgrade check**:

   ```bash
   forge inspect <ContractName> storage-layout
   ```

   Compare implementation versions (if multiple).
4. **Add quick PoCs (Foundry)**:

   * PoC for flash-loan misuse: simulate manipulated oracle + flash loan.
   * PoC for reentrancy: attacker contract + refund/withdraw sequence.
   * PoC for oracle manipulation: mock price feed, call repay/withdraw.
5. **Fuzz & invariants**:

   * Add invariants in Foundry that should always hold (e.g., `totalCollateral >= 0`, `balances sum`).
   * Run Echidna on critical modules if possible.
6. **Write findings.md entries**:
   For each confirmed issue, create `findings/<ID>-<short-title>.md` with:

   * Title + severity (H/M/L), reproducible PoC (test + outputs), suggested fix, risk/priority.
7. **Compile final report**:

   * Once PoCs and fixes tested, compile PDF similar to your previous reports and push to portfolio.

---

## 6) Findings template (use this for each bug you will write)

```
# TITLE (Root Cause + Impact Summary)

## Description  
- Explain *what was expected* vs *what actually happens*.  
- Identify the exact faulty logic (state change, oracle read, math, access control, etc).  
- Mention why this exists: missing check? wrong assumption? incorrect invariant?

## Severity  
- High / Medium / Low  
- Based on financial loss, exploitability, impact radius.

## Risk  
- Explain likelihood (how easy to exploit)  
- Who can exploit (anyone, borrower, liquidator, flash-loan attacker, admin)?  
- Attack surface (atomic → flash loan? multi-step exploit? economic or technical?)

## Impact  
- Describe **what an attacker gains** or  
- **What breaks in the protocol**  
- Funds lost? Collateral stolen? Bypass checks? Permanent lock? DoS?

## Tools Used  
- Foundry Tests (`forge test`)  
- Slither (`slither .`)  
- cast / chisel  
- Manual code review (call graph, storage layout)  
- Fuzzing / invariants (if applicable)  

## Recommended Mitigation  
- Exact fix steps (change check, add require, reorder operations, apply CEI, validate oracle, upgrade layout rule, etc)  
- If architectural fix needed, mention both “quick patch” + “long-term fix”.

## Proof of Concepts  
- Foundry test snippet (minimal reproducible)  
- Transaction simulation if relevant  
- cast output if storage/encoding issue  
- Any reproducible steps an auditor or dev can run  
```

---

## 7) Severity / Prioritization guidance

* **High (H):** immediate funds at risk, reentrancy, flash-loan drain, oracle manipulation, upgrade that leads to admin takeover.
* **Medium (M):** DoS, business-logic allowing exploitation but less likely, unsafe casts (depending on context).
* **Low (L):** NatSpec/Docs, minor gas-waste, non-exploitable edge-case.

---

## 8) Useful commands & checklist (copy into repo)

```bash
# build & test
forge build
forge test -vvv

# static analysis
slither . --json slither-report.json

# storage layout
forge inspect ContractName storage-layout

# cast helpers (if deployed locally)
cast storage <ADDR> <SLOT>
cast parse-bytes32-string <HEX>

# interactive checks
chisel    # (if available)
```

**Audit checklist (short):**

* [ ] Run slither & save output
* [ ] Inspect storage layout & check gaps
* [ ] Search for: loops on dynamic arrays, external calls before state updates, keccak randomness using block vars, direct oracle usage, casts (`uintXX(...)`)
* [ ] Add PoC tests (Foundry) for each high-risk area
* [ ] Add invariants & run echidna fuzzing where possible
* [ ] Draft `findings/*.md` per template

---

## 9) Next steps for me (and you, if you follow)

1. Start hands-on auditing today: clone Section 6 sample & run tests locally.
2. Implement the PoC tests for reentrancy, DoS, oracle manipulation.
3. When a finding is confirmed, write `findings/H-01-<title>.md` and push small PR/branch (so history preserved).
4. After confirming multiple findings, compile final report PDF (like your PasswordStore & PuppyRaffle) and upload to portfolio.

---

## 10) Final notes / learning tips

* Focus on **intent vs implementation**: always ask “what’s this supposed to do?” then check the code path.
* Economical attacks often need both code bug + economic manipulation — don’t ignore on-chain data (liquidity, price depth).
* Keep PoCs small & deterministic (Foundry tests are great).
* Use the course GitHub to compare your approach & findings with peers/instructors.

---

>> Note: Security reviews are not linear.