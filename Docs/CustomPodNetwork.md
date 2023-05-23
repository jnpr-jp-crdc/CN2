# Custom Pod Network
- [Namespace](https://github.com/jnpr-jp-crdc/CN2/blob/main/Docs/Namespace.md)や[Virtual Network](https://github.com/jnpr-jp-crdc/CN2/blob/main/Docs/VirtualNetwork.md)のセクションで説明したPod NetworkはNamespaceに作成されるdefault pod networkがPODのPrimary Interfaceとして接続され、作成したVirtual NetworkにPODを接続する際はSecondary Interfaceとして接続される構成となります。
- Custom Pod Networkを利用することで、Virtual NetworkをPrimary Interfaceとして接続することが可能
- Custom Pod NetworkはNamespaceに設定する方法とPODに設定する方法の２通り
- Custom Pod Networkを利用してもPODへの複数Interface接続可能

## NamespaceにCustom Pod Networkを設定
- NamespaceにCustom Pod Networkを定義することで、Namespace内のPod全てにCustom Pod Networkを適用

### Namespace作成
```
apiVersion: v1
kind: Namespace
metadata:
  name: ns2
  annotations:
    net.juniper.contrail.podnetwork: ns1/vn3
```
### Custom Pod Network作成
VirtualNetworkに"podNetwork: true"を設定
```
apiVersion: core.contrail.juniper.net/v3
kind: Subnet
metadata:
  name: sub1-v4
  namespace: ns1
  annotations:
    core.juniper.net/display-name: sub1-v4
spec:
  cidr: 10.0.0.0/24
  defaultGateway: 10.0.0.1
---
apiVersion: core.contrail.juniper.net/v3
kind: VirtualNetwork
metadata:
  namespace: ns1
  name: vn1
spec:
  podNetwork: true
  v4SubnetReference:
    apiVersion: core.contrail.juniper.net/v3
    kind: Subnet
    namespace: ns1
    name: sub1-v4
```
### NADの場合は以下のように指定
```
apiVersion: "k8s.cni.cncf.io/v1"
kind: NetworkAttachmentDefinition
metadata:
  name: vn1
  namespace: ns1
  annotations:
    juniper.net/networks: '{
      "ipamV4Subnet": "10.0.0.0/24",
      "podNetwork": true
    }'
```
### POD作成
```
apiVersion: v1
kind: Pod
metadata:
  name: pod1
  namespace: ns1
spec:
  containers:
  - name: pod1
    image: docker.io/centos
    command: [ "/bin/bash", "-c" ]
    args:
      - sleep infinity;
    securityContext:
      privileged: true
      capabilities:
        add:
          - NET_ADMIN
```

### POD Description
```
# kubectl describe pod pod1 -n ns1
Name:             pod1
Namespace:        ns1
Priority:         0
Service Account:  default
Node:             cn2-worker1/172.27.115.242
Start Time:       Tue, 23 May 2023 07:04:59 +0000
Labels:           <none>
Annotations:      k8s.v1.cni.cncf.io/network-status:
                    [{
                        "name": "ns1/vn1",
                        "interface": "eth0",
                        "ips": [
                            "10.0.0.2"
                        ],
                        "mac": "02:89:72:a1:fc:b9",
                        "default": true,
                        "dns": {}
                    }]
                  kube-manager.juniper.net/vm-uuid: 1a1389b6-3751-4ac9-a614-e34e042b6721
                  v1.multus-cni.io/default-network: ns1/vn1
Status:           Running
IP:               30.0.0.2
```

## PodにCustom Pod Networkを設定
- Pod単位にCustom Pod Networkを指定可能

上記のCustom Pod Network:vn1を指定したNamespace:ns1に新たにCustom Pod Network:vn2を定義し、vn2をCustom Pod NetworkとしたPODを作成

### Custom Pod Networkの設定
```
apiVersion: core.contrail.juniper.net/v3
kind: Subnet
metadata:
  name: sub2-v4
  namespace: ns1
  annotations:
    core.juniper.net/display-name: sub2-v4
spec:
  cidr: 20.0.0.0/24
  defaultGateway: 20.0.0.1
---
apiVersion: core.contrail.juniper.net/v3
kind: VirtualNetwork
metadata:
  namespace: ns1
  name: vn2
spec:
  podNetwork: true
  v4SubnetReference:
    apiVersion: core.contrail.juniper.net/v3
    kind: Subnet
    namespace: ns1
    name: sub2-v4
```

### Pod作成
```
apiVersion: v1
kind: Pod
metadata:
  name: pod1
  namespace: ns1
  annotations:
    net.juniper.contrail.podnetwork: ns1/vn2
    net.juniper.contrail.podnetwork.ip: 20.0.0.10
    net.juniper.contrail.podnetwork.cni-args: |
      {
        "net.juniper.contrail.interfacegroup": "eth0"
      }
spec:
  containers:
  - name: pod1
    image: docker.io/centos
    command: [ "/bin/bash", "-c" ]
    args:
      - sleep infinity;
    securityContext:
      privileged: true
      capabilities:
        add:
          - NET_ADMIN
```
### POD Description
Namespace内のpod2のみ異なるCustom Pod Networkがアサインされたことがわかる
```
# kubectl describe pod pod2 -n ns2
Name:             pod2
Namespace:        ns1
Priority:         0
Service Account:  default
Node:             cn2-worker1/172.27.115.242
Start Time:       Tue, 23 May 2023 09:09:06 +0000
Labels:           <none>
Annotations:      k8s.v1.cni.cncf.io/network-status:
                    [{
                        "name": "ns1/vn2",
                        "interface": "eth0",
                        "ips": [
                            "20.0.0.10"
                        ],
                        "mac": "02:09:fe:fe:d1:b5",
                        "default": true,
                        "dns": {}
                    }]
                  kube-manager.juniper.net/vm-uuid: e3e4db11-7a30-4299-b247-83bfd5b47088
                  net.juniper.contrail.podnetwork: ns2/vn4
                  net.juniper.contrail.podnetwork.cni-args:
                    {
                      "net.juniper.contrail.interfacegroup": "eth0"
                    }
                  net.juniper.contrail.podnetwork.ip: 20.0.0.10
                  v1.multus-cni.io/default-network: ns1/vn2
Status:           Running
IP:               20.0.0.10
```

## Service for Custom Pod Network
- Custom Pod Networkを作成するとCustom Pod Netowrkに紐付くService Networkが自動生成される
- Custom Pod NetworkとこのService NetworkはVNRによりRouteLeakされているため、互いに疎通が可能
```
# kubectl get vn -n ns1
NAME                     VNI    IP FAMILIES   STATE     AGE
default-podnetwork       4104   v6,v4         Success   152m
default-servicenetwork   4105   v6,v4         Success   152m
vn1                      4106   v4            Success   151m
vn1-servicenetwork       4107   v6,v4         Success   151m
vn2                      4108   v4            Success   27m
vn2-servicenetwork       4109   v6,v4         Success   27m
```

### Custom Pod Networkを指定したServiceを作成
```
apiVersion: v1
kind: Service
metadata:
  name: my-service
  namespace: ns1
  annotations:
    net.juniper.contrail.podnetwork: ns1/vn1
spec:
  selector:
    app: web
  ports:
  - port: 80
    targetPort: 80
```

### Serviceを紐付けたPodを作成
```
apiVersion: v1
kind: Pod
metadata:
  name: nginx1
  namespace: ns1
  labels:
    app: web
spec:
  containers:
  - name: nginx1
    image: docker.io/nginx
    securityContext:
      privileged: true
    ports:
    - containerPort: 80
```

### Service確認
```
# kubectl get svc -n ns1
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
my-service   ClusterIP   10.233.40.252   <none>        80/TCP    14m
```

### 上記作成したPod1から疎通確認
```
[root@pod1 /]# curl 10.233.40.252
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

