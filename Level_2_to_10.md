# Penetration Testing Lab Report: OverTheWire (Bandit Level 2 → 10)

## 1. Executive Summary

### 🕵️‍♂️ Engagement Overview
A simulated internal penetration testing engagement was executed against the `Bandit` architecture hosted by the OverTheWire community. The objective of this assessment was to perform a white-box security audit of the environment's Linux filesystem, network configurations, and access control models. Operations successfully scaled from initial unprivileged access (Level 2) to systematic horizontal and vertical privilege escalation, terminating at the Level 10 boundary node.

### ⚠️ Tactical Risk Assessment
The assessment identified several critical architectural and configuration weaknesses across the target infrastructure:
* **Improper Object-Level Access Controls:** Sensitive privilege tokens were left exposed to unprivileged users via hidden directories and global root-level paths (`/var/`), violating the Principle of Least Privilege (PoLP).
* **Insecure Flag and Input Abstraction:** System utilities failed to properly sanitize special characters (such as leading dashes `--`), allowing boundary-bypass techniques to expose password artifacts.
* **Weak Cryptographic Obfuscation:** The infrastructure relied heavily on security-through-obscurity methodologies, utilizing easily reversible ASCII string strings inside raw binary streams to mask sensitive credential information.

### 🚀 Strategic Conclusion
While the target environment successfully implemented multi-layered boundary defenses, programmatic filesystem enumeration and data-type profiling successfully defeated all operational barriers. Full host compromise was achieved sequentially across eight nodes. Immediate strategic recommendations include restricting global read permissions on system artifact trees and enforcing robust data-at-rest encryption.

---

## 2. Target Identification & Source Reconnaissance

