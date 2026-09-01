# Bandit Over The Wire (BOTW) Log File

This is the log file for BOTW containing notes, commands used, and the underlying thought process.

*   **Current Objective:** Creating a better workflow and showcasing the commands or codes used to optimize the process through trial and error.

---

## 🛠️ Architecture & Workflow Setup

### Trial 1: Native Shell History Tracking
*   **Command:** `tail -f ~/.bash_history | grep --line-buffered "^#"`
*   **Result:** ❌ **FAILED**
*   **Reason:** Bash history buffers in memory; notes were delayed or glitched.

### Trial 2: Dedicated File Stream
*   **Command:** `echo "# note" >> ~/notes.txt`
*   **Result:**  **SUCCESS** (with caveats)
*   **Reason:** The stream worked, but formatting was too tight and single-line aliases glitched on special characters.

### Trial 3: Multi-Line Function Definition
*   **Command:** 
    ```bash
    note() { printf ... }
    ```
*   **Result:** ❌ **FAILED**
*   **Reason:** Threw a `parse error near ()` because an old alias named `note` was conflicting with the new function definition.

### Final Solution: Memory Clearance & Custom Function
*   **Command:** `unalias note && note() { printf ... }`
*   **Result:**  **SUCCESS**
*   **Reason:** Cleared the terminal memory using `unalias` before creating the function, removing the conflict. Used a multi-line bash function for clean spacing and automated dashed lines.

### Trial 4: Local Function via SSH
*   **Command:** `note [message]` *(while logged into bandit@bandit.labs.overthewire.org)*
*   **Result:** ❌ **FAILED**
*   **Reason:** Threw `Permission denied`. The `note` function and `notes.txt` reside on the local VM. Once SSH connects, commands execute on the remote server instead.

### Final Solution: 3-Terminal Tilix Architecture
*   **Setup:**
    *   **Terminal 1 (Left/Top):** Active Bandit SSH Session (Gameplay Only)
    *   **Terminal 2 (Left/Bottom):** Local VM Shell (For typing `note [message]`)
    *   **Terminal 3 (Right):** Live Output Stream (`tail -f ~/notes.txt`)
*   **Result:**  **SUCCESS**
*   **Reason:** Achieved complete separation of game input and note input.

### Trial 5 & 7: Package Manager Interception
*   **System Prompt:** `Command 'note' not found, but can be installed with...`
*   **Action:** Selected 'No'.
*   **Reason:** The Linux package manager intercepted the missing command, assuming it was a known external program rather than recognizing it as our custom local script.
*   **Final Solution:** Declared the local shell function to override the package manager hook. The shell now routes the word `note` to our text file instead of searching the web for an installer.

---

## 🎮 Gameplay Log & Level Progression

### 🔹 Bandit Level 2 → Level 3
*   **Context:** Logged into Level 2 successfully. Ran `ls` and found a file named `spaces in this filename`.
*   **Trial 9:** Tried `cat "spaces in this filename"`
    *   *Result:* ❌ **FAILED** (System could not read or locate the file using manual quotes due to hidden formatting characters).
*   **Trial 10:** Attempted Tab autocomplete with `cat spaces`
    *   *Result:* ❌ **FAILED** (`No such file or directory`; Tab key did nothing because the shell found no matching starting characters).
*   **Discovery:** Ran location discovery commands (`pwd` and `ls -la`). Visual inspection revealed the file is actually named `--spaces in this filename--` (with leading/trailing dashes). The double leading dash (`--`) causes `cat` to misinterpret the filename as an option switch.
*   **Solution:** Prepend the absolute current path specifier `./`.
    ```bash
    cat "./--spaces in this filename--"
    ```
*   **Outcome:**  **SUCCESS**. Level 3 password successfully revealed.

### 🔹 Bandit Level 3 → Level 4
*   **Context:** Authenticated into Bandit Level 3.
*   **Discovery:** Running `ls -la` in the home directory revealed a hidden space directory structure. Navigated inside and located a hidden file explicitly named `...Hiding-From-You`. The three dots (`...`) are a deliberate obfuscation technique used to mimic system navigation pointers (`.` and `..`).
*   **Solution:** Wrap the multi-dot filename in quotes to safely print the text string.
    ```bash
    cat "...Hiding-From-You"
    ```
*   **Outcome:**  **SUCCESS**. Level 4 password successfully revealed.

### 🔹 Bandit Level 4 → Level 5
*   **Context:** Authenticated into Bandit Level 4.
*   **Discovery:** Inspected the `inhere` directory and located ten files named `-file00` through `-file09`. The leading dashes interfere with basic shell commands.
*   **Solution:** Used the `file` utility combined with a wildcard specifier to inspect the data profiles of all files simultaneously.
    ```bash
    file ./-*
    ```
