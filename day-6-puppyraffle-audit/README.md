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