# Cluster IP
- Service(ClusterIP)を定義することにより、ECMPベースで複数PODにアクセスが可能
- Contrail環境ではdefault-podnetworkとdefault-servicenetworkはVNRにより互いにRouteLeakしているため、PODからClusterIPへ接続可
- 新規作成したVirtual Networkからdefault-servicenetworkへの接続は別途RouteLeakが必要

<img src="https://github.com/jnpr-jp-crdc/CN2/blob/main/Docs/Images/ClusterIP.png" width="50%">

[ClusterIP sample yaml](https://github.com/jnpr-jp-crdc/CN2/blob/main/Manifests/ClusterIP.yaml)

## ClusterIP 確認
```
# kubectl get services -n ns1
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
my-service   ClusterIP   10.233.54.224   <none>        80/TCP    77s

# kubectl get floatingips -n ns1
NAME                                                                 IPADDRESS       INTERFACES   PORTMAPPING   STATE     AGE
contrail-k8s-kubemanager-cluster1-juniper-local-my-service-28b76502  10.233.54.224   1            TCP/80->80    Success   8m30s

[root@centos1 /]# curl 10.233.54.224
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
--- snip ---

[root@centos1 /]# curl my-service.ns1.svc.cluster1.juniper.local
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
--- snip ---
```
