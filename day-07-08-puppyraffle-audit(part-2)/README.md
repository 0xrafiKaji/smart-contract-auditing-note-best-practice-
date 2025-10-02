# Day 7–8 of Learning Smart Contract Auditing (Security Review) 🔍

Following @CyfrinUpdraft’s Smart Contract Auditing course 🚀

**Focus:** continued manual review of `PuppyRaffle` (enterRaffle, refund, getActivePlayerIndex, selectWinner) — wrote PoCs and findings.  
**Repo (course):** https://github.com/Cyfrin/4-puppy-raffle-audit (onboarded branch)

---

## Summary of what I completed
- Returned to PuppyRaffle with better DoS context.
- Wrote a DoS PoC showing gas costs increase when checking duplicates with O(n²) loops.
- Inspected `refund()` and discovered **re-entrancy** pattern (external call before state change).
- Checked `getActivePlayerIndex()` and found edge-case (index 0 ambiguity).
- Reviewed `selectWinner()` — found **weak randomness**, potential arithmetic/precision issues, `unsafe casting` on fees.
- Wrote tests (Foundry) locally for PoC.

---

## Quick commands / tooling I used
- `forge test -vvv` / `forge test --mt <name> -vvv` (run specific PoC tests)
- `forge coverage`
- `slither .`
- `chisel` (interactive type queries)
- `cast storage <ADDR> <SLOT>`
- `cast parse-bytes32-string <HEX>`
- `echidna` / fuzzing where needed

---

# Findings (detection → PoC → mitigation)

---

## 1) DoS — Duplicate-check O(n²) loops (enterRaffle)
**Why it’s bad:** nested loops over dynamic `players` array → gas grows quadratically as players increase → later entrants pay much more / can hit block gas limit → can be used to block entrants.

**Mitigations (recommended):**
- Replace O(n) or O(n²) duplicate checks with O(1) mapping lookups.
- Use `EnumerableSet` if you need iterable set but careful with gas.
- Or allow duplicates (business decision).

**How to detect (audit-time checklist):**
- Search for `for (...)` loops iterating `players.length` or other dynamic arrays.
- Look for nested loops `for (i..players.length) for (j..players.length)`.
- Look for External Calls Inside Loops and Think Like an Attacker ☠️.
- Run gas profiling: write tests with increasing array sizes and measure gas (`gasleft()` or Foundry gas-report).

---

## 2) Business Logic Edge Case — `getActivePlayerIndex` returning 0 ambiguity
**Problem:** function returns `0` if not found — but index `0` is valid. So caller/clients can't distinguish "player at index 0" vs "not found".

**Mitigation:**
- Return `(bool found, uint256 index)` or `uint256(-1)` for not found.
- Or document clearly.

**How to detect (audit-time checklist):**
- Search: missing/incorrect invariants, state enums not fully tested.
- Look for: functions that assume single-call flows, empty-array / zero-value handling.
- Test: fuzz 0, duplicates, ownership-transfer scenarios; add invariant tests (e.g., totalSupply == sum(balances)).

---

## 3) Re-entrancy — refund() executes external call before state change
**Why it’s bad:** external call to `msg.sender` before updating state allows attacker fallback to re-enter and call `refund` again.

**How to detect:**
- Look for external calls before state updates.
- Run `slither` or `aderyn` for reentrancy. [Slither and Aderyn can catch this vulnerability (though not always)!]

**Mitigations:**
- Apply CEI: update state before external calls.
- Use `ReentrancyGuard`.

**How to detect (audit-time checklist):**
- Search: external calls (call/transfer/send/token transfer) before state updates, external calls inside loops.
- Look for: payable withdraws, push patterns in loops.
- Test: add Attacker contract test (deposit -> reenter), run Foundry reentrancy checks, add ReentrancyGuard on withdraws.

---

## 4) Weak Randomness — predictable randomness sources
**Why it’s bad:** block/timestamp/difficulty/msg.sender are predictable/manipulable.

**Mitigations:**
- Use Chainlink VRF.
- Use commit-reveal.

**How to detect (audit-time checklist):**
- Search: use of block.timestamp / block.number / blockhash / tx.origin / msg.sender as sole seed.
- Look for: `random % players.length`, single-call draw.
- Test: simulate many draws (10k) to detect bias; mark for Chainlink VRF or commit-reveal.

---

## 5) Integer Overflow / Precision Loss
**Why it’s bad:** casting to smaller ints, arithmetic like `(players.length * entranceFee)` can overflow.

**Mitigations:**
- Use uint256 for monetary values.
- Avoid unchecked or small int casts.

**How to detect (audit-time checklist):**
- Search: arithmetic without checks, `unchecked` blocks, manual downcasts.
- Look for: balances/math used as indices or loop bounds.
- Test: boundary tests with 0, 1, maxUint, maxUint-1; assert reverts or no-wrap (Solidity>=0.8 assumed).

---

## 6) Unsafe Casting
**Why it’s bad:** downcasting loses data.

**Mitigations:**
- Avoid narrowing casts.
- Add precondition checks.

**How to detect (audit-time checklist):**
- Search: explicit casts (uint256->uint32, int<->uint), `address(uintX(...))`.
- Look for: storage using narrow types for amounts/ids.
- Test: downcast boundary tests (value > type.max) and add `require(value <= type.max)` before casts.

---

# Checklist to add in every audit for these vulns
- [ ] Run slither & review report.
- [ ] Run Aderyn & review report.
- [ ] Write targeted Foundry PoC tests.
- [ ] Search code for `call{value:...}`, loops, unsafe casts.
- [ ] Inspect coverage.
- [ ] More...

---

## Next steps (for PuppyRaffle)
1. Add DoS PoC & gas output.
2. Fix refund() with CEI and test again.
3. Replace weak randomness with VRF or commit-reveal.
4. Fix unsafe casts.
5. And continue the audit...

---