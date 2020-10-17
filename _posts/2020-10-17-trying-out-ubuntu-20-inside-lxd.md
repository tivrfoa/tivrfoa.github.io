---
layout: post
title:  "Trying out Ubuntu 20.10 (beta) inside LXD"
date:   2020-10-17 18:00:00 -0300
categories: linux container lxd lxc
---


# Testing LXD

Transcription from:
[Trying out Ubuntu 20.10 (beta) inside LXD](https://www.youtube.com/watch?v=W3EkvU4Arys)


```shell
$ lxc version
Client version: 4.6
Server version: 4.6
```

### Creating a container

```shell
$ lxc launch images:ubuntu/20.10/cloud c1
Creating c1
Starting c1                                 
$ lxc list
+--------------+---------+----------------------+-----------------------------------------------+-----------+-----------+
|     NAME     |  STATE  |         IPV4         |                     IPV6                      |   TYPE    | SNAPSHOTS |
+--------------+---------+----------------------+-----------------------------------------------+-----------+-----------+
| c1           | RUNNING | 10.xx.yyy.83 (eth0)  | fd42:17ae:dbde:9688:216:3eff:fe0f:9607 (eth0) | CONTAINER | 0         |
+--------------+---------+----------------------+-----------------------------------------------+-----------+-----------+
| intent-leech | RUNNING | 10.xx.yyy.209 (eth0) | fd42:17ae:dbde:9688:216:3eff:fe31:fb01 (eth0) | CONTAINER | 0         |
+--------------+---------+----------------------+-----------------------------------------------+-----------+-----------+
```

### Going Inside the Container

```sh
$ lxc exec c1 bash
root@c1:~# lsb_release -a
No LSB modules are available.
Distributor ID:	Ubuntu
Description:	Ubuntu 20.10
Release:	20.10
Codename:	groovy
```

```sh
root@c1:~# apt update
```

### Creating a virtual machine

The only difference is the `--vm` at the end.

```sh
$ lxc launch images:ubuntu/20.10/cloud v1 --vm
```

This download a different image, a virtual machine image, which is quite a bit larger.

Unlike a container where we're just unpacking 60 70 MB tarball with a bunch of files directly into a directory effectively, for a VM image we're downloading something more like 250 MB or so, and that's a very compressed disk image that than needs to be unpacked into a pre-created block, which in this case is about 10GB large.

So not only do we need to decompress the data and unpack, we also need to then do some partition table reshuffling to make sure that the GPT partition table used for virtual machines is all consistent.


### Creating password for user ubuntu in VM

```sh
$ lxc exec v1 -- passwd ubuntu
New password: 
Retype new password: 
passwd: password updated successfully
```

```sh
$ lxc console v1
To detach from the console, press: <ctrl>+a q

v1 login: ubuntu
Password: 
Welcome to Ubuntu 20.10 (GNU/Linux 5.8.0-23-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.

ubuntu@v1:~$ pwd
/home/ubuntu
```

The contents of the image itself is near identical to the container.

Other than the VM imageg having a bootloader and a kernel, the rest is near identical.

```sh
ubuntu@v1:~$ ps fauxww
USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root           2  0.0  0.0      0     0 ?        S    20:55   0:00 [kthreadd]
root           3  0.0  0.0      0     0 ?        I<   20:55   0:00  \_ [rcu_gp]
root           4  0.0  0.0      0     0 ?        I<   20:55   0:00  \_ [rcu_par_gp]
root           6  0.0  0.0      0     0 ?        I<   20:55   0:00  \_ [kworker/0:0H-kblockd]
root           9  0.0  0.0      0     0 ?        I<   20:55   0:00  \_ [mm_percpu_wq]
root          10  0.0  0.0      0     0 ?        S    20:55   0:00  \_ [ksoftirqd/0]
root          11  0.0  0.0      0     0 ?        I    20:55   0:00  \_ [rcu_sched]
root          12  0.0  0.0      0     0 ?        S    20:55   0:00  \_ [migration/0]
root          13  0.0  0.0      0     0 ?        S    20:55   0:00  \_ [idle_inject/0]
root          14  0.0  0.0      0     0 ?        S    20:55   0:00  \_ [cpuhp/0]
root          15  0.0  0.0      0     0 ?        S    20:55   0:00  \_ [kdevtmpfs]
root          16  0.0  0.0      0     0 ?        I<   20:55   0:00  \_ [netns]
root          17  0.0  0.0      0     0 ?        S    20:55   0:00  \_ [rcu_tasks_kthre]
root          18  0.0  0.0      0     0 ?        S    20:55   0:00  \_ [rcu_tasks_rude_]
root          19  0.0  0.0      0     0 ?        S    20:55   0:00  \_ [rcu_tasks_trace]
root          20  0.0  0.0      0     0 ?        S    20:55   0:00  \_ [kauditd]
root          21  0.0  0.0      0     0 ?        S    20:55   0:00  \_ [khungtaskd]
root          22  0.0  0.0      0     0 ?        S    20:55   0:00  \_ [oom_reaper]
root          23  0.0  0.0      0     0 ?        I<   20:55   0:00  \_ [writeback]
root          24  0.0  0.0      0     0 ?        S    20:55   0:00  \_ [kcompactd0]
root          25  0.0  0.0      0     0 ?        SN   20:55   0:00  \_ [ksmd]
root          26  0.0  0.0      0     0 ?        SN   20:55   0:00  \_ [khugepaged]
root          72  0.0  0.0      0     0 ?        I<   20:55   0:00  \_ [kintegrityd]
root          73  0.0  0.0      0     0 ?        I<   20:55   0:00  \_ [kblockd]
root          74  0.0  0.0      0     0 ?        I<   20:55   0:00  \_ [blkcg_punt_bio]
root          75  0.0  0.0      0     0 ?        I<   20:55   0:00  \_ [tpm_dev_wq]
root          76  0.0  0.0      0     0 ?        I<   20:55   0:00  \_ [ata_sff]
root          77  0.0  0.0      0     0 ?        I<   20:55   0:00  \_ [md]
root          78  0.0  0.0      0     0 ?        I<   20:55   0:00  \_ [edac-poller]
root          79  0.0  0.0      0     0 ?        I<   20:55   0:00  \_ [devfreq_wq]
root          80  0.0  0.0      0     0 ?        S    20:55   0:00  \_ [watchdogd]
root          82  0.0  0.0      0     0 ?        I<   20:55   0:00  \_ [pm_wq]
root          84  0.0  0.0      0     0 ?        S    20:55   0:00  \_ [kswapd0]
root          85  0.0  0.0      0     0 ?        S    20:55   0:00  \_ [ecryptfs-kthrea]
root          87  0.0  0.0      0     0 ?        I<   20:55   0:00  \_ [kthrotld]
root          88  0.0  0.0      0     0 ?        S    20:55   0:00  \_ [irq/24-aerdrv]
root          89  0.0  0.0      0     0 ?        S    20:55   0:00  \_ [irq/24-pciehp]
root          90  0.0  0.0      0     0 ?        S    20:55   0:00  \_ [irq/25-aerdrv]
root          91  0.0  0.0      0     0 ?        S    20:55   0:00  \_ [irq/25-pciehp]
root          92  0.0  0.0      0     0 ?        S    20:55   0:00  \_ [irq/26-aerdrv]
root          93  0.0  0.0      0     0 ?        S    20:55   0:00  \_ [irq/26-pciehp]
root          94  0.0  0.0      0     0 ?        S    20:55   0:00  \_ [irq/27-aerdrv]
root          95  0.0  0.0      0     0 ?        S    20:55   0:00  \_ [irq/27-pciehp]
root          96  0.0  0.0      0     0 ?        S    20:55   0:00  \_ [irq/28-aerdrv]
root          97  0.0  0.0      0     0 ?        S    20:55   0:00  \_ [irq/28-pciehp]
root          98  0.0  0.0      0     0 ?        I<   20:55   0:00  \_ [acpi_thermal_pm]
root          99  0.0  0.0      0     0 ?        I<   20:55   0:00  \_ [vfio-irqfd-clea]
root         100  0.0  0.0      0     0 ?        I<   20:55   0:00  \_ [ipv6_addrconf]
root         109  0.0  0.0      0     0 ?        I<   20:55   0:00  \_ [kstrp]
root         112  0.0  0.0      0     0 ?        I<   20:55   0:00  \_ [zswap-shrink]
root         113  0.0  0.0      0     0 ?        I<   20:55   0:00  \_ [kworker/u3:0]
root         120  0.0  0.0      0     0 ?        I<   20:55   0:00  \_ [charger_manager]
root         155  0.0  0.0      0     0 ?        S    20:55   0:00  \_ [scsi_eh_0]
root         156  0.0  0.0      0     0 ?        I<   20:55   0:00  \_ [scsi_tmf_0]
root         157  0.0  0.0      0     0 ?        I<   20:55   0:00  \_ [kworker/0:1H-kblockd]
root         158  0.0  0.0      0     0 ?        S    20:55   0:00  \_ [scsi_eh_1]
root         159  0.0  0.0      0     0 ?        I<   20:55   0:00  \_ [scsi_tmf_1]
root         160  0.0  0.0      0     0 ?        S    20:55   0:00  \_ [scsi_eh_2]
root         161  0.0  0.0      0     0 ?        I<   20:55   0:00  \_ [scsi_tmf_2]
root         162  0.0  0.0      0     0 ?        S    20:55   0:00  \_ [scsi_eh_3]
root         163  0.0  0.0      0     0 ?        I<   20:55   0:00  \_ [scsi_tmf_3]
root         164  0.0  0.0      0     0 ?        S    20:55   0:00  \_ [scsi_eh_4]
root         165  0.0  0.0      0     0 ?        I<   20:55   0:00  \_ [scsi_tmf_4]
root         166  0.0  0.0      0     0 ?        S    20:55   0:00  \_ [scsi_eh_5]
root         167  0.0  0.0      0     0 ?        I<   20:55   0:00  \_ [scsi_tmf_5]
root         168  0.0  0.0      0     0 ?        S    20:55   0:00  \_ [scsi_eh_6]
root         169  0.0  0.0      0     0 ?        I<   20:55   0:00  \_ [scsi_tmf_6]
root         172  0.0  0.0      0     0 ?        I    20:55   0:00  \_ [kworker/u2:4-events_unbound]
root         189  0.0  0.0      0     0 ?        S    20:55   0:00  \_ [jbd2/sda2-8]
root         190  0.0  0.0      0     0 ?        I<   20:55   0:00  \_ [ext4-rsv-conver]
root         201  0.0  0.0      0     0 ?        S    20:55   0:00  \_ [hwrng]
root         287  0.0  0.0      0     0 ?        I<   20:55   0:00  \_ [cryptd]
root         540  0.0  0.0      0     0 ?        I    21:10   0:00  \_ [kworker/0:0-memcg_kmem_cache]
root         547  0.0  0.0      0     0 ?        I    21:15   0:00  \_ [kworker/0:1-events]
root         556  0.0  0.0      0     0 ?        I    21:17   0:00  \_ [kworker/u2:1-ext4-rsv-conversion]
root           1  0.0  1.1 102704 11264 ?        Ss   20:55   0:01 /sbin/init splash
root         232  0.0  1.2  52632 12588 ?        S<s  20:55   0:00 /lib/systemd/systemd-journald
root         254  0.0  0.5  19440  5172 ?        Ss   20:55   0:00 /lib/systemd/systemd-udevd
systemd+     281  0.0  0.6  91384  6676 ?        Ssl  20:55   0:00 /lib/systemd/systemd-timesyncd
root         284  0.0  1.3 715208 13776 ?        Ssl  20:55   0:01 /run/lxd_config/9p/lxd-agent
systemd+     348  0.0  0.8  28012  8104 ?        Ss   20:55   0:00 /lib/systemd/systemd-networkd
systemd+     350  0.0  1.2  24968 12544 ?        Ss   20:55   0:00 /lib/systemd/systemd-resolved
root         449  0.0  0.2   9400  2752 ?        Ss   20:55   0:00 /usr/sbin/cron -f
message+     450  0.0  0.4   8444  4788 ?        Ss   20:55   0:00 /usr/bin/dbus-daemon --system --address=systemd: --nofork --nopidfile --systemd-activation --syslog-only
root         457  0.0  1.7  32280 17760 ?        Ss   20:55   0:00 /usr/bin/python3 /usr/bin/networkd-dispatcher --run-startup-triggers
syslog       458  0.0  0.4 221124  4692 ?        Ssl  20:55   0:00 /usr/sbin/rsyslogd -n -iNONE
root         462  0.0  0.8  18044  8264 ?        Ss   20:55   0:00 /lib/systemd/systemd-logind
root         478  0.0  0.1   8424  1840 tty1     Ss+  20:55   0:00 /sbin/agetty -o -p -- \u --noclear tty1 linux
root         480  0.0  2.0 110488 20332 ?        Ssl  20:55   0:00 /usr/bin/python3 /usr/share/unattended-upgrades/unattended-upgrade-shutdown --wait-for-signal
root         550  0.0  0.4  11564  4444 ttyS0    Ss   21:15   0:00 /bin/login -p --
ubuntu       574  0.0  0.4   9828  4136 ttyS0    S    21:18   0:00  \_ -bash
ubuntu       588  0.0  0.3  12504  3428 ttyS0    R+   21:23   0:00      \_ ps fauxww
ubuntu       567  0.0  0.9  19512  9944 ?        Ss   21:18   0:00 /lib/systemd/systemd --user
ubuntu       568  0.0  0.2 103904  2940 ?        S    21:18   0:00  \_ (sd-pam)
```

But if we look for example the message output you can see the kernel is 5.8.0 and it just booted.

```sh
ubuntu@v1:~$ sudo dmesg | less
[    0.000000] Linux version 5.8.0-23-generic (buildd@lcy01-amd64-020) (gcc (Ubuntu 10.2.0-13ubuntu1) 10.2.0, GNU ld (GNU Binutils for Ubuntu) 2.35.1) #24-Ubuntu SMP Fri Oct 9 16:32:13 UTC 2020 (Ubuntu 5.8.0-23.24-generic 5.8.14)
[    0.000000] Command line: BOOT_IMAGE=/boot/vmlinuz-5.8.0-23-generic root=/dev/sda2 ro quiet splash console=tty1 console=ttyS0 vt.handoff=7
[    0.000000] KERNEL supported cpus:
[    0.000000]   Intel GenuineIntel
[    0.000000]   AMD AuthenticAMD
[    0.000000]   Hygon HygonGenuine
[    0.000000]   Centaur CentaurHauls
[    0.000000]   zhaoxin   Shanghai  
[    0.000000] x86/fpu: Supporting XSAVE feature 0x001: 'x87 floating point registers'
[    0.000000] x86/fpu: Supporting XSAVE feature 0x002: 'SSE registers'
[    0.000000] x86/fpu: Supporting XSAVE feature 0x004: 'AVX registers'
[    0.000000] x86/fpu: Supporting XSAVE feature 0x008: 'MPX bounds registers'
[    0.000000] x86/fpu: Supporting XSAVE feature 0x010: 'MPX CSR'
[    0.000000] x86/fpu: xstate_offset[2]:  576, xstate_sizes[2]:  256
[    0.000000] x86/fpu: xstate_offset[3]:  832, xstate_sizes[3]:   64
[    0.000000] x86/fpu: xstate_offset[4]:  896, xstate_sizes[4]:   64
[    0.000000] x86/fpu: Enabled xstate features 0x1f, context size is 960 bytes, using 'compacted' format.
[    0.000000] BIOS-provided physical RAM map:
[    0.000000] BIOS-e820: [mem 0x0000000000000000-0x000000000009ffff] usable
[    0.000000] BIOS-e820: [mem 0x0000000000100000-0x000000003ee65fff] usable
...
```

The reason why things like `lxc exec` as well as the file transfer api, `file pull` for example, and features work the same way as containers is because we're using an agent in the VM.

So you might see here the `ps` command I just run is a child of bash shell which is itself a child of `lxd-agent`.

```sh
lxc file pull v1/etc/hosts -
127.0.1.1	v1
127.0.0.1	localhost
::1		localhost ip6-localhost ip6-loopback
ff02::1		ip6-allnodes
ff02::2		ip6-allrouters
```


```sh
lxc console v1 --type=vga
LXD automatically uses either spicy or remote-viewer when present.
As neither could be found, the raw SPICE socket can be found at:
  spice+unix:///home/leandro/snap/lxd/17738/.config/lxc/sockets/482866122.spice

sudo apt install spice-client-gtk

^C
$ sudo systemctl reload snap.lxd.daemon
[sudo] password for leandro: 
$ lxc console v1 --type=vga

(spicy:24095): dbind-WARNING **: 19:01:20.777: Couldn't register with accessibility bus: Did not receive a reply. Possible causes include: the remote application did not send a reply, the message bus security policy blocked the reply, the reply timeout expired, or the network connection was broken.
** Message: 19:01:27.193: main channel: opened
** Message: 19:01:46.165: main channel: failed to connect
```

### lxc info

#### Container

```sh
$ lxc info c1
Name: c1
Location: none
Remote: unix://
Architecture: x86_64
Created: 2020/10/17 20:23 UTC
Status: Running
Type: container
Profiles: default
Pid: 20122
Ips:
  eth0:	inet	10.xx.yyy.83	veth8b762177
  eth0:	inet6	fd42:17ae:dbde:9688:216:3eff:fe0f:9607	veth8b762177
  eth0:	inet6	fe80::216:3eff:fe0f:9607	veth8b762177
  lo:	inet	127.0.0.1
  lo:	inet6	::1
Resources:
  Processes: 18
  Disk usage:
    root: 85.47MB
  CPU usage:
    CPU usage (in seconds): 10
  Memory usage:
    Memory (current): 61.01MB
    Memory (peak): 135.85MB
  Network usage:
    eth0:
      Bytes received: 18.91MB
      Bytes sent: 843.71kB
      Packets received: 10343
      Packets sent: 9371
    lo:
      Bytes received: 1.27kB
      Bytes sent: 1.27kB
      Packets received: 12
      Packets sent: 12
```

#### Virtual Machine

```sh
$ lxc info v1
Name: v1
Location: none
Remote: unix://
Architecture: x86_64
Created: 2020/10/17 20:53 UTC
Status: Running
Type: virtual-machine
Profiles: default
Pid: 1393
Ips:
  enp5s0:	inet	10.xx.yyy.245	tap7f7c8767
  enp5s0:	inet6	fd42:17ae:dbde:9688:216:3eff:fec3:57fe	tap7f7c8767
  enp5s0:	inet6	fe80::216:3eff:fec3:57fe	tap7f7c8767
  lo:	inet	127.0.0.1
  lo:	inet6	::1
Resources:
  Processes: 15
  Disk usage:
    root: 50.84MB
  CPU usage:
    CPU usage (in seconds): 20
  Memory usage:
    Memory (current): 315.35MB
    Memory (peak): 361.05MB
  Network usage:
    enp5s0:
      Bytes received: 43.74kB
      Bytes sent: 25.93kB
      Packets received: 374
      Packets sent: 276
    lo:
      Bytes received: 6.32kB
      Bytes sent: 6.32kB
      Packets received: 84
      Packets sent: 84
```

#### Memory Comparison

Container | Virtual Machine
----------|----------------
Memory (current): 61.01MB | 315.35MB
Memory (peak):   135.85MB | 361.05MB


### How VM works compared to containers

The way VM work in LXD, all the operations that you're used to running on containers will work for Virtual Machines. You can turn an existing VM with a snapshot into an image, you can snapshot them, you can restore those snapshots, file transfer works properly, you can do backups.

It's really the exact same command line and in fact rest api that's used to drive VM or containers.

Except VM let you do some extra cool stuff like actually seeing a running desktop environment, which is not realyy something you can do in containers.<br>
You can do individual GUI tools in the container. You can do that with some amout of pass-through.


### References

[https://linuxcontainers.org/lxd/advanced-guide/#edit-a-profile](https://linuxcontainers.org/lxd/advanced-guide/#edit-a-profile)

https://discuss.linuxcontainers.org/t/vnc-support-is-disabled/8735/4

