# Day 4 of Learning Smart Contract Auditing (Security Review) 🔍

Today’s focus: Wrapping up the **PasswordStore Audit** + Writing Findings Report  

---

### 🔑 Key Takeaways
- Completed my first **end-to-end audit** of a protocol 🎉  
- Learned to **structure & write findings reports** clearly  
- Importance of:  
  - Root Cause + Impact in title  
  - Proof of Concept (tests / cast cmds)  
  - Recommended Mitigation (specific, actionable)  
- Reports ≠ just notes → they must be **communicable to devs**  

---

### 🐞 Final Findings (PasswordStore)
1. **[H-01] Missing Access Control in `setPassword`**  
   - Anyone can overwrite the password.  
   - ✅ Fix: Add ownership check.

2. **[M-01] Password Stored in Plaintext**  
   - Data is publicly visible on-chain.  
   - ✅ Fix: Store hash/encrypted ciphertext.

3. **[L-01] Incorrect NatSpec Docs in `getPassword`**  
   - Function docs include invalid `@param`.  
   - ✅ Fix: Correct NatSpec.  

---

### 📝 Findings Report
📂 Full write-up with descriptions, impact, PoCs & mitigations.  
👉 Full Findings Report: [PasswordStore Audit (Onboarded)](https://github.com/0xrafi-kaji/smart-contract-security-portfolio/blob/main/2025-09-20_PasswordStore_report.pdf)

---

### 🚀 Recap
- Finished my first real audit flow (Scoping → Recon → Vuln ID → Reporting).  
- Delivered a findings report with **3 vulnerabilities**.  
- Now comfortable with documenting bugs professionally.  

---