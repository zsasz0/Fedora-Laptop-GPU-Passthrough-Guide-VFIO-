# Fedora Laptop GPU Passthrough (VFIO)

This repository contains a **complete, tested guide** for setting up
GPU passthrough on **Fedora laptops** using KVM, QEMU, and VFIO.

>  Laptop GPU passthrough is more complex than desktop setups.
> Read carefully and **back up your data**.

## What This Guide Covers
- Fedora host setup
- Laptop-specific VFIO issues
- IOMMU & GPU isolation
- libvirt + virt-manager
- Windows VM (UEFI)
- Looking Glass
- Troubleshooting

## Start
 [Requirements](docs/01-requirements.md)
## configure GRUB, enable IOMMU, and Bind the dedicated GPU to vfio-pci and verify isolation .
 [vfio-setup](docs/vfio-setup.md)

## Who This Is For
- Fedora users
- Gaming / GPU compute in VMs
- Laptop owners with iGPU + dGPU
