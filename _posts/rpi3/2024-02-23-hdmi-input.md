---
layout: post
title: Setup Hdmi input module on raspberry pi
description: setup hdmi input
tags: hdmi linux raspberrypi
---

to simulate the different resolution and monitor the cec commands, we need enable the config CONFIG_VIDEO_TC358743_CEC in the kernel, and prepare the hdmi input module
with TC358743XBG chip, most of the modules didn't connect the i2s audio by default, if we need use the i2s signal, it needs some rework for it

# environment

## host

we use ubuntu 20.04 or later versions
```shell
sudo apt update
sudo apt install git bc bison flex libssl-dev make libncurses-dev libncurses5-dev crossbuild-essential-armhf crossbuild-essential-arm64 libc6-dev
sudo apt install v4l-utils
```

for rtsp server used, we also need download the go-1.22.0 or later environment
```shell
wget https://go.dev/dl/go1.22.0.linux-amd64.tar.gz
tar xzvf go1.22.0.linux-amd64.tar.gz
sudo mv go /usr/local/go1.22.0

# it also needs be added into system env variables
export PATH=$PATH:/usr/local/go1.22.0/bin
```

and download the source code and build for rtsp server
```shell
git clone https://github.com/bhaney/rtsp-simple-server
cd rtsp-simple-server
go build
sudo ln -s $(pwd)/rtsp-simple-server /usr/local/bin/rtsp-simple-server
```

## raspberry pi

we can download raspberry pi os from here https://www.raspberrypi.com/software/operating-systems/
after the initial settings, it needs add some packages

```shell
sudo apt install v4l-utils
```

# download the source

download the source code from github
```shell
git clone https://github.com/raspberrypi/linux rpi-kernel
```

# build the kenrel

in this part, you need find the matched case for the raspberry pi and use the related command to build the kernel.

## raspberry pi, zero, zero w, ... for 32bits

```shell
cd rpi-kernel
export ARCH=arm
export KERNEL=kernel
export IMAGE=zImage
export CROSS_COMPILE=arm-linux-gnueabihf-
make bcmrpi_defconfig
```

## raspberry pi 3, 3+, 4, 400, zero 2w, ..., for 64bits

```shell
cd rpi-kernel
export ARCH=arm64
export KERNEL=kernel8
export IMAGE=Image
export CROSS_COMPILE=aarch64-linux-gnu-
make bcm2711_defconfig
```

## raspberry pi 5 for 64bits

```shell
cd rpi-kernel
export ARCH=arm64
export KERNEL=kernel_2712
export IMAGE=Image
export CROSS_COMPILE=aarch64-linux-gnu-
make bcm2712_defconfig
```

## enable the hdmi input cec feature

use the menuconfig to enable the CONFIG_VIDEO_TC358743_CEC

```shell
# Device Drivers
# > Multimedia support (MEDIA_SUPPORT [=m])
#  -> Media ancillary drivers
#    -> Video decoders
#      -> Toshiba TC358743 decoder (VIDEO_TC358743 [=m])
#        -> Enable Toshiba TC358743 CEC support (VIDEO_TC358743_CEC)

make menuconfig
```

## build

in this case, we don't need bulid the device tree, if you want to replace the device tree by yours, please add the term "dtbs" after "make zImage"
and assume the version is x.y.z

```shell
make clean
make $IMAGE modules -j8
mkdir new_modules
make INSTALL_MOD_PATH=$(pwd)/new_modules modules_install
rm $(pwd)/new_modules/lib/modules/x.y.z/build
rm $(pwd)/new_modules/lib/modules/x.y.z/source
```

remove the symbolic link for source and build to avoid recursive copy to the sd card

# put to sdcard

we assume the sdcard is /dev/mmcblk0

```shell
mkdir temp
sudo mount -t vfat /dev/mmcblk0p1 temp
sudo cp temp/$KERNEL.img temp/$KERNEL.img.bak
sudo cp arch/$ARCH/boot/$IMAGE temp/$KERNEL.img

# ignore if you didn't rebuild the device trees
sudo cp arch/$ARCH/boot/dts/broadcom/%.dtb temp/
sudo cp arch/$ARCH/boot/dts/overlay/%.dtb* temp/overlays/
###
sync
sudo umount temp

sudo mount -t ext4 /dev/mmcblk0p2 temp
sudo cp -rf new_modules/lib/modules/x.y.z temp/lib/modules/

sync
sudo umount temp
```


# configure the hdmi input

we need enable the device tree overlay for the hdmi input module and gpu shared memory from the config.txt file

