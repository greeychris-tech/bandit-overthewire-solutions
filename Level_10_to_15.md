# Penetration Testing Lab Report: OverTheWire (Bandit Level 10 → 15)

## 1. Executive Summary

### 🕵️‍♂️ Engagement Overview
A secondary simulated internal penetration testing engagement was executed against the `Bandit` architecture hosted by the OverTheWire community. The primary directive was to advance the active threat footprint from the Level 10 baseline up to Level 15. Operations focused on advanced data decoding, reverse-engineering multi-layered file archives, and bypassing restrictive local loopback firewall configurations through cryptographic asset extraction.

### ⚠️ Tactical Risk Assessment
This phase of the security audit exposed several severe architectural and configuration vulnerabilities:
* **Weak Reversible Encoding Architectures:** Sensitive authentication credentials were obfuscated using easily reversible Base64 matrices and symmetrical character rotation (ROT13), providing zero cryptographic security against unprivileged users.
* **Complex Nested Data Packaging:** Critical systems packed binary artifacts inside deep, cyclical archive wrappers (Gzip, Bzip2, Tar). Programmatic magic-byte inspection successfully decapsulated all layers, proving that complex packaging does not equal defense-in-depth.
* **Insecure Private Key Storage & Perimeter Gaps:** Unsecured private SSH keys (`sshkey.private`) were left readable in the local workspace. While the host implemented strict local loopback (`localhost`) blocks and read-only directory barriers to prevent local lateral movement, these defenses were bypassed by exfiltrating the private key asset to an external workstation and pivoting back over the public domain interface.

### 🚀 Strategic Conclusion
The assessment successfully proved that perimeter-hardening techniques (like loopback port restrictions and write-locks) are entirely defeated if underlying cryptographic assets are left exposed. Horizontal privilege escalation was achieved across all designated nodes, terminating in a successful raw TCP network socket injection at Level 15. Strategic recommendations include applying strict file system ACLs on private cryptographic material and replacing static cleartext network daemons with encrypted, tokenized challenge-response mechanisms.

---

## 2. Target Identification & Source Reconnaissance

