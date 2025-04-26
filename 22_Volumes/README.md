

### Volumes <a name="volumes"></a>

#### Why? <a name="why?"></a>
Before understanding volumes, lets try to understand why is this even required.

Let's create any pod with 2 containers and check the server the pod is running in:

```
ka containers-wo-volume.yaml
pod/containers-wo-volume created

$ kgp -o wide
NAME                   READY   STATUS    RESTARTS   AGE   IP               NODE      NOMINATED NODE   READINESS GATES
containers-wo-volume   2/2     Running   0          16s   172.16.235.138   worker1   <none>           <none>
```
Connect to each containers and add some files to /home.
```
$ kex containers-wo-volume -c container1 -it -- /bin/bash
[root@containers-wo-volume /]# cd /home
[root@containers-wo-volume home]# touch {1..10}.txt
[root@containers-wo-volume home]# exit
$ kex containers-wo-volume -c container1 -it -- /bin/bash
[root@containers-wo-volume /]# cd /home
[root@containers-wo-volume home]# ll
total 0
[root@containers-wo-volume home]# touch {1..10}.txt
[root@containers-wo-volume home]# exit
```

We find that files created in /home for container1 will not be visible in /home for container2.
Now let's see what happens if we kill the container. Before that make note of server where the pod is running.
For me its worker1. So we ssh to worker1 node and follow the next steps.

First we change some configuration of crictl. This can be performed on all worker nodes.
```
sudo su
crictl config --set runtime-endpoint=unix:///run/containerd/containerd.sock
crictl config --set image-endpoint=unix:///run/containerd/containerd.sock
crict version
```
Next steps are for the node where the pod is running.
We check for containers running on this node (for me worker1).
```
root@worker1:# crictl ps -a
CONTAINER           IMAGE               CREATED             STATE               NAME                ATTEMPT             POD ID              POD
b4922af0262f4       eeb6ee3f44bd0       32 seconds ago      Running             container2          0                   79491f6eddc83       containers-wo-volume
a0631290fd56f       eeb6ee3f44bd0       32 seconds ago      Running             container1          0                   79491f6eddc83       containers-wo-volume
36aee24c29a90       5c6ffd2b2a1d0       6 days ago          Running             calico-node         0                   bd16e22ea4450       calico-node-nqq8j
7c0126c9da79d       5c6ffd2b2a1d0       6 days ago          Exited              mount-bpffs         0                   bd16e22ea4450       calico-node-nqq8j
8ff5ba14af393       6527a35581401       6 days ago          Exited              install-cni         0                   bd16e22ea4450       calico-node-nqq8j
a69edacee631c       d699d5830022f       6 days ago          Running             kube-proxy          0                   d041f9d76f4e2       kube-proxy-gdcgd
ecb41aaf7da76       6527a35581401       6 days ago          Exited              upgrade-ipam        0                   bd16e22ea4450       calico-node-nqq8j
root@worker1:# crictl stop a0631290fd56f && crictl rm a0631290fd56f
a0631290fd56f
a0631290fd56f
root@worker1:# crictl ps -a
CONTAINER           IMAGE               CREATED              STATE               NAME                ATTEMPT             POD ID              POD
638f436709116       eeb6ee3f44bd0       About a minute ago   Running             container1          1                   79491f6eddc83       containers-wo-volume
b4922af0262f4       eeb6ee3f44bd0       16 minutes ago       Running             container2          0                   79491f6eddc83       containers-wo-volume
36aee24c29a90       5c6ffd2b2a1d0       6 days ago           Running             calico-node         0                   bd16e22ea4450       calico-node-nqq8j
7c0126c9da79d       5c6ffd2b2a1d0       6 days ago           Exited              mount-bpffs         0                   bd16e22ea4450       calico-node-nqq8j
8ff5ba14af393       6527a35581401       6 days ago           Exited              install-cni         0                   bd16e22ea4450       calico-node-nqq8j
a69edacee631c       d699d5830022f       6 days ago           Running             kube-proxy          0                   d041f9d76f4e2       kube-proxy-gdcgd
ecb41aaf7da76       6527a35581401       6 days ago           Exited              upgrade-ipam        0                   bd16e22ea4450       calico-node-nqq8j
```
A new instance of container1 was started after we stopped and deleted the old instance of container1.
Now we go back to master node and execute inside the container1 and check for the files we created earlier.
```
$ kex containers-wo-volume -c container1 -it -- /bin/bash
[root@containers-wo-volume /]# cd /home/
[root@containers-wo-volume home]# ll
total 0
```
They are gone.

