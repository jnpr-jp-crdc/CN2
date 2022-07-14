# Virtual Network
- default-podnetwork, default-servicenetwork以外にNamespace内に1つ以上のVirtual Networkを作成可能
- Virtual NetworkはRoutingInstance(VRF)が分かれており、Virtual Network間の接続は不可
- VNR(後述) を使用することでVirtual Network間接続が可能
- Virtual Networkに紐づくSubnet(IPv4/IPv6)を作成
- Virtual Networkには以下の3つのForwardingModeがある
  - l2_l3 : IP FIB, MAC FIBをLookup (Default Mode)
  - l2 : MAC FIBのみLookup
  - l3 : IP FIBのみLookup
- Default-podnetwork同様にIP Fabric Forwarding, Fabric SNATの設定が可能
- NAD(Network Attachment Definition)作成時、SubnetとVirtual Networkが自動生成される
- Multusインストール時、POD作成にはNADが必要

<img src="https://github.com/jnpr-jp-crdc/CN2/blob/main/Docs/Images/VirtualNetwork.png" width="50%">

# Virtual Network作成
PODをVirtual Networkに接続するには"Virtual Network"と"Subnet"の作成が必要

## IPv4 Subnet
[IPv4 Subnet sample yaml](https://github.com/jnpr-jp-crdc/CN2/blob/main/Manifests/ipv4_subnet.yaml)

#### Subnet Option
bgpaasPrimaryIP, bgpaasSecondaryIP: BGPaaS参照

disableBGPaaSIPAutoAllocation: trueに設定した場合上記オプション必須 

## IPv6 Subnet
[IPv6 Subnet sample yaml](https://github.com/jnpr-jp-crdc/CN2/blob/main/Manifests/ipv6_subnet.yaml)

## Virtual Network
作成したSubnetとVirtual Networkを紐づける

Virtual NetworkはIPv4 Only, IPv6 Only, DualStackが可能

[Virtual Network sample yaml](https://github.com/jnpr-jp-crdc/CN2/blob/main/Manifests/virtualnetwork.yaml)

#### Virtual Network Option
forwardingMode: l2_l3(Default) or l2 or l3 

rpf: (ReversePathForwarding) trueの場合、RouteTableのValidationCheckが有効となり、RouteTableに存在しない場合通信不可

## Network Attachment Definition
[NAD sample yaml](https://github.com/jnpr-jp-crdc/CN2/blob/main/Manifests/nad.yaml)

Multus未使用時はNADは不要

VirtualNetwork, Subnetが作成されていない場合、NADを作成することでVirtual NetworkとSubnetが自動生成される
```
# kubectl get vn -n ns1
NAME                     VNI   IP FAMILIES   STATE     AGE
default-podnetwork       5     v6,v4         Success   3d20h
default-servicenetwork   6     v6,v4         Success   3d20h
nad1                     9     v6,v4         Success   11s
```
```
# kubectl get subnet -n ns1
NAME      CIDR            USAGE   STATE     AGE
nad1-v4   10.0.0.0/24     1.17%   Success   15s
nad1-v6   2001:db8::/64   0.00%   Success   15s
```





# POD作成
## Single Virtual Network NIC POD
[POD sample yaml](https://github.com/jnpr-jp-crdc/CN2/blob/main/Manifests/pod-nad.yaml)

PODはdefault-networkとvn1に接続される
```
# kubectl describe pod centos1 -n ns1
Name:         centos1
Namespace:    ns1
Priority:     0
Node:         cn2-worker1/172.27.115.242
Start Time:   Mon, 20 Jun 2022 01:44:34 +0000
Labels:       <none>
Annotations:  k8s.v1.cni.cncf.io/network-status:
                [{
                    "name": "ns1/default-podnetwork",
                    "interface": "eth0",
                    "ips": [
                        "10.233.65.8",
                        "fd85:ee78:d8a6:8607::1:108"
                    ],
                    "mac": "02:92:8e:f7:8c:40",
                    "default": true,
                    "dns": {}
                },{
                    "name": "ns1/nad1",
                    "interface": "eth1",
                    "ips": [
                        "10.0.0.2",
                        "2001:db8::2"
                    ],
                    "mac": "02:34:d0:6c:e7:0b",
                    "dns": {}
                }]
              k8s.v1.cni.cncf.io/networks: ns1/nad1
              kube-manager.juniper.net/vm-uuid: 8a08bdf0-5bec-40cb-8e25-c1921e351c93
Status:       Running
IP:           10.233.65.8
IPs:
  IP:  10.233.65.8
  IP:  fd85:ee78:d8a6:8607::1:108
```

## Multi Virtual Network NIC POD
[POD sample yaml](https://github.com/jnpr-jp-crdc/CN2/blob/main/Manifests/pod-multi-vn.yaml)

PODはdefault-networkとvn1, vn2に接続される
```
# kubectl describe pod centos1 -n ns1
Name:         centos1
Namespace:    ns1
Priority:     0
Node:         cn2-worker1/172.27.115.242
Start Time:   Wed, 22 Jun 2022 00:35:11 +0000
Labels:       <none>
Annotations:  k8s.v1.cni.cncf.io/network-status:
                [{
                    "name": "ns1/default-podnetwork",
                    "interface": "eth0",
                    "ips": [
                        "10.233.65.8",
                        "fd85:ee78:d8a6:8607::1:108"
                    ],
                    "mac": "02:f1:0f:c3:ef:17",
                    "default": true,
                    "dns": {}
                },{
                    "name": "ns1/vn1",
                    "interface": "eth1",
                    "ips": [
                        "10.0.0.2"
                    ],
                    "mac": "02:73:fa:01:d7:a3",
                    "dns": {}
                },{
                  "name": "ns1/vn2",
                  "interface": "eth2",
                  "ips": [
                      "20.0.0.2"
                  ],
                  "mac": "02:b2:0f:c0:c6:fb",
                  "dns": {}
              }]
            k8s.v1.cni.cncf.io/networks: ns1/vn1, ns1/vn2
            kube-manager.juniper.net/vm-uuid: fbf38889-0e83-4c37-8e59-29e7672a53c9
Status:       Running
IP:           10.233.65.8
IPs:
IP:  10.233.65.8
IP:  fd85:ee78:d8a6:8607::1:108
```

## POD作成 Virtual Network オプション
PODに対してIP/MAC指定が可能

[POD sample yaml](https://github.com/jnpr-jp-crdc/CN2/blob/main/Manifests/pod-option.yaml)
