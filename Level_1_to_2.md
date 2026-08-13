# Penetration Testing Lab Report: OverTheWire (Bandit 1)

> ⚠️ **Note:** This level was documented using my Phase 1 legacy workflow (prior to engineering my split-screen logging system). 

## 1. Target Identification & Source Reconnaissance

*   **Lab Provider:** OverTheWire Wargames Community
*   **Target Track:** Bandit (Designed for fundamental Linux command line and security principles)
*   **Intelligence Source:** Evaluated the official portal ([OverTheWire](https://overthewire.org)) to extract connection requirements.
*   **Target Host IP/Domain:** `bandit.labs.overthewire.org`
*   **Target Destination Port:** `2220`
*   **Initial Low-Privilege Username:** `bandit1`
*   **Initial Plaintext Password:** `[REDACTED_LEVEL_1_SECRET_TOKEN]`

---

## 2. Infrastructure & Laboratory Pre-Requisites

| Infrastructure Layer | Specifications & Configurations |
| :--- | :--- |
| **Hardware Layer** | ASUS Vivobook Laptop (16GB RAM / 512GB SSD / Windows Host OS) |
| **Virtualization Layer** | Oracle VM VirtualBox Manager |
| **Guest Operating System** | Kali Linux (Allocated 4GB Dedicated RAM / 2 CPU Cores) |
| **Network Infrastructure** | Mobile Phone Hotspot (Configured as Metered Connection; 20GB monthly cap) |

---

## 3. Laboratory Integration & Discovery

### Environment Baseline Audit
Booted the Kali Linux Virtual Machine and initiated a standard desktop terminal session.
> [!NOTE]  
> **Telemetry & Documentation Methodology:** Terminal executions utilize sequential `echo` string formatting to force real-time thought-process tracking and inline commentary directly into the shell session. This logging approach serves as an audit trail for step-by-step verification within isolated target environments.

### Network Execution & Shell Access Sequence
1. **Initiate SSH Connection:** Fired up the native Secure Shell utility:
   ```bash
   ssh bandit1@bandit.labs.overthewire.org -p 2220
   ```
2. **Cryptographic Challenge:** Received the remote host's cryptographic authentication challenge. 
3. **Authentication:** Submitted the plaintext initialization credential (` [REDACTED_LEVEL_1_SECRET_TOKEN] `) at the remote system password prompt.
4. **Access Granted:** Successfully authenticated past the perimeter firewall, landing on the remote server's active text shell environment: `bandit1@bandit~$`.

### Data Extraction Sequence

5. **Directory Inspection:** Executed the `ls` command on the target server filesystem, identifying a target file asset titled `-`.
6. **Payload Extraction:** Executed the shell command string to extract the plaintext payload string hidden within the file layer:
   ```bash
   cat ./-
   ```

---

## 4. Extracted Privilege Escalation Credential

> [!KEY]
> **Bandit Level 2 Access Password:** `[REDACTED_LEVEL_2_SECRET_TOKEN]`

---

## 5. Closing Credential Sanitation Protocols

*   Applied media sanitation protocols to visually redact all plaintext passwords from screenshots.
*   Verified sanitization compliance prior to uploading assets to the GitHub repository.
