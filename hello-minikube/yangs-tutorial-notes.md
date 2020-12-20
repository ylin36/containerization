## Create minikube cluster
    minikube start
    minikube dashboard

## Create a deployment
    kubectl create deployment hello-node --image=k8s.gcr.io/echoserver:1.4

### View the deployment
    kubectl get deployments

### View the pod
    kubectl get pods

### View cluster events
    kubectl get events

### View kubectl configuration
    kubectl config view

## Create a service
* Pod is only acesible by its internal ip within k8s cluster. Expose hello-note as a k8s service to be viewed publically. 
* LoadBalancer type indicates exposing the service outside the cluster. (the demo app listens on port 8080)
```
kubectl expose deployment hello-node --type=LoadBalancer --port=8080
```

### View services
    kubectl get services

### Make service available through minikube
* cloud providers that support load balancers => an external IP address would be provisioned to access the Service. 
* minikube =>the LoadBalancer type makes the Service accessible through the minikube service command.
```
minikube service hello-node
```

### Enable addons
    minikube addons list
    minikube addons enable metrics-server

### View pod and service just created
    kubectl get pod,svc -n kube-system

### Disable metrics-server
    minikube addons disable metrics-server  

### Clean up
    kubectl get service hello-node
    kubectl delete deployment hello-node

### Or stop minikube
    minikube stop

### Or just delete the minikube vm
    minikube delete