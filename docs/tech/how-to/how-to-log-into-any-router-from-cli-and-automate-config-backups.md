---
title: How to Log into Any Router from CLI and Automate Config Backups
date: 2025-06-17T13:05:16.383Z
description: A comprehensive guide for developers on accessing network routers via CLI (SSH, Telnet, Serial) and automating configuration backups using Bash, Python, and Git.
tags: ["Linux", "CLI", "Bash", "Python", "Networking", "Automation", "SSH", "Router"]
categories: ["Networking", "DevOps", "SysAdmin"]
comments: true
---

As developers, we often interact with various systems through the command line. Networks are no exception. While most consumer-grade routers offer a shiny web UI, real power users – and especially those looking to automate – gravitate towards the Command Line Interface (CLI). This guide will walk you through accessing network routers from your Linux/macOS terminal and, critically, how to automate those critical configuration backups.

Forget clicking around; let's get pragmatic.

## Why CLI for Router Management?

1.  **Automation**: Automate repetitive tasks, configurations, and backups. Essential for DevOps and SREs.
2.  **Consistency**: Ensure identical configurations across multiple devices.
3.  **Speed**: Often faster to type a command than navigate multiple web UI menus.
4.  **Troubleshooting**: Access to detailed logs and diagnostic tools not always exposed in the web UI.
5.  **Disaster Recovery**: A backup of your router's configuration is paramount for quick recovery.

Let's dive into the various methods of accessing a router.

## 1. Understanding Router Access Protocols

Routers speak a few different languages for CLI access. Knowing which to use, and when, is key.

### a. SSH (Secure Shell) - The Preferred Standard

SSH is the **gold standard** for secure remote access. It encrypts all communication, protecting your credentials and configuration data from eavesdropping. Most modern, professional-grade routers (Cisco, Juniper, MikroTik, Ubiquiti, etc.) support SSH.

### b. Telnet - The Insecure Legacy

Telnet is an older protocol that transmits all data, including usernames and passwords, in **plain text**. This makes it highly vulnerable to eavesdropping. **Avoid Telnet if SSH is available.** It's typically only found on very old or poorly configured devices. We'll cover it only out of completeness and necessity when no other option exists.

### c. Serial Console - The Lifeline

When things go really wrong (e.g., misconfigured IP, locked out, factory reset), the serial console is your direct, out-of-band access. It requires a physical cable connection (typically a USB-to-serial adapter and a console cable) to a dedicated serial port on the router. This is your "break glass in case of emergency" access.

### d. Web UI (Graphical User Interface) - Not CLI

While most consumer routers rely primarily on a Web UI (accessed via `http://192.168.1.1` or similar), this guide focuses on CLI methods. Web UIs are fine for basic setup but are generally not conducive to automation or advanced troubleshooting.

### Checking for Open Ports with `nmap`

Before attempting to connect, you can check which ports are open on your router's IP address using `nmap`. The default SSH port is 22, and Telnet is 23.

```bash
# Replace 192.168.1.1 with your router's IP address
nmap 192.168.1.1 -p 22,23
```

```output
Starting Nmap 7.93 ( https://nmap.org ) at 2023-10-27 10:30 PDT
Nmap scan report for _gateway (192.168.1.1)
Host is up (0.0028s latency).

PORT   STATE SERVICE
22/tcp open  ssh
23/tcp closed telnet

Nmap done: 1 IP address (1 host up) scanned in 0.04 seconds
```

In this output, port 22 (SSH) is open, and port 23 (Telnet) is closed. Excellent! We should use SSH.

## 2. Logging In via SSH (The Preferred Way)

To use SSH, you'll need:
*   Your router's IP address (e.g., `192.168.1.1`).
*   A username configured on the router (e.g., `admin`, `root`, `cisco`).
*   The password for that user, or an SSH key.

### a. Basic SSH Login (Password-based)