*   **Finding:** `./-file07` was uniquely identified as `ASCII text`, while all other files contained raw non-text data.
*   **Execution:** 
    ```bash
    cat ./-file07
    ```
*   **Outcome:**  **SUCCESS**. Level 5 password successfully revealed.

### 🔹 Bandit Level 5 → Level 6
*   **Context:** Authenticated into Bandit Level 5.
*   **Discovery:** Entered the `inhere` directory. Manual verification showed 19 deeply nested subdirectories.
*   **Solution:** Deployed the `find` command to filter the directory tree tree by specific file type (`f`) and exact byte size (`1033c`).
    ```bash
    find inhere/ -type f -size 1033c
    ```
*   **Finding:** Programmatically isolated the hidden target path: `inhere/maybehere07/.file2`.
*   **Execution:** `cat inhere/maybehere07/.file2`
*   **Behavior Note:** The file output injected a large block of trailing whitespace/newline characters, causing terminal layout distortion. This padding mimics raw data packets and tests shell formatting resilience.
*   **Outcome:**  **SUCCESS**. Level 6 password safely captured.

### 🔹 Bandit Level 6 → Level 7
*   **Context:** Authenticated into Bandit Level 6.
*   **Discovery:** `ls -la` returned only basic system configuration dotfiles. The target file is hidden globally on the root server file system rather than locally.
*   **Trial 11:** Executed global root search: `find / -user bandit7 -group bandit6 -size 33c`
    *   *Result:* ❌ **BLANK** (Zero paths and no errors returned; root system scans can timeout or stall under heavy server load).
*   **Solution:** Modified search criteria to scan target system directories individually. Narrowed root search down to the `/var/` system directory to bypass heavy server load blocks.
    ```bash
    find /var/ -user bandit7 -group bandit6 -size 33c 2>/dev/null
    ```
*   **Finding:** Isolated hidden path: `/var/lib/dpkg/info/bandit7.password`.
*   **Outcome:**  **SUCCESS**. Executed `cat /var/lib/dpkg/info/bandit7.password` to reveal the Level 7 password.

### 🔹 Bandit Level 7 → Level 8
*   **Context:** Authenticated into Bandit Level 7. Located a large data repository file named `data.txt`. Reading it entirely with `cat` would overflow the shell.
*   **Solution:** Used the `grep` keyword filter tool to search `data.txt` for the string `millionth` and isolate the password line.
    ```bash
    grep "millionth" data.txt
    ```
*   **Outcome:**  **SUCCESS**. Level 8 password successfully revealed.

### 🔹 Bandit Level 8 → Level 9
*   **Context:** Exploring Bandit Level 8 environment. Verified that Level 8 does not utilize SQL or databases; it focuses entirely on flat-file text parsing.
*   **Challenge:** `data.txt` contains heavily repeated lines. The password is the only line that appears exactly once. The `uniq` tool requires adjacent matching lines, meaning data must be structurally sorted first.
*   **Solution:** Pipe the output of `sort` into `uniq -u` to cleanly isolate the single unique password string.
    ```bash
    sort data.txt | uniq -u
    ```
*   **Outcome:**  **SUCCESS**. Level 9 password successfully revealed.

### 🔹 Bandit Level 9 → Level 10
*   **Context:** Authenticated into Bandit Level 9. Located `data.txt`.
*   **Challenge:** The file contains raw binary data, which will break or corrupt standard text reading attempts.
*   **Solution:** Use the `strings` utility to strip away the binary junk and pipe the output into `grep` to look for the `==` markers.
    ```bash
    strings data.txt | grep "=="
    ```
*   *Note:* Encountered a minor local error when accidentally running the command in the VM terminal instead of the active SSH connection pane.
*   **Outcome:**  **SUCCESS**. Level 10 password successfully revealed.

---

## 🛑 Current State: End Point - Bandit Level 10

*   **Status:** Successfully logged into Bandit Level 10 using the extracted password hash.
*   **Operational Decision:** Pausing active gameplay at Level 10 to execute administrative and engineering tasks:
    1.  **Asset Sanitization:** Prepare to scrub all raw password strings from the local file repository to protect operational security (OpSec) by generating a clean `notes_sanitized.txt` before publishing.
    2.  **Jira Documentation:** Export the troubleshooting journey and level metrics into Jira tickets using platform-specific panel/code markup.
    3.  **GitHub Integration:** Create a permanent repository file to house these trial-and-error logs using standard GitHub Flavored Markdown (`.md`).
