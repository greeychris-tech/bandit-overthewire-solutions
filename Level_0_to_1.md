1. TARGET IDENTIFICATION & SOURCE RECONNAISSANCE

Lab Provider: OverTheWire Wargames Community

Target Track: Bandit (Designed for fundamental Linux command line and security principles)

Intelligence Source: Evaluated the official portal (https://overthewire.org) to extract connection requirements.

Target Host IP/Domain: 

    OverTheWire: Natas 

    Target Destination Port: 2220

    Initial Low-Privilege Username: bandit0

    Initial Plaintext Password: bandit0

2. INFRASTRUCTURE & LABORATORY PRE-REQUISITES

    Hardware Layer: ASUS Vivobook Laptop (Configured with 16GB Total RAM / 512GB SSD / Windows Host OS).

    Virtualization Layer: Oracle VM VirtualBox Manager application.

    Guest Operating System: Kali Linux Distribution (VM explicitly optimized and re-allocated to 4GB Dedicated RAM and 2 CPU Cores to remediate terminal processing latency).

    Network Infrastructure Layer: Mobile Phone Hotspot. Configured within Windows settings as a Metered Connection to suppress automated background data usage and preserve a 20GB monthly network bandwidth cap.

3. LABORATORY INTEGRATION & DISCOVERY

    Environment Baseline Audit:

        Booted the Kali Linux Virtual Machine and initiated a standard desktop terminal session.

        Executed a complete directory audit (ls -la) to evaluate the local environment.

        Identified a local conflict element: an automated script asset named setup_complete_bandit.sh was present in the working directory, causing environmental path conflicts with standard networking syntax.

    Network Execution & Shell Access Sequence:
    4. Bypassed the local script shortcut conflict by enforcing an explicit system binary path routing configuration to fire up the native Secure Shell utility:
       /usr/bin/ssh bandit0@bandit.labs.overthewire.org -p 2220
    5. Received the remote host's cryptographic authentication challenge. Evaluated the fingerprint hash and explicitly typed yes to authorize the connection and save the key to known_hosts.
    6. Submitted the plaintext initialization credential (bandit0) at the remote system password prompt.
    7. Successfully authenticated past the perimeter firewall, landing on the remote server's active text shell environment: bandit0@bandit:~$.

    Data Extraction Sequence:
    8. Executed the ls directory inspection command on the target server filesystem, identifying a target file asset titled readme.
    9. Executed the file-read system binary command (cat readme) to extract the plaintext payload string hidden within the file layer.

4. EXTRACTED PRIVILEGE ESCALATION CREDENTIAL

* **Bandit Level 1 Access Password:** [REDACTED_LEVEL_1_SECRET_TOKEN] 
