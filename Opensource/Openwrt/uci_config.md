# UCI

## Basic Introduction
UCI is a small utility written in C (a shell script-wrapper is available as well) and is intended to centralize the whole configuration of a device running OpenWrt.
UCI configuration files are located in the directory /etc/config/

## uci command usage

```c
root@NOKIA-10G-SFU:/# uci show network
network.loopback=interface
network.loopback.ifname='lo'
network.loopback.proto='static'
network.loopback.ipaddr='127.0.0.1'
network.loopback.netmask='255.0.0.0'
 
```

```c
uci set network.loopback.ipaddr='127.0.0.2'
uci commit
uci show network

root@TEMP:/# uci show network
network.loopback=interface
network.loopback.ifname='lo'
network.loopback.proto='static'
network.loopback.netmask='255.0.0.0'
network.loopback.ipaddr='127.0.0.2'


uci delete network.loopback
```

## Take effect
Now we finish modifying the uci config file, however it doesn't mean the new configuration take effect.
We need to restart the related service

```c
/etc/init.d/network restart
```

## UCI Example for modifying the hostname

```c
uci set system.@system[0].hostname=JohnsonPC
uci commit system
uci show system

/etc/init.d/system restart

root@JohnsonPC:/#
```

