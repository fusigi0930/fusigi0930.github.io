# bring up raspberry pi3 b+
raspberry start boot sequence from GPU (bootcode.bin, start.elf, ...) then load uboot, let uboot to init the target board ram size and storage.

## 1. download raspberry pi firmware and boot binary
download the raspberry binary from https://github.com/raspberrypi/firmware by command:

```shell
git clone https://github.com/raspberrypi/firmware
```

## 2. create target sd card (USB storage)
the GPU load bootcode.bin from the first partition in target sd with FAT format, so we need create a primary partition with FAT format

## 3. put firmware and binary into the first FAT partition
### 3.0 create file partition
it can use the linux command fdisk/gdisk/sfdisk to create file partition for SD/USB storage.

for example, ubuntu at least need a rootfs partition, so it need 2 partition for the sdcard:
* partition1: FAT32 - boot
* partition2: ext4 - rootfs

it can use the command to create sdcard:
```shell

```
### 3.1 GPU firmware and binary
the boot partition need binaries to boot the target board:
* bootcode.bin
* start.elf start_x.elf start_db.elf start_cd.elf
* fixup.dat fixup_x.dat fixup_db.dat fixup_cd.dat
* config.txt
* cmdline.txt
* (optional, if you want to boot from u-boot) u-boot.bin
* kernel image (kernel8.img)
* device tree

the most files are stored in firmware/boot, you can copy these files to boot partition:
```shell
cp firmware/boot/bootcode.bin <boot>/
cp firmware/boot/start*.elf <boot>/
cp firmware/boot/fixup*.dat <boot>/
```

the config.txt and cmdline.txt are not stored in the github firmware repository, it will stored in my personal github repository: https://github.com/fusigi0930
```shell
git clone https://github.com/fusigi0930
```
then copy the config.txt and cmdline.txt into boot partition:
```shell
cp <>/config.txt <boot>/
cp <>/cmdline.txt <boot>/
```

### 3.2 u-boot for CPU
(optional) because GPU bootcode also configure RAM size, storage, ...; we can use the config.txt and cmdline.txt to load kernel directly.

it also can download the u-boot source from github mirror:
```shell
git clone https://github.com/fusigi0930/rpi-uboot
```

and build u-boot:
```shell
cd rpi-uboot
export ARCH=arm64
export CROSS_COMPILE=aarch64-linux-gnu-
make rpi_3_defconfig
make
```

### 3.3 kernel for CPU
download the kernel for raspberry pi3 b+ from github:
```shell
git clone https://github.com/raspberrypi/linux -b rpi-4.14.y kernel
```
build the kernel and device tree:
```shell
cd kernel
export ARCH=arm64
export CROSS_COMPILE=aarch64-linux-gnu-
make bcmrpi3_defconfig
make -j8
```
then copy the output binary to boot partition:
```shell
cd kernel/arch/arm64/boot/
cp Image <boot>/kernel8.img
cp dts/bcm2710-rpi-3-b-plus.dtb <boot>
```

## 4. set the config.txt file for GPU bootcode

## 5. set the cmdline.txt file

## 6. put the relate os rootfs
for example, the ubuntu-base rootfs can be downloaded from canonical web site:
### 6.1. ubuntu base
```shell
wget cdimage.ubuntu.com/cdimage/ubuntu-base/bionic/daily/current/bionic-baed-arm64.tar.gz
wget cdimage.ubuntu.com/cdimage/ubuntu-base/release/bionic/release/ubuntu-base-18.04-base-arm64.tar.gz
```
extract the rootfs tarball to rootfs partition:
```shell
tar xzvf bionic-base-arm64.tar.gz --directory=<rootfs>
sync
```

### 6.2 ubuntu server
the ubuntu server usually provided by using ISO format, we have to download the iso file first, for example:

```shell
wget http://cdimage.ubuntu.com/ubuntu-server/bionic/daily/current/bionic-server-arm64.iso
```
and extract the rootfs from iso file:
```shell
sudo mount -o loop bionic-server-arm64.iso temp
sudo unsquashfs -d roofs temp/install/filesystem.squashfs
sudo cp rootfs <rootfs mount point> -rfa
```

after the setup, it can get a ubuntu 18.04 (base) for raspberry pi3 b+

## Appendix
### reference site
* https://github.com/sakaki-/gentoo-on-rpi3-64bit
