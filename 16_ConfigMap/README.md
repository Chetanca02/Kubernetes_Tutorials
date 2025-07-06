A ConfigMap in Kubernetes is an object used to store non-confidential configuration data in key-value pairs. It allows you to decouple your application configuration from your application code. ConfigMaps can be used to manage environment variables, command-line arguments, and configuration files in a Kubernetes cluster.

Unlike most Kubernetes objects that have a spec, a ConfigMap has data and binaryData fields. These fields accept key-value pairs as their values. Both the data field and the binaryData are optional. The data field is designed to contain UTF-8 strings while the binaryData field is designed to contain binary data as base64-encoded strings.

The name of a ConfigMap must be a valid DNS subdomain name.

Each key under the data or the binaryData field must consist of alphanumeric characters, -, _ or .. The keys stored in data must not overlap with the keys in the binaryData field.

The scope of a configmap is restricted to the namespace. There is an exception where we write code to run inside the Pod that uses the Kubernetes API to read a ConfigMap. All other usage of configmap restricts to pods inside the same namespace.

By default your namespace should already have a configmap starting with the namespace name.

##### Key Features of ConfigMap:
- Stores plain text data (e.g., configuration files, JSON, YAML, etc.).
- Can be referenced by pods to set environment variables or mount as files.
- Enables dynamic updates without rebuilding container images.

#### Usage a ConfigMap
There are three main ways to use a ConfigMap:
- Filesystem: We can mount a ConfigMap into a Pod. A file is created for each entry based on the key name. The contents of that file are set to the value.
- Environment variable: A ConfigMap can be used to dynamically set the value of an environment variable.
- Command-line argument: Kubernetes supports dynamically creating the command line for a container based on ConfigMap values.

Let's create a configmap using the imperative command. Even in imperative method we can create configmap using --from-literal providing key value pair in the command itself. The other way is to add the key-value pairs as key=value in a file and using --from-file keyword. However, there is a slight difference between the two. Third way is creating env file and use keyword --from-env-file.

```
$ k create configmap my-config --from-literal=k8s.username=admin --from-literal=access.level="1"
configmap/my-config created
$ kgcm
NAME               DATA   AGE
kube-root-ca.crt   1      6d6h
my-config          2      77s
$ kgcm my-config
NAME        DATA   AGE
my-config   2      85s
$ kdcm my-config
Name:         my-config
Namespace:    kube-tut
Labels:       <none>
Annotations:  <none>

Data
====
access.level:
----
1
k8s.username:
----
admin

BinaryData
====

Events:  <none>

$ kubectl create configmap app-config --from-file=appConfig.properties 
configmap/app-config created
$ kdcm app-config
Name:         app-config
Namespace:    kube-tut
Labels:       <none>
Annotations:  <none>

Data
====
appConfig.properties:
----
app.color=Dark
app.mode=debug

BinaryData
====

Events:  <none>

$ kubectl create configmap app-config2 --from-env-file=appConfig.env 
configmap/app-config2 created
chetan@master:~/kube-tut/16_ConfigMap$ kdcm app-config2
Name:         app-config2
Namespace:    kube-tut
Labels:       <none>
Annotations:  <none>

Data
====
app.color:
----
Dark
app.mode:
----
debug

BinaryData
====

Events:  <none>
```

Lets use the configmap in the pod.

```
$ ka pod-cm.yaml 
pod/configmap-pod created
$ kgp
NAME            READY   STATUS    RESTARTS   AGE
configmap-pod   1/1     Running   0          12s
$ kex -it configmap-pod -- printenv
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=configmap-pod
NGINX_VERSION=1.27.3
NJS_VERSION=0.8.7
NJS_RELEASE=1~bookworm
PKG_RELEASE=1~bookworm
DYNPKG_RELEASE=1~bookworm
app.mode=debug
app.color=Dark
KUBERNETES_PORT_443_TCP_PORT=443
KUBERNETES_PORT_443_TCP_ADDR=10.96.0.1
KUBERNETES_SERVICE_HOST=10.96.0.1
KUBERNETES_SERVICE_PORT=443
KUBERNETES_SERVICE_PORT_HTTPS=443
KUBERNETES_PORT=tcp://10.96.0.1:443
KUBERNETES_PORT_443_TCP=tcp://10.96.0.1:443
KUBERNETES_PORT_443_TCP_PROTO=tcp
TERM=xterm
HOME=/root
```

If we need to use only 1 key from the configmap, instead of using the whole configmap, this can also be done. In the below example, you will see that only the value of `app.color` key is used as `color` in the environment varaibles, `app.mode` is not used.

```
$ ka pod-cm_2.yaml 
pod/configmap-pod2 created
$ kgp
NAME            READY   STATUS    RESTARTS   AGE
configmap-pod2   1/1     Running   0          10s
$ kex -it configmap-pod -- printenv
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=configmap-pod2
NGINX_VERSION=1.27.3
NJS_VERSION=0.8.7
NJS_RELEASE=1~bookworm
PKG_RELEASE=1~bookworm
DYNPKG_RELEASE=1~bookworm
color=Dark
KUBERNETES_PORT_443_TCP_PORT=443
KUBERNETES_PORT_443_TCP_ADDR=10.96.0.1
KUBERNETES_SERVICE_HOST=10.96.0.1
KUBERNETES_SERVICE_PORT=443
KUBERNETES_SERVICE_PORT_HTTPS=443
KUBERNETES_PORT=tcp://10.96.0.1:443
KUBERNETES_PORT_443_TCP=tcp://10.96.0.1:443
KUBERNETES_PORT_443_TCP_PROTO=tcp
TERM=xterm
HOME=/root
```

** Please note that in the same command we can create a configmap with multiple files using --from-file and --from-env-file. Also a combination of --from-literal, --from-file and --from-env-file is also permitted. --from-file and --from-env-file also take folder as arguments.**

```
k create cm multi-cm --from-env-file file1.env --from-env-file file2.env --from-env-file file3.env
```