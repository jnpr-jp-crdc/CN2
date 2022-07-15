# Loadbalancer
- Service(LoadBalancer)を定義することにより、FloatingIPを使用し外部からPODへECMPアクセス可能
- FloatingIPはExternal Netから払い出され、ServiceIPにアタッチされる
- FloatingIPは外部RouterへAdvertise可能
  - External Netのfabricforwardingをtrueにすることで、Contrail Controller/外部Router間でBGP/ipv4経路がAdvertiseされ、Underlay接続となる
  - External Netのfabricforwardingをfalseにし、外部RouterのRoutingInstanceとRouteLeakすることにより、Contrail Controller/外部Router間でBGP/vpnv4経路がAdvertiseされ、Overlay接続となる

<img src="https://github.com/jnpr-jp-crdc/CN2/blob/main/Docs/Images/Loadbalancer.png" width="50%">

## Externet Networkの定義
[Externel Network sample yaml](https://github.com/jnpr-jp-crdc/CN2/blob/main/Manifests/ExternalNet.yaml)

Defaultで使用するExternal Netを定義
```
# kubectl edit kubemanager contrail-k8s-kubemanager -n contrail
Spec箇所に以下を追記

spec:
  externalNetworkSelectors:
    default-external:
      networkSelector:
        matchLabels:
          service.contrail.juniper.net/externalNetwork: default-external
      namespaceSelector:
        matchLabels:
          ns: ns1
```

## Loadbalancer - Default External Network
[Loadbalancer sample yaml](https://github.com/jnpr-jp-crdc/CN2/blob/main/Manifests/Loadbalancer.yaml)

### Loadbalancer確認
```
# kubectl get svc -n ns1
NAME         TYPE                  CLUSTER-IP     EXTERNAL-IP    PORT(S)                   AGE
my-service   LoadBalancer   10.233.33.83   100.0.0.4            8080:30597/TCP   5m21s

# kubectl get fip -n ns1
NAME                                                                                                                  IPADDRESS       INTERFACES     PORTMAPPING       STATE     AGE
contrail-k8s-kubemanager-cluster1-juniper-local-my-service-28b76502    10.233.33.83    1                          TCP/8080->80         Success   5m16s
contrail-k8s-kubemanager-cluster1-juniper-local-my-service-d7fd4d39     100.0.0.4           1                          TCP/8080->80         Success   5m12s

# curl 100.0.0.4:8080
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
--- snip ---
```

外部ルータ接続している場合は、FloatingIPが自動的に外部ルータへ広報される
```
root@vmx1> show route table inet.0

inet.0: 7 destinations, 7 routes (7 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

--- snip ---

100.0.0.4/32       *[BGP/170] 00:01:42, MED 100, localpref 200, from 172.27.115.241
                      AS path: ?, validation-state: unverified
                    >  to 172.27.115.242 via ge-0/0/0.0
```

## Loadbalancer - External Network Selector
Defaultで定義したExternal Network以外のExternal Networkを指定可能

[Loadbalancer sample yaml](https://github.com/jnpr-jp-crdc/CN2/blob/main/Manifests/Loadbalancer-ENS.yaml)

### Loadbalancer確認
```
# kubectl get svc -n ns1
NAME         TYPE                  CLUSTER-IP     EXTERNAL-IP   PORT(S)                   AGE
my-service   LoadBalancer   10.233.27.32   200.0.0.2           8080:31670/TCP   2m29s
```
