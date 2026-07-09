# Dell-EMC-Server-Infrastructure---CNCF-Kube-Vert-Project
# 🖥️ Dell EMC Server Infrastructure

A virtualized server setup running **KubeVirt** on Ubuntu, providing isolated Linux and Windows desktop environments to thin clients via **NComputing** — secured with SSH 2FA + CAPTCHA.

---

## 📋 Table of Contents

- [Architecture Overview](#architecture-overview)
- [System Flow](#system-flow)
- [Components](#components)
  - [Host OS](#host-os)
  - [Virtualization Layer](#virtualization-layer)
  - [Client Access](#client-access)
  - [Remote Management](#remote-management)
- [Tools & Dependencies](#tools--dependencies)
- [Infrastructure](#infrastructure)
- [Security](#security)
- [Getting Started](#getting-started)
- [Prerequisites](#prerequisites)

---

## Architecture Overview

```
DELL EMC Server
└── Ubuntu (main OS)
    ├── SSH Access (2FA + CAPTCHA)
    └── KubeVirt (virtualization layer)
        ├── Ubuntu VM  ──(XRDP)──► NComputing thin clients
        └── Windows VM ──(RDP)───► NComputing thin clients
```

The server runs Ubuntu as the host OS, with KubeVirt managing two virtual machines — one Ubuntu (Linux desktop) and one Windows — both accessible to end users through NComputing thin clients.

---

## System Flow

```
                        ┌──────────────────┐
                        │  DELL EMC Server │
                        └────────┬─────────┘
                                 │
                        ┌────────▼─────────┐        ┌──────────────────────┐
                        │  Ubuntu (main OS)├───────►│  SSH Access          │
                        └────┬─────────┬───┘        │  2FA + CAPTCHA login │
                             │         │             └──────────────────────┘
                          Ubuntu    Windows
                            VM        VM
                        ┌───▼─────────▼───────────────────────┐  ┌──────────────┐
                        │     KubeVirt (virtualization layer)  ├─►│ KubeVirt GUI │
                        │  ┌──────────────┐ ┌───────────────┐ │  │ Start/Stop   │
                        │  │  Ubuntu VM   │ │  Windows VM   │ │  │ CPU / RAM    │
                        │  │  Linux desktop│ │ Win desktop  │ │  │ Console      │
                        │  └──────┬───────┘ └──────┬────────┘ │  └──────────────┘
                        └─────────│────────────────│──────────┘
                                XRDP              RDP
                        ┌─────────▼────────────────▼──────────┐
                        │           NComputing                 │
                        │  Thin clients — connect via GUI      │
                        └──────────────────────────────────────┘
```

---

## Components

### Host OS

| Component | Details |
|-----------|---------|
| **Hardware** | Dell EMC Server |
| **Operating System** | Ubuntu (main OS) |
| **SSH Access** | Secured with Two-Factor Authentication (2FA) + CAPTCHA |

Ubuntu serves as the host operating system on top of the Dell EMC hardware. All SSH logins require both 2FA and CAPTCHA verification to prevent unauthorized access.

---

### Virtualization Layer

**KubeVirt** handles the virtualization layer, running two VMs concurrently on the host Ubuntu system.

#### Ubuntu VM
- **Environment:** Linux desktop environment
- **Protocol:** XRDP (for remote desktop access)
- **Use case:** Linux-based user sessions and workflows

#### Windows VM
- **Environment:** Windows desktop environment
- **Protocol:** RDP (Remote Desktop Protocol)
- **Use case:** Windows-based user sessions and applications

#### KubeVirt GUI
A web-based management interface exposing the following controls:
- **Start / Stop** — power management for each VM
- **CPU / RAM** — resource monitoring and allocation
- **Console** — direct VM console access

---

### Client Access

**NComputing** thin clients connect to the virtual machines through the GUI:

- Linux users connect to the **Ubuntu VM** via **XRDP**
- Windows users connect to the **Windows VM** via **RDP**

This allows multiple users to share server resources without requiring dedicated hardware per user.

---

### Remote Management

| Interface | Purpose |
|-----------|---------|
| **KubeVirt GUI** | VM lifecycle management (start/stop, resources, console) |
| **SSH** | Host OS administration (secured with 2FA + CAPTCHA) |

---

## Tools & Dependencies

| # | Tool | Purpose |
|---|------|---------|
| 1 | **Docker** | Container runtime, base for K3S and KubeVirt |
| 2 | **K3S** | Lightweight Kubernetes distribution |
| 3 | **Kube-Virt** | VM management on top of Kubernetes |
| 4 | **Kube-Virt GUI** | Web UI for managing virtual machines |
| 5 | **NComputing** | Thin client software for multi-user VM access |
| 6 | **XRDP** | Remote desktop gateway for the Ubuntu VM |
| 7 | **SSH 2FA + CAPTCHA** | Secure host OS access |

---

## Infrastructure

| Component | Description |
|-----------|-------------|
| **Server Hardening** | OS-level security hardening applied to the Ubuntu host |
| **RAID 1** | Disk mirroring for data redundancy and fault tolerance |
| **DHCP** | Automatic IP address assignment for network devices |
| **DNS** | Domain name resolution within the network |

---

## Security

This setup implements multiple security layers:

- **SSH hardening** — 2FA (Two-Factor Authentication) and CAPTCHA required for every SSH login
- **VM isolation** — Each VM runs in an isolated KubeVirt environment
- **Server hardening** — The Ubuntu host OS is hardened at the OS level
- **RAID 1** — Disk mirroring protects against data loss from hardware failure
- **Network controls** — DHCP and DNS managed internally for network control

---

## Getting Started

### Prerequisites

Make sure the following are installed and configured on your Dell EMC server running Ubuntu:

- Ubuntu Server (LTS recommended)
- Docker
- K3S (Lightweight Kubernetes)
- KubeVirt operator
- NComputing software
- XRDP (for the Ubuntu VM)

### Installation Order

```bash
# 1. Install Docker
curl -fsSL https://get.docker.com | sh

# 2. Install K3S
curl -sfL https://get.k3s.io | sh -

# 3. Install KubeVirt
export KUBEVIRT_VERSION=$(curl -s https://api.github.com/repos/kubevirt/kubevirt/releases/latest | grep tag_name | cut -d '"' -f 4)
kubectl apply -f https://github.com/kubevirt/kubevirt/releases/download/${KUBEVIRT_VERSION}/kubevirt-operator.yaml
kubectl apply -f https://github.com/kubevirt/kubevirt/releases/download/${KUBEVIRT_VERSION}/kubevirt-cr.yaml

# 4. Install KubeVirt GUI (virtink or kubevirt-manager)
# Follow the official KubeVirt UI documentation for your preferred GUI

# 5. Set up XRDP on the Ubuntu VM
sudo apt install xrdp -y
sudo systemctl enable xrdp

# 6. Configure SSH 2FA + CAPTCHA
# Install Google Authenticator PAM module
sudo apt install libpam-google-authenticator -y
# Follow PAM configuration steps for SSH
```

> **Note:** Refer to the official documentation for each tool for detailed configuration and production hardening.

---

## License

This project is for internal infrastructure use. Modify as needed for your environment.
