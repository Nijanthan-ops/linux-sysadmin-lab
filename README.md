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

## 2: Users, Permissions & Intrusion Prevention
 - Created a secondary user (testuser) with adduser
 - Granted sudo privileges with usermod -aG sudo testuser (using -a to append, not overwrite, existing group membership)
 - Reviewed /etc/sudoers via visudo to understand how group-based sudo access is granted
 - Practiced file permission management:
 - chmod 640 to set explicit owner/group/other read-write access
 - chown to reassign file ownership between users
 - Installed and enabled fail2ban to automatically detect and block brute-force SSH attempts
 - Verified fail2ban functionally, not just installed:
 - Temporarily re-enabled password authentication in a controlled test window
 - Deliberately triggered repeated failed logins against testuser from the host
 - Confirmed via fail2ban-client status sshd that fail2ban logged and banned the offending IP (Total banned: 1)
 - Reverted PasswordAuthentication back to no and confirmed the setting afterward, restoring the Day 1 security posture

## Result: Working access-control model (sudo group, per-file permissions) plus a verified, functioning intrusion-prevention layer — not just a service installed and left unchecked.

## 3: Networking Fundamentals

 - Focused on the core networking concepts a sysadmin role expects, verified with real diagnostic commands rather than just theory.

 - IP addressing & subnetting — how a subnet mask (e.g. /24) defines the boundary between network and host portions of an address
 - DNS resolution flow — traced the path from a domain name query to resolver → root server → TLD server → authoritative server → IP address
 - Common ports — 22 (SSH), 53 (DNS), 80 (HTTP), 443 (HTTPS), 3306 (MySQL)
 - Hands-on diagnostics, run and interpreted against real targets:
 - ping — reachability and round-trip latency
 - traceroute — hop-by-hop path to a destination
 - dig — direct DNS resolution lookup
 - ss -tulnp — locally listening ports and the process bound to each

## Result: Comfortable explaining and using the core commands a sysadmin uses daily to diagnose connectivity and name-resolution issues.

## Key Commands Reference
  - ssh-keygen -t ed25519              # Generate SSH key pair
  -  ssh-copy-id user@host              # Deploy public key to a remote server
  - sudo ufw allow OpenSSH             # Allow SSH before enabling firewall
  -  sudo ufw enable                    # Enable default-deny firewall
  -  sudo usermod -aG sudo user         # Grant sudo access (append, not overwrite)
  -  sudo visudo                        # Safely edit sudoers file
  -  chmod 640 file                     # Set explicit rw/r/– permissions
  -  sudo fail2ban-client status sshd   # Check jail status and banned IPs
  -  journalctl -u ssh                  # View SSH service logs
  -  dig example.com                    # Query DNS resolution
  -  ss -tulnp                          # Show locally listening ports
