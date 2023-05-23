# Service Health Check
- Vitual Machine Interface(POD Interface)に対するHealthCheck
- Ping, HTTP, BFDのHealthCheckが可能
- HealthCheckにFailした場合、監視対象InterfaceがRouteTableから削除される
- PODのMulti-Intefaceに対して異なるHealthCheckが可能

## Service Health Check設定
### PING
```
apiVersion: core.contrail.juniper.net/v1alpha1
kind: ServiceHealthCheck
metadata:
  name: ping-hc
  namespace: ns1
spec:
  serviceHealthCheckProperties:
    delay: 2
    enabled: true
    healthCheckType: end-to-end
    maxRetries: 5
    monitorType: PING
    timeout: 5
```
### BFD
```
apiVersion: core.contrail.juniper.net/v1alpha1
kind: ServiceHealthCheck
metadata:
  name: bfd-hc
  namespace: ns1
spec:
  serviceHealthCheckProperties:
    delay: 2
    enabled: true
    healthCheckType: link-local
    maxRetries: 5
    monitorType: BFD
    timeout: 5
```
### HTTP
```
apiVersion: core.contrail.juniper.net/v1alpha1
kind: ServiceHealthCheck
metadata:
  name: http-hc
  namespace: ns1
spec:
  serviceHealthCheckProperties:
    delay: 2
    enabled: true
    healthCheckType: end-to-end
    maxRetries: 5
    monitorType: HTTP
    timeout: 5
    httpMethod: GET
    expectedCodes: 200
    urlPath: /health
```
各種パラメータの詳細は[こちら](https://www.juniper.net/documentation/us/en/software/cn-cloud-native23.1/cn-cloud-native-feature-guide/cn-cloud-native-network-feature/topics/topic-map/cn-cloud-native-health-check-object.html#task_e15_krd_qjb__table_cjh_mm1_q5b)を参照

## VMIにService Health Checkをアサイン
### Default POD Intefaceにアサイン
```
apiVersion: v1
kind: Pod
metadata:
  name: hc_pod
  namespace: hc_ns
  annotations:
     core.juniper.net/health-check: '[{"name": "ping-hc", "namespace": "ns1"}]'
```
### NAD / VNに接続されているPOD Interfaceにアサイン
```
content_copy zoom_out_map
apiVersion: v1
kind: Pod
metadata:
  name: hc_pod
  namespace: hc_ns
  annotations:
          k8s.v1.cni.cncf.io/networks: |
      [
        {
          "name": "hc-vn",
          "namespace": "ns1",
          "cni-args": {
            "core.juniper.net/health-check": "[{\"name\": \"ping-hc\", \"namespace\": \"ns1\"}]"
          }
        }
      ]
```

## Service Health Check Status確認
https://[PODが稼働しているWorkerNodeIP]:8085/Snh_HealthCheckSandeshReq?

running:trueとなっていればHealth Check稼働中、active:trueとなっていればHealthCheckに成功している状態