**Conclusion: On-disk files in a container are ephemeral, which presents some problems for non-trivial applications when running in containers. One problem occurs when a container crashes or is stopped. Container state is not saved so all of the files that were created or modified during the lifetime of the container are lost. During a crash, kubelet restarts the container with a clean state. Another problem occurs when multiple containers are running in a Pod and need to share files. It can be challenging to setup and access a shared filesystem across all of the containers.**

This is a big issue! Let's say we have some important files that have been created in the whole duration that our container was running. Now they are gone.
Although not all pods require the data to be persisted, but many use cases involve persisting the data like databases.

This is where Volume comes.

#### What? <a name="what?"></a>
Kubernetes volumes are an abstraction that allow containers in a pod to access and share storage. Volumes provide a way to persist data beyond the lifecycle of a container and enable data sharing between containers within a pod. They solve the problem of ephemeral container storage, where data is lost when a container crashes or is restarted.

#### Key Characteristics <a name="key-characteristics"></a>
- Pod-Level Scope: Volumes are defined at the pod level, meaning all containers in a pod can access them.
- Lifecycle: A volume’s lifecycle is tied to the pod. When the pod is deleted, the volume is destroyed unless it’s backed by persistent storage (e.g., PersistentVolume).
- Multiple Volume Types: Kubernetes supports various volume types, each suited for different use cases (e.g., emptyDir, hostPath, local, nfs). Volumes can also be ephemeral (not persistant after restarts) or persistent. 

#### Ephemeral volumes
Ephemeral volumes follow the Pod's lifetime and get created and deleted along with the Pod.
Ephemeral volumes are specified inline in the Pod spec, which simplifies application deployment and management.
Use cases:
- Some applications need additional storage but don't care whether that data is stored persistently across restarts. For example, caching services are often limited by memory size and can move infrequently used data into storage that is slower than memory with little impact on overall performance.
- Other applications expect some read-only input data to be present in files, like configuration data or secret keys.

##### emptyDir
A volume that is created empty and deleted when the pod terminates. It is stored on the node’s local filesystem and can be used for temporary storage or sharing data between containers. Data is deleted when the pod is removed.
<u>Use case:</u> Sharing temporary files between two containers in the same pod.

Let's understand with an example:

