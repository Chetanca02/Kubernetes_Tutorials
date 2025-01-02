A Secret in Kubernetes is a resource that securely stores sensitive information, such as passwords, OAuth tokens, SSH keys, or any confidential data. Secrets are similar to ConfigMaps but designed specifically for sensitive information, ensuring that they are not exposed as plain text in the configuration files or container environment variables.
Secret is not landed to the disk; instead, it's stored in a per-node tmpfs filesystem. Kubelet on the mode will create a tmpfs filesystem to store secret. Secret is not designed to store large amounts of data due to storage management consideration. The current size limit of one secret is 1MB.

##### Types of Secrets
- Opaque (default): Generic key-value pairs.
- kubernetes.io/service-account-token: Used to associate a Secret with a service account.
- kubernetes.io/dockerconfigjson: Used for Docker registry authentication. Image pull secrets leverage the secrets API to automate the distribution of private registry credentials. Image pull secrets are stored just like normal secrets but are consumed through the spec.imagePullSecrets Pod specification field.
- kubernetes.io/tls: Stores TLS certificates and keys.

##### Why Use Secrets?
- Secure Storage: Secrets are encoded (base64), though not encrypted. However, they are protected by Kubernetes and underlying storage mechanisms.
- Flexibility: Secrets can be mounted as files or exposed as environment variables.
- Dynamic Updates: Secrets can be updated without restarting the pods.
- Access Control: Using RBAC, you can define which users or pods can access specific Secrets.

Secrets just like configmap, can be created using --from-lietral, --from-file and --from-env-file with the imperative method.

Let's create some secrets.

```
$ kubectl create secret generic my-secret \
  --from-literal=username=admin \
  --from-literal=password=password
secret/my-secret created
$ kgsec
NAME        TYPE     DATA   AGE
my-secret   Opaque   2      10s
$ kdsec my-secret
Name:         my-secret
Namespace:    kube-tut
Labels:       <none>
Annotations:  <none>

Type:  Opaque

Data
====
password:  8 bytes
username:  5 bytes

$ kgsec my-secret -o yaml
apiVersion: v1
data:
  password: cGFzc3dvcmQ=
  username: YWRtaW4=
kind: Secret
metadata:
  creationTimestamp: "2025-01-02T16:40:32Z"
  name: my-secret
  namespace: kube-tut
  resourceVersion: "690139"
  uid: bd146347-b023-46fc-98d6-b35054cdd002
type: Opaque
```

The secret is opaque, meaning secret are not transparent. Even in the yaml format we can see that it is encoded, whereas in describe it is not visible at all. But this doesn't mean it is completely safe.

```
$ echo "cGFzc3dvcmQ=" | base64 --decode
password

echo -n "password" | base64
cGFzc3dvcmQ=
```

Let's create secrets from files.
```
$ echo "username" > username.txt
$ echo "password" > password.txt
$ k create secret generic my-secret2 --from-file=username=username.txt --from-file=password=password.txt
secret/my-secret2 created
$ kgsec my-secret2 -o yaml
apiVersion: v1
data:
  password: cGFzc3dvcmQK
  username: dXNlcm5hbWUK
kind: Secret
metadata:
  creationTimestamp: "2025-01-02T16:52:21Z"
  name: my-secret2
  namespace: kube-tut
  resourceVersion: "691219"
  uid: ca79f76a-8c84-4046-8dd7-53307419fc02
type: Opaque
$ echo -n "cGFzc3dvcmQK" | base64 --decode
password
$ echo -n "dXNlcm5hbWUK" | base64 --decode
username
```

Let's use the created secret in a pod. For this I have created a manifest file.
```
$ ka pod-with-secret.yaml 
pod/pod-with-secret created
$ kgp
NAME              READY   STATUS    RESTARTS   AGE
pod-with-secret   1/1     Running   0          7s
$ kex pod-with-secret -it -- printenv | grep -E 'USERNAME|PASSWORD'
USERNAME=admin
PASSWORD=password
```


```
$ kubectl create secret docker-registry my-image-pull-secret \
--docker-username=<username> \
--docker-password=<password> \
--docker-email=<email-address>
```



##### Best Practices
- Encryption: Use Kubernetes Secrets Encryption at Rest to encrypt Secrets in etcd.
- RBAC: Apply Role-Based Access Control (RBAC) to limit access.
- Least Privilege: Grant pods access only to the required Secrets.
- Periodic Rotation: Rotate Secrets periodically for security.
- Avoid Embedding Secrets in Images: Never hardcode Secrets in application images.
