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
