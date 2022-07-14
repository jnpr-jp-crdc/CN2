# Namespace概要

CN2では以下の2つのタイプのNamespaceを利用することが可能

- Non-Isolated Namespace (一般的なNamespace)
  - default-pod-network, default-service-networkはNamespace間で共有
  - Podはクラスタ内の他のPODとNATなしで接続が可能
  - Podは”IP Fabric Forwarding”, “Fabric Source NAT”機能を利用することでUnderlay Networkへ接続が可能

- Isolated Namespace
  - deault-pod-network, default-service-networkはNamespace毎に自動生成される
  - Podは同一Namespace内のPODのみ接続が可能
  - PodはNon-Isolated NamespaceのServiceにアクセス可能
  - Pod/ServiceのIP AddressはクラスタのPod/Service Subnetと同じSubnetが使用される
  - Podは”IP Fabric Forwarding”, “Fabric Source NAT”機能を利用することでUnderlay Networkへ接続が可能
  - “Virtual Network Router(VNR)”機能を利用することでNamespace間接続が可能

<img src="https://www.juniper.net/documentation/us/en/software/cn-cloud-native22/cn-cloud-native-feature-guide/cn-cloud-native-network-feature/images/jn-000222.png" width="70%">

# Namespace作成

## Isolated Namespace
[Isolated Namespace Sample yaml](https://github.com/jnpr-jp-crdc/CN2/blob/main/Manifests/IsolatedNamespace.yaml)

## Isolated Namespace with Fabric SNAT
- Fabric SNATをtrueにセットした場合、PODから外部への通信はPODが稼働するWorkerNodeのInterface IP(vhost0 Interface IP)でSNATされる
- SNATのため、外部からPODへの接続は不可
- SNATはNamespace内のdefault-podnetworkにのみ有効

<img src="https://github.com/jnpr-jp-crdc/CN2/blob/main/Docs/Images/IsolatedNamespace_SNAT.png" width="50%">

[Isolated Namespace with Fabric SNAT Sample yaml](https://github.com/jnpr-jp-crdc/CN2/blob/main/Manifests/IsolatedNamespace_with_SNAT.yaml)

## Isolated Namespace with Fabric Forwarding
- Fabric Forwardingをtrueにセットした場合、PODから外部への通信はそのままUnderlayへ接続される
- Fabric forwardingはNamespace内のdefault-podnetworkにのみ有効
- Contrail Control Nodeと外部ルータ間でBGP接続をしている場合、PODのホストルートがBGP/inet unicastにより外部ルータへAdvertiseされる
  - 外部ルータ接続　参照
<img src="https://github.com/jnpr-jp-crdc/CN2/blob/main/Docs/Images/IsolatedNamespace_with_FabricForwarding.png" width="50%">

[Isolated Namespace with Fabric Forwarding Sample yaml](https://github.com/jnpr-jp-crdc/CN2/blob/main/Manifests/IsolatedNamespace_with_FabricForwarding.yaml)

# Isolated Namespace確認
新規作成したIsolated Namespaceにdefault-podnetwork, default-servicenetworkが自動作成される

```
# kubectl get vn -A
NAMESPACE                                                  NAME                     VNI   IP FAMILIES   STATE     AGE
contrail-k8s-kubemanager-cluster1-juniper-local-contrail   default-podnetwork       2     v6,v4         Success   78m
contrail-k8s-kubemanager-cluster1-juniper-local-contrail   default-servicenetwork   4     v6,v4         Success   78m
contrail                                                   ip-fabric                3                   Success   78m
contrail                                                   link-local               1                   Success   78m
ns1                                                        default-podnetwork       5     v6,v4         Success   10m
ns1                                                        default-servicenetwork   6     v6,v4         Success   10m
```

cluster namespaceのsubnetが使用されるため、Isolated Namespace毎にsubnetは作成されない
```
# kubectl get subnet -A
NAMESPACE                                                  NAME                                   CIDR                            USAGE   STATE     AGE
contrail-k8s-kubemanager-cluster1-juniper-local-contrail   default-podnetwork-pod-v4-subnet       10.233.64.0/18                  0.15%   Success   99m
contrail-k8s-kubemanager-cluster1-juniper-local-contrail   default-podnetwork-pod-v6-subnet       fd85:ee78:d8a6:8607::1:0/112    0.04%   Success   99m
contrail-k8s-kubemanager-cluster1-juniper-local-contrail   default-servicenetwork-pod-v4-subnet   10.233.0.0/18                   0.12%   Success   99m
contrail-k8s-kubemanager-cluster1-juniper-local-contrail   default-servicenetwork-pod-v6-subnet   fd85:ee78:d8a6:8607::1000/116   0.07%   Success   99m
```

# POD作成
[pod sample yaml](https://github.com/jnpr-jp-crdc/CN2/blob/main/Manifests/IsolatedNamespace_pod.yaml)

作成したPODはdefault-podnetworkに接続される
```
# kubectl describe pod centos1 -n ns1
Name:         centos1
Namespace:    ns1
Priority:     0
Node:         cn2-worker1/172.27.115.242
Start Time:   Mon, 20 Jun 2022 01:32:56 +0000
Labels:       <none>
Annotations:  kube-manager.juniper.net/vm-uuid: 2b940bcb-41e2-4b70-934f-f29b06023966
Status:       Running
IP:           10.233.65.8
IPs:
  IP:  10.233.65.8
  IP:  fd85:ee78:d8a6:8607::1:108
```
