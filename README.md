# ssh from x86 to rg35xx-plus
This guide tells you how to connect to a rg35xx-plus via SSH to manage your ROMs via wifi or to do other fun stuff. I hope it doesn't but it probably breaks things so be careful. The guide is written for Ubuntu 23.10.

## install qemu
Qemu is required to run an emulated chroot on your x86 machine.
Here you can read more about it
https://unix.stackexchange.com/questions/41889/how-can-i-chroot-into-a-filesystem-with-a-different-architechture
Or just run this command:
```bash
apt-get install qemu-user-static
cp $(which qemu-arm-static) /mnt/usr/bin
```
## chroot from x86
First, you have to insert the stock SD-Card and find the linuxrootfs and the boot partition. For me this was /dev/mmcblk0p5 and /dev/mmcblk0p6.
The following commands will chroot onto the SD card.
```bash
sudo mount /dev/mmcblk0p5 /mnt
sudo mount /dev/mmcblk0p6 /mnt/boot
for dir in /dev /dev/pts /proc /sys /run; do sudo mount --bind $dir /mnt$dir; done
sudo cp /proc/mounts /mnt/etc/mtab
sudo mount -o bind /etc/resolv.conf /mnt/etc/resolv.conf
sudo chroot /mnt qemu-arm-static /bin/bash
```
now execute the following commands in the chroot environment
```bash
apt install openssh-server
systemctl enable ssh
```

## Permit root user for ssh
After installing ssh, edit /etc/ssh/sshd_config and add the following to the bottom of the file:
```
PermitRootLogin yes
```
Now you can close the chroot and insert the SD Card back into your device.
You can enable wifi and login with username root password root.

## Optional and probably bad:
For some reasons, the sudo binaries and config files belong to the game user on this device. Change it like this:
```bash
chown root:root /usr/bin/sudo && chmod 4755 /usr/bin/sudo
chown root:root /usr/lib/sudo/sudoers.so && chmod 4755 /usr/lib/sudo/sudoers.so
chown root:root /etc/sudoers
```

hf and good luck ;)
