like deployment. create one or more pods. it'll watch these pods to make sure they do what they're intended, and exist successfully. when completed the job is marked as completed and work is done.
```
# echo-job.yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: echo-job
spec:
  template:
    spec:
      restartPolicy: OnFailure
      containers:
      - name: echo
        image: busybox
        command: ["echo", "Running in a job"]
```

```
kubectl apply -f echo-job.yaml
# job.batch/echo-job created

kubectl get pods
# NAME             READY   STATUS
# echo-job-sttd5   0/1     Completed

kubectl logs echo-job-sttd5
# Running in a job

kubectl get jobs
# NAME       DESIRED   SUCCESSFUL
# echo-job   1         1
```

```
cronjob
```
apiVersion: batch/v1
kind: CronJob
metadata:
  name: echo-cronjob
spec:
  schedule: "* * * * *" # Every minute
  jobTemplate:
    spec:
      template:
        spec:
          restartPolicy: OnFailure
          containers:
          - name: echo
            image: busybox
            command: ["echo", "Triggered by a CronJob"]