This is the simplest way, but less secure for automation as it typically requires entering a password manually or using `sshpass` (which we'll cover later, with caveats).

```bash
# Syntax: ssh <username>@<router_ip>
ssh admin@192.168.1.1
```

```output
The authenticity of host '192.168.1.1 (192.168.1.1)' can't be established.
ED25519 key fingerprint is SHA256:aBcDeFgHiJkLmNoPqRsTuVwXyZ0123456789.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '192.168.1.1' (ED25519) to the list of known hosts.
admin@192.168.1.1's password:
RouterOS 7.10.2 (stable)
admin@Router>
```

Once logged in, the prompt will change, indicating you are now on the router's CLI. Commands are vendor-specific (e.g., `show running-config` on Cisco, `print` on MikroTik, `status` on UniFi devices).

### b. SSH Key-based Authentication (Highly Recommended)

For security and automation, SSH keys are superior. You generate a pair of keys: a private key (kept secret on your local machine) and a public key (uploaded to the router). When connecting, your client uses the private key to prove its identity, and the router verifies it with the public key. No password needs to be sent over the network.

**Step 1: Generate an SSH Key Pair**

If you don't have one, generate a strong ED25519 key.

```bash
ssh-keygen -t ed25519 -C "your_email@example.com"
```

```output
Generating public/private ed25519 key pair.
Enter file in which to save the key (/home/youruser/.ssh/id_ed25519): 
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/youruser/.ssh/id_ed25519
Your public key has been saved in /home/youruser/.ssh/id_ed25519.pub
The key fingerprint is:
SHA256:AbCdEfGhIjKlMnOpQrStUvWxFgHiJkL your_email@example.com
The key's randomart image is:
+--[ED25519 256]--+
|      .o*o..     |
|     . o=*=.     |
|    . . =++.     |
|   . + = E.      |
|. . o + .        |
|.o o + o         |
|==+ o .          |
|*o = o           |
|o.+.             |
+----[SHA256]-----+
```

*   **Note**: It's good practice to set a passphrase for your private key, especially if your local machine might be compromised. For automation, you might leave it empty, but be aware of the security implications.
*   Your public key will be in `~/.ssh/id_ed25519.pub`.

**Step 2: Copy Public Key to Router**

This step is **router-specific**. You'll usually log in to the router (via its web UI or initial password-based SSH) and paste your `id_ed25519.pub` content into the user's SSH key configuration.

*   **Cisco IOS/IOS XE**:
    ```
    username <username> privilege 15 secret <password>
    username <username> sshpubkey-data <paste_your_public_key_here>
    ```
*   **MikroTik RouterOS**:
    ```
    /user ssh-keys add user=<username> public-key="<paste_your_public_key_here>"
    ```
*   **Ubiquiti EdgeRouter/UniFi OS**: Usually involves copying the key to `~/.ssh/authorized_keys` for a specific user.

**Step 3: Configure SSH Client (Optional but Recommended)**

Edit `~/.ssh/config` to simplify connections.

```bash
# Open or create the SSH config file
nano ~/.ssh/config
```

Add an entry like this:

```
Host myrouter
    Hostname 192.168.1.1
    User admin
    IdentityFile ~/.ssh/id_ed25519
    Port 22 # Optional, 22 is default
```

Now, you can connect using the alias:

```bash
ssh myrouter
```

```output
RouterOS 7.10.2 (stable)
admin@Router>
```

This is much cleaner and secure.

## 3. Dealing with Telnet (When You Have No Choice)

As stated, Telnet is insecure. Use it only if:
1.  It's the *only* available method.
2.  You are on a highly trusted, isolated network.
3.  You understand the risks.

The command is simple:

```bash
telnet 192.168.1.1
```

```output
Trying 192.168.1.1...
Connected to 192.168.1.1.
Escape character is '^]'.
MikroTik RouterOS 7.10.2
Login: admin
Password:
RouterOS 7.10.2 (stable)
admin@Router>
```

Once connected, it behaves much like an SSH session, but without encryption.

## 4. Serial Console Access (The Lifeline)

This method requires physical access and specific hardware:

*   **USB-to-Serial Adapter**: If your computer doesn't have a built-in serial port (most don't), you'll need one. Prolific (PL2303) and FTDI (FT232) chipsets are common.
*   **Console Cable**: Typically an RJ45-to-DB9 (or USB-C to RJ45 for newer routers) cable that connects the router's console port to your serial adapter.

### a. Identifying the Serial Port

On Linux, serial devices usually appear as `/dev/ttyUSB0`, `/dev/ttyS0`, etc. After plugging in your USB-to-serial adapter, you can check `dmesg`:

```bash
dmesg | grep tty
```

```output
[   10.123456] usb 1-1: pl2303 converter now attached to ttyUSB0
```

This confirms `/dev/ttyUSB0` is our port.

### b. Connecting with `screen`

`screen` is a powerful terminal multiplexer that can also be used for serial connections. You'll need to know the router's console speed (baud rate), data bits, parity, and stop bits (usually 9600 8N1: 9600 baud, 8 data bits, No parity, 1 stop bit).