*   **Lab Provider:** OverTheWire Wargames Community
*   **Target Track:** Bandit (Linux Command Line & Security Fundamentals)
*   **Intelligence Source:** Official Portal ([OverTheWire Bandit Track](https://overthewire.org))
*   **Target Host IP/Domain:** `bandit.labs.overthewire.org`
*   **Target Destination Port:** `2220`
*   **Authentication Username:** `bandit2` through `bandit9`
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

## 4. Engineering Workflows & Telemetry Separation

During initial setup, standard logging techniques like tracking native shell history directly (`tail -f ~/.bash_history`) failed due to in-memory buffering bottlenecks. Attempting local command logging inside the remote SSH target environment also failed due to context/permission mismatches.

To resolve this, a **3-Pane Tilix Architecture** was engineered to separate gameplay traffic from telemetry capture:
1. **Pane 1 (Remote Context):** Active Bandit SSH Session (Target Gameplay Only).
2. **Pane 2 (Local Context):** Local VM Shell running a cleared `unalias note` custom logging function block.
3. **Pane 3 (Analysis Stream):** Real-time monitoring via `tail -f ~/notes.txt` to capture operational notes flawlessly.

---

## 5. Laboratory Integration & Discovery Sequences

### 🛡️ Bandit Level 2 → Level 3: Argument/Flag Abstraction Bypass
* **Target File Discovery:** Visual inspection via `ls -la` revealed a file named `--spaces in this filename--`. The double leading dash (`--`) caused standard utilities to misinterpret the filename as a command flag/option switch, breaking tab-autocomplete and basic reads.
* **Enumeration Command:** `ls -la`
* **Filtering & Path Resolution:** Prepend an explicit relative path reference `./` to force the shell to handle the dashes as a literal file path instead of an operational flag.
* **Payload Extraction Command:** 
  ```bash
  cat "./--spaces in this filename--"
  ```

### 🛡️ Bandit Level 3 → Level 4: Hidden Directory Traversal
* **Target File Discovery:** Explored the workspace using recursive flags and uncovered a directory structure holding a hidden artifact named `...Hiding-From-You`. The three-dot (`...`) naming convention was an obfuscation technique designed to blend with native system navigation pointers (`.` and `..`).
* **Enumeration Command:** `ls -la`
* **Filtering & Path Resolution:** Identified the unique multi-dot hidden path string.
* **Payload Extraction Command:**
  ```bash
  cat "...Hiding-From-You"
  ```

### 🛡️ Bandit Level 4 → Level 5: Programmatic Data-Type Profiling
* **Target File Discovery:** The `inhere` working directory contained ten identically looking files (`-file00` through `-file09`). To avoid opening non-text or binary files manually, data profiling was required to find human-readable text.
* **Enumeration Command:** `ls`
* **Filtering & Path Resolution:** Deployed the `file` utility alongside a wildcard operator (`./*`) to inspect and determine the underlying data profiles of all assets simultaneously. It isolated `./-file07` as the only `ASCII text` block.
* **Payload Extraction Command:**
  ```bash
  file ./*
  cat ./-file07
  ```

### 🛡️ Bandit Level 5 → Level 6: Automated Multi-Parameter Filesystem Filtering
* **Target File Discovery:** The target area contained a deeply nested framework of 19 subdirectories. Sifting through this manually was highly inefficient, necessitating automated criteria filtering.
* **Enumeration Command:** `ls -la`
* **Filtering & Path Resolution:** Executed a scoped search using the `find` engine, filtering explicitly for regular files (`-type f`) measuring exactly 1033 bytes (`-size 1033c`). This programmatically isolated the hidden path: `inhere/maybehere07/.file2`.
* **Payload Extraction Command:**
  ```bash
  find inhere/ -type f -size 1033c
  cat inhere/maybehere07/.file2
  ```

### 🛡️ Bandit Level 6 → Level 7: Global Root-Level Privilege Search
* **Target File Discovery:** Running `ls -la` in the local home directory yielded no level assets. The target artifact was stashed globally across the root server filesystem under strict User/Group ownership permissions.
* **Enumeration Command:** `ls -la`
* **Filtering & Path Resolution:** Executed a global query starting from root (`/`) looking for assets matching user `bandit7`, group `bandit6`, and size `33c`. To maintain clear console visibility, standard error streams were redirected. Restricting search operations down to the mutable system branch (`/var/`) bypassed server load limitations and successfully pinpointed `/var/lib/dpkg/info/bandit7.password`.
* **Payload Extraction Command:**
  ```bash
  find / -user bandit7 -group bandit6 -size 33c 2>/dev/null
  cat /var/lib/dpkg/info/bandit7.password
  ```

### 🛡️ Bandit Level 7 → Level 8: High-Density Keyword Filtering
* **Target File Discovery:** Encountered a massive flat-file data repository named `data.txt`. Attempting to read it completely would result in terminal buffer overflow.
* **Enumeration Command:** `ls -l`
* **Filtering & Path Resolution:** Utilized a text pattern matcher to parse the flat-file and target the precise line containing a known signature keyword.
* **Payload Extraction Command:**
  ```bash
  grep "millionth" data.txt
  ```

### 🛡️ Bandit Level 8 → Level 9: Structured Stream Deduplication
* **Target File Discovery:** The `data.txt` repository contained thousands of heavily duplicated text entries. The target privilege escalation credential was explicitly defined as the only string occurring exactly once.
* **Enumeration Command:** `ls -l`
* **Filtering & Path Resolution:** Because the `uniq` command requires identical lines to be natively adjacent to process them, a command pipeline was built. The file stream was alphabetized first before running unique deduplication flags (`-u`).
* **Payload Extraction Command:**
  ```bash
  sort data.txt | uniq -u
  ```

### 🛡️ Bandit Level 9 → Level 10: Binary Data Striping & Regex Validation
* **Target File Discovery:** The `data.txt` file was deployed as a raw binary blob. Opening it natively would corrupt standard terminal text layouts and wrap human-readable strings inside unreadable machine code junk.
* **Enumeration Command:** `file data.txt`
* **Filtering & Path Resolution:** Processed the binary file using a utility that strips out non-printable elements to isolate ASCII character sequences. The clean text output stream was then piped directly into a regular expression filter looking for consecutive equal sign (`==`) markers.
* **Payload Extraction Command:**
  ```bash
  strings data.txt | grep "=="
  ```

---

## 6. Extracted Privilege Escalation Credentials

> [!TIP]
> **Bandit Level 10 Access Verified:** Connection established successfully using the Level 10 authentication credential token. Active gameplay paused at this boundary node to conduct administrative data mapping.

---

## 7. Closing Credential Sanitation Protocols

* Applied media sanitation protocols to visually redact plaintext password hashes and tokens from all active logs and markdown documentation strings.

