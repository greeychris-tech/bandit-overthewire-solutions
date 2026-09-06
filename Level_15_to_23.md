# Penetration Testing Lab Report: OverTheWire (Bandit Level 15 → 23)

## 1. Executive Summary

### 🕵️‍♂️ Engagement Overview
A tertiary simulated internal penetration testing engagement was executed against the **Bandit architecture** hosted by the **OverTheWire community**. The primary directive was to advance the active threat footprint from the **Level 15 baseline up to Level 23**. Operations focused on validating secure SSL/TLS cryptographic communication layers, mapping local network services, bypassing loopback network firewalls via external asset exfiltration, evading interactive shell restrictions, abusing SetUID binary configurations, and leveraging vulnerabilities within automated infrastructure scheduling systems.

### ⚠️ Tactical Risk Assessment
This phase of the security audit exposed several critical architectural, configuration, and privilege isolation vulnerabilities:

* **Internal Infrastructure Information Masking & Weak Transport Isolation:** Network services relied on undocumented bindings across the `31000-32000` port range. Some systems used insecure local SSL endpoints that output raw, unencrypted private keys to unprivileged local accounts upon a successful network handshake.
* **Deficient Interactive Shell Control Filters:** Anti-tamper controls designed to restrict interactive console access via custom environmental profiles (`.bashrc` / `.profile`) were entirely bypassed. Passing direct execution strings to the secure shell daemon allowed execution paths to initialize before the profile logout script could fire.
* **Loosely Applied SetUID Privileges:** Multiple binary utilities (`bandit20-do`, `suconnect`) were configured with active SetUID bits owned by higher-tier accounts. These binaries lacked robust argument validation or environment isolation, creating direct bridges for arbitrary local file system disclosure and inter-process network race condition exploits.
* **Insecure Shared Workspaces & Deterministic Automation Scripts:** Critical background infrastructure scripts driven by the Linux Cron daemon leaked sensitive information into globally readable directories (`/tmp`). Furthermore, scripts relied on deterministic, un-salted cryptographic hashing functions (`md5sum`) to obscure target file paths, allowing an analyst to easily replicate the calculations and uncover hidden assets.

### 🚀 Strategic Conclusion
The assessment proved that privilege separation boundaries (such as restrictive shells, localized firewalls, and cryptographic path hashing) are fundamentally compromised when system binaries are granted loose SUID permissions or when background automation scripts leak operational outputs into public directories. Horizontal privilege escalation was successfully completed across all nodes, culminating in a programmatic target path reconstruction at Level 23. Strategic recommendations include restricting SetUID execution permissions, auditing all automated scripts for data leakage, and migrating system automation routines away from public shared directories like `/tmp`.

---

## 2. Target Identification & Source Reconnaissance

* **Lab Provider:** OverTheWire Wargames Community
* **Target Track:** Bandit (Linux Command Line & Security Fundamentals)
* **Intelligence Source:** Official Portal (OverTheWire Bandit Track)
* **Target Host IP/Domain:** `bandit.labs.overthewire.org`
* **Target Destination Port:** `2220`
* **Authentication Username:** `bandit15` through `bandit22`
* **Plaintext Access Password:** `[SECURE_TOKEN_REDACTED]`

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

### 🛡️ Bandit Level 15 → Level 16: Encrypted TLS Network Socket Credential Injection
* **Target File Discovery:** Environment analysis shifted focus from disk assets to a localized SSL/TLS service listening internally on port `30001`. Kernel-level visibility limits masked direct socket enumeration (`ss -tl`), necessitating active protocol probing.
* **Enumeration Command:** `ss -tl | grep 30001`
* **Filtering & Path Resolution:** Initialized an explicit SSL/TLS handshake with the OpenSSL engine. The client successfully parsed the self-signed `SnakeOil` certificate error and passed the active Level 14 system token into the secure channel to satisfy the application challenge.
* **Payload Extraction Command:**
```bash
echo "[SECURE_TOKEN_REDACTED]" | openssl s_client -connect localhost:30001 -quiet
```

### 🛡️ Bandit Level 16 → Level 17: Local Port Scanning & Private RSA Key Harvesting
* **Target File Discovery:** Directory triage exposed only baseline POSIX files and the legacy password. Threat intelligence indicated an active authentication service bound to an anonymous port within the `31000-32000` TCP range.
* **Enumeration & Profiling Commands:** 
```bash
nmap -sV -p 31000-32000 localhost
```
* **Filtering & Deep Extraction Loop:** The network exploration scan mapped five open sockets. Sequential cryptographic profiling pinpointed port `31790` as the target gateway, which accepted the Level 15 token and dumped a raw RSA Private Key block structure.
* **Payload Extraction Command:**
```bash
# Key exfiltrated locally to Kali Linux due to remote filesystem write restrictions
chmod 600 ~/kali_bandit17.key
ssh -i ~/kali_bandit17.key -o PreferredAuthentications=publickey -o UserKnownHostsFile=/dev/null -o StrictHostKeyChecking=no bandit17@bandit.labs.overthewire.org -p 2220
```

