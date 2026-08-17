*This project has been created as part of the 42 curriculum by maherraz.*

# Born2beRoot

## Table of Contents. 
1. [Description](#description)
    * [Operating System Selection](#operating-system-selection)
        * [Why Debian ?](#why-debian)
        * [Limitations of Debian](#constraints--limitations)
    * [Virtualization Infrastructure](#virtualization-infrastructure)
        * [What?](#what)

4. [Storage Architecture & Encryption](#storage-architecture--encryption)
5. [Privilege Management](#privilege-management)
6. [Authentication Policy](#authentication-policy)
7. [Network & Service Security](#network--service-security)
8. [Automated Tary Script](#automated-telemetry-script)
9. [Technical Comparisons](#technical-comparisons)
10. [Operation Manual](#operation-manual)

---

## Description

Born2beRoot is a system administration project focused on deploying a secure Linux distribution within a virtual machine. The implementation enforces strict security baselines, comprehensive monitoring, and a fully functional multi-user environment operating entirely without a graphical user interface (GUI).

### Operating System Selection
I chose Debian Trixie  ([debian-13.6.0-amd64-netinst.iso](https://cdimage.debian.org/debian-cd/current/amd64/iso-cd/debian-13.6.0-amd64-netinst.iso)).
Debian is a volunteer organization founded in 1993 by Ian Murdock to promote free software. Supported by around 1,000 developers, the project maintains software packages based on core guidelines like the Social Contract and DFSG. Debian also contributes to key initiatives like the Filesystem Hierarchy Standard and Debian Jr.

### Why Debian?
* **Stability & Predictability:** 
Debian offers a rock-solid base system with minimal background noise, ideal for high-availability production baselines.<br/>
* **Resource Optimization:** The headless netinst installation ensures zero footprint overhead from unused desktop environments.<br/>
* **Package Management Efficiency:** 
The APT package manager provides reliable dependency resolution and rapid security patching.
* **Standardization:** A clean, upstream Linux ecosystem allows for standard POSIX compliance without proprietary vendor layers.
### Constraints & Limitations
* **Package Lifecycle:** Official repositories favor stable, thoroughly tested versions over bleeding-edge releases.
* **Support Model:** Relies on community-driven support structures rather than enterprise-level commercial service level agreements (SLAs).

---

## Virtualization Infrastructure
### What ?
Virtualizationis a technique that allows you to run one or more operating systems (such as Linux, Windows or macOS) inside your own computer, along with your main system and without having to restart.<br/>
The deployment utilizes a [VirtualBox](https://www.virtualbox.org/wiki/Downloads) to abstract the physical hardware layers and isolate the operating system.

### Architectural Advantages
* **Strict Sandbox Isolation:**
* **Hardware Efficiency:** Enables granular allocation of physical CPU, memory, and storage pools dynamically shared across a single host.
* **State Preservation:** Employs precise snapshot mechanics allowing immediate system rollback to known stable states during testing and evaluation.
---

## Storage Architecture & Encryption
The storage subsystem relies on **LVM (Logical Volume Manager)** wrapped inside a secure cryptographic abstraction layer managed by LUKS.
### Partition Topology

```text
sda (12G)
├─sda1 (791M)  /boot
├─sda2 (1K)
└─sda5 (11.2G)
  └─sda5_crypt (LUKS Encrypted Volume)
    ├─maherraz42--vg-root    7.1G  /
    ├─maherraz42--vg-swap_1  620M  [SWAP]
    └─maherraz42--vg-home    3.4G  /home
```

### Cryptographic Implementation* Storage layer encryption utilizes `dm-crypt` with **AES-256-XTS-plain64** mapping.*
Logical Volume Management facilitates dynamic partition resizing and protects the kernel from system-wide failures due to single-mount directory saturation.

---

## Privilege Management
* Direct administrative `root` authentication is explicitly disabled across all interactive shells.
* Elevation is handled exclusively through a strictly audited `sudo` configuration.
* **Audit Trails:** Complete command tracking (both standard input and output state) is logged systematically to `/var/log/sudo/`.* **Environment Restraints:** TTY execution modes are strictly enforced, and default environment paths are explicitly restricted to secure system binaries.
---## Authentication Policy
System authentication properties are defined via `libpam-pwquality` to enforce enterprise-grade password compliance.
### Operational Ruleset* **Minimal Length:** 10 characters.* **Character Complexity:** Mandatory inclusion of uppercase, lowercase, and numerical characters.* **Repetition Threshold:** Maximum of 3 consecutive identical characters allowed.* **Identity Protection:** Rejection of any string matching or containing the local user login identifier.* **Lifecycle Rules:** Passwords must be rotated every 30 days, with a 2-day mandatory minimum operational lifespan between changes.* **Proactive Warning:** Active terminal notifications start 7 days prior to credential expiration.* **Difference Threshold:** Non-root password changes must differ by at least 7 structural characters from the previous entry.
---## Network & Service Security### OpenSSH Hardening* The standard daemon listener is remapped from default port 22 to **port 4242**.
* Remote root logins (`PermitRootLogin`) are explicitly denied to mitigate automated brute-force attacks.
### Firewall Configuration (UFW)* Packet filtering is enabled and enforced immediately at boot level.* **Default Policy:** Strict drop/deny policy for all unrequested ingress traffic.* **Allowed Entry:** Inbound traffic is exclusively restricted to the designated SSH control port (4242).
---## Automated Telemetry Script
A hardened Bash script (`monitoring.sh`) executes on a strict schedule to broadcast runtime infrastructure analytics across all active interactive terminal sessions via `wall`.
### Monitored Metrics* System architecture and underlying kernel version.* Count of discrete physical and virtual processor cores.* Available RAM volume and precise percentage consumption.* Storage volume layout capacities and percent usage.* Current aggregate CPU load index.* Absolute timestamp of last system initialization.* Operational state of LVM components (active status detection).* Enumeration of active TCP sockets.* Count of concurrently authenticated users.* Network interface layer identification (IPv4 and MAC addresses).
* Aggregate count of executions processed via the `sudo` subsystem.
### Automation Schedule```bash
# crontab -l
*/10 * * * * /home/maherraz/monitoring.sh
```
### Process Validation
```text
systemctl status cron
● cron.service - Regular background program processing daemon
   Loaded: loaded (/usr/lib/systemd/system/cron.service; enabled)
   Active: active (running) since ...
```
*Note: The standard system message "No MTA installed, discarding output" is expected behavior as telemetry relies purely on direct TTY broadcasting.*


## Technical Comparisons
| Module | Active Selection | Evaluated Alternative | Architectural Rationale |
| :--- | :--- | :--- | :--- |
| **OS** | Debian | Rocky Linux | Debian utilizes `apt` providing standard stability for baseline administrative setups; Rocky uses `dnf` and is geared heavily toward enterprise RHEL lifecycles. |
| **MAC** | AppArmor | SELinux | AppArmor isolates profiles natively by path strings, making rules highly readable. SELinux enforces label-based nodes, increasing security control but introducing significant rule configuration complexity. |
| **Firewall** | UFW | firewalld | UFW provides predictable, stateful firewall adjustments for standalone hosts. firewalld focuses on zoning targets meant for highly dynamic networking. |
| **Hypervisor** | VirtualBox | UTM | VirtualBox serves as the x86 cross-platform baseline standard. UTM relies natively on Apple’s Hypervisor Framework for bare-metal performance optimization on ARM hardware. |
---

## Operation Manual
This quick reference guide outlines control commands required to audit and interact with the server infrastructure during technical evaluations.
### Remote Access Authentication
```bash
ssh -p 4242 <username>@<IP_address>
```
* Replace `<username>` with the respective user context (e.g., `maherraz`).
* Query target interface parameters with `ip a` to discover active IP routing.* Direct root access attempts will fail due to security policy configurations.
### Firewall Operation (UFW)* **Inspect active filtering matrix:**
  ```bash
  sudo ufw status verbose
  ```* **Inject network access rules (e.g., open port 8080):**
  ```bash
  sudo ufw allow 8080
  ```* **Revoke rules by explicit application index or port:**
  ```bash
  sudo ufw status numbered
  sudo ufw delete <rule-number>
  ```
### System Daemon Control (systemd)* **Audit service runtime states:**
  ```bash
  sudo systemctl status ssh
  sudo systemctl status ufw
  sudo systemctl status cron
  ```* **Process lifecycle manipulation:**
  ```bash
  sudo systemctl restart ssh
  ```
### Credential Matrix Control* **Modify local user account secret keys:**
  ```bash
  passwd
  ```* **Administrative override of user credentials:**
  ```bash

sudo passwd 
```

* Review password aging metadata constraints:
bash sudo chage -l <username> 

## Account & Group Management

* Provison new user identities:
bash sudo adduser <newuser> 
* Append existing user into structural security groups:
bash sudo usermod -aG <groupname> <username> 
* Inspect user access vectors:
bash id <username> groups <username> 

## System Identity Modification

* Query active hostname metrics:
bash hostnamectl 
* Enforce persistent system hostname changes:
bash sudo hostnamectl set-hostname <new-hostname> 

## Monitoring Control

* Inspect script content:
bash cat /home/<login>/monitoring.sh 
* Inspect active crontab configuration:
bash sudo crontab -u root -l 


