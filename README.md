<h2>
  Table of Contents
</h2>

* [Introduction](#introduction)
* [Hardware Requirements](#hardware-requirements)
* [Tutorial](#tutorial)
  * [Part 1: Hardware Requirements](#part-1-hardware-requirements)
  * [Part 2: Enable IOMMU & VFIO Binding](#part-2-enable-iommu--vfio-binding)
  * [Part 3: Virtualization Setup (libvirt & KVM)](#part-3-virtualization-setup-libvirt--kvm)


<h2 id="introduction">
  Introduction
</h2>

This repository contains a **complete, tested guide** for setting up  
**GPU passthrough on Fedora laptops** using **KVM, QEMU, and VFIO**.

> ⚠️ Laptop GPU passthrough is more complex than desktop setups.  
> Back up important data before continuing.

### What This Guide Covers

- Fedora host configuration
- Laptop-specific VFIO issues
- IOMMU and GPU isolation
- libvirt and KVM setup

---

<h2 id="hardware-requirements">
  Hardware Requirements
</h2>

- **CPU with IOMMU support**
  - Intel: VT-d
  - AMD: AMD-Vi (SVM + IOMMU)
- **Two GPUs**
  - Integrated GPU (Intel/AMD) → host
  - Dedicated GPU (NVIDIA/AMD) → virtual machine
- UEFI firmware
- Secure Boot **disabled**
- External monitor (recommended)
- 16 GB RAM or more recommended

---

<h2 id="tutorial">
  Tutorial
</h2>

<h3 id="part-1-hardware-requirements">
  Part 1: Hardware Requirements
</h3>

Before proceeding, ensure:

- Virtualization is enabled in BIOS/UEFI
- IOMMU is enabled in BIOS/UEFI
- Secure Boot is disabled
- Your laptop has both an iGPU and dGPU

Do **not** continue if any requirement is missing.

---

<h3 id="part-2-enable-iommu--vfio-binding">
  Part 2: Enable IOMMU & VFIO Binding
</h3>

### Enable IOMMU in the Kernel

Edit GRUB:
```bash
$ sudo nano /etc/default/grub
```
Intel CPUs

GRUB_CMDLINE_LINUX="intel_iommu=on iommu=pt"


AMD CPUs

GRUB_CMDLINE_LINUX="amd_iommu=on iommu=pt"


Regenerate GRUB:
```bash
$ sudo grub2-mkconfig -o /boot/efi/EFI/fedora/grub.cfg
```

Reboot:
```bash
$ sudo reboot
```

Verify IOMMU:

$ dmesg | grep -i iommu


Expected:

IOMMU enabled

Identify GPU PCI IDs
$ lspci -nn


Example:

01:00.0 VGA compatible controller [0300]: NVIDIA Corporation GA107M [10de:25a2]
01:00.1 Audio device [0403]: NVIDIA Corporation GA107 Audio Controller [10de:2291]

Bind GPU to VFIO
$ sudo nano /etc/modprobe.d/vfio-pci.conf

add:

options vfio-pci ids=10de:25a2,10de:2291

Load VFIO Before NVIDIA Drivers
$ sudo nano /etc/modprobe.d/nvidia-vfio-softdep.conf

add:

softdep nvidia pre: vfio-pci
softdep nvidia_drm pre: vfio-pci

Add VFIO Drivers to Initramfs
$ sudo nano /etc/dracut.conf.d/vfio.conf

add:

add_drivers+=" vfio vfio_iommu_type1 vfio_pci "


Rebuild initramfs:

$ sudo dracut -f
$ sudo reboot

Verify VFIO Binding
$ lspci -nnk | grep -A3 -E "VGA|Audio"


Expected:

Kernel driver in use: vfio-pci

<h3 id="part-3-virtualization-setup-libvirt--kvm"> Part 3: Virtualization Setup (libvirt & KVM) </h3>
Install Virtualization Stack
$ sudo dnf update -y
$ sudo dnf install -y @virtualization

Enable & Start libvirtd
$ sudo systemctl enable --now libvirtd
$ sudo systemctl status libvirtd

Add User to Required Groups (CRITICAL)
$ sudo usermod -aG libvirt,kvm,input,disk $(whoami)
$ newgrp libvirt


Verify:

$ groups

Fix Default libvirt Network
$ sudo virsh net-list --all


If inactive:

$ sudo virsh net-start default
$ sudo virsh net-autostart default


Verify:

$ sudo virsh net-list


Expected:

default   active   yes

Verify KVM Acceleration
$ lsmod | grep kvm


Expected:

kvm_intel or

kvm_amd

Optional validation:

$ virt-host-validate

Final Status Check
$ systemctl is-active libvirtd
$ virsh net-list
$ lspci -nnk | grep -A3 VGA


✔ libvirtd running
✔ default network active
✔ GPU bound to VFIO

End

You are now ready to create a UEFI virtual machine and attach your
passthrough GPU.


