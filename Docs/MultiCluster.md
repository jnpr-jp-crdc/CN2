# Multi Cluster
Contrailでは、Central K8S ClusterにインストールされたContrail Controllerから複数のDistributed K8S Clusterを管理することが可能です。

Central K8S ClusterにはContrail Controller及びContrail vRouterをインストールし、Distributed K8S ClusterにはContrail vRouterのみがインストールされます。

K8S Cluster毎にCNIをインストールする必要がなく、Central K8S Clusterからネットワークの統合管理及び、Cluster間接続定義が可能です。

## Multi Cluster インストール
以下は、1つ目のK8S Cluster(Contrail含む)をインストール済みの状態で、2つ目のK8S Clusterをインストールする手順となります。
※Central ClusterのContrail deployer yamlは"central_cluster_deployer_example.yaml"を使用

1. Central K8S Cluster同様、K8Sをインストール
  - CNI Pluginなし
  - Local DNS無効
  - Service Network, Pod NetworkのIPv4/IPv6 SubnetはCentral Clusterと別Subnetを指定
  - Node名はCentral Clusterと別名を指定

2. Distributed ClusterにContrail インストール

Central ClusterのkubeconfigをDistributed ClusterのMaster Nodeにコピー
```
# scp user@central:~/.kube/config central-cluster-kubeconfig
```

central-cluster-kubeconfigの"server"箇所をIPを環境に合わせて編集
```
server: https://172.27.115.180:6443
```

Central Clusterにアクセスするためのsecretを定義
```
# kubectl create ns contrail-deploy
# kubectl create secret generic central-kubeconfig -n contrail-deploy --from-file=kubeconfig=/root/central-cluster-kubeconfig
```

distributed_cluster_deployer_example.yaml内のsecretを編集
Central Cluster同様hub.juniper.netのアカウントのbase64 encodeを使用
```
- .dockerconfigjson: <base64-encoded-credential>
+ .dockerconfigjson: xxxxxxxxxxxxxxxxxxxxxxxxxxx
```

contrail deployerをapply
```
# kubectl apply -f distributed_cluster_deployer_example.yaml
```

contrail-k8s-deployerがrunningになっていることを確認
```
# kubectl get pods -n contrail-deploy
NAME                                     READY   STATUS    RESTARTS   AGE
contrail-k8s-deployer-6458859585-xhwx6   1/1     Running   0          6m
```

Central ClusterにDistributed Clusterのkubeconfigをコピー
```
# scp user@workload:~/.kube/config workload-cluster-kubeconfig
```

workload-cluster-kubeconfigの"server"箇所をIPを環境に合わせて編集
```
server: https://172.27.115.183:6443
```

Central Cluster側で、Distributed Clusterにアクセスするためのsecretを定義
```
kubectl create secret generic workload-kubeconfig -n contrail --from-file=kubeconfig=/root/workload-cluster-kubeconfig
```

Central Cluster側で、kubemanager-cluster2.yamlを定義
```
apiVersion: configplane.juniper.net/v1alpha1
kind: Kubemanager
metadata:
# 任意の名前
  name: kubemanager-cluster2
  namespace: contrail
spec:
  common:
    containers:
    - image: hub.juniper.net/cn2/contrail-k8s-kubemanager:22.2.0.93
      name: contrail-k8s-kubemanager
    nodeSelector:
      node-role.kubernetes.io/master: ""
# Distributed ClusterのSubnetを指定
  podV4Subnet: 10.234.128.0/17
  serviceV4Subnet: 10.234.0.0/17
  podV6Subnet: fd85:ee78:d8a6:8607::3:0000/112
  serviceV6Subnet: fd85:ee78:d8a6:8607::2:1000/116
# Distribute Cluster名
  clusterName: cluster2
  kubeconfigSecretName: workload-kubeconfig
  enableNad: true
# 最初のDistribute Clusterは19446。複数Clusterがある場合は以降のClusterは+1
  listenerPort: 19446
# 最初のDistribute Clusterは7699。複数Clusterがある場合は以降のClusterは+100
  constantRouteTargetNumber: 7699
```

Central Cluster側で、kubemanager-cluster2.yamlをApply
```
# kubectl apply -f manifests/kubemanager-cluster2.yaml
```

Distributed Cluster側で、vRouterをインストール
```
# kubectl apply -f distributed_cluster_vrouter_example.yaml
```

