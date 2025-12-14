Virtualization Setup (libvirt & KVM)

This section installs and configures libvirt, KVM, and networking required for GPU passthrough on Fedora.

1. Install Virtualization Stack

Update the system:

$ sudo dnf update -y


Install Fedora’s virtualization group:

$ sudo dnf install -y @virtualization


This installs:

QEMU / KVM

libvirt

virt-manager

OVMF (UEFI firmware)

Virtual networking tools

2. Enable & Start libvirtd

Enable and start the libvirt daemon:

$ sudo systemctl enable --now libvirtd


Verify status:

$ sudo systemctl status libvirtd


The service should be active (running).

3. Add User to Required Groups (CRITICAL)

Add your user to required groups:

$ sudo usermod -aG libvirt,kvm,input,disk $(whoami)


Apply group changes:

$ newgrp libvirt


(Or log out and log back in.)

Verify:

$ groups

4. Fix Default libvirt Network (Very Common Issue)

List libvirt networks:

$ sudo virsh net-list --all


If the default network is inactive or missing, start it:

$ sudo virsh net-start default
$ sudo virsh net-autostart default


Verify:

$ sudo virsh net-list


Expected output:

Name      State    Autostart
--------------------------------
default   active   yes

5. Verify KVM Acceleration

Check loaded kernel modules:

$ lsmod | grep kvm


You should see one of the following:

kvm_intel (Intel CPUs)

kvm_amd (AMD CPUs)

Optional full validation:

$ virt-host-validate


Important checks should report PASS.

Quick Verification Checklist
$ systemctl is-active libvirtd
$ virsh net-list
$ lsmod | grep kvm


✔ libvirtd running
✔ default network active
✔ KVM acceleration enabled

Notes for Laptop Users

Do not remove the default NAT network

Bridged networking is optional

Wayland is recommended on the host
