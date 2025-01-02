#### Node Selector
Node Selector is a simple mechanism in Kubernetes that assigns pods to specific nodes by matching key-value pairs defined in the pod specification with those on the nodes' labels.

##### Node Labels:
Labels are key-value pairs attached to nodes.
These labels describe node characteristics, like their purpose, hardware, or other attributes.
for example disktype=ssd
Adding labels to nodes allows you to target Pods for scheduling on specific nodes or groups of nodes. You can use this functionality to ensure that specific Pods only run on nodes with certain isolation, security, or regulatory properties.

```
$ k label nodes worker1 disktype=ssd
node/worker1 labeled
$ kgno --show-labels
NAME      STATUS   ROLES           AGE    VERSION   LABELS
master    Ready    control-plane   6d2h   v1.29.9   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=master,kubernetes.io/os=linux,node-role.kubernetes.io/control-plane=,node.kubernetes.io/exclude-from-external-load-balancers=
worker1   Ready    <none>          6d2h   v1.29.9   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,disktype=ssd,kubernetes.io/arch=amd64,kubernetes.io/hostname=worker1,kubernetes.io/os=linux
worker2   Ready    <none>          6d2h   v1.29.9   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=worker2,kubernetes.io/os=linux
```

To remove the node label:
```
$ k label nodes worker1 disktype-
node/worker1 labeled
chetan@master:~$ kgno --show-labels
NAME      STATUS   ROLES           AGE    VERSION   LABELS
master    Ready    control-plane   6d2h   v1.29.9   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=master,kubernetes.io/os=linux,node-role.kubernetes.io/control-plane=,node.kubernetes.io/exclude-from-external-load-balancers=
worker1   Ready    <none>          6d2h   v1.29.9   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=worker1,kubernetes.io/os=linux
worker2   Ready    <none>          6d2h   v1.29.9   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=worker2,kubernetes.io/os=linux
```

##### NodeSelector:
The nodeSelector field in the pod specification is used to specify which nodes a pod can be scheduled on.
Pods will only be scheduled on nodes whose labels match the nodeSelector.
Syntax:
```
nodeSelector:
  <key>: <value>
```

Let's see this with some examples:
```
$ ka example-pod.yaml 
pod/example-pod created
$ kgp -o wide
NAME          READY   STATUS    RESTARTS   AGE   IP               NODE      NOMINATED NODE   READINESS GATES
example-pod   1/1     Running   0          8s    172.16.235.164   worker1   <none>           <none>

$ ka example-deployment.yaml 
deployment.apps/nginx-deployment created
$ kg all -o wide
NAME                                    READY   STATUS              RESTARTS   AGE     IP               NODE      NOMINATED NODE   READINESS GATES
pod/example-pod                         1/1     Running             0          5m33s   172.16.235.164   worker1   <none>           <none>
pod/nginx-deployment-56859997bb-66hdg   0/1     ContainerCreating   0          11s     <none>           worker1   <none>           <none>
pod/nginx-deployment-56859997bb-86wqv   0/1     ContainerCreating   0          11s     <none>           worker1   <none>           <none>
pod/nginx-deployment-56859997bb-bnnrn   0/1     ContainerCreating   0          11s     <none>           worker1   <none>           <none>
pod/nginx-deployment-56859997bb-bpr6c   0/1     ContainerCreating   0          11s     <none>           worker1   <none>           <none>
pod/nginx-deployment-56859997bb-dg9mt   0/1     ContainerCreating   0          11s     <none>           worker1   <none>           <none>

NAME                               READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS   IMAGES         SELECTOR
deployment.apps/nginx-deployment   0/5     5            0           11s   nginx        nginx:1.21.6   app=nginx

NAME                                          DESIRED   CURRENT   READY   AGE   CONTAINERS   IMAGES         SELECTOR
replicaset.apps/nginx-deployment-56859997bb   5         5         0       11s   nginx        nginx:1.21.6   app=nginx,pod-template-hash=56859997bb

$ kg all -o wide
NAME                                    READY   STATUS    RESTARTS   AGE     IP               NODE      NOMINATED NODE   READINESS GATES
pod/example-pod                         1/1     Running   0          7m18s   172.16.235.164   worker1   <none>           <none>
pod/nginx-deployment-56859997bb-66hdg   1/1     Running   0          116s    172.16.235.166   worker1   <none>           <none>
pod/nginx-deployment-56859997bb-86wqv   1/1     Running   0          116s    172.16.235.168   worker1   <none>           <none>
pod/nginx-deployment-56859997bb-bnnrn   1/1     Running   0          116s    172.16.235.167   worker1   <none>           <none>
pod/nginx-deployment-56859997bb-bpr6c   1/1     Running   0          116s    172.16.235.165   worker1   <none>           <none>
pod/nginx-deployment-56859997bb-dg9mt   1/1     Running   0          116s    172.16.235.169   worker1   <none>           <none>

NAME                               READY   UP-TO-DATE   AVAILABLE   AGE    CONTAINERS   IMAGES         SELECTOR
deployment.apps/nginx-deployment   5/5     5            5           116s   nginx        nginx:1.21.6   app=nginx

NAME                                          DESIRED   CURRENT   READY   AGE    CONTAINERS   IMAGES         SELECTOR
replicaset.apps/nginx-deployment-56859997bb   5         5         5       116s   nginx        nginx:1.21.6   app=nginx,pod-template-hash=56859997bb
```

*If a label is used in pod specification which is not assigned to any node, the pod will be in "Pending state".*