### 🛡️ Bandit Level 17 → Level 18: Cryptographic File Diffing and Extraction
* **Target File Discovery:** Environment triage identified two major structural artifacts within the user space: `passwords.old` and `passwords.new` (both 3300 bytes).
* **Enumeration Command:** `ls -la`
* **Filtering & Path Resolution:** The native `diff` utility was invoked to compare line entries between the files, programmatically bypassing identical string buffers to isolate the modified password vector at line entry 42.
* **Payload Extraction Command:**
```bash
diff passwords.old passwords.new
```

### 🛡️ Bandit Level 18 → Level 19: SSH Shell Profile Disruption and Terminal Escape
* **Target File Discovery:** Direct remote authentication attempts triggered an instant automated logout routine driven by a modified `.bashrc` template.
* **Enumeration Command:** `ssh bandit18@bandit.labs.overthewire.org -p 2220`
* **Filtering & Path Resolution:** To circumvent the interactive profile kill switch, a non-interactive SSH execution payload was deployed, forcing the daemon to read the target `readme` asset before the environment initialized its shell constraints.
* **Payload Extraction Command:**
```bash
ssh bandit18@bandit.labs.overthewire.org -p 2220 cat readme
```

### 🛡️ Bandit Level 19 → Level 20: SUID Executable Privilege Abuse
* **Target File Discovery:** Environment evaluation exposed an explicit 14880-byte binary named `bandit20-do` featuring an active SetUID bit (`-rwsr-x---`) owned by `bandit20`.
* **Enumeration Command:** `ls -la`
* **Filtering & Path Resolution:** After running the binary without parameters to verify the syntax, the executable context was tested via `whoami` to confirm kernel privilege escalation. A file exfiltration command argument was appended to bypass local Discretionary Access Control (DAC) boundaries.
* **Payload Extraction Command:**
```bash
./bandit20-do whoami
./bandit20-do cat /etc/bandit_pass/bandit20
```

### 🛡️ Bandit Level 20 → Level 21: Inter-Process Network Socket Synchronization
* **Target File Discovery:** Triage isolated an explicit SUID binary named `suconnect` (`-rwsr-x---`), owned by user `bandit21`.
* **Enumeration Command:** `./suconnect`
* **Filtering & Path Resolution:** The binary parameters required connecting to a local port to validate the active tier password. A background Netcat thread was initialized to host the password stream, and a standalone client command was executed to overcome an initial socket binding race condition.
* **Payload Extraction Command:**
```bash
echo "[SECURE_TOKEN_REDACTED]" | nc -l -p 4545 &
./suconnect 4545
```

### 🛡️ Bandit Level 21 → Level 22: Scheduled Task Automation Tracking & Exploit
* **Target File Discovery:** The filesystem lacked direct exploit wrappers. Trajectory shifted to the global scheduled task repository path `/etc/cron.d/` to audit background daemon routines.
* **Enumeration Command:** `ls -la /etc/cron.d/`
* **Filtering & Path Resolution:** Source analysis of `cronjob_bandit22` exposed a script (`/usr/bin/cronjob_bandit22.sh`) running every minute. Code inspection revealed a data leak vulnerability where the privileged token was being systematically written to a globally readable file inside `/tmp/`.
* **Payload Extraction Command:**
```bash
cat /etc/cron.d/cronjob_bandit22
cat /usr/bin/cronjob_bandit22.sh
cat /tmp/[SECURE_TOKEN_REDACTED]
```

### 🛡️ Bandit Level 22 → Level 23: Deterministic Obfuscation Routine Reversals
* **Target File Discovery:** Automated process analysis tracking led to the inspection of the next logical cron configuration asset, `cronjob_bandit23`.
* **Enumeration Command:** `cat /etc/cron.d/cronjob_bandit23`
* **Filtering & Path Resolution:** Dissection of `/usr/bin/cronjob_bandit23.sh` revealed a variable-driven tracking algorithm using an un-salted MD5 hash loop. The exact string manipulation syntax was programmatically simulated locally to calculate the target destination file descriptor within the public `/tmp/` tree.
* **Payload Extraction Command:**
```bash
cat /usr/bin/cronjob_bandit23.sh
# Simulating target hash calculation:
echo "I am user bandit23" | md5sum | cut -d ' ' -f 1
cat /tmp/[SECURE_TOKEN_REDACTED]
```

---

## 5. Extracted Privilege Escalation Credentials

NOTE: The linear chain of custody from Level 15 through Level 23 has been completely mapped, validated, and officially closed. The exfiltrated private key asset (~/kali_bandit17.key) serves as the singular proof-of-work baseline for the Level 16 → 17 transition.

---

## 6. Closing Credential Sanitation Protocols

Applied regular expression stream scrubbing (`sed`) to programmatically clean active logs, converting live forensic tracking metrics into a secure, sanitized report suitable for public distribution.
