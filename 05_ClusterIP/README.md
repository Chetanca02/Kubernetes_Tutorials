### ClusterIP
ClusterIP exposes the Service on a cluster-internal IP. Choosing this value makes the Service only reachable from within the cluster. *This is the default that is used if no Service type is explicitly specified.* 
This default Service type assigns an IP address from a pool of IP addresses that the cluster has reserved for that purpose.
Several of the other types for Service build on the ClusterIP type as a foundation.
To specify the cluster IP address as part of a Service creation request, set the .spec.clusterIP field.
If a Service is defined with the .spec.clusterIP set to "None" then Kubernetes does not assign an IP address.

#### Key Features of ClusterIP Services
- Internal Access Only: The service is only accessible from within the cluster. It is not exposed to external traffic.
- Service Discovery: Kubernetes assigns a stable virtual IP (ClusterIP) to the service. Pods can discover this service using its name via the cluster's DNS.
- Load Balancing: Distributes traffic across the pods selected by the service, ensuring high availability.
- Selector and Endpoints: Uses selectors to match pods based on labels and creates endpoints pointing to those pods.
- Pods resolve the service name to its ClusterIP using Kubernetes DNS.


Understanding ClusterIP Services in Kubernetes
ClusterIP is the default type of service in Kubernetes and is used to expose services within the cluster. It provides a stable internal IP address that allows pods in the cluster to communicate with one another.

Key Features of ClusterIP Services
Internal Access Only:

The service is only accessible from within the cluster. It is not exposed to external traffic.
Service Discovery:

Kubernetes assigns a stable virtual IP (ClusterIP) to the service. Pods can discover this service using its name via the cluster's DNS.
Load Balancing:

Distributes traffic across the pods selected by the service, ensuring high availability.
Selector and Endpoints:

Uses selectors to match pods based on labels and creates endpoints pointing to those pods.
Use Cases:

Internal communication between microservices in a cluster.
Backend components like databases or APIs.
How ClusterIP Services Work
Service Definition:
A YAML file defines the service with type: ClusterIP. For example:

yaml
Copy code
apiVersion: v1
kind: Service
metadata:
  name: my-clusterip-service
spec:
  selector:
    app: my-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
ClusterIP Assignment:

Kubernetes assigns a unique IP address from the cluster's internal range to this service.
DNS Resolution:

Pods resolve the service name to its ClusterIP using Kubernetes DNS.
Comparison with Other Service Types
Feature	ClusterIP	NodePort	LoadBalancer	ExternalName
Access Scope	Internal to cluster	External via node IP and NodePort	External with a cloud provider's LB	Resolves to an external DNS name
Traffic Access	Pods inside the cluster	External clients on specific node ports	External clients via load balancer IP	External clients via DNS resolution
Use Case	Inter-pod communication	Expose services externally (simple)	Expose services with cloud LB support	Proxy to external services
IP Address	Cluster-internal IP	Node IP + port	Load balancer IP	No IP (DNS-based resolution)
Kubernetes Object	Service with type: ClusterIP	Service with type: NodePort	Service with type: LoadBalancer	Service with type: ExternalName
Complexity	Simple	Slightly more complex	Requires cloud integration	Very simple but limited in functionality.

#### When to Use ClusterIP?
- Internal Microservices: Applications that communicate with each other internally, e.g., frontend calling a backend API.
- Databases: For internal access by other services within the cluster.
- Security: Helps isolate services by restricting exposure to the cluster's internal network.

Let's create a ClusterIP service. First make sure a pod is running or spin a new pod. You can also customize your pod a liitle bit. I have a nginx pod up and running with custom index page.

``` $ k run nginx-pod --image=nginx:latest```

Let's expose the pod.
```
$ k expose pod nginx-pod --type=ClusterIP --port=80 --name=nginx-service
service/nginx-service exposed
$ kg all
NAME            READY   STATUS    RESTARTS   AGE
pod/nginx-pod   1/1     Running   0          8m7s

NAME                    TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)   AGE
service/nginx-service   ClusterIP   10.96.8.190   <none>        80/TCP    6s
```

Now we can access our pod 'nginx-pod' from our kubernetes cluster using this cluster-IP.
```
$ curl 10.96.8.190
Welcome to NGINX
```

Let's access our pod via created service in another pod.
```
$ kubectl run centos-pod --image=centos:8 -- sleep 3600
pod/centos-pod created

$ kex centos-pod -- /bin/bash
[root@centos-pod /]# curl http://nginx-service:80
Welcome to NGINX
[root@centos-pod /]# exit
```

You can create and modify the manifest fle for this pod.

Lets cleanup the service.
``` krms nginx-service```