```
$ ka emptyDir.yaml
pod/container-share-volume created
$ kg all
NAME                         READY   STATUS              RESTARTS   AGE
pod/container-share-volume   0/2     ContainerCreating   0          3s
$ kex container-share-volume -c container1 -it -- /bin/bash
[root@container-share-volume /]# cd /tmp/xchange/
[root@container-share-volume xchange]# ll
total 0
[root@container-share-volume xchange]# touch file{1..10}.txt
[root@container-share-volume xchange]# ll
total 0
-rw-r--r-- 1 root root 0 Jan 12 10:43 file1.txt
-rw-r--r-- 1 root root 0 Jan 12 10:43 file10.txt
-rw-r--r-- 1 root root 0 Jan 12 10:43 file2.txt
-rw-r--r-- 1 root root 0 Jan 12 10:43 file3.txt
-rw-r--r-- 1 root root 0 Jan 12 10:43 file4.txt
-rw-r--r-- 1 root root 0 Jan 12 10:43 file5.txt
-rw-r--r-- 1 root root 0 Jan 12 10:43 file6.txt
-rw-r--r-- 1 root root 0 Jan 12 10:43 file7.txt
-rw-r--r-- 1 root root 0 Jan 12 10:43 file8.txt
-rw-r--r-- 1 root root 0 Jan 12 10:43 file9.txt
[root@container-share-volume xchange]# exit
exit
$ kex container-share-volume -c container2 -it -- /bin/bash
[root@container-share-volume /]# cd /tmp/data/
[root@container-share-volume data]# ll
total 0
-rw-r--r-- 1 root root 0 Jan 12 10:43 file1.txt
-rw-r--r-- 1 root root 0 Jan 12 10:43 file10.txt
-rw-r--r-- 1 root root 0 Jan 12 10:43 file2.txt
-rw-r--r-- 1 root root 0 Jan 12 10:43 file3.txt
-rw-r--r-- 1 root root 0 Jan 12 10:43 file4.txt
-rw-r--r-- 1 root root 0 Jan 12 10:43 file5.txt
-rw-r--r-- 1 root root 0 Jan 12 10:43 file6.txt
-rw-r--r-- 1 root root 0 Jan 12 10:43 file7.txt
-rw-r--r-- 1 root root 0 Jan 12 10:43 file8.txt
-rw-r--r-- 1 root root 0 Jan 12 10:43 file9.txt
[root@container-share-volume data]# touch file_{11..20}.txt
[root@container-share-volume data]# ll
total 0
-rw-r--r-- 1 root root 0 Jan 12 10:43 file1.txt
-rw-r--r-- 1 root root 0 Jan 12 10:43 file10.txt
-rw-r--r-- 1 root root 0 Jan 12 10:43 file2.txt
-rw-r--r-- 1 root root 0 Jan 12 10:43 file3.txt
-rw-r--r-- 1 root root 0 Jan 12 10:43 file4.txt
-rw-r--r-- 1 root root 0 Jan 12 10:43 file5.txt
-rw-r--r-- 1 root root 0 Jan 12 10:43 file6.txt
-rw-r--r-- 1 root root 0 Jan 12 10:43 file7.txt
-rw-r--r-- 1 root root 0 Jan 12 10:43 file8.txt
-rw-r--r-- 1 root root 0 Jan 12 10:43 file9.txt
-rw-r--r-- 1 root root 0 Jan 12 10:46 file_11.txt
-rw-r--r-- 1 root root 0 Jan 12 10:46 file_12.txt
-rw-r--r-- 1 root root 0 Jan 12 10:46 file_13.txt
-rw-r--r-- 1 root root 0 Jan 12 10:46 file_14.txt
-rw-r--r-- 1 root root 0 Jan 12 10:46 file_15.txt
-rw-r--r-- 1 root root 0 Jan 12 10:46 file_16.txt
-rw-r--r-- 1 root root 0 Jan 12 10:46 file_17.txt
-rw-r--r-- 1 root root 0 Jan 12 10:46 file_18.txt
-rw-r--r-- 1 root root 0 Jan 12 10:46 file_19.txt
-rw-r--r-- 1 root root 0 Jan 12 10:46 file_20.txt
[root@container-share-volume data]# exit
exit
$ kex container-share-volume -c container1 -it -- /bin/bash
[root@container-share-volume /]# cd /tmp/xchange/
[root@container-share-volume xchange]# ll
total 0
-rw-r--r-- 1 root root 0 Jan 12 10:43 file1.txt
-rw-r--r-- 1 root root 0 Jan 12 10:43 file10.txt
-rw-r--r-- 1 root root 0 Jan 12 10:43 file2.txt
-rw-r--r-- 1 root root 0 Jan 12 10:43 file3.txt
-rw-r--r-- 1 root root 0 Jan 12 10:43 file4.txt
-rw-r--r-- 1 root root 0 Jan 12 10:43 file5.txt
-rw-r--r-- 1 root root 0 Jan 12 10:43 file6.txt
-rw-r--r-- 1 root root 0 Jan 12 10:43 file7.txt
-rw-r--r-- 1 root root 0 Jan 12 10:43 file8.txt
-rw-r--r-- 1 root root 0 Jan 12 10:43 file9.txt
-rw-r--r-- 1 root root 0 Jan 12 10:46 file_11.txt
-rw-r--r-- 1 root root 0 Jan 12 10:46 file_12.txt
-rw-r--r-- 1 root root 0 Jan 12 10:46 file_13.txt
-rw-r--r-- 1 root root 0 Jan 12 10:46 file_14.txt
-rw-r--r-- 1 root root 0 Jan 12 10:46 file_15.txt
-rw-r--r-- 1 root root 0 Jan 12 10:46 file_16.txt
-rw-r--r-- 1 root root 0 Jan 12 10:46 file_17.txt
-rw-r--r-- 1 root root 0 Jan 12 10:46 file_18.txt
-rw-r--r-- 1 root root 0 Jan 12 10:46 file_19.txt
-rw-r--r-- 1 root root 0 Jan 12 10:46 file_20.txt
[root@container-share-volume xchange]# exit
exit
```
Conclusion:
- For a Pod that defines an emptyDir volume, the volume is created when the Pod is assigned to a node.
- As the name says, the emptyDir volume is initially empty.
- All containers in the Pod can read and write the same files in the emptyDir volume, though that volume can be mounted at the same or different paths in each container.
- When a Pod is removed from a node for any reason, the data in the emptyDir is deleted permanently.

