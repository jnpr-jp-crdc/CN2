# Ingress
- Ingressを定義することによりL7 LoadBalancingが可能
- Contrailでは、HAProxy, NGINX, ContourのIngress Controllerをサポート
- FloatingIPは外部RouterへAdvertise可能
  - External Netのfabricforwardingをtrueにすることで、Contrail Controller/外部Router間でBGP/ipv4経路がAdvertiseされ、Underlay接続となる
  - External Netのfabricforwardingをfalseにし、外部RouterのRoutingInstanceとRouteLeakすることにより、Contrail Controller/外部Router間でBGP/vpnv4経路がAdvertiseされ、Overlay接続となる

<img src="https://github.com/jnpr-jp-crdc/CN2/blob/main/Docs/Images/Ingress.png" width="50%">

## 外部ルータ定義
設定方法は[Loadbalancer](https://github.com/jnpr-jp-crdc/CN2/blob/main/Docs/Loadbalancer.md)セクション参照

## Ingress定義
[Ingress(NGINX) sample yaml](https://github.com/jnpr-jp-crdc/CN2/blob/main/Manifests/Ingress-nginx.yaml)

### Ingress確認
```
# kubectl get svc -n ns1
NAME   TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE
web    ClusterIP   10.233.49.40   <none>        8080/TCP   7m27s

# kubectl get ingress -n ns1
NAME       CLASS   HOSTS   ADDRESS     PORTS   AGE
ingress1     nginx     *             100.0.0.5      80           8m7s

# kubectl get fip -n ns1
NAME                                                                                                                                       IPADDRESS     INTERFACES   PORTMAPPING                           STATE     AGE
contrail-k8s-kubemanager-cluster1-juniper-local-ingress-nginx-controller-bf14c2bc     100.0.0.5          1                       TCP/80->80,TCP/443->443      Success   118m

# curl 100.0.0.5/testpath
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

100.0.0.5/32       *[BGP/170] 00:01:42, MED 100, localpref 200, from 172.27.115.241
                      AS path: ?, validation-state: unverified
                    >  to 172.27.115.242 via ge-0/0/0.0
```
