# 外部ルータ接続
- 仮想ネットワークの外部接続には物理ルータを使用可能となり、ソフトウェアGWのボトルネックを解消
- 仮想ネットワークのVPN網への延伸、NATによるInternet接続が可能

<img src="https://github.com/jnpr-jp-crdc/CN2/blob/main/Docs/Images/%E5%A4%96%E9%83%A8%E3%83%AB%E3%83%BC%E3%82%BF%E6%8E%A5%E7%B6%9A1.png" width="100%">

## 外部ルータ定義
- Contrail Control Nodeと外部ルータ間でBGP Peeringするために、Contrail側で外部ルータを定義
<img src="https://github.com/jnpr-jp-crdc/CN2/blob/main/Docs/Images/%E5%A4%96%E9%83%A8%E3%83%AB%E3%83%BC%E3%82%BF%E6%8E%A5%E7%B6%9A2.png" width="50%">

[外部ルータ定義 sample yaml](https://github.com/jnpr-jp-crdc/CN2/blob/main/Manifests/ExternalRouter.yaml)

bgpRouterParameters内でBGP接続する外部ルータのIP, RouterIDをを指定

### 外部ルータ定義確認
```
# kubectl get bgprouters -A
NAMESPACE   NAME          TYPE           IDENTIFIER       STATE     AGE
contrail    cn2-master1   control-node   172.27.115.241   Success   12d
contrail    vmx1          router         1.1.1.1          Success   14m

# kubectl describe bgprouters vmx1 -n contrail
-- snip --
Spec:
  Bgp Router Parameters:
    Address:  1.1.1.1
    Address Families:
      Family:
        route-target
        inet-vpn
        e-vpn
        inet
    Auth Data:
    Autonomous System:  64512
    Identifier:         1.1.1.1
    Port:               179
    Router Type:        router
    Vendor:             contrail
```

## 外部ルータ接続 - Fabric Forwarding
- Contrail Control Nodeと外部ルータ間でBGP/IPv4 Peeringし、Contrailで作成したVirtual Networkの経路を外部ルータへAdvertise
- PODのhost routeが外部ルータへadvertiseされる
<img src="https://github.com/jnpr-jp-crdc/CN2/blob/main/Docs/Images/%E5%A4%96%E9%83%A8%E3%83%AB%E3%83%BC%E3%82%BF%E6%8E%A5%E7%B6%9A3.png" width="50%">

[Fabric Forwarding sample yaml](https://github.com/jnpr-jp-crdc/CN2/blob/main/Manifests/ExternalRouter-FabricForwarding.yaml)

### 外部ルータ設定(JUNOS)
```
set interfaces ge-0/0/0 unit 0 family inet address 172.27.115.245/22
set interfaces lo0 unit 0 family inet address 1.1.1.1/32
set routing-options route-distinguisher-id 1.1.1.1
set routing-options resolution rib bgp.rtarget.0 resolution-ribs inet.0
set routing-options resolution rib bgp.l3vpn.0 resolution-ribs inet.3
set routing-options resolution rib bgp.l3vpn.0 resolution-ribs inet.0
set routing-options router-id 1.1.1.1
set routing-options autonomous-system 64512
set routing-options autonomous-system loops 2
set protocols router-advertisement interface lo0.0
set protocols bgp group contrail_asn-64512 type internal
set protocols bgp group contrail_asn-64512 local-address 1.1.1.1
set protocols bgp group contrail_asn-64512 hold-time 90
set protocols bgp group contrail_asn-64512 family inet unicast
set protocols bgp group contrail_asn-64512 local-as 64512
set protocols bgp group contrail_asn-64512 local-as loops 2
set protocols bgp group contrail_asn-64512 multipath
set protocols bgp group contrail_asn-64512 neighbor 172.27.115.241 peer-as 64512
```

### 外部ルータ RouteTable
```
root@vmx1> show route table inet.0

inet.0: 7 destinations, 7 routes (7 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

0.0.0.0/0          *[Static/5] 00:08:02
                    >  to 172.27.112.1 via ge-0/0/0.0
1.1.1.1/32         *[Direct/0] 22:58:03
                    >  via lo0.0
10.0.0.2/32        *[BGP/170] 04:07:09, MED 100, localpref 200, from 172.27.115.241
                      AS path: ?, validation-state: unverified
                    >  to 172.27.115.242 via ge-0/0/0.0
```

## 外部ルータ接続 - Virtual Network
- Contrail Control Nodeと外部ルータ間でBGP/VPNv4 Peeringし、Contrailで作成したVirtual Networkの経路を外部ルータへAdvertise
- PODのhost routeが外部ルータへadvertiseされる

<img src="https://github.com/jnpr-jp-crdc/CN2/blob/main/Docs/Images/%E5%A4%96%E9%83%A8%E3%83%AB%E3%83%BC%E3%82%BF%E6%8E%A5%E7%B6%9A4.png" width="50%">

[Virtual Network sample yaml](https://github.com/jnpr-jp-crdc/CN2/blob/main/Manifests/ExternalRouter-VN.yaml)

### 外部ルータ設定(JUNOS)
```
set interfaces ge-0/0/0 unit 0 family inet address 172.27.115.245/22
set interfaces lo0 unit 0 family inet address 1.1.1.1/32
set interfaces lo0 unit 1 family inet address 100.0.0.1/32
set routing-options route-distinguisher-id 1.1.1.1
set routing-options resolution rib bgp.rtarget.0 resolution-ribs inet.0
set routing-options resolution rib bgp.l3vpn.0 resolution-ribs inet.3
set routing-options resolution rib bgp.l3vpn.0 resolution-ribs inet.0
set routing-options router-id 1.1.1.1
set routing-options autonomous-system 64512
set routing-options autonomous-system loops 2
set routing-options dynamic-tunnels contrail_gre_tunnel source-address 1.1.1.1
set routing-options dynamic-tunnels contrail_gre_tunnel gre
set routing-options dynamic-tunnels contrail_gre_tunnel destination-networks 172.27.112.0/22
set protocols router-advertisement interface fxp0.0
set protocols router-advertisement interface lo0.0
set protocols bgp group contrail_asn-64512 type internal
set protocols bgp group contrail_asn-64512 local-address 1.1.1.1
set protocols bgp group contrail_asn-64512 hold-time 90
set protocols bgp group contrail_asn-64512 family inet unicast
set protocols bgp group contrail_asn-64512 family inet-vpn unicast
set protocols bgp group contrail_asn-64512 family route-target
set protocols bgp group contrail_asn-64512 local-as 64512
set protocols bgp group contrail_asn-64512 local-as loops 2
set protocols bgp group contrail_asn-64512 multipath
set protocols bgp group contrail_asn-64512 neighbor 172.27.115.241 peer-as 64512
set protocols bgp group contrail_asn-64512 vpn-apply-export
set routing-instances vn1 instance-type vrf
set routing-instances vn1 interface lo0.1
set routing-instances vn1 vrf-target target:64512:1
set routing-instances vn1 vrf-table-label
```

### 外部ルータ RouteTable
```
root@vmx1> show route table vn1.inet.0

vn1.inet.0: 2 destinations, 2 routes (2 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

10.0.0.2/32        *[BGP/170] 00:01:16, MED 100, localpref 200, from 172.27.115.241
                      AS path: ?, validation-state: unverified
                    >  via gr-0/3/0.32770, Push 67
99.0.0.1/32        *[Direct/0] 17:16:05
                    >  via lo0.1

root@vmx1> show route table inet.3

inet.3: 4 destinations, 4 routes (4 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

172.27.112.0/22    *[Tunnel/305] 18:20:48
                       Tunnel
172.27.115.241/32  *[Tunnel/300] 00:01:09
                    >  via gr-0/3/0.32771
172.27.115.242/32  *[Tunnel/300] 17:51:57
                    >  via gr-0/3/0.32770
172.27.115.243/32  *[Tunnel/300] 17:51:57
                    >  via gr-0/3/0.32769
```
