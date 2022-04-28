* Check number of nodes in the cluster
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

you can use regular portfoward to test a pod/deployment
```
kubectl port-forward --address 0.0.0.0 nginx 3000:80 
```