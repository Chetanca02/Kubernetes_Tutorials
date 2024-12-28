### NodePort
ClusterIP exposes the pod internally in the cluster. What if we want to access it from outside the cluster?
This is the next stage of the ClusterIP where we want to deploy the application or service that should be accessible to the world without any interruption. In this Service, the node port exposes the service or application through the static port on each node’s IP. If you don’t care what port it is, don’t specify one and your cluster will randomly assign one for you.
NodePort must be within the port range 30000-32767.

#### How NodePort Works
- Port Allocation: Kubernetes assigns a port from a pre-defined range (default: 30000–32767) on each node. This port is referred to as the NodePort.
- Traffic Flow: External users send requests to the Node's IP address and the NodePort. The request is forwarded to the backend Pods selected by the service using its label selectors.
- Cluster-wide Exposure: The service is accessible via the NodePort on all nodes in the cluster, regardless of where the Pods are running.

#### Terminologies used while creating NodePort service
- type: NodePort: Indicates the service type.
- port: The port exposed by the service within the cluster.
- targetPort: The port on the container.
- nodePort: The port exposed on the node (optional; if not specified, Kubernetes will allocate one from the default range).

Once again we will use the same pod 'nginx-pod' and expose it. Please spin one for you if you do not have one up and running.
```
$ k expose pod nginx-pod --type=NodePort --port=80 --name=nginx-service
service/nginx-service exposed

$ kg all
NAME             READY   STATUS    RESTARTS   AGE
pod/centos-pod   1/1     Running   0          14m
pod/nginx-pod    1/1     Running   0          34m

NAME                    TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
service/nginx-service   NodePort   10.111.31.255   <none>        80:31303/TCP   3s

$ curl localhost:31303
Welcome to NGINX

$ kex centos-pod -- /bin/bash
[root@centos-pod /]# curl http://nginx-service:80
Welcome to NGINX
[root@centos-pod /]# exit
```
