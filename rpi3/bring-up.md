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
### 3.1 GPU firmware and binary

### 3.2 u-boot for CPU

### 3.3 kernel for CPU

## 4. set the config.txt file for GPU bootcode

## 5. set the u-boot boot environment

## 6. put the relate os rootfs

PS. https://github.com/sakaki-/gentoo-on-rpi3-64bit
