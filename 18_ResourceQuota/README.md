Resource quotas in Kubernetes are mechanisms used to manage and limit the resource consumption of a namespace. They ensure fair resource distribution and prevent a single namespace from monopolizing cluster resources. By defining resource quotas, administrators can control the amount of compute resources (like CPU and memory), storage, and object counts (like pods, services, and persistent volume claims) that a namespace can use.

#### Resource Quota Features
- Per Namespace Scope: Resource quotas apply to a single namespace.
- Controlled Resource Types:
    - Compute resources: CPU and memory
    - Storage resources: Persistent volume claims
    - Object counts: Pods, services, ConfigMaps, secrets, etc.
- Limit Ranges: Can work alongside resource quotas to define minimum and maximum resource requests/limits per pod or container.

#### Key Benefits
- Fair Resource Allocation: Prevents a namespace from exhausting the cluster’s resources.
- Cost Management: Helps organizations control cloud costs by capping resource usage.
- Predictable Behavior: Ensures that workloads have enough resources without over-provisioning.

#### Kubernetes Resource Requests and Limits

| **Aspect**                | **Requests**                                                                                          | **Limits**                                                                                      |
|---------------------------|-------------------------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------|
| **Definition**            | Minimum amount of resources guaranteed for a container or pod.                                       | Maximum amount of resources a container or pod can use.                                       |
| **Purpose**               | Determines scheduling of the pod to a node.                                                          | Restricts resource usage to prevent overconsumption.                                          |
| **Enforcement**           | Ensures the pod is scheduled on a node with enough resources to meet the request.                    | Enforced by the Kubernetes runtime; usage above this limit is throttled (for CPU) or terminated (for memory). |
| **Unit**                  | Specified in CPU cores or memory (MiB, GiB, etc.).                                                   | Specified in the same units as requests.                                                     |
| **Usage Impact**          | If not set, the scheduler assumes zero resources are required.                                       | If not set, no upper limit is enforced.                                                      |
| **Behavior Under High Load** | Ensures the pod gets the requested amount of resources even if the node is under pressure.          | Limits the pod's usage to the defined value to prevent affecting other workloads.            |

When we specify a Pod, we can optionally specify how much of each resource a container needs. When we specify the resource request for containers in a Pod, the kube-scheduler uses this information to decide which node to place the Pod on. When we specify a resource limit for a container, the kubelet enforces those limits so that the running container is not allowed to use more of that resource than the limit we set. The kubelet also reserves at least the request amount of that system resource specifically for that container to use.

Starting in Kubernetes 1.32, we can also specify resource requests and limits at the Pod level. At the Pod level, Kubernetes 1.32 only supports resource requests or limits for specific resource types: cpu and / or memory.
For a Pod, we can specify resource limits and requests for CPU and memory by including the following:
- spec.resources.limits.cpu
- spec.resources.limits.memory
- spec.resources.requests.cpu
- spec.resources.requests.memory

#### Resource units in Kubernetes
##### CPU resource units
Limits and requests for CPU resources are measured in cpu units. In Kubernetes, 1 CPU unit is equivalent to 1 physical CPU core, or 1 virtual core, depending on whether the node is a physical host or a virtual machine running inside a physical machine.
Fractinal requests such as 0.5 or 0.1 are also allowed. 0.5 CPU means requesting half as much CPU time compared to 1.0 CPU. For CPU resource units, the quantity expression 0.1 is equivalent to the expression 100m, which can be read as "one hundred millicpu"/"one hundred millicore". Kubernetes doesn't allow to specify CPU resources with a precision finer than 1m or 0.001 CPU. 

##### Memory resource units
Limits and requests for memory are measured in bytes. We can express memory as a plain integer or as a fixed-point number using one of these quantity suffixes: E, P, T, G, M, k. We can also use the power-of-two equivalents: Ei, Pi, Ti, Gi, Mi, Ki. Please make note that 400M and 400m are not the same. 400M is 400 mebibytes whereas 400m is 400millibytes or 0.4 bytes.

Lets use resource quota in our existing namespace.
```
$ kubectl create resourcequota resource-quota --hard=requests.cpu=2,
requests.memory=4Gi,pods=6,persistentvolumeclaims=5,limits.cpu=4,limits.memory=8Gi
resourcequota/resource-quota created
```

