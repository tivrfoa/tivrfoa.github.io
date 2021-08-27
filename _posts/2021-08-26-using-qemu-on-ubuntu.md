---
layout: post
title:  "Using QEMU on Ubuntu"
date:   2021-08-26 09:15:44 -0300
categories: linux qemu virtual machine
---


>What is QEMU?
>
>QEMU is a generic and open source machine emulator and virtualizer.
>
>source: [https://www.qemu.org/](https://www.qemu.org/)


## Install a Virtual Machine Manager GUI for KVM

`sudo apt-get install virt-manager`

[https://www.ubuntubuzz.com/2016/05/how-to-install-kvm-with-gui-virt-manager-in-ubuntu.html](https://www.ubuntubuzz.com/2016/05/how-to-install-kvm-with-gui-virt-manager-in-ubuntu.html)


## How To Install Ubuntu in QEMU/KVM Virtual Machine

[https://www.ubuntubuzz.com/2016/05/how-to-install-ubuntu-in-qemukvm-virtual-machine.html](https://www.ubuntubuzz.com/2016/05/how-to-install-ubuntu-in-qemukvm-virtual-machine.html)

## What is QCOW2?

>QCOW2 is a storage format for virtual disks. QCOW stands for QEMU copy-on-write. The QCOW2 format decouples the physical storage layer from the virtual layer by adding a mapping between logical and physical blocks. Each logical block is mapped to its physical offset, which enables storage over-commitment and virtual machine snapshots, where each QCOW volume only represents changes made to an underlying virtual disk.
>
>The initial mapping points all logical blocks to the offsets in the backing file or volume. When a virtual machine writes data to a QCOW2 volume after a snapshot, the relevant block is read from the backing volume, modified with the new information and written into a new snapshot QCOW2 volume. Then the map is updated to point to the new place.

source: [https://access.redhat.com/solutions/641193](https://access.redhat.com/solutions/641193)

## from: [Kernel Recipes 2015 - Speed up your kernel development cycle with QEMU - Stefan Hajnoczi](https://www.youtube.com/watch?v=PBY9l97-lto)

### How to boot a development kernel

```sh
qemu-system-x86_64 -enable-kvm -m 1024 \
    -kernel /boot/vmlinuz \
    -initrd /boot/initrd.img \
    -append 'console=ttyS0' \
    -nographic
```

`-m 1024` means 1GB RAM

In the video, `initramfs.img` is used, but on my machine it's `initrd.img`

This text showed after running the commmand above:

`No root device specified. Boot arguments must include a root= parameter.`

Some things that may solve it, but I didn't investigate it further yet ...:

[Unable to run linux kernel image on qemu](https://stackoverflow.com/questions/18441102/unable-to-run-linux-kernel-image-on-qemu)

### Build process

Compile your kernel modules:

```sh
make M=drivers/virtio \
	CONIG_VIRTIO_PCI=m modules
```

Build initramfs:

```sh
usr/gen_init_cpio input | gzip >initramfs.img
```

Run virtual machine:

```sh
qemu-system-x86_64 -m 1024 -enable-kvm \
	-kernel arch/x86_64/boot/bzImage \
	-initrd initramfs.img \
	-append 'console=ttyS0' \
	-nographic
```

## initramfs vs initrd

[The difference between initrd and initramfs?](https://stackoverflow.com/a/10603984/339561)



## How to use custom image kernel for ubuntu in qemu?

[https://stackoverflow.com/questions/65951475/how-to-use-custom-image-kernel-for-ubuntu-in-qemu](https://stackoverflow.com/questions/65951475/how-to-use-custom-image-kernel-for-ubuntu-in-qemu)

### References

[https://www.qemu.org/download/](https://www.qemu.org/download/)

[https://qemu-project.gitlab.io/qemu/](https://qemu-project.gitlab.io/qemu/)

[https://wiki.qemu.org/Documentation](https://wiki.qemu.org/Documentation)

[https://gitlab.com/qemu-project/qemu/-/tree/master/docs](https://gitlab.com/qemu-project/qemu/-/tree/master/docs)

[https://www.qemu-advent-calendar.org](https://www.qemu-advent-calendar.org)

[How to run Ubuntu desktop on QEMU?](https://askubuntu.com/questions/884534/how-to-run-ubuntu-desktop-on-qemu)
