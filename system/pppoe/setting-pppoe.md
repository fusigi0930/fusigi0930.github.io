# setting pppoe on ubuntu 18.04

## 1. install pppoeconf
```shell
sudo apt-get install pppoeconf
```

## 2. config pppoe
```shell
sudo pppoeconf
```
in fact, it just following the prompt to config your pppoe configuration, and put the current settings in your pppoeconf:
* username
* password
* network interface
* init in system startup

after the settings is completed, all of configuration should be stored in files:
* /etc/ppp/peers/dsl-provider
* /etc/ppp/pap-secrets
* /etc/ppp/chap-secrets

## 3. enable/disable pppoe by manual
it has commands can enable/disable pppoe connection:
* enable

```shell
pon /etc/ppp/peers/dsl-provider
```

* disable

```shell
poff
```
