# Installation of ArchLinux on Raspberry Pi 4B---
These are instructions on how I installed Arch Linux on my Raspberry Pi.
This documentation primarily serves me to remember all the steps I took in case I want/need to run through the installation process again at a later date, but may help others as well.

## Put Arch Linux on an SD Card
This chapter explain how one installs Arch Linux on an SD card so it can be used with a Raspberry Pi.
### Identify the SD Card
```bash
lsblk
```
output
```
NAME        MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
loop0         7:0    0  54,8M  1 loop /snap/gtk-common-themes/1502
loop1         7:1    0   199M  1 loop /snap/gitkraken/154
loop2         7:2    0  29,9M  1 loop /snap/snapd/8542
loop3         7:3    0   199M  1 loop /snap/gitkraken/153
loop4         7:4    0    55M  1 loop /snap/core18/1880
loop5         7:5    0  29,9M  1 loop /snap/snapd/8790
loop6         7:6    0  62,1M  1 loop /snap/gtk-common-themes/1506
loop7         7:7    0  95,8M  1 loop /snap/rpi-imager/76
loop8         7:8    0  55,3M  1 loop /snap/core18/1885
mmcblk0     179:0    0 119,1G  0 disk 
├─mmcblk0p1 179:1    0   200M  0 part 
└─mmcblk0p2 179:2    0 118,9G  0 part 
nvme0n1     259:0    0 238,5G  0 disk 
├─nvme0n1p1 259:1    0   512M  0 part /boot/efi
└─nvme0n1p2 259:2    0   238G  0 part /
```
&rarr; my SD card is called `mmcblk0`.
Let's call it `sdX` in the following for generalization.

### Format/Partition the SD Card
We got the instructions primarily from [here](https://archlinuxarm.org/platforms/armv6/raspberry-pi) and adapted them where necessary/faulty.

Start fdisk to partition the SD card:
```bash
fdisk /dev/sdX
```

At the fdisk prompt, delete old partitions and create a new one:
```
> o             # clear out any partitions on the drive
> p             # list partitions; there should be no partitions left
> n             # new partition
> p             # for primary
> 1 or ENTER    # for the first partition on the drive
> ENTER         # to accept the default first sector
> +200M         # for the last sector
> t             # change partition type
> c             # to set the first partition to type W95 FAT32 (LBA)
> n             # new partition
> p             # for primary
> 2 or ENTER    # for the second partition on the drive
> ENTER         # to accept the default first sector
> ENTER         # to accept the default last sector
> w             # write the partition table and exit
```

If you are prompted
```
Partition #1 contains a vfat signature.
Do you want to remove the signature? [Y]es/[N]o: Y
The signature will be removed by a write command.
```
for partition 1 or 2, you can remove it (like above), then type
```
> t             # change partition type
> 2 or ENTER    # for the second partition on the drive
> 83            # Linux/ext4
```
before the `w` write to set the 2nd partition to Linux type (ext4).
Or you can not remove it since we take care of partition 1 (vFAT) ourselves anyway, and the default is ext4 (necessary for partition 2.

### Download Arch Linux
64Bit is OK for a Raspberry Pi Model 4B
```bash
wget http://os.archlinuxarm.org/os/ArchLinuxARM-rpi-aarch64-latest.tar.gz
```
alternatively
```bash
wget http://os.archlinuxarm.org/os/ArchLinuxARM-rpi-4-latest.tar.gz
```

### Writing to the SD Card
You shoud run the following as root:
```bash
sudo su -
```

Create and mount the FAT filesystem:
```bash
mkfs.vfat /dev/sdX1
mkdir boot
mount /dev/sdX1 boot
```

Create and mount the ext4 filesystem:
```bash
mkfs.ext4 /dev/sdX2
mkdir root
mount /dev/sdX2 root
```

Unarchive the ArchLinux package into the root directory
```bash
tar zxvf <path-to-downloaded-arch-tar-gz> -C root
```
Then move all that is in the `boot/` directory in `root` to the `boot` partition
```bash
mv root/boot/* boot
```
Now synchronize the mount directories with the actual SD card
```bash
sync
```
and unmount both directories/partitions:
```bash
umount boot root
```

Congrats, the installation is done.
You may now safely remove the SD card and insert it into your RasPi.

### References & Useful Links
* [ArchLinux Doc for RasPi](https://archlinuxarm.org/platforms/armv6/raspberry-pi)
* [itsFoss](https://itsfoss.com/install-arch-raspberry-pi/)
* [linuxize](https://linuxize.com/post/how-to-install-arch-linux-on-raspberry-pi/)
* [instructables](https://www.instructables.com/id/Arch-Linux-on-Raspberry-Pi/)
