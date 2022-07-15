# VLAN Sub Interface
- PODとvRouter間でTrunk接続
- PODにはParent Interface(Tagなし)とSub Interface(Tagあり)を設定
- Parenet / Sub InterfaceはInterface Groupによって紐付ける
- Network PolicyはWhiteList形式(”except” optionはDeny Ruleとなる)

<img src="https://github.com/jnpr-jp-crdc/CN2/blob/main/Docs/Images/VLAN-SubInterface.png" width="50%">

[VLAN Sub Interface sample yaml](https://github.com/jnpr-jp-crdc/CN2/blob/main/Manifests/VLAN-SubInterface.yaml)

POD Interface確認
```
[root@centos1 /]# ip a
2: eth1.100@eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether 02:3e:63:aa:a4:f3 brd ff:ff:ff:ff:ff:ff
    inet 20.0.0.2/24 brd 20.0.0.255 scope global eth1.100
       valid_lft forever preferred_lft forever
    inet6 fe80::a816:32ff:feaa:1ac4/64 scope link
       valid_lft forever preferred_lft forever
3: eth1.200@eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether 02:54:6a:9e:f3:c5 brd ff:ff:ff:ff:ff:ff
    inet 30.0.0.2/24 brd 30.0.0.255 scope global eth1.200
       valid_lft forever preferred_lft forever
    inet6 fe80::a816:32ff:feaa:1ac4/64 scope link
       valid_lft forever preferred_lft forever
401: eth0@if402: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether 02:16:89:02:08:2f brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 10.233.65.8/18 brd 10.233.127.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fd85:ee78:d8a6:8607::1:108/112 scope global
       valid_lft forever preferred_lft forever
    inet6 fe80::40f:acff:fed2:5e46/64 scope link
       valid_lft forever preferred_lft forever
403: eth1@if404: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether 02:dc:c7:d9:ae:0f brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 10.0.0.2/24 brd 10.0.0.255 scope global eth1
       valid_lft forever preferred_lft forever
    inet6 fe80::a816:32ff:feaa:1ac4/64 scope link
       valid_lft forever preferred_lft forever
```
