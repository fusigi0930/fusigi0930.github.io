# setting l2tp server on ubuntu 18.04

## 1. install xl2tpd

in ubuntu 18.04 arm64, the xl2tpd package is not stred in apt server, you have to download it by yourself and use dpkg install it.
```shell
dpkg -i xl2tpd.deb
```

## 2. config l2tp service
