# BGPaaS
- BGPaaSにより、PODとContrail Control Node間でBGP接続が可能
- PODはContrail Control Nodeとの間でRouteのImport/Exportが可能
- BGP Peerを張るPODの指定方法は2つ
  - virtualMachineInterfacesSelector
    - PODに設定したLabelを元にPeerを決定
  - virtualMachineInterfacesReferences
    - PODのVMIを指定しPeering

<img src="https://github.com/jnpr-jp-crdc/CN2/blob/main/Docs/Images/BGPaaS.png" width="50%">

## BGPaaS virtualMachineInterfacesSelector
[BGPaaS VMIS sample yaml](https://github.com/jnpr-jp-crdc/CN2/blob/main/Manifests/BGPaaS-VMIS.yaml)

Bird BGP Status
```
bird> show protocols
Name       Proto      Table      State  Since         Info
device1    Device     ---        up     02:35:15.724
direct1    Direct     ---        up     02:35:15.724
kernel1    Kernel     master4    up     02:35:15.724
bgp1_1     BGP        ---        up     02:35:20.312  Established

bird> show route
Table master4:
10.0.0.5/32          unicast [bgp1_1 02:38:09.537 from 10.0.0.2] ! (100) [AS64512?]
	via 10.0.0.1 on eth1
10.0.0.4/32          unicast [bgp1_1 02:35:20.315 from 10.0.0.2] ! (100) [AS64512?]
	via 10.0.0.1 on eth1
```

Contrail Status
```
# kubectl get BGPRouter -n ns1
NAME                    TYPE            IDENTIFIER   STATE     AGE
ns1-vn1-bgpaas-server   bgpaas-server                Success   6m37s
ns1-vn1-bird-cb1d9a9c   bgpaas-client   10.0.0.4     Success   6m37s
```

## BGPaaS virtualMachineInterfacesReferences
[BGPaaS VMIR 1 sample yaml](https://github.com/jnpr-jp-crdc/CN2/blob/main/Manifests/BGPaaS-VMIR-1.yaml)

 上記Sample yamlにて作成したPODのVirtualMachineInterface(VMI)を確認
 ```
 # kubectl get vmi -n ns1
CLUSTERNAME                                       NAME            NETWORK              PODNAME   IFCNAME   STATE     AGE
contrail-k8s-kubemanager-cluster1-juniper-local   bird-cb1d9a9c   vn1                  bird      eth1      Success   8m12s
contrail-k8s-kubemanager-cluster1-juniper-local   bird-d655861b   default-podnetwork   bird      eth0      Success   8m12s
 ```

以下のSample yamlでBGP接続するVMIを指定
 [BGPaaS VMIR 1 sample yaml](https://github.com/jnpr-jp-crdc/CN2/blob/main/Manifests/BGPaaS-VMIR-2.yaml)

Bird Status
```
bird> show protocols
Name       Proto      Table      State  Since         Info
device1    Device     ---        up     02:35:15.724
direct1    Direct     ---        up     02:35:15.724
kernel1    Kernel     master4    up     02:35:15.724
bgp1_1     BGP        ---        up     02:35:20.312  Established

bird> show route
Table master4:
10.0.0.5/32          unicast [bgp1_1 02:38:09.537 from 10.0.0.2] ! (100) [AS64512?]
	via 10.0.0.1 on eth1
10.0.0.4/32          unicast [bgp1_1 02:35:20.315 from 10.0.0.2] ! (100) [AS64512?]
	via 10.0.0.1 on eth1
```

## Contrail BGP Peer確認 (GUI)
Contrail側のBGP Peer, RouteなどはGUIから確認可能

http://controller_ip:8083
<img src="https://github.com/jnpr-jp-crdc/CN2/blob/main/Docs/Images/Contrail_BGP_Nei.png">
