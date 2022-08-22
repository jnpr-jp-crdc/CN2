# Contrail Status インストール
```
chmod +x contrail-tools/kubectl-contrailstatus
cp contrail-tools/kubectl-contrailstatus /usr/local/bin
```

# Contrail Status
“kubectl contrail status”でContrailコンポーネントの状態確認が可能

```
# kubectl contrailstatus --all
PODNAME(CONFIG)                          	STATUS	NODE       	IP            	MESSAGE
contrail-k8s-apiserver-5f7c4cb9f9-bnnvx  	ok    	cn2-master1	172.27.115.241
contrail-k8s-controller-687f5565f4-x9fhv 	ok    	cn2-master1	172.27.115.241
contrail-k8s-kubemanager-6998b8448f-s5dvg	ok    	cn2-master1	172.27.115.241


PODNAME(CONTROL)  	STATUS	NODE       	IP            	MESSAGE
contrail-control-0	ok    	cn2-master1	172.27.115.241


LOCAL BGPROUTER	NEIGHBOR BGPROUTER	ENCODING	STATE         	POD
cn2-master1    	vmx1              	BGP     	Established ok	contrail-control-0
cn2-master1    	cn2-master1       	XMPP    	Established ok	contrail-control-0
cn2-master1    	cn2-worker1       	XMPP    	Established ok	contrail-control-0
cn2-master1    	cn2-worker2       	XMPP    	Established ok	contrail-control-0


PODNAME(DATA)                 	STATUS	NODE       	IP            	MESSAGE
contrail-vrouter-masters-ptjng	ok    	cn2-master1	172.27.115.241
contrail-vrouter-nodes-2tpnt  	ok    	cn2-worker2	172.27.115.243
contrail-vrouter-nodes-xv425  	ok    	cn2-worker1	172.27.115.242

# kubectl contrailstatus resource routinginstance
NAME                                	NAMESPACE                                               	STATUS
DefaultIPFabricNetwork              	contrail                                                	ok
DefaultPodServiceIPFabricNetwork    	contrail-k8s-kubemanager-cluster1-juniper-local-contrail	ok
DefaultPodServiceNetwork            	contrail-k8s-kubemanager-cluster1-juniper-local-contrail	ok
DefaultServiceNetwork               	contrail-k8s-kubemanager-cluster1-juniper-local-contrail	ok
IsolatedNamespaceIPFabricNetwork    	ns1                                                     	ok
IsolatedNamespacePodServiceNetwork  	ns1                                                     	ok
IsolatedNamespacePodToDefaultService	ns1                                                     	ok
default                             	contrail                                                	ok
default-podnetwork                  	ns1                                                     	ok
default-podnetwork                  	contrail-k8s-kubemanager-cluster1-juniper-local-contrail	ok
default-servicenetwork              	contrail-k8s-kubemanager-cluster1-juniper-local-contrail	ok
default-servicenetwork              	ns1                                                     	ok
ip-fabric                           	contrail                                                	ok
link-local                          	contrail                                                	ok
vn-external1                        	ns1                                                     	ok
vn1                                 	ns1                                                     	ok

# kubectl contrailstatus resource -h
This command can be used to probe statuses of contrail resources.

Available resources:
  bgprouter, virtualnetwork, routinginstance, globalsystemconfig

```
