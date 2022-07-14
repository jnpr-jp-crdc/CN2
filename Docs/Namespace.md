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

![Namespace](https://www.juniper.net/documentation/us/en/software/cn-cloud-native22/cn-cloud-native-feature-guide/cn-cloud-native-network-feature/images/jn-000222.png)

# Namespace作成

## Isolated Namespace
[Isolated Namespace Sample yaml](https://github.com/jnpr-jp-crdc/CN2/blob/main/Manifests/IsolatedNamespace.yaml)

## Isolated Namespace with Fabric SNAT
- Fabric SNATをtrueにセットした場合、PODから外部への通信はPODが稼働するWorkerNodeのInterface IP(vhost0 Interface IP)でSNATされる
- SNATのため、外部からPODへの接続は不可
- SNATはNamespace内のdefault-podnetworkにのみ有効

![FabricSNAT](https://github.com/jnpr-jp-crdc/CN2/blob/main/Docs/Images/IsolatedNamespace_SNAT.png)

[Isolated Namespace with Fabric SNAT Sample yaml](https://github.com/jnpr-jp-crdc/CN2/blob/main/Manifests/IsolatedNamespace_with_SNAT.yaml)

## Isolated Namespace with Fabric Forwarding
- Fabric Forwardingをtrueにセットした場合、PODから外部への通信はそのままUnderlayへ接続される
- Fabric forwardingはNamespace内のdefault-podnetworkにのみ有効
- Contrail Control Nodeと外部ルータ間でBGP接続をしている場合、PODのホストルートがBGP/inet unicastにより外部ルータへAdvertiseされる
  - 外部ルータ接続　参照
![FabricForwarding](https://github.com/jnpr-jp-crdc/CN2/blob/main/Docs/Images/IsolatedNamespace_with_FabricForwarding.png)

[Isolated Namespace with Fabric Forwarding Sample yaml](https://github.com/jnpr-jp-crdc/CN2/blob/main/Manifests/IsolatedNamespace_with_FabricForwarding.yaml)
