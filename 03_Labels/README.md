## Labels
Labels are key-value pairs that are used to group together sets of objects, very often pods. This is important for several other concepts, such as replication controller, replica sets, and services that operate on dynamic groups of objects and need to identify the members of the group. There is a NxN relationship between objects and labels. Each object may have multiple labels, and each label may be applied to different objects. There are certain restrictions by design on labels. Each label on an object must have a unique key. The label key must adhere to a strict syntax. It has two parts: prefix and name. The prefix is optional. If it exists then it is separated from the name by a forward slash (/) and it must be a valid DNS sub-domain. The prefix must be 253 characters long at most. The name is mandatory and must be 63 characters long at most. Names must start and end with an alphanumeric character (a-z, A-Z, 0-9) and contain only alphanumeric characters, dots, dashes, and underscores. Values follow the same restrictions as names. Note that labels are dedicated for identifying objects and not for attaching arbitrary metadata to objects.
Labels syntax in object spec is:
labels:
```json
"metadata": {
  "labels": {
    "key1" : "value1",
    "key2" : "value2"
  }
}
```

Labels enable users to map their own organizational structures onto system objects in a loosely coupled fashion, without requiring clients to store these mappings.
Example labels:
```
- "release" : "stable", "release" : "canary"
- "environment" : "dev", "environment" : "qa", "environment" : "production"
- "tier" : "frontend", "tier" : "backend", "tier" : "cache"
- "partition" : "customerA", "partition" : "customerB"
- "track" : "daily", "track" : "weekly"
```

## Label Selectors
Label Selectors is used to filter the set of objects. Separated by commas, multiple requirements will be joined by the logical AND (&&) operator. There are two ways to filter:
- Equality-based requirement
    Equality- or inequality-based requirements allow filtering by label keys and values. Matching objects must satisfy all of the specified label constraints, though they may have additional labels as well. Three kinds of operators are admitted =,==,!=. The first two represent equality (and are synonyms), while the latter represents inequality. For example:
    ```
    - environment = production
    - tier != frontend
    ```
    The former selects all resources with key equal to environment and value equal to production. The latter selects all resources with key equal to tier and value distinct from frontend, and all resources with no labels with the tier key. One could filter for resources in production excluding frontend using the comma operator: environment=production,tier!=frontend

    One usage scenario for equality-based label requirement is for Pods to specify node selection criteria. For example, the sample Pod below selects nodes where the accelerator label exists and is set to nvidia-tesla-p100.
    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
    name: cuda-test
    spec:
    containers:
        - name: cuda-test
        image: "registry.k8s.io/cuda-vector-add:v0.1"
        resources:
            limits:
            nvidia.com/gpu: 1
    nodeSelector:
        accelerator: nvidia-tesla-p100
    ```

- Set-based requirement: in, notin, exists 
    Set-based label requirements allow filtering keys according to a set of values. Three kinds of operators are supported: in,notin and exists (only the key identifier). For example:
    ```
    environment in (production, qa)
    tier notin (frontend, backend)
    partition
    !partition
    ```
    The first example selects all resources with key equal to environment and value equal to production or qa. The second example selects all resources with key equal to tier and values other than frontend and backend, and all resources with no labels with the tier key. The third example selects all resources including a label with key partition; no values are checked. The fourth example selects all resources without a label with key partition; no values are checked. Similarly the comma separator acts as an AND operator. So filtering resources with a partition key (no matter the value) and with environment different than qa can be achieved using partition,environment notin (qa). The set-based label selector is a general form of equality since environment=production is equivalent to environment in (production); similarly for != and notin. Set-based requirements can be mixed with equality-based requirements. For example: partition in (customerA, customerB),environment!=qa.

The set of pods that a service targets is defined with a label selector. Similarly, the population of pods that a replicationcontroller should manage is also defined with a label selector.
Label selectors for both objects are defined in json or yaml files using maps, and only equality-based requirement selectors are supported:
```yaml
selector: {
    component : redis
}
```
Resources, such as Job, Deployment, ReplicaSet, and DaemonSet, support set-based requirements as well.

```yaml
selector:
  matchLabels:
    component: redis
  matchExpressions:
    - { key: tier, operator: In, values: [cache] }
    - { key: environment, operator: NotIn, values: [dev] }
```
