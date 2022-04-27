# Steps

1. If installing on Windows for the first time, use https://rufus.akeo.ie/ to create a bootable USB from the linux ISO. Make space on the Windows hard drive by shrinking the existing Microsoft partition and work out how to boot from the USB on the machine.

Gotcha - be sure to disable secure boot if booting from USB isn't working properly. eg. https://docs.microsoft.com/en-us/windows-hardware/manufacture/desktop/disabling-secure-boot?view=windows-11

2. Follow https://wiki.archlinux.org/title/installation_guide.

3. Be sure to install the packages listed in `new-packages/pkglist.txt` as these are needed to install stuff on the new arch installation. Swap out `amd-ucode` as needed - https://wiki.archlinux.org/title/microcode.

4. Configure boot loader. I use `systemd-boot` - https://wiki.archlinux.org/title/systemd-boot

Example configuration:

```
title Arch Linux
linux /vmlinuz-linux
initrd /amd-ucode.img
initrd /initramfs-linux.img
options root="/dev/nvme0n1p5" rw
```

Or obtain PARTUUID for root partition with `blkid -s PARTUUID -o value /dev/xxx`:

```
title Arch Linux
linux /vmlinuz-linux
initrd /intel-ucode.img
initrd /initramfs-linux.img
options root=PARTUUID=acae0845-1747-9644-a2b2-e74dbaf9135f rw
```

Basically how this works: the linux kernel and .img files are generated automatically or you can generate manually with `mkinitcpio -p linux`. If the `/etc/fstab` file of the installation describes mounting the ESP at `/boot`, these files will be placed in the ESP alongside other stuff such as loader conf when the root partition is booted.


5. Reboot. Hopefully everything is working properly and the arch linux installation runs correctly. Run `systemctl start dhcpcd.service` and connect to wifi with `iwd` https://wiki.archlinux.org/title/Iwd (as on the installation OS) to get internet. Install all the desired packages - `new-packages/pkglist2.txt` has a list of basic things that are good to have such as plasma, VS code and Chromium.