```bash
# Syntax: screen /dev/ttyUSB0 <baud_rate>
screen /dev/ttyUSB0 9600
```

```output
(Screen clears, you might see boot messages or a login prompt from the router)

Press Enter to activate the console...
User: admin
Password:
admin@Router>
```

To exit `screen`, press `Ctrl+A` then `k`, then `y` to confirm.

### c. Other Tools

*   **`minicom`**: Another popular serial client for Linux. More configuration options than `screen`.
*   **`putty`**: Excellent multi-protocol client for Windows, also available on Linux. Has a dedicated Serial section.

## 5. Automating Configuration Backups

This is where CLI truly shines. Regular configuration backups are essential for disaster recovery, auditing, and change management.

### a. Simple SSH + Scripting (Bash)

For basic backups, a simple shell script combined with SSH keys can be highly effective. This assumes the router's CLI can output the running configuration to standard output.

```bash
#!/bin/bash

ROUTER_IP="192.168.1.1"
ROUTER_USER="admin"
BACKUP_DIR="/opt/router_configs"
TIMESTAMP=$(date +"%Y%m%d_%H%M%S")
CONFIG_FILE="${BACKUP_DIR}/router_config_${TIMESTAMP}.txt"
LOG_FILE="${BACKUP_DIR}/backup.log"

# Ensure backup directory exists
mkdir -p "$BACKUP_DIR"

echo "[$(date)] Starting backup for $ROUTER_IP..." >> "$LOG_FILE"

# Cisco-like command: show running-config
# MikroTik-like command: export compact
# Ubiquiti EdgeRouter: show configuration commands
# Replace 'show running-config' with the appropriate command for your router
ssh -o BatchMode=yes -o StrictHostKeyChecking=no "$ROUTER_USER@$ROUTER_IP" "show running-config" > "$CONFIG_FILE" 2>> "$LOG_FILE"

if [ $? -eq 0 ]; then
    echo "[$(date)] Backup successful: $CONFIG_FILE" >> "$LOG_FILE"
    echo "Backup successful: $CONFIG_FILE"
else
    echo "[$(date)] Backup failed for $ROUTER_IP" >> "$LOG_FILE"
    echo "Backup failed. Check $LOG_FILE for details."
fi

# Optional: Keep only the last 7 backups
find "$BACKUP_DIR" -type f -name "router_config_*.txt" -mtime +7 -delete
```

**Explanation:**
*   `ssh -o BatchMode=yes`: Prevents SSH from asking for a password, essential for automation. Requires SSH key authentication.
*   `-o StrictHostKeyChecking=no`: **Use with caution!** This bypasses the host key check. Only use if you've already manually connected and accepted the host key, or in controlled environments. Better to set `StrictHostKeyChecking=yes` and ensure the router's host key is in `~/.ssh/known_hosts`.
*   `"show running-config"`: The command executed on the router. **Change this to your router's specific command!**
*   `>`: Redirects the router's output to `CONFIG_FILE`.
*   `2>>`: Appends standard error to `LOG_FILE`.

**Example Usage:**

```bash
# Save the script as backup_router.sh
chmod +x backup_router.sh
./backup_router.sh
```

```output
Backup successful: /opt/router_configs/router_config_20231027_104500.txt
```

**Sample Log File (`/opt/router_configs/backup.log`):**

```
[Fri Oct 27 10:45:00 PDT 2023] Starting backup for 192.168.1.1...
[Fri Oct 27 10:45:01 PDT 2023] Backup successful: /opt/router_configs/router_config_20231027_104500.txt
```

### b. Automating with `cron`

You can schedule the above script to run daily or weekly using `cron`.

```bash
crontab -e
```

Add the following line to run the script every day at 3:00 AM:

```
0 3 * * * /path/to/your/backup_router.sh > /dev/null 2>&1
```

Replace `/path/to/your/backup_router.sh` with the actual path to your script.

### c. Python with `paramiko` (Robust, Cross-Platform)

For more complex interactions, error handling, or when dealing with multiple devices, Python with `paramiko` (an SSHv2 protocol implementation) or `netmiko` (built on `paramiko`, specifically for network devices) is excellent.

First, install `paramiko`:

```bash
pip install paramiko
```

**Python Script Example (`backup_paramiko.py`):**

