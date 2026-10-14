# KVM / QEMU Hypervisor & Virtualization Setup

## Objective
Deploy a lean, enterprise-grade virtualization host on bare-metal Fedora Workstation using native Linux kernel virtualization (KVM), QEMU hardware emulation, and `libvirt` management tools. This environment provides the hypervisor infrastructure for enterprise Windows Server, Windows workstation, and Linux server virtual machines.

---

## Hardware & Host Specifications

| Component | Specification |
| :--- | :--- |
| **Host System** | Dell Inspiron 23 AIO |
| **Processor** | Intel x86_64 |
| **Host Operating System** | Fedora Workstation 4x |
| **Host Memory** | 16 GB RAM |
| **Hypervisor Stack** | KVM (Kernel-based Virtual Machine) + QEMU |
| **Management Daemon / API** | `libvirtd` |
| **Management GUI** | Virtual Machine Manager (`virt-manager`) |
| **CLI Provisioning** | `virt-install` |

---

## 1. Hypervisor Installation & Configuration

To maintain a minimal footprint without desktop virtualization bloat, only the essential virtualization binaries and management utilities were installed:

```bash
# 1. Install minimal KVM/QEMU and libvirt packages
sudo dnf install qemu-kvm libvirt virt-manager virt-install

# 2. Enable and start the libvirt daemon
sudo systemctl enable --now libvirtd

# 3. Add current user to the libvirt group for non-root management
sudo usermod -aG libvirt $(whoami)