Verify if the resource qouta is created successfully or not:
```
$ kg resourcequota
NAME             AGE   REQUEST                                                                             LIMIT
resource-quota   6s    persistentvolumeclaims: 0/5, pods: 0/6, requests.cpu: 0/2, requests.memory: 0/4Gi   limits.cpu: 0/4, limits.memory: 0/8Gi
$ k describe resourcequota resource-quota
Name:                   resource-quota
Namespace:              kube-tut
Resource                Used  Hard
--------                ----  ----
limits.cpu              0     4
limits.memory           0     8Gi
persistentvolumeclaims  0     5
pods                    0     6
requests.cpu            0     2
requests.memory         0     4Gi
```

Let's create a ReplicaSet with resource requests and limits. You can create your own manifest file or use the file 'nginx-rq.yaml '. Then describe the pod to check the resource quota.

```
$ ka nginx-rq.yaml 
replicaset.apps/nginx-replicaset created
$ kg all
NAME                         READY   STATUS    RESTARTS   AGE
pod/nginx-replicaset-2gx6r   1/1     Running   0          19s
pod/nginx-replicaset-4vsm4   1/1     Running   0          19s
pod/nginx-replicaset-75rvk   1/1     Running   0          19s
pod/nginx-replicaset-bgk7g   1/1     Running   0          19s
pod/nginx-replicaset-wfb9l   1/1     Running   0          19s

NAME                               DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-replicaset   5         5         5       19s
$ k describe po -l app=nginx
```

5 pods are created as per the replicaset, as the resource required is well within the limit of the set quota.

Next, we will try to exceed the quota.

Create a copy of the manifest file, and just change the name of the replicaset (for me it is nginx-replicaset2). Then apply this manigest file and check if all pods are created.

```
$ ka nginx-rq2.yaml 
replicaset.apps/nginx-replicaset unchanged
$ vi nginx-rq2.yaml
$ ka nginx-rq2.yaml 
replicaset.apps/nginx-replicaset2 created
$ kg all
NAME                          READY   STATUS    RESTARTS   AGE
pod/nginx-replicaset-2gx6r    1/1     Running   0          55m
pod/nginx-replicaset-4vsm4    1/1     Running   0          55m
pod/nginx-replicaset-75rvk    1/1     Running   0          55m
pod/nginx-replicaset-bgk7g    1/1     Running   0          55m
pod/nginx-replicaset-wfb9l    1/1     Running   0          55m
pod/nginx-replicaset2-r6bm8   1/1     Running   0          6s

NAME                                DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-replicaset    5         5         5       55m
replicaset.apps/nginx-replicaset2   5         1         1       6s
```

We can see that only one pod has been created for the new ReplicaSet. If I were to delete the old Replicaset, 4 more pods for new replicaset will be created.

```
krm replicaset nginx-replicaset
```
After some time:

```
$ kgp
NAME                      READY   STATUS    RESTARTS   AGE
nginx-replicaset2-9nkkd   1/1     Running   0          117s
nginx-replicaset2-pjv7k   1/1     Running   0          117s
nginx-replicaset2-r6bm8   1/1     Running   0          6m51s
nginx-replicaset2-rngtk   1/1     Running   0          117s
nginx-replicaset2-xpnb9   1/1     Running   0          117s
```

Lets try to create a pod that request more resources.
```
ka nginx-big-pod.yaml 
Error from server (Forbidden): error when creating "nginx-big-pod.yaml": pods "nginx-big-pod" is forbidden: exceeded quota: resource-quota, requested: limits.cpu=15,limits.memory=12Gi,requests.cpu=10,requests.memory=9Gi, used: limits.cpu=1,limits.memory=1000Mi,requests.cpu=500m,requests.memory=500Mi, limited: limits.cpu=4,limits.memory=8Gi,requests.cpu=2,requests.memory=4Gi
```

Let's perform the cleanup:
```
krm rs nginx-replicaset
krm rs nginx-replicaset2
krm resourcequota resource-quota
```

##### Use Cases:
- Workload Isolation: Assign specific workloads to particular nodes based on hardware requirements or isolation needs. Example - Run GPU-based workloads on nodes labeled gpu=true.
- Resource Optimization: Use nodes with different hardware configurations (e.g., SSDs for high I/O workloads, large memory nodes for memory-intensive applications).
- Compliance: Ensure specific workloads run only on nodes meeting regulatory or organizational compliance requirements.

##### Limitations:
- No Advanced Logic: NodeSelector supports only exact matches. You cannot use operators like !=, <, or >.
- Static Nature: If node labels change or nodes with matching labels are unavailable, pods remain unscheduled until nodes become available.

For more complex scheduling requirements, consider using node affinity or taints and tolerations for greater flexibility.