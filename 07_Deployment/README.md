A Deployment is a Kubernetes workload used to manage stateless applications. It provides declarative updates for Pods and ReplicaSets, allowing you to define your desired state for an application and ensuring it runs as specified.

**Key Features of Deployment**
- Declarative Updates: You declare the desired state of your application (e.g., number of replicas, image version), and Kubernetes ensures that this state is achieved.

- Rolling Updates: Deployment supports rolling updates to minimize downtime during application updates.

- Self-Healing: If a Pod crashes or a node fails, Kubernetes automatically replaces the failed Pods to maintain the desired state.

- Scalability: Easily scale the application up or down by adjusting the replicas field.

- Rollback: You can roll back to a previous version of the Deployment if an update causes issues.

**When to Use Deployment**
- Stateless applications like web servers (e.g., Nginx, Apache).
- Applications where scaling and rolling updates are critical.
- Microservices that don’t require persistent storage or ordered deployments.

Let's look into more details with an example:

```
$ kubectl create deployment nginx-deployment --image=nginx:1.14.2 --replicas=2
deployment.apps/nginx-deployment created

$ kg all
NAME                                    READY   STATUS    RESTARTS   AGE
pod/nginx-deployment-75c7fcd9ff-j2xqr   1/1     Running   0          4s
pod/nginx-deployment-75c7fcd9ff-vhn27   1/1     Running   0          4s

NAME                               READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx-deployment   2/2     2            2           4s

NAME                                          DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-deployment-75c7fcd9ff   2         2         2       4s
```

We have created a deployment for nginx using image for version 1.14.2. Currently 2 replicas have been created. The deployment creates a replicaset with desired replicas. We will try to change the image to latest image and increase the replica. But before performing this update lets check the current rollout history.

```
$ kubectl rollout history deployment/nginx-deployment
deployment.apps/nginx-deployment 
REVISION  CHANGE-CAUSE
1         <none>
```

Currently we have only one rollout (and one replicaset corresponding to that).
CHANGE-CAUSE is copied from the Deployment annotation kubernetes.io/change-cause to its revisions upon creation. You can specify the CHANGE-CAUSE message by:
- Annotating the Deployment with 
```
$ kubectl annotate deployment/nginx-deployment kubernetes.io/change-cause="image updated to latest"
```
- Manually editing the manifest of the resource.

Let's see the first method of annotating first:
```
$ kubectl annotate deployment/nginx-deployment kubernetes.io/change-cause="initial deployment"
deployment.apps/nginx-deployment annotated

$ kubectl rollout history deployment/nginx-deployment
deployment.apps/nginx-deployment 
REVISION  CHANGE-CAUSE
1         initial deployment
```

Let's try to edit this deployment with latest nginx version and 3 replicas. Also we will use the secong method for annotating the deployment rollout.

We will change the revision and change cause under metadata-->annotations, replicas under spec and image under spec-->template-->spec-->containers:
```metadata:
  annotations:
    deployment.kubernetes.io/revision: "2"
    kubernetes.io/change-cause: change to latest image, add one replica
```
```
spec:
  replicas: 3
  template:
    spec:
      containers:
      - image: nginx:latest
```

```
$ k edit deployment/nginx-deployment
deployment.apps/nginx-deployment edited

$ kg all
$ while true; do kg all; sleep 2; done
NAME                                    READY   STATUS              RESTARTS   AGE
pod/nginx-deployment-75c7fcd9ff-j2xqr   1/1     Running             0          3m40s
pod/nginx-deployment-75c7fcd9ff-vhn27   0/1     Terminating         0          3m40s
pod/nginx-deployment-dc589f6f6-6q78w    1/1     Running             0          9s
pod/nginx-deployment-dc589f6f6-bb7xd    0/1     ContainerCreating   0          5s
pod/nginx-deployment-dc589f6f6-rmsnx    1/1     Running             0          7s

NAME                               READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx-deployment   3/3     3            3           3m40s

NAME                                          DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-deployment-75c7fcd9ff   1         1         1       3m40s
replicaset.apps/nginx-deployment-dc589f6f6    3         3         2       10s
NAME                                    READY   STATUS        RESTARTS   AGE
pod/nginx-deployment-75c7fcd9ff-j2xqr   1/1     Terminating   0          3m43s
pod/nginx-deployment-dc589f6f6-6q78w    1/1     Running       0          12s
pod/nginx-deployment-dc589f6f6-bb7xd    1/1     Running       0          8s
pod/nginx-deployment-dc589f6f6-rmsnx    1/1     Running       0          10s

NAME                               READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx-deployment   3/3     3            3           3m43s

NAME                                          DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-deployment-75c7fcd9ff   0         0         0       3m43s
replicaset.apps/nginx-deployment-dc589f6f6    3         3         3       13s
NAME                                    READY   STATUS        RESTARTS   AGE
pod/nginx-deployment-75c7fcd9ff-j2xqr   1/1     Terminating   0          3m45s
pod/nginx-deployment-dc589f6f6-6q78w    1/1     Running       0          14s
pod/nginx-deployment-dc589f6f6-bb7xd    1/1     Running       0          10s
pod/nginx-deployment-dc589f6f6-rmsnx    1/1     Running       0          12s

NAME                               READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx-deployment   3/3     3            3           3m45s

NAME                                          DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-deployment-75c7fcd9ff   0         0         0       3m45s
replicaset.apps/nginx-deployment-dc589f6f6    3         3         3       15s
NAME                                    READY   STATUS        RESTARTS   AGE
pod/nginx-deployment-75c7fcd9ff-j2xqr   1/1     Terminating   0          3m47s
pod/nginx-deployment-dc589f6f6-6q78w    1/1     Running       0          16s
pod/nginx-deployment-dc589f6f6-bb7xd    1/1     Running       0          12s
pod/nginx-deployment-dc589f6f6-rmsnx    1/1     Running       0          14s

NAME                               READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx-deployment   3/3     3            3           3m47s

NAME                                          DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-deployment-75c7fcd9ff   0         0         0       3m47s
replicaset.apps/nginx-deployment-dc589f6f6    3         3         3       17s
NAME                                   READY   STATUS    RESTARTS   AGE
pod/nginx-deployment-dc589f6f6-6q78w   1/1     Running   0          19s
pod/nginx-deployment-dc589f6f6-bb7xd   1/1     Running   0          15s
pod/nginx-deployment-dc589f6f6-rmsnx   1/1     Running   0          17s

NAME                               READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx-deployment   3/3     3            3           3m50s

NAME                                          DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-deployment-75c7fcd9ff   0         0         0       3m50s
replicaset.apps/nginx-deployment-dc589f6f6    3         3         3       20s
```

