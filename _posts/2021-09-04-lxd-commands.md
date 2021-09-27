---
layout: post
title:  "LXD Commands"
date:   2021-09-04 09:15:44 -0300
categories: linux container vm lxd lxc
---

## What is LXD?

>LXD is a next generation system container and virtual machine manager. It offers a unified user experience around full Linux systems running inside containers or virtual machines.
>
> [https://linuxcontainers.org/lxd/introduction/](https://linuxcontainers.org/lxd/introduction/)

> - it's a container manager;
> - it allows you to run Linux Containers (LXC)
> - But it gives you additional features, such as clustering
>
> [Getting started with LXD Containerization (Full Guide!)](https://www.youtube.com/watch?v=aIwgPKkVj8s)

## Installation on Ubuntu

`sudo snap install lxd`

For other distributions, see [https://linuxcontainers.org/lxd/getting-started-cli/#installation](https://linuxcontainers.org/lxd/getting-started-cli/#installation)

## Initial Configuration

`sudo lxd init`


### Storage backend to use

>I'm not going to choose zfs in my case. That's highly recommended if
> you can choose that or if you have the hardware to support that. I
> would actually need another dedicated drive in order to effectively
> manage zfs. So what I'm going to do instead is set it to `directory` or `dir`
>for to use a directory on the file system. Again, zfs is highly recommended,
>but it's better to use zfs with a dedicated block device which I don't currently
>have available.
>
>  - LearnLinuxTV - [Getting started with LXD Containerization (Full Guide!)](https://www.youtube.com/watch?v=aIwgPKkVj8s)

## List remote servers

`lxc remote list`

## List images available from the server 'images'

`lxc image list images:`

## List images that have 'debian' in the name

`lxc image list images: debian`

## Create Ubuntu image VM

```sh
lxc launch images:ubuntu/21.10/desktop ubuntu2110 --vm -c limits.cpu=4 -c limits.memory=4GiB --console=vga
```

`--console=vga` immediately get the video output once the instance is created.

`--vm` tells it's a virtual machine. To create a container instead, run:<br>

```sh
lxc launch images:ubuntu/21.10 ubuntu-21.10-container
```

If Ubuntu asks you for a password, then open a terminal on your host machine and set it with:<br>
`lxc exec ubuntu2110 -- passwd ubuntu`

source: [https://askubuntu.com/questions/932976/what-are-root-and-ubuntu-password-in-lxc-ubuntu16-04-container](https://askubuntu.com/questions/932976/what-are-root-and-ubuntu-password-in-lxc-ubuntu16-04-container)

## Create Arch Linux

```sh
lxc launch images:archlinux/desktop-gnome archlinux --vm -c limits.cpu=4 -c limits.memory=4GiB -c security.secureboot=false --console=vga
```

>Arch Linux is a bit different, because they don't really have a default desktop environment, so as a result the variant there is called `desktop-gnome`.
>
>The other difference with Arch Linux is that it is not signed with a valid secure boot key, which means we need to disable secure boot so that the VM can actually boot.
>  - Stéphane Graber [Arch Linux and Ubuntu Desktop in LXD VMs](https://youtu.be/pEUsTMiq4B4?t=379)

## List containers and virtual machines

```sh
lxc list

+------------+---------+------+------+-----------------+-----------+
|    NAME    |  STATE  | IPV4 | IPV6 |      TYPE       | SNAPSHOTS |
+------------+---------+------+------+-----------------+-----------+
| archlinux  | STOPPED |      |      | VIRTUAL-MACHINE | 0         |
+------------+---------+------+------+-----------------+-----------+
| ubuntu2110 | STOPPED |      |      | VIRTUAL-MACHINE | 0         |
+------------+---------+------+------+-----------------+-----------+
```

## List your local images

`lxc image list local:`

| ALIAS | FINGERPRINT  | PUBLIC |               DESCRIPTION                | ARCHITECTURE |      TYPE       |   SIZE    |         UPLOAD DATE          |
|       | 71ba5f5629fe | no     | Ubuntu impish amd64 (20210902_07:42)     | x86_64       | VIRTUAL-MACHINE | 1528.84MB | Sep 3, 2021 at 11:42am (UTC) |
|       | d4341a461a32 | no     | Archlinux current amd64 (20210904_04:18) | x86_64       | VIRTUAL-MACHINE | 1262.39MB | Sep 4, 2021 at 2:15pm (UTC)  |

As you can see, by they default they don't have an alias, so let's create an alias for them!

## Creating an alias for an existing image

>lxc image alias create
>Description:
>  Create aliases for existing images
>
>Usage:
>  lxc image alias create [<remote>:]<alias> <fingerprint> [flags]

```sh
lxc image alias create local:ubuntu-21-10 71ba5f5629fe
```

You now can create containers/VM using this image!<br>
No need to download gigabytes again ...

## Creating a VM (I thought it would create a container ... :/) from a local image

```sh
$ lxc launch local:ubuntu-21-10 ubuntu-21-10-container
Creating ubuntu-21-10-container
Starting ubuntu-21-10-container
```

It has the same type as the base image ...<br>
It shows it as a Virtual Machine ...

## Execute a command in the running containter/VM

```sh
lxc exec instance-name -- apt install apache2
```

## Get instance info

```sh
$ lxc info ubuntu2110
Name: ubuntu2110
Status: STOPPED
Type: virtual-machine
Architecture: x86_64
Created: 2021/09/03 11:42 UTC
Last Used: 2021/09/04 11:02 UTC
```

## Start instance

```sh
lxc start instace-name
```

## Stop instance

```sh
lxc stop instace-name
```

## Access VM shell

```sh
lxc console ubuntu2110
```

## Access VM GUI

```sh
lxc console ubuntu2110 --type=vga
```

## How do I copy a file from host into a LXD container?

[https://discuss.linuxcontainers.org/t/how-do-i-copy-a-file-from-host-into-a-lxd-container/2066](https://discuss.linuxcontainers.org/t/how-do-i-copy-a-file-from-host-into-a-lxd-container/2066)

```sh
lxc file push myfile.txt mycontainer/home/ubuntu/
```

## List Storages

```sh
$ lxc storage list
```

```txt
+---------+--------+--------------------------------------------+-------------+---------+
|  NAME   | DRIVER |                   SOURCE                   | DESCRIPTION | USED BY |
+---------+--------+--------------------------------------------+-------------+---------+
| default | zfs    | /var/snap/lxd/common/lxd/disks/default.img |             | 5       |
+---------+--------+--------------------------------------------+-------------+---------+
```

## Describe storage

```sh
$ lxc storage show default
```

```txt
To start your first instance, try: lxc launch ubuntu:18.04

config:
  size: 20GB
  source: /var/snap/lxd/common/lxd/disks/default.img
  zfs.pool_name: default
description: ""
name: default
driver: zfs
used_by:
- /1.0/images/20512f15fed33bb25a83fd4ee9a6aa2fd72d8bb97c1d3323007580a1fee7c619
- /1.0/images/71ba5f5629fe4489309fb2948467976e3a1cfd1446e692c40709f4d9778ee741
- /1.0/instances/archlinux
- /1.0/instances/ubuntu2110
- /1.0/profiles/default
status: Created
locations:
- none
```

## Create an snapshot

>Snapshots are very useful if you want to try something out,
>maybe you want to install a piece of software, see how it
>affects the container and then if you don't like the changes,
>roll that back as if nothing ever happened.
>
>  - LearnLinuxTV - [Getting started with LXD Containerization (Full Guide!)](https://www.youtube.com/watch?v=aIwgPKkVj8s)

```sh
lxc snapshot instance-name snapshot-name
```

## Delete snapshot

```sh
lxc delete instance-name/snapshot-name
```

## Restore snapshot

```sh
lxc restore instance-name snapshot-name
```

## Start the container automatically

```sh
lxc config set instance-name boot.autostart 1
```

## Setting ceiling for memory

```sh
lxc config set instance-name limits.memory 1GB
```

## Show container configuration

```sh
lxc config show instance-name

architecture: x86_64
config:
  image.architecture: amd64
  image.description: Ubuntu impish amd64 (20210902_07:42)
  image.os: Ubuntu
  image.release: impish
  image.serial: "20210902_07:42"
  image.type: disk-kvm.img
  image.variant: desktop
  limits.cpu: "4"
  limits.memory: 4GiB
  volatile.base_image: 71ba5f5629fe4489309fb2948467976e3a1cfd1446e692c40709f4d9778ee741
  volatile.eth0.hwaddr: 00:16:3e:b7:ef:f4
  volatile.last_state.power: STOPPED
  volatile.uuid: 5283dec6-3083-4999-b11b-1fd9bef3257b
  volatile.vsock_id: "4"
devices: {}
ephemeral: false
profiles:
- default
stateful: false
description: ""
```

## TODO change storage location

By default it creates images in `/var/snap/lxd/common/lxd/disks`,
but I don't have much space in `/`, so I need to find out how
to create the images in `/home`.

Maybe there's info on how to do it here:<br>
[Storage configuration](https://linuxcontainers.org/lxd/docs/master/storage.html)

## Creating a new storage pool

[LXD Change Storage Location](https://discuss.linuxcontainers.org/t/lxd-change-storage-location/3749)

```sh
$ lxc storage create sdb1 btrfs source=/home/lesco/dev/lxd/sdb1
```

*Error: Provided path does not reside on a btrfs filesystem*

## Storage pool vs Storage volume

From **Simos Xenitellis** - [Storage volumes vs disk devices: questions](https://discuss.linuxcontainers.org/t/storage-volumes-vs-disk-devices-questions/3374):

*A storage volume 26 is disk space that has been allocated from a storage pool, to be used by a container. Container images are also stored in storage volumes.*

*To list your storage pools, run the following. Here, there is a single storage pool which happens to be called lxd.*

```
$ lxc storage list
```

| NAME | DESCRIPTION | DRIVER | SOURCE | USED BY |
| lxd  |             | zfs    | lxd    | 6       |

*Let’s list the storage volumes in the storage pool. I have not created a separate storage volume, so we are seeing the volumes created by LXD for my containers, plus the storage volumes for any cached container images. There are two containers and three container images, in total five. I do not know why lxc storage list lxd shows six instead of five. Probably because when I sudo zfs list, I can see a deleted image so that may account for the sixth storage volume.*

```
$ lxc storage volume list lxd
```

|   TYPE    |                               NAME                               | DESCRIPTION | USED BY |
| container | c1                                                               |             | 1       |
| container | c2                                                               |             | 1       |
| image     | 018d083aec1332a90bf9ae851c9790a17cfe51a2a83ed4a2d041956363a6aada |             | 1       |
| image     | 2e4d679ce33b8f1ec2ee4d82cd2a413093e978d037a84b3b30a11161f679d2fc |             | 1       |
| image     | 8f9da4cd832ba0235749caa2249c1ecfcee0cee052c4647fb502955fcec70072 |             | 1       |

<i>But what are these container images? Can we easily see what those hashes correspond to?
We do that with lxc image list. Indeed, the hashes correspond with the hashes of the storage volumes.</i>

```
$ lxc image list
```

|                ALIAS                 | FINGERPRINT  | PUBLIC |                    DESCRIPTION                    |  ARCH  |   SIZE   |          UPLOAD DATE          |
| openwrt-chaos_calmer-20181120-212658 | 2e4d679ce33b | no     | openwrt 15.05.1 x86_64 (default) (20181120_21:26) | x86_64 | 2.17MB   | Nov 20, 2018 at 7:29pm (UTC)  |
|                                      | 018d083aec13 | no     | ubuntu 16.04 LTS amd64 (release) (20181114)       | x86_64 | 158.12MB | Nov 15, 2018 at 11:03pm (UTC) |
|                                      | 8f9da4cd832b | no     | ubuntu 18.04 LTS amd64 (release) (20181101)       | x86_64 | 174.44MB | Nov 13, 2018 at 12:19am (UTC) |

*In retrospect, a disk device 21 is disk space from the host filesystem that has been shared to a container.*

## TODO run a custom Linux kernel

## Run ISO with LXD

>Absolutely. For this you'd follow instructions similar to what I showed in the Windows 11 video.<br>
>Essentially:
> - lxc init some-name --vm -c limits.cpu=4 -c limits.memory=4GiB
> - lxc config device override some-name root size=50GiB
> - lxc config device add some-name install disk source=/path/to/the/iso/file boot.priority=10
> - lxc start some-name --console=vga
>
>This creates a blank instance with 4 vCPU and 4GiB of RAM, grows its disk to 50GiB and then attaches a cdrom drive with your ISO. It then starts it with you attached to the VGA console so you can go through the install process.
>
>  - Stéphane Graber comment on [Arch Linux and Ubuntu Desktop in LXD VMs](https://www.youtube.com/watch?v=pEUsTMiq4B4)

## References

[Getting started with LXD Containerization (Full Guide!)](https://www.youtube.com/watch?v=aIwgPKkVj8s)

[LXD Getting started - command line](https://linuxcontainers.org/lxd/getting-started-cli/)

[Arch Linux and Ubuntu Desktop in LXD VMs](https://www.youtube.com/watch?v=pEUsTMiq4B4)

[Running virtual machines with LXD 4.0](https://discuss.linuxcontainers.org/t/running-virtual-machines-with-lxd-4-0/7519)

[Container or Virtual Machine](https://discuss.linuxcontainers.org/t/container-or-virtual-machine/8293)

[LXD Change Storage Location](https://discuss.linuxcontainers.org/t/lxd-change-storage-location/3749)

[A little missundertanding for ‘lxd init’](https://discuss.linuxcontainers.org/t/a-little-missundertanding-for-lxd-init/11995)


