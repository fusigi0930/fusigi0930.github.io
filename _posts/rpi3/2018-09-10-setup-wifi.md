---
layout: post
title: setup ubuntu wifi in raspberry pi 3 b+
description: setup wifi
tags: wifi ubuntu linux system
---

# setup wifi for raspberry pi3 b+
it can use open source broadcom wifi drvier -- brcmfmac with 43445 firmware

it uses ubuntu 18.04 for the following describe:

PS. the ubuntu 18.04 server without wpa_supplicant package as default.
## 0. Disable the biosname in your ubuntu
in ubuntu 18.04, the network interface name becomes to xxxxxxx not eth0, 1, .., wlan0, 1, ...
if you confuse about this, you can disable it from the cmdline.txt in boot partitions, just add "net.ifnames=0 biosdevname=0" in the end of cmdline.
```shell
cmdline=....... net.ifnames=0 biosdevname=0
```

## 1. kernel and drivers
in this case, please set the following kernel CONFIG to "*", let the kernel preload some feaures:
* CONFIG_NET
* CONFIG_WIRELESS
* CONFIG_RFKILL
* CONFIG_CFG80211
* CONFIG_CFG80211_WEXT
* CONFIG_MAC80211


then set the broadcom full mac wifi driver to "m":
* CONFIG_BRCMFMAC

then rebuild the kernel image and drivers and updates them to boot, rootfs partitions likes:
```shell
cd <kernel>
make menuconfig <== add the CONFIGs
make clean
make -j8

cp arch/arm64/boot/Image <boot>/kernel8.img
cp drivers/net/wireless/broadcom/brcm80211/brcmfmac/brcmfmac.ko <rootfs>/lib/modules
cp drivers/net/wireless/broadcom/brcm80211/brcmutil/brcmutil.ko <rootfs>/lib/modules
```

## 2. wifi firmware
in raspberry pi3 b+, it can use the wifi firmware binrary as:
* brcmfmac43455-sdio.bin
* brcmfmac43455-sdio.clm_blob
* brcmfmac43455-sdio.txt

to download these firmware files, you can find in github armbian firmware repository, and put the files to rootfs partitions:
```shell
cp brcmfmac43455-sdio.bin <rootfs>/lib/firmware/brcm/
cp brcmfmac43455-sdio.txt <rootfs>/lib/firmware/brcm/
```

## 3. manual install driver in your system
to bring up the wlan0, use the commands:
```shell
insmod /lib/modules/brcmutil.ko
insmod /lib/modules/brcmfmac.ko
```
normally, the wlan0 will be created and we can use it to do next setting for wpa supplicant.

## 4. wpa supplicant configure
the ubuntu 18.04 server (arm64 version) dose not preload the wpa_supplicant, it needs install wpa_supplicant first.

### 4.1. install wpa_supplicant
it can find the wpa_supplicant package and relates package from http://ports.ubuntu.com/pool/main
* libnl-3-200_3.2.21-1_arm64.deb
* libnl-genl-3-200_3.2.21-1_arm64.deb
* multiarch-support_2.27-3ubuntu1_arm64.deb
* libpcsclite1_1.8.23-1_arm64.deb
* wpasupplicant_2.6-18ubuntu1_arm64.deb

then, it can use dpkg to install them:
```shell
dpkg -i multiarch-support_2.27-3ubuntu1_arm64.deb
dpkg -i libpcsclite1_1.8.23-1_arm64.deb
dpkg -i libnl-3-200_3.2.21-1_arm64.deb
dpkg -i libnl-genl-3-200_3.2.21-1_arm64.deb
dkpg -i wpasupplicant_2.6-18ubuntu1_arm64.deb
```

### 4.2. initial wpa_supplicant
first， it needs a wpa_supplicant.conf file with content:
```shell
ctrl_interface=/var/run/wpa_supplicant
update_config=1
```

then, launch the wpa_supplicant with arguments:
```shell
wpa_supplicant -B -D nl80211 -i wlan0 -c wpa_supplicant.conf
```

after this, the wpa_supplicant should be started and it can use wpa_cli to control it:
```shell
wpa_cli -i wlan0
scan
scan_result
```

### 4.3. setting the wifi ssid and password
use the wpa_passphrase can generate the wpa_supplicant configure for a ssid:
```shell
wpa_passphrase <ssid> <pass>
```
for example:
```shell
wpa_passphrase ssid-doraemon doraemon

network={
        ssid="ssid-doraemon"
        #psk="doraemon"
        psk=5c0ea2621a502d013e9e1b4a85ffa5811eb77564f125582268c7bbf0c69c6683
}
```
## 5. integrate into ubuntu system
set start wifi service after start system
### 5.1. create a init.d scirpt
create a script to install wifi driver with content:
```shell
#!/bin/sh
### BEGIN INIT INFO
# Provides:          wifi
# Required-Start:
# Required-Stop:
# Should-Start:
# Default-Start:     2 3 4 5
# Default-Stop:
# X-Interactive:
# Short-Description: wifi
### END INIT INFO

PATH=/bin:/sbin:/usr/bin:/usr/sbin
MODFILE=/lib/modules/brcmfmac.ko
MODUTIL=/lib/modules/brcmutil.ko
MODUTIL_NAME=brcmutil
MODFILE_NAME=brcmfmac

case $1 in
        start)
                if [ "$(lsmod | grep $MODFILE_NAME | wc -l)" != "0" ] || [ "$(lss
mod | grep $MODFILE_NAME | wc -l)" != "0" ]; then
                        $0 stop
                fi

                insmod $MODUTIL
                insmod $MODFILE
                ;;
        stop)
                rmmod $MODFILE_NAME 2>&1 > /dev/null
                rmmod $MODUTIL_NAME 2>&1 > /dev/null
                ;;
        restart|force-reload)
                $0 stop
                sleep 1
                $0 start
                ;;
        *)
                echo not support command argument $1
                exit 1
                ;;
esac

exit 0
```

### 5.2. update rc.d
let ubuntu install wifi driver while starting up:
```shell
update-rc.d wifi defaults
```

### 5.3. setup netplan
for now, wpa_supplicant will be called from netplan, setup wifi network in netplan as file: /etc/netplan/01-wifi.yaml
```shell
network:
  version: 2
  renderer: networkd
  wifis:
    wlan0:
      dhcp4: yes
      dhcp6: no
      gateway4: 192.168.10.1
      nameservers:
        addresses: [192.168.10.1, 8.8.8.8]
      access-points:
        "ssid-dormaemon":
          password: "doraemon"
```  
