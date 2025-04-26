A Job is a workload resource used to manage the execution of one or more pods that run until a specific task is completed. Jobs are typically used for tasks that are finite or short-lived in nature, such as batch processing, data analysis, or backups.
Unlike other Kubernetes controllers (e.g., Deployments or StatefulSets), a Job ensures that a certain number of pods complete successfully. Once the desired number of successful completions is reached, the Job terminates.

##### Key Features of Jobs
- Finite Workload: Jobs are designed to perform a task that eventually completes.
- Pod Management: Kubernetes manages the pods, ensuring that failed ones are restarted until the task is completed.
- Parallelism: Jobs can run tasks sequentially or in parallel using multiple pods.

##### Key Job Fields
spec.completions: Number of successful completions required.
spec.parallelism: Number of pods to run concurrently.
spec.template: Pod template defining the container and its configuration.

##### Types of Job Patterns
- Non-Parallel Jobs (Single Pod)
  - Runs a single pod until completion unless pod fails.
  - Example: Backing up a database.
  - spec.completions & spec.parallelism are not required; default to 1.
- Parallel Jobs with a Fixed Completion Count
  - Runs multiple pods in parallel, completing when a specified number of pods finish successfully.
  - Example: Processing a batch of tasks.
  - spec.completions is set to desired number of completion needed
  - spec.parallelism defaults to 1 (can be left unset). Can be increaed to attain parallelism.
- Parallel Jobs with Work Queue
  - Creates pods that consume tasks from a queue. The queue ensures each task is executed only once.
  - When any Pod from the Job terminates with success, no new Pods are created.
  - Once at least one Pod has terminated with success and all Pods are terminated, then the Job is completed with success.
  - Example: Distributed data processing.
  - spec.completions is left unset.
  - spec.parallelism defaults to 1 (can be left unset). Can be increaed to attain parallelism.

##### Use Cases of Jobs
- Database Backups: Run a job to back up a database at scheduled intervals.
- Data Processing: Process large datasets using parallel pods.
- Batch Processing: Execute tasks like sending emails, generating reports, or running scripts.
- Testing: Perform integration or load testing as part of CI/CD pipelines.
- Maintenance Tasks: Cleanup temporary files or compress logs periodically.


Lets create each of these pods:

Non-Parallel Jobs
```
$ ka simple_job.yaml 
job.batch/simple-job created
$ kgp -w
NAME               READY   STATUS              RESTARTS   AGE
simple-job-r2pk2   0/1     ContainerCreating   0          3s
simple-job-r2pk2   1/1     Running             0          5s
simple-job-r2pk2   0/1     Completed           0          55s
simple-job-r2pk2   0/1     Completed           0          57s

$ job_name='simple-job'
$ pods_name=$(kgp --selector=job-name=$job_name -o jsonpath='{.items[*].metadata.name}')
$ for pod_name in $pods_name; do echo FROM POD $pod_name; k logs $pod_name; done
FROM POD simple-job-r2pk2
Hello World!
```

This job runs a single pod that prints "Hello World!", waits for 50 seconds and then terminates.

Parallel Jobs with a Fixed Completion Count
```
$ ka parallel_completion_job.yaml 
job.batch/parallel-completion-job created
$ kgp -w
NAME                            READY   STATUS              RESTARTS   AGE
parallel-completion-job-nx5ss   0/1     ContainerCreating   0          2s
parallel-completion-job-sdmft   0/1     ContainerCreating   0          2s
parallel-completion-job-z6js9   0/1     ContainerCreating   0          3s
parallel-completion-job-fx25j   0/1     ContainerCreating   0          3s
parallel-completion-job-w6d54   0/1     ContainerCreating   0          3s
parallel-completion-job-sdmft   1/1     Running             0          6s
parallel-completion-job-z6js9   1/1     Running             0          7s
parallel-completion-job-nx5ss   1/1     Running             0          8s
parallel-completion-job-fx25j   1/1     Running             0          9s
parallel-completion-job-w6d54   1/1     Running             0          12s
parallel-completion-job-sdmft   0/1     Completed           0          57s
parallel-completion-job-nx5ss   0/1     Completed           0          60s
parallel-completion-job-z6js9   0/1     Completed           0          60s
parallel-completion-job-fx25j   0/1     Completed           0          61s
parallel-completion-job-w6d54   0/1     Completed           0          65s

$ job_name='parallel-completion-job'

$ pods_name=$(kgp --selector=job-name=$job_name -o jsonpath='{.items[*].metadata.name}')

$ for pod_name in $pods_name; do echo FROM POD $pod_name; k logs $pod_name; do
ne
FROM POD parallel-completion-job-fx25j
Hello from pod
FROM POD parallel-completion-job-nx5ss
Hello from pod
FROM POD parallel-completion-job-sdmft
Hello from pod
FROM POD parallel-completion-job-w6d54
Hello from pod
FROM POD parallel-completion-job-z6js9
Hello from pod
```
This job runs 5 pods concurrently, and the job is considered complete after 5 successful completions. We can play with parallelism and completion counts.

Cleanup:
krm job simple-job
krm job parallel-completion-job