```shell
sudo mount -t vfat /dev/mmcblk0p1 temp

sudo vim temp/config.txt

# add parameters to the end of file
dtoverlay=tc358743
gpu_mem=128
```

# How to use

after finish the setup, you can try it on your raspberry pi device, you should find the 2 device nodes 

```
/dev/video0 # hdmi input streaming
/dev/cec0   # hdmi input cec control
```

## set the hdmi

before you set the edid to hdmi module, please copy one of the following edid and save to a file myedid

```
# 1080p 30 fps
 00 FF FF FF FF FF FF 00 52 62 88 88 00 88 88 88
 1C 15 01 03 80 A0 5A 78 0A 0D C9 A0 57 47 98 27
 12 48 4C 00 00 00 01 01 01 01 01 01 01 01 01 01
 01 01 01 01 01 01 01 1D 80 18 71 38 2D 40 58 2C
 45 00 80 38 74 00 00 1E 01 1D 80 18 71 38 2D 40
 58 2C 45 00 80 38 74 00 00 1E 00 00 00 FC 00 44
 43 44 5A 2D 48 32 43 20 4D 4F 44 0A 00 00 00 FD
 00 14 78 01 FF 10 00 0A 20 20 20 20 20 20 01 B9
 02 03 1A 71 47 A2 22 22 22 22 22 22 23 09 07 01
 83 01 00 00 65 03 0C 00 10 00 01 1D 80 18 71 38
 2D 40 58 2C 45 00 80 38 74 00 00 1E 01 1D 80 18
 71 38 2D 40 58 2C 45 00 80 38 74 00 00 1E 01 1D
 80 18 71 38 2D 40 58 2C 45 00 80 38 74 00 00 1E
 01 1D 80 18 71 38 2D 40 58 2C 45 00 80 38 74 00
 00 1E 00 00 00 00 00 00 00 00 00 00 00 00 00 00
 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 03

# 1080p 25 fps
 00 FF FF FF FF FF FF 00 52 62 88 88 00 88 88 88
 1C 15 01 03 80 A0 5A 78 0A 0D C9 A0 57 47 98 27
 12 48 4C 00 00 00 01 01 01 01 01 01 01 01 01 01
 01 01 01 01 01 01 01 1D 80 D0 72 38 2D 40 10 2C
 45 80 80 38 74 00 00 1E 01 1D 80 18 71 38 2D 40
 58 2C 45 00 80 38 74 00 00 1E 00 00 00 FC 00 44
 43 44 5A 2D 48 32 43 20 4D 4F 44 0A 00 00 00 FD
 00 14 78 01 FF 10 00 0A 20 20 20 20 20 20 01 C8
 02 03 1A 71 47 A1 22 22 22 22 22 22 23 09 07 01
 83 01 00 00 65 03 0C 00 10 00 01 1D 80 18 71 38
 2D 40 58 2C 45 00 80 38 74 00 00 1E 01 1D 80 18
 71 38 2D 40 58 2C 45 00 80 38 74 00 00 1E 01 1D
 80 18 71 38 2D 40 58 2C 45 00 80 38 74 00 00 1E
 01 1D 80 18 71 38 2D 40 58 2C 45 00 80 38 74 00
 00 1E 00 00 00 00 00 00 00 00 00 00 00 00 00 00
 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 04

# 720p 60 fps
 00 FF FF FF FF FF FF 00 52 62 88 88 00 88 88 88
 1C 15 01 03 80 A0 5A 78 0A 0D C9 A0 57 47 98 27
 12 48 4C 00 00 00 01 01 01 01 01 01 01 01 01 01
 01 01 01 01 01 01 01 1D 00 72 51 D0 1E 20 6E 28
 55 00 80 38 74 00 00 1E 01 1D 80 18 71 38 2D 40
 58 2C 45 00 80 38 74 00 00 1E 00 00 00 FC 00 44
 43 44 5A 2D 48 32 43 20 4D 4F 44 0A 00 00 00 FD
 00 14 78 01 FF 10 00 0A 20 20 20 20 20 20 01 74
 02 03 1A 71 47 84 22 22 22 22 22 22 23 09 07 01
 83 01 00 00 65 03 0C 00 10 00 01 1D 80 18 71 38
 2D 40 58 2C 45 00 80 38 74 00 00 1E 01 1D 80 18
 71 38 2D 40 58 2C 45 00 80 38 74 00 00 1E 01 1D
 80 18 71 38 2D 40 58 2C 45 00 80 38 74 00 00 1E
 01 1D 80 18 71 38 2D 40 58 2C 45 00 80 38 74 00
 00 1E 00 00 00 00 00 00 00 00 00 00 00 00 00 00
 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 21

```

set the edid by using the commands

