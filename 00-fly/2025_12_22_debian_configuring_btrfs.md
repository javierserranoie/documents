# Debian: Configuring BTRFS

BTRFS is a filesystem that introduce some advantages, the most interesting ones for my use case are:

* Capacity to define subvolumes on the fly (kind of logical partitions)
* Ability to produce snapshots of the system and recover from failures
* Copy on Write (CoW): creating kind of immutable filesystem at block level that is the basis for snapshots and disaster recovery.

There are many other advantages (as well as disadvantages that are kind of mitigated by some BTRFS features, ie: higher writing stress on SSDs due to CoW, etc.).

I've been using BTRFS on archlinux for a while now with very good results but I will be trying it on Debian now, I will build a base Debian image for homelabbing and despite for this use case there might be better and more lightweight filsesystems, I wanted to check what's the Debian adoption and support for this filesystem.

## Debian Installation

I will install Debian base image from non-graphic install (I think that you could do graphical but the results would be identical). 

Some of the problems that I found during installation is the lack of support to create BTRFS volumes, I tried to use the embedded terminal and do it manually but the BTRFS tool wasn't present (or I couldn't find it in the path). This is not a big problem as volumes can be redefined later on but it will require some work.

I didn't encrypt the disk but the option here would be create an encrypted physical paritions with a mapper device and add BTRFS on top as this filesystem doesn't support native encryption as of today but for homelabbing I didn't believe it was necessary, perhaps in another chapter I will do.

 ## Troubleshooting

After deleting @rootfs if the grubconfig wasnt' working well perhaps you end up in the grub console (grub>) If that's the case, follow these steps:

1. find where the btrfs volumes are with `ls`, ie:

```
grub> ls

(hd) (hd0) (hd0,gpt1) (hd0,gpt2)

grub> ls (hd0,gpt2)

@ @home @logs @cache

grub> linux (hd0,gpt2)/@/boot/vmlinuz-<tab>...  root=/dev/vda2 rootflags=subvol=@ rw
grub> initrd (hd0,gpt2)/@/boo/initramfs-<tab>...
grub> boot
```

**NOTE:** After these commands, you should be in your system again but we are not done, we need to rebuild grub with:

2. install grub and update config:

```
$ grub-install
$ grub-mkconfig -o /boot/grub/grub.cfg
```

if you are using EFI:

```
$ grub-install --target=x86_64-efi --efi-directory=/boot/efi
$ grub-mkconfig -o /boot/grub/grub.cfg
```
