# Day 3 of Learning Smart Contract Auditing (Security Review) 🔍

Following @CyfrinUpdraft’s Smart Contract Auditing course 🚀

Today’s focus: Recon → Understanding Code, Exploits, and Writing Findings

---

### 🔑 Key Takeaways
- First step: **Understand the codebase** (functions + architecture) before hunting bugs.  
- Take organized notes (`.notes.md`, inline `// Q`, `// I`, `// @Audit`)  
- Tools: `cloc`, Solidity Metrics, Tincho method → small → large  
- NatSpec comments = valuable documentation, but must be accurate!  
- Always ask: *Does the implementation match the intent?*

📂 Full codebase: [PasswordStore Audit Repo (onboarded branch)](https://github.com/Cyfrin/3-passwordstore-audit/tree/onboarded)

---

### 📝 Writing Findings
Report Structure:  
- **Title** = Root Cause + Impact  
- **Description** = clear explanation w/ references (`PasswordStore::s_password`)  
- **Impact** = plain language effect  
- **Proof of Concept** = code or `cast` commands  
- **Recommended Mitigation** = concrete solution  

---

### 🐞 Findings & Exploits

#### [H-01] Missing Access Control on `setPassword`
**Description:**  
The function `setPassword(string memory newPassword)` has **no access control**.  
Anyone can overwrite the stored password. NatSpec incorrectly claims "only the owner" can call it, but the check is missing.  

**Impact:**  
Any random address can change the password, breaking contract integrity and rendering the system unusable.  

**Proof of Concept (PoC):**
```solidity
function test_anyone_can_set_password(address randomUser) public {
    vm.assume(randomUser != owner);
    vm.prank(randomUser);
    passwordStore.setPassword("hacked");
    assertEq(passwordStore.getPassword(), "hacked");
}
```

**Recommended Mitigation:**  
Add explicit ownership check:
```solidity
if (msg.sender != s_owner) {
    revert PasswordStore__NotOwner();
}
```

---

#### [H-02] Password Stored in Plaintext On-Chain
**Description:**  
The contract stores `s_password` directly in storage. Ethereum storage is public, so anyone can read the password directly without calling `getPassword()`.  

**Impact:**  
Exposes confidential information. Anyone can recover the plaintext password using on-chain data.  

**Proof of Concept (PoC):**
```bash
cast storage <PASSWORD_STORE_ADDRESS> 1
cast parse-bytes32-string <hex_value>
# Output reveals actual password
```

**Recommended Mitigation:**  
- Do not store sensitive plaintext data on-chain.  
- Store hashed/encrypted values only.  
- Consider client-side encryption and on-chain ciphertext storage.

---

#### [I-01] Incorrect NatSpec Documentation in `getPassword`
**Description:**  
The NatSpec for `getPassword()` includes a `@param` tag for `newPassword`, but the function takes no parameters.  

**Impact:**  
Causes confusion for developers and auditors. Documentation mismatch makes audits harder and may mislead integrators.  

**Proof of Concept (PoC):**
```solidity
/*
 * @notice This allows only the owner to retrieve the password.
 * @param newPassword The new password to set.   <-- incorrect
 */
function getPassword() external view returns (string memory) {
    return s_password;
}
```

**Recommended Mitigation:**  
Remove or correct NatSpec comments so they align with the actual function signature.

---

### 🧪 Protocol Tests
- Ran `forge build`, `forge test`, `forge coverage`  
- Coverage can be misleading (e.g., critical access control bug missed!)  
- Added fuzz test: `test_anyone_can_set_password(address randomUser)`  

---

### 🚀 Recap
- Recon isn’t bug hunting → it’s about **understanding intent vs implementation**.  
- Documented 3 vulnerabilities today:  
  1. Missing Access Control (High)  
  2. Public Data Exposure (High)  
  3. NatSpec Documentation Bug (Info)  
- Learned how to structure a **proper Findings Report**.  

---

## 📌 Foundry cast Cheatsheet (extra )

### ✅ String → Bytes32

```bash
cast --to-bytes32 "Rafi"
```

👉 Output: `0x5261666900000000000000000000000000000000000000000000000000000000`

---

### ✅ Bytes32 → String

```bash
cast parse-bytes32-string 0x5261666900000000000000000000000000000000000000000000000000000000
```

👉 Output: `Rafi`

---

### ✅ String → Hex (dynamic length, not fixed 32 bytes)

```bash
cast --to-hex "Rafi"
```

👉 Output: `0x52616669`

---

### ✅ Hex → String

```bash
cast --from-utf8 0x52616669
```

👉 Output: `Rafi`

---

### ✅ Bytes → Hex

```bash
cast --to-hex "Hello World"
```

👉 Output: `0x48656c6c6f20576f726c64`

---

### ✅ Hex → Bytes (Human-readable string)

```bash
cast --from-utf8 0x48656c6c6f20576f726c64
```

👉 Output: `Hello World`

---

📒 **Remember**:

* `--to-bytes32` → always padding **fixed 32 bytes**.
* `--to-hex` / `--from-utf8` → for dynamic string encoding/decoding.
* `parse-bytes32-string` → only for **bytes32 → string**.

---

