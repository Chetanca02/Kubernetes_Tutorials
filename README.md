# Kubernetes_Tutorials
## Author: Chetan

## Table of Contents
- [Kubernetes\_Tutorials](#kubernetes_tutorials)
  - [Author: Chetan](#author-chetan)
  - [Table of Contents](#table-of-contents)
  - [Kubernetes: Introduction](#kubernetes-introduction)
  - [Kubernetes: Components](#kubernetes-components)
    - [Control Plane](#control-plane)
    - [Worker Nodes](#worker-nodes)




## Kubernetes: Introduction
Kubernetes is an open source orchestrator for deploying containerized applications. Kubernetes was originally developed by Google, inspired by a decade of experience deploying scalable, reliable systems in containers via application-oriented APIs. Kubernetes is a platform for managing application containers across multiple hosts. It provides lots of management features for contain-oriented applications, such as auto scaling, rolling deployment, compute resource, and volume management. Same as the nature of containers, it's designed to run anywhere, so we're able to run it on a bare metal, in our data center, on the public cloud, or even hybrid cloud.
Kubernetes has grown to be the product of a rich and growing open source community.

## Kubernetes: Components
Kubernetes includes two major players:
- Masters: The Master is the heart of Kubernetes, which controls and schedules all the activities in the cluster. Also called the Control plane
- Nodes: Nodes are the workers that run our containers. A node is a single host. It may be a physical or virtual machine. Its job is to run pods.

### Control Plane
Manage the overall state of the cluster.
Control Plane Components

- kube-apiserver: The core component server that exposes the Kubernetes HTTP API. The API server provides an HTTP/HTTPS server, which provides a RESTful API for all the components in the Kubernetes master. For example, we could GET resource status, such as pod, POST to create a new resource and also watch a resource. API server reads and updates etcd, which is Kubernetes' backend data store.
- etcd: Consistent and highly-available key value store for all API server data. etcd is an open source distributed key-value store (https://coreos.com/etcd). Kubernetes stores all the RESTful API objects here. etcd is responsible for storing and replicating data.
- kube-scheduler: Scheduler decides which node is suitable for pods to run on, according to the resource capacity or the balance of the resource utilization on the node. It also considers spreading the pods in the same set to different nodes.
- kube-controller-manager: Runs controllers to implement Kubernetes API behavior. The Controller Manager controls lots of different things in the cluster. Replication Controller Manager ensures all the ReplicationControllers run on the desired container amount. Node Controller Manager responds when the nodes go down, it will then evict the pods. Endpoint Controller is used to associate the relationship between services and pods. Service Account and Token Controller are used to control default account and API access tokens.
- cloud-controller-manager (optional): Integrates with underlying cloud provider(s).

All the components can run on different hosts with clustering.

![Control Plane Components](image.png)

### Worker Nodes
Node components need to be provisioned and run on every node, which report the runtime status of the pod to the maste
Worker Node Components
- Kubelet: The kubelet is the Kubernetes representative on the node. It oversees communicating with the master components and manage the running pods. That includes the following:
• Download pod secrets from the API server
• Mount volumes
• Run the pod's container (Docker or Rkt)
• Report the status of the node and each pod
• Run container liveness probes
- kube-proxy: The kube proxy does low-level network housekeeping on each node. It refects the Kubernetes services locally and can do TCP and UDP forwarding. It finds cluster IPs via environment variables or DNS. Proxy handles the routing between pod load balancer (a.k.a. service) and pods, it also provides the routing from outside to service. There are two proxy modes, userspace and iptables. Userspace mode creates large overhead by switching kernel space and user space. Iptables mode, on the other hand, is the latest default proxy mode. It changes iptables NAT in Linux to achieve routing TCP and UDP packets across all containers.
- Container runtime: Software responsible for running containers. Common container runtime engines are:
• containerd
• CRI-O
• Docker Engine
• Mirantis Container Runtime

![Kubernetes Components](image-1.png)




