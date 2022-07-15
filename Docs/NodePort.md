# NodePort
- K8S Cluster外部からWorkerNode IPでPortForwardingによる接続
- Contrail環境ではFloatingIPが生成され、VMIにアタッチされる

<img src="https://github.com/jnpr-jp-crdc/CN2/blob/main/Docs/Images/NodePort.png" width="50%">

[NodePort sample yaml](https://github.com/jnpr-jp-crdc/CN2/blob/main/Manifests/NodePort.yaml)

NodePort Range: 30000 - 32767

## NodePort確認
```
# kubectl get services -n ns1
NAME         TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
my-service   NodePort   10.233.12.191   <none>        80:30010/TCP   8m24s

# kubectl get floatingips -n ns1
NAME                                                                 IPADDRESS       INTERFACES   PORTMAPPING   STATE     AGE
contrail-k8s-kubemanager-cluster1-juniper-local-my-service-28b76502  10.233.12.191   1            TCP/80->80    Success   21m

# curl worker_node_vhost0_ip:30010
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
--- snip ---
```