Distribute Clusterが以下の状態になっていることを確認
```
# kubectl get pods -A
NAMESPACE         NAME                                     READY   STATUS              RESTARTS   
contrail-deploy   contrail-k8s-deployer-6c88f6798f-jjx96   1/1     Running             3         
contrail          contrail-vrouter-masters-phttx           3/3     Running             0       
contrail          contrail-vrouter-nodes-xkkk8             3/3     Running             0       
kube-system       coredns-657959df74-26jrk                 1/1     Running             0       
kube-system       dns-autoscaler-b5c786945-m4vtz           1/1     Running             0       
kube-system       kube-apiserver-k8s-cp0                   1/1     Running             0       
kube-system       kube-controller-manager-k8s-cp0          1/1     Running             0       
kube-system       kube-proxy-5rbp4                         1/1     Running             0       
kube-system       kube-proxy-kr95n                         1/1     Running             0       
kube-system       kube-scheduler-k8s-cp0                   1/1     Running             0       
kube-system       nginx-proxy-k8s-worker0                  1/1     Running             0     
```

## Multi Cluster NAMESPACE
- Distributed Cluster側で定義したNamespaceはCentral Clusterにシンクされます。

Distributed Clusterで新規Namespaceを作成
```
apiVersion: v1
kind: Namespace
metadata:
  name: ns1
  labels:
    core.juniper.net/isolated-namespace: "true"
```
```
kubectl apply -f ns1.yaml
```

Central Cluster及び、Distributed ClusterのNamespaceを確認
```
Central# kubectl get ns
NAME                                                       STATUS   AGE
contrail                                                   Active   23h
contrail-analytics                                         Active   23h
contrail-deploy                                            Active   23h
contrail-k8s-kubemanager-cluster1-juniper-local-contrail   Active   23h
contrail-system                                            Active   23h
default                                                    Active   23h
kube-node-lease                                            Active   23h
kube-public                                                Active   23h
kube-system                                                Active   23h
kubemanager-cluster2-cluster2-contrail                     Active   4h59m
kubemanager-cluster2-cluster2-contrail-analytics           Active   4h54m
kubemanager-cluster2-cluster2-contrail-deploy              Active   4h54m
kubemanager-cluster2-cluster2-contrail-system              Active   4h54m
kubemanager-cluster2-cluster2-default                      Active   4h54m
kubemanager-cluster2-cluster2-kube-node-lease              Active   4h54m
kubemanager-cluster2-cluster2-kube-public                  Active   4h54m
kubemanager-cluster2-cluster2-kube-system                  Active   4h54m
kubemanager-cluster2-cluster2-ns1                          Active   4h33m


Distributed# kubectl get ns
NAME                 STATUS   AGE
contrail             Active   5h19m
contrail-analytics   Active   5h19m
contrail-deploy      Active   5h20m
contrail-system      Active   5h19m
default              Active   21h
kube-node-lease      Active   21h
kube-public          Active   21h
kube-system          Active   21h
ns1                  Active   4h37m
```

## Multi Cluster Virtual Network / Subnet
- Distributed Clusterのdefault namespace及び作成したNamespaceのdefault-pod-network, default-service-networkはCentral Cluster側にシンクされます。
- Distributed Cluster側ではVirtual Network / Subnet / VNRの定義はできません。

