---
layout: blog
title: Hibernate your linux
date: 2021-08-13 14:49:42
tags:
---

# abstract

- **Suspend to RAM** method, usually called **suspend**, cuts power to most parts of the machine aside from the RAM, which is required to restore the machine's state. Because of the large power savings, it is advisable for laptops to automatically enter this mode when the computer is running on batteries and the lid is closed (or the user is inactive for some time).

- **Suspend to disk** method, usually called **hibernate**, saves the machine's state into swap space and `completely powers off` the machine. When the machine is powered on, the state is restored. Until then, there is zero power consumption.

- **Suspend to both** method, usually called **hybrid suspend**, saves the machine's state into swap space, but does not power off the machine. Instead, it invokes usual suspend to RAM. Therefore, if the battery is not depleted, the system can resume from RAM. If the battery is depleted, the system can be resumed from disk, which is much slower than resuming from RAM, but the machine's state has not been lost.


# Low level interfaces

## kernel (swsusp)

The most straightforward approach is to directly inform the in-kernel software suspend code (swsusp) to enter a suspended state; the exact method and state depends on the level of hardware support. On modern kernels, writing appropriate strings to /sys/power/state is the primary mechanism to trigger this suspend.

## uswsusp

Userspace Software Suspend


# High level interfaces

## systemd

systemd provides native commands  for suspend, hibernate, hybrid suspend


# Prerequisite

A swap partition or swap file is need. Then you will point the kernel to the *swap* using the `resume=` kernel parameter in oot loader. To configure the initramfs is also needed to tell the kernel.

According to [kernel documentation](https://www.kernel.org/doc/Documentation/power/interface.txt) of system sleep, a small swap partitaion is also very likely to hibernate successfully. By the way, you are *strongly* recommended to read the kernel documentation mentioned previously, which offers numerous help in a straightforward way.

## configure initramfs

`resume` hook should be added in `/etc/mkinitcpio.conf` after `base` and `udev`.

If `systemd` hook is used, the hibernation mechanism is already provided, and no further hooks are needed.

## configure boot loader

resume=*swap_device*

- resume=PART\_UUID
- resume="PARTLABEL=Swap Partition"

Here lists several boot loaders for demonstration. You can get PARTUUID using `blkid`

### systemd boot

edit /boot/oader/entries/arch.conf

options root=UUID=... rw resume=PARTUUID=*swap\_area*

### grub

edit /etc/default/grub

GRUB\_CMDLINE\_LINUX\_DEFAULT="resume=PARTUUID=*swap\_area*"

then regenerate the grub.cfg:

```bash
# grub-mkconfig -o /boot/grub/grub.cfg
```

### efibootmgr

same

resume=PARTUUID=*swap_area*
