# Port Base Mirroring
- PODの通信を特定のPODへMirroring可能

<img src="https://github.com/jnpr-jp-crdc/CN2/blob/main/Docs/Images/PortBaseMirroring.png" width="50%">

[Port Base Mirroring sample yaml](https://github.com/jnpr-jp-crdc/CN2/blob/main/Manifests/PortBaseMirroring.yaml)

### Mirror設定内容
```
apiVersion: core.contrail.juniper.net/v1alpha1
kind: MirrorDestination
metadata:
  name: mirrordestinationprofile1                        # TrafficのMirror元PODに紐付ける名前
  namespace: ns1
  labels:
    core.juniper.net/analyzer-pod-selector: analyzerpod  # TrafficのMirror先のPODの識別
spec:
  trafficDirection: both                                 # both or ingress or egress
  juniperHeader: false                                   # Udp headerを付与する場合はtrue
  udpPort: 9098
  nextHopMode: dynamic                                   # dynamic or static. static(Mirror先を指定)の場合、以降のオプションを設定
    staticNhHeader: <null for dynamic nh|vtep tunnel destip, mac, vxlanid for static nh>
    nextHopMode: <static|dynamic>
    nicAssistedMirroring: <boolean>
      nicAssistedVlanID: 
    staticNextHopHeader: 
      vTEPDestinationIP: 
      vTEPDestinationMac: 
      vxlanID:
---
apiVersion: v1                                            # TrafficのMirror先のPOD
kind: Pod
metadata:
  name: analyzerpod                                     
  namespace: ns1
  labels:
    core.juniper.net/analyzer-pod: analyzerpod            # analyzer-pod-selectorで定義したLabel名
  annotations:
    core.juniper.net/analyzer-interface: "true"           # default-podnetworkでMirror Trafficを受信
    k8s.v1.cni.cncf.io/networks: |
      [
        {
          "name": "vn1",
          "namespace": "ns1",
          "cni-args": {
            "core.juniper.net/analyzer-interface": "true" # VirtualNetworkでMirror Trafficを受信
          }
        }
      ]
spec:
  containers:
  - name: analyzerpod
    image: centos
    imagePullPolicy: IfNotPresent
    command: [ "/bin/bash", "-c" ]
    args:
      - ip route del default;
        ip route add default via 10.0.0.1;
        sleep infinity;
---
apiVersion: v1                                            # TrafficのMirror元のPOD
kind: Pod
metadata:
  name: mirrored-pod
  namespace: ns1
  annotations:
    core.juniper.net/mirror-destination: "mirrordestinationprofile1"   # default-podnetworkのTrafficをMirror先へ送信.MirrorDestinationリソースの名前を指定
    k8s.v1.cni.cncf.io/networks: |
      [
        {
          "name": "vn1",
          "namespace": "ns1",
          "cni-args": {
            "core.juniper.net/mirror-destination": "mirrordestinationprofile1"  # Virtual NetworkのTrafficをMirror先へ送信. MirrorDestinationリソースの名前を指定
          }
        }
      ]
spec:
  containers:
  - name: mirrored-pod
    image: centos
    imagePullPolicy: IfNotPresent
    command: [ "/bin/bash", "-c" ]
    args:
      - ip route del default;
        ip route add default via 10.0.0.1;
        sleep infinity;
```