*   **Lab Provider:** OverTheWire Wargames Community
*   **Target Track:** Bandit (Linux Command Line & Security Fundamentals)
*   **Intelligence Source:** Official Portal ([OverTheWire Bandit Track](https://overthewire.org))
*   **Target Host IP/Domain:** `bandit.labs.overthewire.org`
*   **Target Destination Port:** `2220`
*   **Authentication Username:** `bandit10` through `bandit14`
*   **Plaintext Access Password:** `[REDACTED_PER_OPSEC_SANITIZATION]`

---

## 3. Infrastructure & Laboratory Pre-Requisites

| Infrastructure Layer | Specifications & Configurations |
| :--- | :--- |
| **Hardware Layer** | ASUS Vivobook Laptop (16GB RAM / 512GB SSD / Windows Host OS) |
| **Virtualization Layer** | Oracle VM VirtualBox Manager |
| **Guest Operating System** | Kali Linux Distribution (4GB Dedicated RAM / 2 CPU Cores) |
| **Network Infrastructure** | Mobile Phone Hotspot (Metered Connection / 20GB Cap) |

---

## 4. Laboratory Integration & Discovery Sequences

### 🛡️ Bandit Level 10 → Level 11: Reversible Data Encoding Reversal
* **Target File Discovery:** Environment triage identified an asset named `data.txt` (69 bytes). File properties and structural metrics indicated the presence of a Base64-encoded character vector.
* **Enumeration Command:** `ls -la`
* **Filtering & Path Resolution:** Invoked a native decoding engine to reverse the encoding matrix and strip the ASCII representation down to raw plaintext.
* **Payload Extraction Command:**
  ```bash
  base64 -d data.txt
  ```

### 🛡️ Bandit Level 11 → Level 12: Symmetrical Alphabet Rotation (ROT13) Cryptanalysis
* **Target File Discovery:** Located `data.txt` (49 bytes). Structural signatures and character patterns confirmed a symmetrical 13-character shift cipher (ROT13) was active.
* **Enumeration Command:** `ls -la`
* **Filtering & Path Resolution:** Rather than performing manual cryptanalysis, the stream was piped into the native translation (`tr`) utility to programmatically rotate uppercase and lowercase characters back by 13 positions, reconstructing the original ASCII baseline.
* **Payload Extraction Command:**
  ```bash
  cat data.txt | tr 'A-Za-z' 'N-ZA-Mn-za-m'
  ```

### 🛡️ Bandit Level 12 → Level 13: Automated Multi-Layered Magic-Byte Decapsulation
* **Target File Discovery:** Located `data.txt` (2,641 bytes). Due to its inflated size, running a raw text read threatened terminal stability. Diagnostic profiling was mandated.
* **Enumeration & Profiling Commands:** 
  ```bash
  file data.txt
  head -n 5 data.txt
  ```
* **Filtering & Deep Extraction Loop:** The asset was identified as an ASCII representation of a binary object (a hexdump). `head` profiling exposed the leading magic bytes `1f8b`, mapping directly to a Gzip compression signature. Because the home folder lacked file creation privileges, a sandbox workspace was established in `/tmp`. The forensic decapsulation loop was executed systematically across 8 distinct layers:
  1. *Hex Reversal:* Reconstructed the raw binary from the hexdump using `xxd -r data.txt /tmp/data2.gz`.
  2. *Gzip Layer 1:* Extracted via `gzip -d data2.gz` (resulting in `data2`, a Bzip2 file).
  3. *Bzip2 Layer 2:* Extracted via `bzip2 -d data2` (resulting in `data2.out`, a Gzip file).
  4. *Gzip Layer 3:* Extracted via `gzip -d data2.out` (resulting in `data2`, a Tar archive).
  5. *Tar Layer 4:* Unpacked via `tar -xvf data2` (extracting `data5.bin`, a Tar archive).
  6. *Tar Layer 5:* Unpacked via `tar -xvf data5.bin` (extracting `data6.bin`, a Bzip2 file).
  7. *Bzip2 Layer 6:* Extracted via `bzip2 -d data6.bin` (resulting in `data6`, a Tar archive).
  8. *Tar Layer 7:* Unpacked via `tar -xvf data6` (extracting `data8.bin`, a Gzip file).
  9. *Gzip Layer 8 (Final Node):* Extracted via `gzip -d data8.bin` to reveal a 49-byte plaintext ASCII asset.
* **Payload Extraction Command:**
  ```bash
  cat data8.bin
  ```

### 🛡️ Bandit Level 13 → Level 14: Cryptographic Key Exfiltration & Lateral Port Pivot
* **Target File Discovery:** Environment scanning exposed an unsecured, private SSH cryptographic asset named `sshkey.private` (2,602 bytes). Direct read access against the Level 14 system password repository was blocked by user privilege boundaries.
* **Enumeration Command:** `ls -la`
* **Filtering & Path Resolution:** The exposed private key presented an authentication bypass vector. Attempts to establish an internal local loopback connection (`ssh -i sshkey.private bandit14@localhost -p 2220`) were hard-locked by the infrastructure firewall to prevent resource exhaustion, and home directory write-locks prevented SSH fingerprint caching. To bypass these restrictions, the `sshkey.private` asset was exfiltrated directly to the local Kali Linux VM workstation, permissioned securely to `chmod 600`, and used to authenticate directly against the server's public perimeter interface.
* **Payload Extraction Command:**
  ```bash
  # Executed locally on Kali Linux VM after key exfiltration:
  chmod 600 kali_bandit14.key
  ssh -i kali_bandit14.key bandit14@bandit.labs.overthewire.org -p 2220
  cat /etc/bandit_pass/bandit14
  ```

### 🛡️ Bandit Level 14 → Level 15: Raw TCP Socket Credential Injection
* **Target File Discovery:** Target intelligence specified that the privilege escalation mechanism for this node shifted away from the local file system and was managed by a closed network daemon listening internally on port 30000.
* **Enumeration Command:** `netstat -ant` or network socket verification.
* **Filtering & Path Resolution:** Deployed Netcat (`nc`) to open a raw TCP streaming socket directly to `localhost` on port 30000. The newly exfiltrated Level 14 password token string was piped into the open network socket interface to satisfy the authentication daemon's validation challenge.
* **Payload Extraction Command:**
  ```bash
  echo "[REDACTED_L14_HASH]" | nc localhost 30000
  ```

---

## 5. Extracted Privilege Escalation Credentials

> [!TIP]
> **Bandit Level 15 Dynamic Session Established:** The remote gateway verified the network socket injection, returning a positive validation state (`Correct!`). Active investigation is paused at this terminal boundary baseline to preserve operational telemetry.

---

## 6. Closing Credential Sanitation Protocols

* Applied regular expression stream scrubbing to remove all raw 32-character plaintext authentication hashes and tokens from all active markdown documentation assets.
* Verified total operational security (OpSec) compliance before initializing the repository push sequence.
