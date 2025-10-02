# Day 9–11 of Learning Smart Contract Auditing - (PuppyRaffle Audit) Final Findings & Best Practices

**Repo (course):** [Cyfrin PuppyRaffle Audit](https://github.com/Cyfrin/4-puppy-raffle-audit)  
**Final Report (my portfolio):** [2025-09-30_PuppyRaffle_report.pdf](https://github.com/0xrafiKaji/smart-contract-security-portfolio/blob/main/2025-09-30_PuppyRaffle_report.pdf)

---

## 📝 Recap
- Completed full audit of `PuppyRaffle` protocol.
- Documented findings in structured format (Title → Description → Impact → PoC → Recommendation).
- Compiled into a final professional-style report and published to portfolio repo.

---

## 🔑 Key Findings
### 1. Denial of Service (DoS) in `enterRaffle`
- **Root cause:** Nested duplicate-check loops → O(n²) gas growth.
- **Impact:** Later entrants can be blocked or face excessive gas cost.
- **PoC:** Foundry test with increasing player array size.
- **Mitigation:** Use O(1) mapping-based checks or redesign business logic.

### 2. Business Logic Edge Case in `getActivePlayerIndex`
- **Root cause:** Returns `0` on “not found”, but `0` can be a valid index.
- **Impact:** Ambiguity leads to potential refund / logic errors.
- **Mitigation:** Return `(bool found, uint256 index)` or `type(uint256).max`.

### 3. Reentrancy in `refund()`
- **Root cause:** External call before state change (CEI violation).
- **Impact:** Attacker fallback can drain contract.
- **PoC:** Attacker contract + Foundry test.
- **Mitigation:** CEI (Checks-Effects-Interactions) pattern, `ReentrancyGuard`.

### 4. Weak Randomness in `selectWinner`
- **Root cause:** Uses predictable on-chain variables (`msg.sender`, `block.timestamp`, etc).
- **Impact:** Outcome can be biased by miners/callers.
- **Mitigation:** Use Chainlink VRF or commit-reveal scheme.

### 5. Integer Overflow & Unsafe Casting
- **Root cause:** Casting monetary values to `uint64`, potential precision/overflow issues.
- **Impact:** Silent truncation or miscalculated balances.
- **Mitigation:** Keep values in `uint256`, assert before downcast.

---

## 📌 Best Practices (for future audits)
- **Recon:** Start with scoping → docs → architecture notes.  
- **Findings:** Always write reports in structured format: Title, Description, Impact, PoC, Mitigation.  
- **Testing:** Use Foundry for fuzz tests & gas profiling; Slither for static analysis.  
- **Report:** Export findings to PDF & keep both raw `.md` + final report in repo.  
- **Checklist:**  
  - [ ] Search for nested loops on dynamic arrays (DoS)  
  - [ ] Validate access controls  
  - [ ] Check external calls for CEI/reentrancy  
  - [ ] Inspect randomness sources  
  - [ ] Check unsafe casting / integer math  
  - [ ] Match implementation with NatSpec & docs  

---

✅ **Audit completed:** PuppyRaffle.  
📂 Notes here will serve as a reusable template for future smart contract audits.
