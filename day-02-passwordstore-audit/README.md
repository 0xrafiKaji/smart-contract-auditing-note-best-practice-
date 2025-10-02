# Day 2 of Learning Smart Contract Auditing (Security Review) 🔍

Following @CyfrinUpdraft’s Smart Contract Auditing course 🚀

My first audit (security review) | PasswordStore Audit


### 🔗 Security Review Code References
- **V1**: [Etherscan link](https://sepolia.etherscan.io/address/0x2ecf6ad327776bf966893c96efb24c9747f6694b)  
- **V2**: [GitHub minimal onboarding](https://github.com/Cyfrin/3-passwordstore-audit)  
- **V3**: [Onboarded branch](https://github.com/Cyfrin/3-passwordstore-audit/tree/onboarded)  
- **Final Audit**: [Audit Data](https://github.com/Cyfrin/3-passwordstore-audit/tree/audit-data)  

---

### 🔑 Key Phases (Recap)
**Initial Review**
1. Scoping  
2. Reconnaissance  
3. Vulnerability Identification  
4. Reporting  

**Skipped in demo**
- Protocol Fixes  
- Mitigation Review  

---

### 🛠 The Setup (Scoping)
#### V1
- "Hey, here is my link to Etherscan, can I get an audit?"  
  - [Coinbase asset listing guide](https://www.coinbase.com/blog/a-guide-to-listing-assets-on-coinbase)

#### V2
- Client onboarding: Minimal  

#### V3
- `cloc` (Count Lines of Code)
- Cloc >>> csv file for notion:
```
cloc . --by-file --csv --quiet | awk 'NR==1{print "language,filename,blank,comment,code"; next} {print}' > scope.csv
```
---

### 📚 The Tincho Method
- Read docs 📖  
- Note taking in-code 📝  
- Small → Large approach  
- Tooling: [Solidity Metrics](https://github.com/Consensys/solidity-metrics)  
- Example: [Tincho’s ENS Review (YouTube)](https://www.youtube.com/watch?app=desktop&v=A-T9F0anN1E)  
