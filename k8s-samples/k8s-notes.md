## Sample k8s commands
Check number of nodes in the cluster
```
kubectl get nodes
```

Apply pod
```
kubectl apply -f pods.yaml
```

Get pod
```
kubectl get pods
```

Create the deployment
```
kubectl apply -f deployment.yaml
```

Get deployments
```
kubectl get deployments
```

Create service
```
kubectl apply -f service.yaml
```

Get service
```
kubectl get service
```

## Sample helm commands
Download helm then install
```
./get_helm.sh
helm --help
```

Use Kind for dev k8s cluster
```
kind create cluster
```

Create a chart (helm package)
```
helm create demo2-chart
```

change the values.yaml, and dryrun the app
```
helm install --dry-run --debug demo2-chart demo2-chart
```

make sure the deployment port is setup correctly in deployment.yaml

Install the helm chart, verify the pod status
```
helm install your-app-name your-chart-name
```
```
helm install demo2 demo2-chart
kubectl get pods
```

Check status of service. Access internal kubernetes app over the internet by mapping service port to 31111
```
kubectl get service
kubectl port-forward svc/your-service-name --address 0.0.0.0 31111:31111
```

## Additional k8s beginner notes
you can use regular portfoward to test a pod/deployment
```
kubectl port-forwrd nginx 3001:80
or
kubectl port-forward --address 0.0.0.0 nginx 3000:80 
```

Streaming logs
```
kubectl logs --follow nginx
               ^        
               |        
                ----------- tells kubectl to keep
                            streaming the logs live,
                            instead of just
                            printing and exiting 
```

Executing commands
```
kubectl exec nginx -- ls
```
Enter interactive session and start bash session
```
kubectl exec -it nginx -- bash
```
Killing pod by name
```
kubectl delete pod nginx
```
Or kill it by sending the same manifest
```
kubectl delete -f nginx.yaml
```

MaxSurge (how many pods we can have exceeding our desired replica count
and MaxUnavailable (how many pods we can have below this count) (absolute number or percent%). K8s will ensure during rollout minimum of x (desired - maxUnavailable) and maximum of y(desired + maxSurge) replicas.
These help rolling release faster by doing more than 1 down 1 up at a time.
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hellok8s
spec:
  strategy:
     rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
  replicas: 3
  selector...
```

Using recreate to take down all pods and bring up new version (cause downtime)
```
spec:
  strategy:
    type: Recreate
  replicas: 3
  selector:
    matchLabels:
```

Rollback release with rollout undo deployment hellok8s
```
kubectl rollout undo deployment hellok8s
```

Use readiness probe to block bad deployments.
have k8s consider this pod to only start receiving reqs after 5 successful
responses from GET to path / on port 4567
```
        app: hellok8s
    spec:
      containers:
      - image: brianstorti/hellok8s:v2 # Still using v2
        name: hellok8s-container
        readinessProbe: # New readiness probe
          periodSeconds: 1
          successThreshold: 5
          httpGet:
            path: /
            port: 4567
```

livenessprobe works similar to readinessprobe, only diff is k8s
will keep calling this probe periodically to make sure the pod is healthy.
restart it in case it is not

```
    spec:
      containers:
      - image: brianstorti/hellok8s:v2
        name: hellok8s-container
        readinessProbe:
          periodSeconds: 1
          successThreshold: 5
          httpGet:
            path: /
            port: 4567
        livenessProbe:
          httpGet:
            path: /
            port: 4567
```