```shell
v4l2-ctl --set-edid=file=myedid --fix-edid-checksums

# plug the hdmi cable
v4l2-ctl --query-dv-timings
v4l2-ctl --set-dv-bt-timings query
v4l2-ctl --verbose
```

you should see the active dv information

```
# 720p
VIDIOC_QUERYCAP: ok
VIDIOC_QUERY_DV_TIMINGS: ok
        Active width: 1280
        Active height: 720
        Total width: 1650
        Total height: 750
        Frame format: progressive
        Polarities: -vsync -hsync
        Pixelclock: 74250000 Hz (60.00 frames per second)
        Horizontal frontporch: 0
        Horizontal sync: 370
        Horizontal backporch: 0
        Vertical frontporch: 0
        Vertical sync: 30
        Vertical backporch: 0
        Standards:
        Flags:

# 1080p
VIDIOC_QUERYCAP: ok
VIDIOC_QUERY_DV_TIMINGS: ok
        Active width: 1920
        Active height: 1080
        Total width: 2200
        Total height: 1125
        Frame format: progressive
        Polarities: -vsync -hsync
        Pixelclock: 148500000 Hz (60.00 frames per second)
        Horizontal frontporch: 0
        Horizontal sync: 280
        Horizontal backporch: 0
        Vertical frontporch: 0
        Vertical sync: 45
        Vertical backporch: 0
        Standards:
        Flags:
```

## get the hdmi input data

we can directly use the ffmpeg utilities to do this like:

```shell
# playback the hdmi input screen
ffplay -i /dev/video0

# playback on network over ssh
ssh <user>@<ip addr> "ffmpeg -i /dev/video0  -f v4l2 -c copy -f nut pipe:1" | ffplay -i pipe:0

# playback on network over rtps
# setup rtsp server
rtsp-simple-server

# send data to rtsp server
ffmpeg -rtsp_transport tcp -c:v libx264  -c:a copy -f rtsp rtsp://192.168.10.112:8554/stream -i /dev/video0

# get data from rtsp server
ffplay -rtsp_transport tcp rtsp://192.168.10.112:8554/stream

```

## cec control

we can use the command to log the cec commands

```shell
sudo cec-ctl -d /dev/cec0 --tv -S -r -M -w
```

it should response like

```shell
Driver Info:
        Driver Name                : tc358743
        Adapter Name               : 10-000f
        Capabilities               : 0x0000003e
                Logical Addresses
                Transmit
                Passthrough
                Remote Control Support
                Monitor All
        Driver version             : 6.1.73
        Available Logical Addresses: 4
        Connector Info             : None
        Physical Address           : 0.0.0.0
        Logical Address Mask       : 0x0001
        CEC Version                : 2.0
        Vendor ID                  : 0x000c03 (HDMI)
        OSD Name                   : 'TV'
        Logical Addresses          : 1 (Allow RC Passthrough)

          Logical Address          : 0 (TV)
            Primary Device Type    : TV
            Logical Address Type   : TV
            All Device Types       : TV
            RC TV Profile          : None
            Device Features        :
                None


        Topology:

            0.0.0.0: TV


(warn: State Change events were lost)
Event: State Change: PA: 0.0.0.0, LA mask: 0x0001, Conn Info: no
Received from Playback Device 1 to TV (4 to 0): TEXT_VIEW_ON (0x0d)
        Raw: 0x40 0x0d (@ )
Transmitted by TV to Playback Device 1 (0 to 4): FEATURE_ABORT (0x00):
        abort-msg: 13 (0x0d, TEXT_VIEW_ON)
        reason: unrecognized-op (0x00)
        Tx, Not Acknowledged (4), Max Retries
        Raw: 0x04 0x00 0x0d 0x00 (    )
Received from Playback Device 1 to all (4 to 15): ACTIVE_SOURCE (0x82):
        phys-addr: 1.0.0.0
        Raw: 0x4f 0x82 0x10 0x00 (O   )
Received from Playback Device 1 to TV (4 to 0): TEXT_VIEW_ON (0x0d)
        Raw: 0x40 0x0d (@ )
Transmitted by TV to Playback Device 1 (0 to 4): FEATURE_ABORT (0x00):
        abort-msg: 13 (0x0d, TEXT_VIEW_ON)
        reason: unrecognized-op (0x00)
        Tx, Not Acknowledged (4), Max Retries
        Raw: 0x04 0x00 0x0d 0x00 (    )
Received from Playback Device 1 to all (4 to 15): ACTIVE_SOURCE (0x82):
        phys-addr: 1.0.0.0
        Raw: 0x4f 0x82 0x10 0x00 (O   )
```