```python
import paramiko
import os
import datetime

ROUTER_IP = "192.168.1.1"
ROUTER_USER = "admin"
# Ensure your SSH private key is correctly specified
PRIVATE_KEY_PATH = os.path.expanduser("~/.ssh/id_ed25519") 
BACKUP_DIR = "/opt/router_configs"

# Command to get the running config. Adjust for your router type.
# Cisco: "show running-config"
# MikroTik: "/export compact"
# Juniper: "show configuration | display set"
CONFIG_COMMAND = "show running-config"

def backup_router_config():
    if not os.path.exists(BACKUP_DIR):
        os.makedirs(BACKUP_DIR)

    timestamp = datetime.datetime.now().strftime("%Y%m%d_%H%M%S")
    config_file = os.path.join(BACKUP_DIR, f"router_config_{timestamp}.txt")

    try:
        ssh_client = paramiko.SSHClient()
        ssh_client.set_missing_host_key_policy(paramiko.AutoAddPolicy()) # AVOID IN PRODUCTION, use WarningPolicy or known_hosts
        
        # Load the private key
        private_key = paramiko.RSAKey.from_private_key_file(PRIVATE_KEY_PATH) # Or paramiko.Ed25519Key, paramiko.ECDSAKey

        print(f"Connecting to {ROUTER_IP}...")
        ssh_client.connect(hostname=ROUTER_IP, username=ROUTER_USER, pkey=private_key)
        print("Connected. Executing command...")

        stdin, stdout, stderr = ssh_client.exec_command(CONFIG_COMMAND)
        output = stdout.read().decode('utf-8')
        errors = stderr.read().decode('utf-8')

        if errors:
            print(f"Error executing command: {errors}")
            return False

        with open(config_file, "w") as f:
            f.write(output)
        
        print(f"Backup successful: {config_file}")
        return True

    except paramiko.AuthenticationException:
        print("Authentication failed, please check your credentials/key.")
        return False
    except paramiko.SSHException as e:
        print(f"SSH error: {e}")
        return False
    except Exception as e:
        print(f"An unexpected error occurred: {e}")
        return False
    finally:
        if 'ssh_client' in locals() and ssh_client:
            ssh_client.close()

if __name__ == "__main__":
    backup_router_config()
```

**Note on `paramiko.AutoAddPolicy()`**: This line will automatically add the router's host key to your `~/.ssh/known_hosts` file if it's not already there. While convenient for initial setup, in a production environment, you should use `paramiko.WarningPolicy()` (which warns if the key is unknown) or, ideally, pre-populate your `known_hosts` file and use `ssh_client.load_system_host_keys()`.

**Example Usage:**

```bash
python backup_paramiko.py
```

```output
Connecting to 192.168.1.1...
Connected. Executing command...
Backup successful: /opt/router_configs/router_config_20231027_105500.txt
```

### d. Python with `netmiko` (Network Automation Library)

`netmiko` is built on `paramiko` but provides a simplified API specifically for interacting with various network device vendors. It handles prompt detection, pagination, and various vendor quirks automatically. This is generally the best choice for network automation in Python.

First, install `netmiko`:

```bash
pip install netmiko
```

**Python Script Example (`backup_netmiko.py`):**

```python
from netmiko import ConnectHandler
import os
import datetime

# Device type specifies the vendor and OS (e.g., 'cisco_ios', 'mikrotik_routeros', 'juniper_junos')
# Find the correct device type in Netmiko documentation for your router.
ROUTER_DEVICE_TYPE = "cisco_ios" # or "mikrotik_routeros", "ubiquiti_edgerouter", etc.
ROUTER_IP = "192.168.1.1"
ROUTER_USER = "admin"
PRIVATE_KEY_PATH = os.path.expanduser("~/.ssh/id_ed25519") 
BACKUP_DIR = "/opt/router_configs"

# Command to get the running config. Netmiko often has built-in methods too.
# For Cisco, "show running-config" is common. For MikroTik, "/export compact".
CONFIG_COMMAND = "show running-config" 

def backup_router_config_netmiko():
    if not os.path.exists(BACKUP_DIR):
        os.makedirs(BACKUP_DIR)

    timestamp = datetime.datetime.now().strftime("%Y%m%d_%H%M%S")
    config_file = os.path.join(BACKUP_DIR, f"router_config_{timestamp}.txt")

    device = {
        "device_type": ROUTER_DEVICE_TYPE,
        "host": ROUTER_IP,
        "username": ROUTER_USER,
        "use_keys": True,
        "key_file": PRIVATE_KEY_PATH,
        # "password": "your_password_if_not_using_keys", # Use password if not using keys
        "global_delay_factor": 2, # Increase for slow devices/connections
    }

    try:
        print(f"Connecting to {ROUTER_IP} ({ROUTER_DEVICE_TYPE})...")
        with ConnectHandler(**device) as net_connect:
            print("Connected. Executing command...")
            output = net_connect.send_command(CONFIG_COMMAND, delay_factor=5) # Add delay_factor for slow devices

            # Netmiko can also directly get specific configs
            # output = net_connect.send_command("show startup-config") # Example for startup config
            # output = net_connect.send_command("show version") # Example for version info
            
            # Or use built-in methods if supported by device type:
            # output = net_connect.send_config_from_file(config_file='commands.txt') # For applying configs
            # net_connect.save_config() # If you want to save running-config to startup-config

            with open(config_file, "w") as f:
                f.write(output)
            
            print(f"Backup successful: {config_file}")
            return True

    except Exception as e:
        print(f"An error occurred: {e}")
        return False

if __name__ == "__main__":
    backup_router_config_netmiko()
```