##### configMap
Description: A volume type used to inject configuration data into containers. Data in a ConfigMap is provided as files in the mounted volume.
<u>Characteristics:</u>
- Useful for externalizing application configuration.
- Can be updated without restarting the pod.

```
$ ka cm-volume.yaml 
configmap/my-config created
pod/configmap-example created
$ kgp
NAME                READY   STATUS    RESTARTS   AGE
configmap-example   1/1     Running   0          32s
$ k logs configmap-example
key1=value1
key2=value2
```
<u>Use Case:</u> Providing application configurations or environment-specific settings.

##### Secret
Similar to ConfigMap, but used to store sensitive information like passwords, tokens, or keys.
<u>Characteristics:</u>
- Data is base64-encoded.
- Can be mounted as volumes or exposed as environment variables.

```
$ ka secret-vol.yaml 
secret/my-secret created
pod/secret-example created
$ k logs secret-example
admin$ 
```

<u>Use Case:</u> Storing credentials securely and injecting them into pods.
  
##### DownwardAPI
Allows exposing pod metadata (e.g., labels, annotations) to containers as files.
<u>Characteristics:</u>
- Useful for making pod-specific information available to applications.

```
$ ka downwardAPI.yaml 
pod/downwardapi-example created
$ k logs downwardapi-example
app="my-app"
```
<u>Use Case:</u> Passing metadata dynamically to applications.

##### projected
Combines multiple volume sources (e.g., ConfigMap, Secret, DownwardAPI) into a single volume.
<u>Characteristics:</u>
- Simplifies the management of multiple ephemeral sources.

```
$ ka projected.yaml 
configmap/my-config created
secret/my-secret created
pod/projected-volume-example created
$ k logs projected-volume-example
key1=value1
key2=value2
admins3cr3t$
```

#### Persisted Volumes
Kubernetes Persistent Volumes (PV) are storage resources that are independent of the lifecycle of pods. They provide a way to persist data beyond the lifecycle of pods, allowing Kubernetes workloads to maintain state even if the pods are deleted, rescheduled, or restarted. Persistent Volumes are a key component of Kubernetes' storage architecture, enabling dynamic and static provisioning of storage.

##### Key Components of Persistent Volumes

- PersistentVolume (PV)
    - A cluster-wide storage resource that represents actual storage, such as NFS shares, cloud block storage, or local storage.
    - Administrators define PVs, and they remain available for use by pods via PersistentVolumeClaims.
- PersistentVolumeClaim (PVC)
    - A request by a user for storage, specifying desired size, access modes, and storage class.
    - PVCs abstract the details of the underlying storage and allow users to focus on their storage needs.
- StorageClass
    - A way to define different types of storage (e.g., SSD, HDD) and policies (e.g., dynamic provisioning, reclaim policy).
    - Supports dynamic provisioning of PVs.