```
# kubectl get vn -A
NAMESPACE                                                  NAME                     VNI   IP FAMILIES   STATE     AGE
contrail-k8s-kubemanager-cluster1-juniper-local-contrail   default-podnetwork       3     v6,v4         Success   24h
contrail-k8s-kubemanager-cluster1-juniper-local-contrail   default-servicenetwork   1     v6,v4         Success   24h
contrail                                                   ip-fabric                4                   Success   24h
contrail                                                   link-local               2                   Success   24h
kubemanager-cluster2-cluster2-contrail                     default-podnetwork       5     v6,v4         Success   6h3m
kubemanager-cluster2-cluster2-contrail                     default-servicenetwork   6     v6,v4         Success   6h3m
kubemanager-cluster2-cluster2-ns1                          default-podnetwork       7     v6,v4         Success   5h37m
kubemanager-cluster2-cluster2-ns1                          default-servicenetwork   8     v6,v4         Success   5h37m
ns1                                                        default-podnetwork       9     v6,v4         Success   10m
ns1                                                        default-servicenetwork   10    v6,v4         Success   10m

# kubectl get subnet -A
NAMESPACE                                                  NAME                                   CIDR                              USAGE   STATE     AGE
contrail-k8s-kubemanager-cluster1-juniper-local-contrail   default-podnetwork-pod-v4-subnet       10.233.128.0/17                   0.02%   Success   24h
contrail-k8s-kubemanager-cluster1-juniper-local-contrail   default-podnetwork-pod-v6-subnet       fd85:ee78:d8a6:8607::1:0/112      0.01%   Success   24h
contrail-k8s-kubemanager-cluster1-juniper-local-contrail   default-servicenetwork-pod-v4-subnet   10.233.0.0/17                     0.02%   Success   24h
contrail-k8s-kubemanager-cluster1-juniper-local-contrail   default-servicenetwork-pod-v6-subnet   fd85:ee78:d8a6:8607::1000/116     0.07%   Success   24h
kubemanager-cluster2-cluster2-contrail                     default-podnetwork-pod-v4-subnet       10.234.128.0/17                   0.02%   Success   6h3m
kubemanager-cluster2-cluster2-contrail                     default-podnetwork-pod-v6-subnet       fd85:ee78:d8a6:8607::3:0/112      0.01%   Success   6h3m
kubemanager-cluster2-cluster2-contrail                     default-servicenetwork-pod-v4-subnet   10.234.0.0/17                     0.01%   Success   6h3m
kubemanager-cluster2-cluster2-contrail                     default-servicenetwork-pod-v6-subnet   fd85:ee78:d8a6:8607::2:1000/116   0.07%   Success   6h3m
```


## Multi Cluster間接続(default-pod-network)
- 各Clusterのdefault namespaceのdefault-pod-network(defaultでfabric-snatが有効)に接続されたPOD間または、新規作成したNamespace(fabric-snatを有効にする必要あり)のdefault-pod-networkに接続されたPOD間は、特別な設定なしで疎通可能です。
- Pod間の通信はWorkerNodeでSNATされ、WorkerNode間はMPLSoGREで通信します。

```
# Central ClusterのPOD
[root@centos1 /]# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
20: eth0@if21: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether 02:b1:0c:a1:23:a8 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 10.233.129.1/17 brd 10.233.255.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fd85:ee78:d8a6:8607::1:101/112 scope global
       valid_lft forever preferred_lft forever
    inet6 fe80::4cf0:4eff:fead:844d/64 scope link
       valid_lft forever preferred_lft forever

# Distributed ClusterのPOD
[root@centos2 /]# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
10: eth0@if11: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether 02:25:01:dd:63:b1 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 10.234.129.0/17 brd 10.234.255.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fd85:ee78:d8a6:8607::3:100/112 scope global
       valid_lft forever preferred_lft forever
    inet6 fe80::7c39:54ff:fe6c:d1b2/64 scope link
       valid_lft forever preferred_lft forever

# Central ClusterのPODからDistributed ClusterのPODへPing
[root@centos1 /]# ping 10.234.129.0
PING 10.234.129.0 (10.234.129.0) 56(84) bytes of data.
64 bytes from 10.234.129.0: icmp_seq=1 ttl=63 time=1.94 ms
64 bytes from 10.234.129.0: icmp_seq=2 ttl=63 time=0.891 ms
64 bytes from 10.234.129.0: icmp_seq=3 ttl=63 time=0.719 ms
```

## Multi Cluster間接続(Virtual Network)
- 各Clusterで作成したVirtual Network間接続はVNRを使用して接続します。
- default-pod-networkによる接続とは異なり、Pod間の通信はSNATなしでWorker Node間でOverlay接続されます。

作成方法を以下に記載します。
1. Distributed ClusterにてNamespaceを作成(以降で作成するVNR識別のため、"cluster2-ns1"のlabelを付与します)
```
apiVersion: v1
kind: Namespace
metadata:
  name: ns1
  labels:
    core.juniper.net/isolated-namespace: "true"
    ns: cluster2-ns1
```

2. Central ClusterにてCentral Cluster用のNamespace/Virtual Network/VNR/POD, Distributed Cluster用のVirtual Network/VNRを作成

  [VN-VNR Sample yaml](https://github.com/jnpr-jp-crdc/CN2/blob/main/Manifests/MultiCluster-VN-VNR-POD.yaml)

3. Distrited ClusterにてPODを作成
 
  [Distributed Cluster POD Sample yaml](https://github.com/jnpr-jp-crdc/CN2/blob/main/Manifests/MultiCluster-Dist-POD.yaml)
