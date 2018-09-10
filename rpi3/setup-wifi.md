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

## 5. integrate into ubuntu system
