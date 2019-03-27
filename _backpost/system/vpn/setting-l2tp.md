# setting l2tp server on ubuntu 18.04

## 1. kernel config

## 2. install xl2tpd

in ubuntu 18.04 arm64, the xl2tpd package is not stred in apt server, you have to download it by yourself and use dpkg install it.
```shell
dpkg -i xl2tpd.deb
```

### 2.1. ipsec package
StrongSwan is a opensource ipsec vpn solution in ubuntu, we can install it by using this command:

```shell
sudo apt-get install strongswan
```

## 3. config l2tp service
### 3.1. config L2TP without IPSec

### 3.2. config L2TP with IPSec
