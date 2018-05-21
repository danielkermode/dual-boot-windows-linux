# Steps

1. If installing on Windows for the first time, use https://rufus.akeo.ie/ to create a bootable USB from the linux ISO. Make space on the Windows hard drive and work out how to boot from the USB.

2. If installing for the first time (eg. not from an existing linux installation), follow https://wiki.archlinux.org/index.php/installation_guide#Pre-installation. Once in a linux shell, create a partition with fdisk.

3. Once the partition has been created (eg. sda9 50GB) it must be formatted with a file system. ext4 seems to be standard. sudo mkfs.ext4 /dev/sda9 (note - you don't need sudo in a root shell but I include it to show that root permissions are needed. "su -" gets you into root shell)

4. Mount the file system. sudo mount /dev/sda9 /mnt

5. If on an existing arch linux, arch-install-scripts is needed. If booting from the USB it should be there already.

6. sudo pacstrap /mnt base to get base packages on the new file system.

7. genfstab -U /mnt >> /mnt/etc/fstab to get fstab (need to be in root shell for this one). https://wiki.archlinux.org/index.php/Fstab. This determines how partitions (or other block devices or remote filesystems) should be mounted into the filesystem. The simplest case is mounting the whole partition onto the root filesystem (ie. the partition mirrors the filesystem, with no extra partitions for things like swap or other directories - /dev/sda9 == /).

8. arch-chroot /mnt to "change root" into the /mnt directory.

9. Set up the new linux: ln -sf /usr/share/zoneinfo/NZ /etc/localtime for NZ time. "en_US.UTF-8 UTF-8" uncommented in /etc/locale.gen and run locale-gen. Create a new file /etc/locale.conf and add "LANG=en_US.UTF-8".

10. Set up networking stuff: /etc/hostname, eg. dankplayground
/etc/hosts, eg.

127.0.0.1	localhost
::1		localhost
127.0.1.1	myhostname.localdomain	myhostname

11. To get internet on the new system, pacman -S dialog wpa_supplicant iw and run wifi-menu. Note that if internet is already running, you won't need to connect until reboot.

12. Configure boot loader. I use systemd-boot. In case you're installing a new linux with an existing ESP, get name of ESP (eg. /dev/sda2) and mount /dev/sda2 /mnt.

Note: /boot should contain "initramfs-linux-fallback.img  initramfs-linux.img  vmlinuz-linux" (and probably lts versions).
If this is the first linux, "bootctl install" will copy required .efi files to the esp, provided it is mounted at /boot (not /mnt!). Note that I use systemd-boot with config options (linux, initrd and PARTUUID) for linux startup, but the .efi is used for systemd-boot itself by the ESP.

Obtain PARTUUID for root partition with blkid -s PARTUUID -o value /dev/sda9 and write a *.conf file in /entries of the ESP:

title Dank Arch
linux /vmlinuz-linux
initrd /intel-ucode.img
initrd /initramfs-linux.img
options root=PARTUUID=acae0845-1747-9644-a2b2-e74dbaf9135f rw

Basically how this works: the linux kernel and .img files are generated automatically or by "mkinitcpio -p linux". If the /etc/fstab file of the installation describes mounting the ESP at /boot, these files will be placed in the ESP alongside other stuff such as loader conf when the root partition is booted. If there is no mounting of the ESP on startup, then the files simply reside in /boot by themselves but need to be in the ESP as well, described in the config of the boot loader.

https://wiki.archlinux.org/index.php/Systemd-boot#Configuration has more info about boot stuff.
