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

2. **[H-01] Password Stored in Plaintext**  
   - Data is publicly visible on-chain.  
   - ✅ Fix: Store hash/encrypted ciphertext.

3. **[I-01] Incorrect NatSpec Docs in `getPassword`**  
   - Function docs include invalid `@param`.  
   - ✅ Fix: Correct NatSpec.  

---

### 📝 Findings Report
📂 Full write-up with descriptions, impact, PoCs & mitigations.  
👉 Full Findings Report: [PasswordStore Audit Report](https://github.com/0xrafi-kaji/smart-contract-security-portfolio/blob/main/2025-09-20_PasswordStore_report.pdf)

---

### How to generate a PDF audit report

1. Add all your findings to a markdown file like `report.md`
   1. Add the metadata you see at the top of that file
2. Install [pandoc](https://pandoc.org/installing.html) & [LaTeX](https://www.latex-project.org/get/)
   1. You might also have to install [one more package](https://github.com/Wandmalfarbe/pandoc-latex-template/issues/141) if you get `File 'footnotebackref.sty' not found.`
4. Download `eisvogel.latex` and add to your templates directory (should be `~/.pandoc/templates/`)
5. Add your logo to the directory as a pdf named `logo.pdf`
6. Run this command:
```
pandoc report.md -o report.pdf --from markdown --template=eisvogel --listings
```
---

### 🚀 Recap
- Finished my first real audit flow (Scoping → Recon → Vuln ID → Reporting).  
- Delivered a findings report with **3 vulnerabilities**.  
- Now comfortable with documenting bugs professionally.  

---
