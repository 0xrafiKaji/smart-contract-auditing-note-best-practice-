# Day 6 of Learning Smart Contract Auditing (Security Review) 🔍

Following @CyfrinUpdraft’s Smart Contract Auditing course 🚀

Today’s focus: **Starting a new audit → PuppyRaffle** 🎲🐶

---

### 🔑 Key Takeaways
- Moved on from **PasswordStore Audit** → starting **PuppyRaffle Audit**.  
- PuppyRaffle is larger than PasswordStore → **more bugs** expected (at least 4 HIGHs!).  
- Learned differences between **private audits** vs **competitive audits (CodeHawks)**.  
- CodeHawks “First Flights” = great platform for practicing live audits.  

---

### 📂 Scoping Phase
- Repo: [PuppyRaffle Audit](https://github.com/Cyfrin/4-puppy-raffle-audit)  
- Don’t peek into `audit-data` branch (answer key).  
- README already provides:  
  - About  
  - Setup (`make` → installs Foundry, OZ, base64)  
  - Scope (contracts in `src/`)  
  - Compatibilities (odd Solc version noted)  
  - Roles (Owner, Player)  
- Ran `forge coverage` → poor coverage = immature codebase → more opportunities for bugs.  

---

### 🛠️ Tools Introduced
- **Static Analysis**:  
  - [Slither](https://github.com/crytic/slither) (Python-based, checks for many vuln types)  
  - [Aderyn](https://github.com/Cyfrin/aderyn) (Rust-based AST traversal tool, outputs `report.md`)  
- **Solidity Metrics** / **Solidity Visual Developer** (VS Code extension) → inheritance graph, call graph, contract summary.  

---

### 📑 Recon / Documentation
- PuppyRaffle lets users **enter a raffle to win an NFT**.  
- Functions of interest:  
  - `enterRaffle(address[] newPlayers)`  
  - `refund(uint256 playerIndex)`  
  - `changeFeeAddress(address newFeeAddress)`  
- Noticed early issues:  
  - `entranceFee` immutable → poor naming convention.  
  - Duplicate check loop in `enterRaffle` smells like **DoS vulnerability**.  
  - `pragma solidity ^0.7.6;` (very outdated).  

---

### 🧪 First Exploit Exploration
- **Duplicate check loop → Denial of Service (DoS)**  
  - Gas usage grows with `players.length`.  
  - Eventually makes entering impractical.  
- Validated with minimized DoS contract + Foundry test (`DoSTest.t.sol`).  

---

### 🚀 Recap
- Finished PasswordStore yesterday → today started PuppyRaffle Audit.  
- Scoped repo, understood roles & architecture.  
- Learned & used static analysis tools (Slither, Aderyn, Solidity Metrics).  
- Found first potential bug: **DoS via duplicate check loop in enterRaffle**.  

---

---

# 🛑 How to Identify DoS (Denial of Service) Vulnerabilities in Smart Contracts

1. **Check for Loops (For / While)**
   - Loops that iterate over **dynamic arrays** (like `players`) are suspicious.  
   - Gas cost grows with array length → if unbounded, can block execution.  

   Example (PuppyRaffle):
   ```solidity
   for (uint256 i = 0; i < players.length; i++) {
       if (players[i] == newPlayer) revert DuplicateNotAllowed();
   }

🔍 **Red flag** → `players.length` can grow without limit.

---

2. **Look for External Calls Inside Loops**

   * If a loop performs storage writes or calls external contracts → **DoS risk increases**.
   * Attackers can force high gas cost or make loop execution fail.

---

3. **Gas Profiling**

   * Run `forge test --gas-report` or check manually with `forge snapshot`.
   * Track functions whose gas cost grows with input size.

---

4. **Think Like an Attacker**

   * “What if I add 1,000 players before the victim joins?”
   * Victim’s `enterRaffle()` call will **revert** due to exceeding block gas limit.

---

5. **Proof of Concept**

   * Write a Foundry test that simulates a long `players` array.
   * Example:

   ```solidity
   function test_DoS_on_enterRaffle() public {
       for (uint256 i = 0; i < 1000; i++) {
           players.push(address(uint160(i+1)));
       }
       vm.expectRevert();
       raffle.enterRaffle(address(0x123)); // should fail due to gas
   }
   ```

---

### ✅ Mitigation

* Use **mappings** or **sets** for duplicate checks → O(1) lookup instead of O(n) loop.
* Avoid unbounded loops inside critical functions.
* Use events instead of on-chain storage when possible.

---
