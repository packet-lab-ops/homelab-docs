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
Note: Group permissions take effect upon session logout/login.

2. Storage Pool & SELinux Permissions
KVM/libvirt enforces mandatory access controls via SELinux. ISO installation media and dynamic virtual disk images (.qcow2) are stored in the default libvirt image pool with correct file security contexts:

bash



# Move downloaded ISOs to the libvirt storage pool
sudo mv ~/Downloads/*.iso /var/lib/libvirt/images/

# Correct ownership and SELinux contexts
sudo chown root:root /var/lib/libvirt/images/*.iso
sudo restorecon -Rv /var/lib/libvirt/images
Installation Media Acquired
Windows Server 2022 Datacenter Evaluation (ISO)
Windows 11 Enterprise Evaluation (ISO)
Ubuntu Server 24.04 LTS (Minimized) (ISO)
VirtIO Windows Driver Media (virtio-win.iso from Fedora Project)
3. Virtual Networking Architecture (virbr0)
The host uses libvirt's default NAT virtual bridge:

Interface: virbr0
Subnet: 192.168.122.0/24
Gateway / DNS Forwarder: 192.168.122.1
Default DHCP Pool: Handled internally by dnsmasq managed by libvirtd
Isolation: Guest VMs are firewalled behind host NAT; they can establish outbound internet access but outside LAN/WAN devices cannot initiate inbound traffic to VM subnets.
Network persistence was verified via terminal:

bash



sudo virsh net-list --all
sudo virsh net-autostart default
4. Hardware Optimization for Windows Guests (VirtIO)
Standard emulated IDE/SATA controllers introduce unnecessary CPU overhead. Windows guest VMs were provisioned using paravirtualized Red Hat VirtIO storage and network drivers:

Storage Controller: VirtIO Block / SCSI Bus (high-throughput, low-latency disk I/O).
Network Interface: virtio NIC (paravirtualized gigabit interface).
Firmware / Security: Emulated TPM 2.0 (required for Windows 11 guest validation).
Driver Injection: During Windows initial setup, the Red Hat VirtIO SCSI controller driver was loaded from virtio-win.iso (amd64 directory) to expose the virtual storage target.
Guest Integration: Post-boot execution of virtio-win-gt-x64.exe to deploy display drivers, QEMU Guest Agent, and automatic window resolution scaling.
5. Verification & Screenshots
Screenshot	Description	File Path
01	KVM hypervisor hardware specifications and VM resource allocation	../screenshots/01-kvm-vm-specs.png
03	Paravirtualized Red Hat VirtIO storage driver loaded during OS install	../screenshots/03-virtio-storage-driver.png
05	Complete multi-VM environment active on Fedora host	../screenshots/05-enterprise-lab-overview.png
6. Key Takeaways
Paravirtualization vs. Full Emulation: Implementing VirtIO drivers demonstrates how modern hypervisors maximize guest performance by bypassing emulated physical controller limits.
Enterprise Red Hat Alignment: KVM and libvirt form the foundation of enterprise virtualization (RHEL, oVirt, Proxmox VE), offering direct hands-on experience with native Linux infrastructure.
