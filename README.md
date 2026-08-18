*This project has been created as part of the 42 curriculum by maherraz.*

# Born2beRoot

## Table des matières

- [Description](#description)
  - [Operating System Selection](#operating-system-selection)
  - [Virtualization Infrastructure](#virtualization-infrastructure)
  - [Storage Architecture & Encryption](#storage-architecture--encryption)
  - [Privilege Management (Sudo)](#privilege-management-sudo)
  - [Authentication Policy (PAM)](#authentication-policy-pam)
  - [Network & Service Security](#network--service-security)
  - [Automated Telemetry Script](#automated-telemetry-script)
  - [Technical Comparisons](#technical-comparisons)
- [Instructions](#instructions)
- [Resources](#resources)
- [AI Usage Description](#ai-usage-description)

## Description

### Project Overview

Born2beRoot is a secure Linux VM designed to respect a strict system-administration baseline. The server is configured without a graphical interface and focuses on isolation, access control, encrypted storage, and monitoring.

### Operating System Selection

I chose Debian Trixie because it is simple, stable, and well adapted to a minimal server setup.

#### Debian advantages
- Stable and predictable release cycle.
- `apt` is easy to use and reliable for package management.
- Minimal installation fits a headless server perfectly.

#### Debian disadvantages
- Packages are not the newest ones.
- Community support is good, but not comparable to a full enterprise support contract.

#### Rocky Linux comparison
Rocky Linux is a solid alternative for enterprise environments, especially with RHEL compatibility and a strong ecosystem. However, it is usually more complex to configure and less suitable for this beginner-level project, especially because SELinux adds extra configuration complexity.

### Main Design Choices

#### Partitioning and encryption
The VM uses LUKS + LVM to encrypt the system disk and separate volumes for `/`, `/home`, `/boot`, and swap. This ensures data confidentiality and easier management of storage.

#### Security policies
- `sudo` is used instead of direct `root` login.
- `AppArmor` is enabled to restrict application access.
- SSH is configured on port `4242` and root login is disabled.
- UFW blocks all incoming traffic except SSH.
- Password rules are enforced with `libpam-pwquality`.

#### User and group management
Users are created with limited privileges. Admin tasks are delegated through `sudo`, and group membership is controlled to keep privileges explicit and auditable.

#### Installed services
The VM runs only the essential services needed for secure administration: `ssh`, `sudo`, `ufw`, `cron`, and monitoring utilities.

### Technical Comparisons

| Module | Chosen option | Alternative | Short rationale |
| :--- | :--- | :--- | :--- |
| **OS** | Debian | Rocky Linux | Debian is lighter, simpler, and better suited to a minimal secure server. Rocky is stronger for enterprise RHEL-style environments, but more complex. |
| **MAC** | AppArmor | SELinux | AppArmor is easier to understand and configure for a small system. SELinux is stricter and more powerful, but harder to manage correctly. |
| **Firewall** | UFW | firewalld | UFW is simple and ideal for a standalone VM. firewalld is more suited to dynamic multi-zone network setups. |
| **Hypervisor** | VirtualBox | UTM | VirtualBox is easy, stable, and widely used for x86 VMs. UTM is optimized for Apple hardware but is less universal. |

---

### Virtualization Infrastructure

The system is installed inside a virtual machine using VirtualBox. This isolates the environment from the host machine and allows easy snapshots, rollback, and testing without damaging the main operating system.

### Storage Architecture & Encryption

The storage subsystem relies on **LVM (Logical Volume Manager)** wrapped inside a secure cryptographic abstraction layer managed by LUKS.

#### Partition Topology
```
sda (12G)
├─sda1 (791M)  /boot
├─sda2 (1K)
└─sda5 (11.2G)
  └─sda5_crypt (LUKS encrypted)
    ├─maherraz42--vg-root    7.1G  /
    ├─maherraz42--vg-swap_1  620M  [SWAP]
    └─maherraz42--vg-home    3.4G  /home
```

#### Cryptographic Implementation
- Storage layer encryption utilizes `dm-crypt` with **AES-256-XTS-plain64** mapping.
- At least 2 encrypted partitions are used, as required by the project.
- Logical Volume Management allows easier volume resizing and improves fault containment.

### Privilege Management (Sudo)
- Direct administrative `root` authentication is explicitly disabled across interactive shells.
- Elevation is handled exclusively through a strictly audited `sudo` configuration.
- Command logging is stored in `/var/log/sudo/`.
- TTY restrictions and limited environment paths improve security.

### Authentication Policy (PAM)

System authentication policies are enforced with `libpam-pwquality`.

#### Operational Ruleset
- Minimum password length: 10 characters.
- Must include uppercase, lowercase, and numbers.
- No more than 3 identical consecutive characters.
- Cannot contain the username.
- Passwords expire every 30 days.
- Minimum 2 days before a new password can be changed.
- Warning starts 7 days before expiration.
- Non-root password differences must be significant enough to prevent trivial reuse.

### Network & Service Security

#### OpenSSH Hardening
- SSH runs on port `4242` instead of `22`.
- Root login is disabled.

#### Firewall Configuration (UFW)
- UFW is enabled at boot.
- Default policy is to deny incoming traffic.
- Only the SSH service is allowed through.

### Automated Telemetry Script

A bash monitoring script runs periodically and displays system information on all active terminals using `wall`.

#### Monitored Metrics
- OS and kernel version.
- CPU cores and load.
- RAM usage.
- Disk usage and LVM status.
- Active TCP connections.
- Logged-in users.
- Network addresses.
- `sudo` command count.

#### Automation Schedule
```bash
# crontab -l
*/10 * * * * /home/maherraz/monitoring.sh
```

#### Process Validation
```text
systemctl status cron
● cron.service - Regular background program processing daemon
   Loaded: loaded (/usr/lib/systemd/system/cron.service; enabled)
   Active: active (running) since ...
```

*Note:* The message `No MTA installed, discarding output` is expected because the script broadcasts directly to terminals and does not use an email daemon.

---

## Instructions

This quick reference guide outlines control commands required to audit and 
interact with the server infrastructure during technical evaluations.

### Remote Access (SSH)
```bash
ssh -p 4242 <username>@<IP_address>
```
- Replace `<username>` with the respective user context (e.g., `maherraz`).
- Query target interface parameters with `ip a` to discover the active IP address.
- Direct root access attempts will fail due to security policy configurations.

### Firewall Operation (UFW)
- **Inspect active filtering matrix:**
  ```bash
  sudo ufw status verbose
  ```
- **Inject network access rules (e.g., open port 8080):**
  ```bash
  sudo ufw allow 8080
  ```
- **Revoke rules by explicit application index or port:**
  ```bash
  sudo ufw status numbered
  sudo ufw delete <rule-number>
  ```

### System Daemon Control (systemd)
- **Audit service runtime states:**
  ```bash
  sudo systemctl status ssh
  sudo systemctl status ufw
  sudo systemctl status cron
  ```
- **Process lifecycle manipulation (start/stop/restart):**
  ```bash
  sudo systemctl restart ssh
  ```

### Credential & Password Management
- **Modify your own password:**
  ```bash
  passwd
  ```
- **Administrative override of user credentials:**
  ```bash
  sudo passwd <username>
  ```
- **Review password aging metadata constraints:**
  ```bash
  sudo chage -l <username>
  ```

### Account & Group Management
- **Provision new user identities:**
  ```bash
  sudo adduser <newuser>
  ```
- **Append existing user into structural security groups:**
  ```bash
  sudo usermod -aG <groupname> <username>
  ```
- **Inspect user access vectors:**
  ```bash
  id <username>
  groups <username>
  ```
- **Create a new group:**
  ```bash
  sudo groupadd <groupname>
  ```

### System Identity (Hostname)
- **Query active hostname metrics:**
  ```bash
  hostnamectl
  ```
- **Enforce persistent system hostname changes:**
  ```bash
  sudo hostnamectl set-hostname <new-hostname>
  ```
- **Verify after reboot** – the change must persist. To revert, use the same command with the original hostname.

### Monitoring Script Control
- **Inspect script content:**
  ```bash
  cat /home/<login>/monitoring.sh
  ```
- **Inspect active crontab configuration:**
  ```bash
  crontab -l                     # for current user
  sudo crontab -l                # for root (if installed as root)
  ```
- **Force script execution manually:**
  ```bash
  /home/<login>/monitoring.sh
  ```
- **Stop script from running at boot** (without deleting it):
  - Remove or comment out the cron line using `crontab -e`.
  - The script will **not run again** after reboot if the cron job is removed.

---

## Resources

### Official Documentation
- [Debian Installation Guide](https://www.debian.org/releases/stable/installmanual)
- [Securing Debian Manual](https://www.debian.org/doc/manuals/securing-debian-howto)
- [LVM HOWTO](https://tldp.org/HOWTO/LVM-HOWTO)
- [UFW Guide](https://help.ubuntu.com/community/UFW)

### Community References
- Debian Handbook – LVM chapter
- AppArmor wiki
- VirtualBox / UTM user manuals

### AI Usage Description

An AI assistant was used as a **technical collaborator** during this project for:

- **Conceptual Clarification** – Explaining LVM, AppArmor vs SELinux, virtualization principles, and system administration theories.
- **Documentation Structuring** – Writing and polishing the English prose, creating comparison tables, and formatting the README to meet 42 standards.
- **Review of Configuration Syntax** – Verifying correct options for SSH, sudo, and PAM configuration files.

#### Strict Exclusions
- The AI did **not** generate any production code, configuration files, or the monitoring script.
- All system commands, file edits, and cron setup were performed **manually** on the terminal.
- The implementation remains entirely my own work.