We can see that newer pods have been created with the edited specifications, where as the older pods get terminated.

Lets check the rollout history:
```
$ kubectl rollout history deployment/nginx-deployment
deployment.apps/nginx-deployment 
REVISION  CHANGE-CAUSE
1         initial deployment
2         change to latest image, add one replica
```

Now let's try to do a rollback.

```
$ k rollout undo deployment/nginx-deployment
deployment.apps/nginx-deployment rolled back

$ while true; do kg all; sleep 2; done
NAME                                    READY   STATUS              RESTARTS   AGE
pod/nginx-deployment-75c7fcd9ff-7j8g4   0/1     ContainerCreating   0          0s
pod/nginx-deployment-75c7fcd9ff-7tmlp   1/1     Running             0          2s
pod/nginx-deployment-dc589f6f6-6q78w    1/1     Running             0          77s
pod/nginx-deployment-dc589f6f6-bb7xd    1/1     Running             0          73s
pod/nginx-deployment-dc589f6f6-rmsnx    1/1     Terminating         0          75s

NAME                               READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx-deployment   3/3     2            3           4m48s

NAME                                          DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-deployment-75c7fcd9ff   2         2         1       4m48s
replicaset.apps/nginx-deployment-dc589f6f6    2         2         2       78s
NAME                                    READY   STATUS              RESTARTS   AGE
pod/nginx-deployment-75c7fcd9ff-7j8g4   1/1     Running             0          2s
pod/nginx-deployment-75c7fcd9ff-7tmlp   1/1     Running             0          4s
pod/nginx-deployment-75c7fcd9ff-rxzgl   0/1     ContainerCreating   0          0s
pod/nginx-deployment-dc589f6f6-6q78w    1/1     Terminating         0          79s
pod/nginx-deployment-dc589f6f6-bb7xd    1/1     Running             0          75s

NAME                               READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx-deployment   3/3     3            3           4m50s

NAME                                          DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-deployment-75c7fcd9ff   3         3         2       4m50s
replicaset.apps/nginx-deployment-dc589f6f6    1         1         1       80s
NAME                                    READY   STATUS        RESTARTS   AGE
pod/nginx-deployment-75c7fcd9ff-7j8g4   1/1     Running       0          5s
pod/nginx-deployment-75c7fcd9ff-7tmlp   1/1     Running       0          7s
pod/nginx-deployment-75c7fcd9ff-rxzgl   1/1     Running       0          3s
pod/nginx-deployment-dc589f6f6-bb7xd    0/1     Terminating   0          78s

NAME                               READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx-deployment   3/3     3            3           4m53s

NAME                                          DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-deployment-75c7fcd9ff   3         3         3       4m53s
replicaset.apps/nginx-deployment-dc589f6f6    0         0         0       83s
NAME                                    READY   STATUS    RESTARTS   AGE
pod/nginx-deployment-75c7fcd9ff-7j8g4   1/1     Running   0          7s
pod/nginx-deployment-75c7fcd9ff-7tmlp   1/1     Running   0          9s
pod/nginx-deployment-75c7fcd9ff-rxzgl   1/1     Running   0          5s

NAME                               READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx-deployment   3/3     3            3           4m55s

NAME                                          DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-deployment-75c7fcd9ff   3         3         3       4m55s
replicaset.apps/nginx-deployment-dc589f6f6    0         0         0       85s
```

When we perform rollback, kubernetes starts to terminate pods from newer replicaset and to shoot up new pods for the older replicaset. Finally the newer replicaset has no pod, where as the older one has all 3 pods up and running.
Please note that the number of replicas is not changed.

Update rollout history:
```
$ kubectl rollout history deployment/nginx-deployment
deployment.apps/nginx-deployment 
REVISION  CHANGE-CAUSE
2         change to latest image, add one replica
3         initial deployment
```
The current revision is 3.

Performing the rollback again we will create revision 4 which will be a copy of revision 2.

Lets cleanup the deployement:

```
$ krmd nginx-deployment
```