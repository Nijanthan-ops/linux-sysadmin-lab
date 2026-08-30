# linux-sysadmin-lab
## Environment
- Host: Ubuntu 24.04 (VirtualBox 7.0.16)
- Guest VM: Ubuntu Server, Bridged networking

## 1: SSH Hardening & Firewall

- Generated an ed25519 SSH key pair on host, deployed to VM via ssh-copy-id
- Verified key-based login before disabling password auth (to avoid lockout)
- Edited /etc/ssh/sshd_config:
  - PasswordAuthentication no
  - PermitRootLogin no
- Restarted sshd and confirmed key-only login in a fresh session
- Installed and configured ufw:
  - Allowed OpenSSH explicitly before enabling
  - Enabled default-deny firewall policy
  - Verified rules with `ufw status verbose`
 ### Result: VM only accepts SSH key auth (no passwords, no root login), and only exposes port 22, with everything else blocked by default.

## 2: Users, Permissions & Intrusion Prevention
 - Created a secondary user (testuser) with **adduser**
 - Granted **sudo** privileges with **usermod -aG sudo testuser** (using -a to append, not overwrite, existing group membership)
 - Reviewed /etc/sudoers via **visudo** to understand how group-based sudo access is granted
 - Practiced file permission management:
   - **chmod 640** to set explicit owner/group/other read-write access
   - **chown** to reassign file ownership between users
   - Installed and enabled **fail2ban** to automatically detect and block brute-force SSH attempts
 - Verified fail2ban functionally, not just installed:
   - Temporarily re-enabled password authentication in a controlled test window
   - Deliberately triggered repeated failed logins against testuser from the host
   - Confirmed via **fail2ban-client status sshd** that fail2ban logged and banned the offending IP (Total banned: 1)
   - Reverted PasswordAuthentication back to no and confirmed the setting afterward, restoring the Day 1 security posture

### Result: Working access-control model (sudo group, per-file permissions) plus a verified, functioning intrusion-prevention layer — not just a service installed and left unchecked.

## 3: Networking Fundamentals

 - Focused on the core networking concepts a sysadmin role expects, verified with real diagnostic commands rather than just theory.

 - IP addressing & subnetting — how a subnet mask (e.g. /24) defines the boundary between network and host portions of an address
 - DNS resolution flow — traced the path from a domain name query to resolver → root server → TLD server → authoritative server → IP address
 - Common ports — 22 (SSH), 53 (DNS), 80 (HTTP), 443 (HTTPS), 3306 (MySQL)
 - Hands-on diagnostics, run and interpreted against real targets:
 - **ping** — reachability and round-trip latency
 - **traceroute** — hop-by-hop path to a destination
 - **dig** — direct DNS resolution lookup
 - **ss -tulnp** — locally listening ports and the process bound to each

### Result: Comfortable explaining and using the core commands a sysadmin uses daily to diagnose connectivity and name-resolution issues.

## 4: Log Monitoring & Automated Alerts
 - Read and interpreted SSH activity two ways: **journalctl -u ssh** (systemd's centralized journal) and /var/log/auth.log (flat-file authentication log)
 - Identified and explained real log entries: successful key-based logins, **sudo** command execution with full audit trail, and service restarts
 - Deliberately generated a real failed-login event and located it in the logs
 - Built an automated monitoring mechanism using **cron**:
   - A job running every 5 minutes, scanning auth.log for failed login attempts and writing the most recent matches to an alert file
   - Verified it end-to-end — confirmed real failed-login entries were automatically captured without manual log checking

### Result: A working, self-built log-monitoring and alerting pipeline — a simplified version of what production monitoring tools do at scale.

## 5: Backup, Restore & Patch Management
 - Practiced a full backup/restore cycle using **rsync**:
   - Took a dated backup (**rsync -av**) of test data, preserving permissions and timestamps
   - Deliberately deleted a file to simulate data loss
   - Restored the missing file from the backup and verified its contents matched the original
 - Performed system patch management:
   - Reviewed pending updates with **apt list --upgradable** before applying anything
   - Applied updates with **apt upgrade**, patching dozens of packages including security-relevant ones (**openssl, apparmor, libgcrypt20**)
   - Observed Ubuntu's phased rollout behavior firsthand (some packages deferred with "Not upgrading yet due to phasing")
   - Reviewed /var/log/apt/history.log as a timestamped, self-documenting audit trail of every install/upgrade performed across the week, each attributed to the requesting user

### Result: Demonstrated not just applying updates, but reviewing what's pending first, understanding staged rollout behavior, and having an audit trail to reference — plus a tested (not just assumed) backup/restore process.


## Key Commands Reference
  - ssh-keygen -t ed25519              # Generate SSH key pair
  - ssh-copy-id user@host              # Deploy public key to a remote server
  - sudo ufw allow OpenSSH             # Allow SSH before enabling firewall
  - sudo ufw enable                    # Enable default-deny firewall
  - sudo usermod -aG sudo user         # Grant sudo access (append, not overwrite)
  - sudo visudo                        # Safely edit sudoers file
  - chmod 640 file                     # Set explicit rw/r/– permissions
  - sudo fail2ban-client status sshd   # Check jail status and banned IPs
  - journalctl -u ssh                  # View SSH service logs
  - dig example.com                    # Query DNS resolution
  - ss -tulnp                          # Show locally listening ports
  - rsync -av src/ dest/                # Backup/sync files, preserving metadata
  - sudo apt list --upgradable         # Review pending patches before applying
  - cat /var/log/apt/history.log       # Review patch/install audit trail