**Example Usage:**

```bash
python backup_netmiko.py
```

```output
Connecting to 192.168.1.1 (cisco_ios)...
Connected. Executing command...
Backup successful: /opt/router_configs/router_config_20231027_110000.txt
```

`netmiko` abstracts away a lot of the complexities of interacting with different network operating systems, making it highly suitable for multi-vendor environments.

## 6. Version Control for Configurations (Git)

Once you're regularly backing up configs, version controlling them with Git is the next logical step. This allows you to track changes, see who changed what, and revert to previous versions if needed.

**Basic Git Workflow for Router Configs:**

```bash
# Initialize a Git repository in your backup directory
cd /opt/router_configs
git init
```

```output
Initialized empty Git repository in /opt/router_configs/.git/
```

```bash
# First commit (optional: add a .gitignore for logs or temporary files)
echo "backup.log" > .gitignore
git add .gitignore
git commit -m "Add .gitignore"
```

```output
[master (root-commit) 8c4f2b1] Add .gitignore
 1 file changed, 1 insertion(+)
 create mode 100644 .gitignore
```

Now, after each successful backup script run, add these lines to your backup script (e.g., `backup_router.sh` or within Python):

```bash
# After a successful config file creation:
cd "$BACKUP_DIR"
git add "$CONFIG_FILE"
git commit -m "Automated backup for $(basename "$CONFIG_FILE")"
# Optional: Push to a remote Git repository (e.g., GitHub, GitLab, private server)
# git push origin master
```

**Example of the Bash script (updated section):**

```bash
# ... (previous script content) ...

if [ $? -eq 0 ]; then
    echo "[$(date)] Backup successful: $CONFIG_FILE" >> "$LOG_FILE"
    echo "Backup successful: $CONFIG_FILE"
    
    # Git Integration
    cd "$BACKUP_DIR"
    git add "$CONFIG_FILE"
    git commit -m "Automated backup for $ROUTER_IP - ${TIMESTAMP}" >> "$LOG_FILE" 2>&1
    echo "[$(date)] Config committed to Git." >> "$LOG_FILE"
else
    # ... (error handling) ...
fi

# ... (rest of the script) ...
```

**Checking Git History:**

```bash
cd /opt/router_configs
git log --oneline
```

```output
2a1b3c4 (HEAD -> master) Automated backup for 192.168.1.1 - 20231027_110000
9d8e7f6 Automated backup for 192.168.1.1 - 20231027_105500
8c4f2b1 Add .gitignore
```

You can now easily see what changed between backups:

```bash
git diff HEAD~1 HEAD
```

```output
diff --git a/router_config_20231027_105500.txt b/router_config_20231027_110000.txt
index a1b2c3d..e4f5g6h 100644
--- a/router_config_20231027_105500.txt
+++ b/router_config_20231027_110000.txt
@@ -10,4 +10,5 @@
 interface Ethernet0/1
  ip address 192.168.10.1 255.255.255.0
 !
-route 0.0.0.0 0.0.0.0 192.168.1.254
+route 0.0.0.0 0.0.0.0 192.168.1.254 name DefaultRoute
+! A new comment added to config
```

This makes managing router configurations just like managing code – a powerful paradigm shift for network operations.

## Conclusion

Mastering CLI access to your network devices is a fundamental skill for any developer or operations professional. It unlocks capabilities far beyond what a typical web UI offers, especially when it comes to automation.

By leveraging SSH (with keys!), scripting with Bash or Python (`paramiko`, `netmiko`), and integrating with version control like Git, you can transform your network management from a manual chore into a robust, automated, and auditable process. Start small, understand your router's specific commands, and scale up from there. Happy networking!
