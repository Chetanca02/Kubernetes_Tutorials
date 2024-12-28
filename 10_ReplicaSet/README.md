Replica Sets are the kubrenetes workload, a level above pods that ensures a certain number of pods are always running. A Replica Set allows you to define the number of pods that need to be running at all times and this number could be “1”. If a pod crashes, it will be recreated to get back to the desired state. For this reason, replica sets are preferred over a naked pod because they provide some high availability.

Let's create a replicaset using the manifest file. 

```
$ ka 10_ReplicaSet/rs.yaml 
replicaset.apps/frontend created
$ kg all
NAME                 READY   STATUS              RESTARTS   AGE
pod/frontend-cfzrb   0/1     ContainerCreating   0          5s
pod/frontend-k5htg   1/1     Running             0          5s
pod/frontend-q77hx   0/1     ContainerCreating   0          5s

NAME                       DESIRED   CURRENT   READY   AGE
replicaset.apps/frontend   3         3         1       5s
$ kg all
NAME                 READY   STATUS              RESTARTS   AGE
pod/frontend-cfzrb   0/1     ContainerCreating   0          7s
pod/frontend-k5htg   1/1     Running             0          7s
pod/frontend-q77hx   0/1     ContainerCreating   0          7s

NAME                       DESIRED   CURRENT   READY   AGE
replicaset.apps/frontend   3         3         1       7s
$ kg all
NAME                 READY   STATUS    RESTARTS   AGE
pod/frontend-cfzrb   1/1     Running   0          2m48s
pod/frontend-k5htg   1/1     Running   0          2m48s
pod/frontend-q77hx   1/1     Running   0          2m48s

NAME                       DESIRED   CURRENT   READY   AGE
replicaset.apps/frontend   3         3         3       2m48s

$ kg replicaset
NAME       DESIRED   CURRENT   READY   AGE
frontend   3         3         3       43s

$ krm po frontend-q77hx
pod "frontend-q77hx" deleted
$ kg all
NAME                 READY   STATUS    RESTARTS   AGE
pod/frontend-7hgpz   1/1     Running   0          5s
pod/frontend-cfzrb   1/1     Running   0          3m12s
pod/frontend-k5htg   1/1     Running   0          3m12s

NAME                       DESIRED   CURRENT   READY   AGE
replicaset.apps/frontend   3         3         3       3m12s
```
When any pods get killed, the replicaset recreated another one.

Let's cleanup the replicaset:
```
$ krmf 10_ReplicaSet/rs.yaml 
replicaset.apps "frontend" deleted
```

ReplicaSet is a lower-level abstraction that manages the desired number of replicas of a pod with basic scaling and self-healing mechanisms. It doesnot provide other features like rolling updates and rollbacks of the application; these need to be performed manually. This is the reason, replicaset are not directly used in most cases. 

We use a high level abstraction called deployment instead.