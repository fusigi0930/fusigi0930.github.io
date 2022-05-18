---
layout: post
title: Create a virtual box disk file from physical partition
description: simuate a similar your current system in virtual box
tags: linux vbox
---
# To simulate your linux system in windows while you are using the multiple boot in your laptop


## 1. dump your linux rootfs partition
if passiable, use USB boot to dump your partition, and make sure your target storage avaliable is enough.

```shell
sudo dd if=<dev node> of=<stored filepath> status=progress
sync
sudo chown 1000:1000 <stored filepath>
```

## 2. reduce the image file size
the dump file should be very large, we use the commands to reduce the size:

```shell
e2fsck -f -y <stored filepath>
resize2fs -M <stored filepath>
```

## 3. adjust the fstab
that is the point to simulate your physical system in your windows virtual box, because the rootfs and home directory
are different with your physical system.

we just adjust the mount device with rootfs to /dev/sda1,
and the swap device to /swapfile, the others we just mart them to comments

we can use the command to mount image file

```shell
sudo mount -o loop <stored filepath> <mount point>
```

after the modification is finished, umount it by this command:

```shell
sudo umount <mount point>
```

## 4. create a virtual disk image file
next, create a disk image file let it contains grub and others setting

```shell
truncate -s <size> <disk filepath>
parted <disk filepath> --script mklabel msdos
parted <disk filepath> --script mkpart primary ext4 5M 100%
LODEV=$(sudo losetup -f)
sudo losetup $LODEV <disk filepath> -o 5M
sudo dd if=<stred filepath> of=$LODEV
```

then resize again, to make sure the virtual disk's data can be modified

```shell
sudo e2fsck -f -y $LODEV
sudo resize2fs $LODEV
```

## 5. convert to vdi file
if you don't install virtualbox before, please install virtualbox first

```shell
sudo apt update
sudo apt install virtualbox
```

we use the command convert the virtual disk image file to virtual box vdi file

```shell
vboxmanage convertfromraw <disk filepath> <vdi filepath>
```

## 6. set boot information for the vdi file
the following actions should be run under Windows system
open virtualbox and set the <vdi filepath> into the virtual media manager (ctrl+d), and add <vdi filepath>
then set it to your virtual machine storage

mount the ubuntu Live CD (iso file) and boot the virtual machine, after the boot procedure is completed, use "Try Ubuntu"

### 6.1 boot repair
install the boot repair utility into the Live system

```shell
sudo add-apt-repository ppa:yannubuntu/boot-repair && sudo apt-get update
sudo apt-get install -y boot-repair && sudo boot-repair
```

then use the boot repair to create boot information.

### 6.2 /etc/fstab
don't forget adjust the fstab such as mount /dev/sda1 to root folder, and use swapfile to replace the original one,
mark the /home directory information to comments

```shell
/dev/sda1 / ext4 errors=remount-ro 0 1
/swapfile none swap sw 0 0
# xxxxxxx /home xxxxxxxxxxxx
```

## 7. create physical disk vmdk in 
get the pyhsical drive information by using Windows Power shell with command:

```cmd
get-physicaldisk
```

then the drive information should be listed as:

```
Number FriendlyName       SerialNumber         MediaType CanPool OperationalStatus HealthStatus Usage            Size
------ ------------       ------------         --------- ------- ----------------- ------------ -----            ----
0      TOSHIBA DT01ACA300 675K33NAS            HDD       False   OK                Healthy      Auto-Select   2.73 TB
1      CT500P2SSD8        6479_A74D_B000_0089. SSD       False   OK                Healthy      Auto-Select 465.76 GB
```

we assume the pyhsical drive you want to use in virtualbox is number 1,
use the command in cmd.exe to create a vmdk file that will be used in virtualbox

```cmd
"%ProgramFiles%\Oracle\VirtualBox\vboxmanage.exe" internalcommands createrawvmdk -filename D:\PhyHDD.vmdk -rawdisk \\.\PhysicalDrive1
```

the command means we create a vmdk file in drive D, the file can be used in virtualbox

before launch virtualbox, make sure launch virtualbox as administrator if you need to use the physical drive in vmdk file

## 8. add vmdk file in to virtualbox

1. add the vmdk that is created in section 7 by using virtualbox media manager, use Ctrl+D to launch media manager
then click add button, choice the vmdk file.
your physical disk information should be shown in the List box.

2. add the vmdk media into your virtual machine stoarge

## 8. add physical disk partition for home directory
assume the vmdk media in your vm linux device node should be /dev/sdb, the your oringial home directory is in partition 5,
add it into /etc/fstab

```shell
/dev/sdb5 /home ext4 default 0 2
```

after these steps, your vm linux can access the partition in your physical drive.