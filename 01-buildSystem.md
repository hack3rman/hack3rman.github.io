## Creating a build system for armhf in qemu

This document should support in running an armhf build system on windows. The commands however should work on any architecture targeting any system more or less. 

Booting a system through QEMU involves a few more steps that just installing a distribution with VirtualBox. With QEMU being a hardware virtualizer, there are a lot of options for peripherals and devices. The whole emulation also needs a way to get to main kernel of the OS. As I understand it, the first step when powering up any system is an initial boot code that does the initialization. This needs to be specific since it needs to initialize the hardware that is present on the system which might have different options, running configuration and so forth. This initial boot code can then proceed in loading further firmware or bootloaders. In many case, the next firmware is UEFI, and attempt to standardize firmware for computer platforms. 

I have used two modes with QEMU
1. Having two UEFI devices mounted. The UEFI image is usually already built and found in the distribtion we are installing. QEMU will use these devices to execute the UEFI code. UEFI will then check the main block devices for any kernel images that will be loaded.
2. QEMU can boot the kernel directly but this means that the kernel image must be provided separately from the root image. This kernel image can be extracted from the root image.

This guide will follow installing Debian on a Windows host.

### Steps to build the system
Install MSYS2 [doc](https://www.msys2.org/docs/installer/)

Install QEMU from within the MSYS2 environment [doc](https://www.qemu.org/download/#windows)

Download deb packages for OVMF [link](https://packages.debian.org/bookworm/ovmf). These are UEFI specialized images for QEMU. Unzip the deb and look for two files: `OVMF.fd` and `OVMF_VARS_4M.fd`. One of them is the UEFI code and the other one is a special space in flash space where UEFI stores variables and allows for configuration to be modified and recalled.

The next step is to create raw image data from these built images.
```
dd of="QEMU_EFI-pflash.raw" if="/dev/zero" bs=1M count=64
dd of="QEMU_EFI-pflash.raw" if="OVMF.fd" conv=notrunc
dd of="QEMU_VARS-pflash.raw" if="/dev/zero" bs=1M count=64
dd of="QEMU_VARS-pflash.raw" if="OVMF_VARS_4M.fd" conv=notrunc
```

We also create a qcow2 image that will hold the root filesystem.
```
qemu-img create -f qcow2 debian32.img 128G
```

