# ssh from x86 to rg35xx-plus
This guide tells you how to connect to a rg35xx-plus via ssh to manage your ROMs via wifi our to do other fun stuff. I hope is doesnt but it probably breaks thinks so be careful. The guide is written for Ubuntu 23.10.

## install qemu
Qemu is required to run a emulated chroot on you x86 machine.
Here you can read more about it
https://unix.stackexchange.com/questions/41889/how-can-i-chroot-into-a-filesystem-with-a-different-architechture
Or just run this command:
```bash
apt-get install qemu-user-static
cp $(which qemu-arm-static) /mnt/usr/bin
```
## chroot from x86
First you have to insert the stock SD-Card and find the linuxrootfs and the boot partition. For me this was /dev/mmcblk0p5 and /dev/mmcblk0p6.
The following commands will chroot onto the sdcard.
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
apt install nano openssh-server
systemctl enable ssh
adduser ssh-user
```
This will install nano (the best editor for linux) and openssh-server. Feel free to install another editor.
Now you need to add your usergroup to sudoers. Do this by editing /etc/sudoers:
```
nano /etc/sudoers
```
Add this line to the file where it belongs
```
ssh-user    ALL=(ALL:ALL) ALL
```

for some reasons the sudo binaries and configfiles belong to the game user on this device. Change it like this:
```bash
chown root:root /usr/bin/sudo && chmod 4755 /usr/bin/sudo
chown root:root /usr/lib/sudo/sudoers.so && chmod 4755 /usr/lib/sudo/sudoers.so
chown root:root /etc/sudoers
```
Now you can close the chroot and insert the SD-Card back to your device.
You can eneable wifi and ssh onto it.
Since the sdcard on the device is mounted such that only root can change it you need to login over ssh via root to manage the roms. There are more elegant and safer ways to do it but this is probably the fastes.
You can enable the root user for ssh in the /etc/ssh/sshd_config file. And you can edit this file via ssh. Restart ssh after you edited it:
```bash
sudo nano /etc/ssh/sshd_config
sudo systemctl restart ssh
```
hf and good luck ;)
