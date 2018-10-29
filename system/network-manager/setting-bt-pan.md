# connect to bluetooth PAN network
in ubuntu 18.04, the network is managed by network-manager (nm), and bluetooth is controlled by bluez.
that means, we need setup the bluetooth first then set the network-manager.

## 1. pair the bluetooth device
pair the bluetooth device with PAN profile and enable the bluetooth PAN.

## 2. configure the bluez
for example:
after pair the bluetooth device, and it mac address is 7c:2e:bd:9c:b1:ab
```shell
sudo dbus-send --system --type=method_call --dest=org.bluez /org/bluez/hci0/dev_7C_2E_BD_9C_B1_AB org.bluez.Network1.Connect string:'nap'
```

## 3. connect the bluetooth PAN network
follow the previous example:
we can use the command to show the network status like:
```shell
nmcli device status
```
and it should output like:
```shell
wlan0              wifi      connected     wifi-ap          
lxcbr0             bridge    connected     lxcbr0             
eth0               ethernet  connected     Wired connection 1
7C:2E:BD:9C:B1:AB  bt        disconnected  --                 
vethT6EPN0         ethernet  unmanaged     --                 
vethURCRXT         ethernet  unmanaged     --                 
lo                 loopback  unmanaged     --
```
we need control the connection by using connection id, we can use the command:
```shell
nmcli con show
```
and it should output like:
```shell
....
BTNET    a7b6ea93-1444-493d-b449-e2c92fb1aa04  bluetooth  --
...
```
now, we can use the command to connect the bluetooth network:
```shell
sudo nmcli con up "BTNET"
```
