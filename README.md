# Virtual Home SOC Lab

A reproducible, documented home Security Operations Center (SOC) lab built with **VMware Workstation**.  
This repo describes how I downloaded and configured the VMs, set up an isolated NAT network on the **10.0.0.0/24** subnet with DHCP starting at **10.0.0.10**, and solved common hiccups encountered during the build.

---

## Table of contents
- [Overview](#overview)  
- [Objectives](#objectives)  
- [Prerequisites](#prerequisites)  
- [Files & repo layout](#files--repo-layout)  
- [VMware NAT / Network configuration](#vmware-nat--network-configuration)  
- [VM list & recommended hardware](#vm-list--recommended-hardware)  
- [Step-by-step build runbook](#step-by-step-build-runbook)  
- [Post-install checks & useful commands](#post-install-checks--useful-commands)  
- [Troubleshooting & hiccups I encountered](#troubleshooting--hiccups-i-encountered)  
- [Snapshots, backups & safety notes](#snapshots-backups--safety-notes)  
- [Resume / LinkedIn friendly summary](#resume--linkedin-friendly-summary)  
- [Appendix: netplan example, Wazuh basics, .gitignore](#appendix-netplan-example-wazuh-basics-gitignore)

---

## Overview
This repository documents the build of a small, multi-VM SOC lab for practicing offensive and defensive operations, malware analysis, and SIEM log ingestion. The lab **network is isolated** via VMware NAT and uses a 10.0.0.0/24 address space to keep the host and VMs segmented from other networks.

**Key software used**
- VMware Workstation (host)
- Kali Linux (attacker)
- Metasploitable (vulnerable target)
- Ubuntu (Wazuh Manager / infrastructure)
- Wazuh (detection & SIEM)
- Flare VM (Windows analysis)

---

## Objectives
- Create an isolated environment for safely practicing scanning, exploitation, and detection.
- Centralize logs and telemetry in Wazuh for analysis.
- Document the setup and troubleshooting steps so it’s reproducible.

---

## Prerequisites
- Windows host with VMware Workstation (Pro or Player with Virtual Network Editor access).
- >= 100 GB free disk space recommended, and 8–16+ GB RAM (more is better).
- ISOs and VM images saved in a dedicated folder (e.g., `C:\VMs\SOC-Lab`).

**Files you should download before starting**
- Kali Linux ISO (VM-ready)
- Metasploitable2 VM image
- Ubuntu ISO (22.04 LTS or current LTS)
- Flare VM (or Flare installer script)
- Wazuh install instructions/packages (we install manager on Ubuntu)

---

## Files & repo layout (suggested)

---

## VMware NAT / Network configuration
I used VMware Workstation's Virtual Network Editor to create/modify the NAT network (VMnet8):

1. Open **VMware Workstation** → `Edit` → **Virtual Network Editor`.
2. Select the NAT network (commonly `VMnet8`) or create a new NAT network.
3. Set:
   - **Subnet IP**: `10.0.0.0`
   - **Subnet mask**: `255.255.255.0`
4. Configure DHCP:
   - **Start IP**: `10.0.0.10`
   - **End IP**: `10.0.0.200` (or other range you prefer)
5. Save changes.
6. Reboot or toggle network adapters on running VMs so they get new leases.

> **Note:** VMware assigns a gateway IP on the subnet (e.g., `10.0.0.2`) — verify in NAT settings if you need the exact gateway.

### Simple ASCII network diagram

---

## VM list & recommended hardware
Tweak to match your host capacity.

### Kali Linux (Attacker)
- vCPU: 2
- RAM: 4–6 GB
- Disk: 40 GB
- Network: NAT (VMnet8)

### Metasploitable (Vulnerable target)
- vCPU: 1
- RAM: 1–2 GB
- Disk: 8–20 GB (image)
- Network: NAT (VMnet8)

### Ubuntu (Wazuh Manager / SIEM host)
- vCPU: 2
- RAM: 4–8 GB
- Disk: 40–60 GB
- Network: NAT (VMnet8)

### Wazuh Agent (optional separate VM / or install agent on existing VMs)
- vCPU: 1
- RAM: 1–2 GB
- Disk: 20 GB
- Network: NAT (VMnet8)

### Flare VM (Windows analysis)
- vCPU: 2
- RAM: 4–8 GB
- Disk: 60–80 GB
- Network: NAT (VMnet8)

> **Tip:** Start only the VMs you need during a test to conserve host resources.

---

## Step-by-step build runbook
1. Create `C:\VMs\SOC-Lab` (store ISOs and notes).
2. Configure VMware NAT network as described above.
3. Create new VM for each image:
   - In VM wizard select ISO (or import VM image).
   - Allocate CPU/RAM/disk per recommended configs.
   - Set `Network Adapter` to **NAT** and confirm VMnet (VMnet8).
4. Install OS and fully update (`apt update && apt upgrade` on Linux, Windows Update on Windows).
5. Install VMware Tools:
   - Linux: `sudo apt install open-vm-tools -y`
   - Windows: Install VMware Tools from the VM menu.
6. Snapshot: take a snapshot after a clean OS install.
7. Install Wazuh Manager on Ubuntu (follow Wazuh docs then snapshot).
8. Install Wazuh agents on other VMs and register them to the manager.
9. Verify connectivity: `ping`, `ip addr`, `ipconfig` as needed.
10. Run test activities (scans from Kali, exploit Metasploitable, generate host changes) and confirm Wazuh receives alerts.

---

## Post-install checks & useful commands

### Check IP and routes (Linux)
```bash
ip addr show
ip route show
ping -c 3 10.0.0.10   # quick ping to test DHCP-assigned address
