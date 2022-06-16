- [1. Dirty way to spin up cluster](#1-dirty-way-to-spin-up-cluster)
  - [1.1 Spin up pod like it's docker](#11-spin-up-pod-like-its-docker)
  - [1.2 Delete the pod](#12-delete-the-pod)
- [2. Pods](#2-pods)
  - [2.1 Get more details about the pod](#21-get-more-details-about-the-pod)
  - [2.2 (informational) sequence of events after applying the pod yaml](#22-informational-sequence-of-events-after-applying-the-pod-yaml)
  - [2.3 Executing process inside a Pod's container](#23-executing-process-inside-a-pods-container)
  - [2.4 Get logs of a pod](#24-get-logs-of-a-pod)
  - [2.5 Delete the pod](#25-delete-the-pod)
  - [2.6 Communication within pod](#26-communication-within-pod)
  - [2.7 Health Monitor](#27-health-monitor)

<small><i><a href='http://ecotrust-canada.github.io/markdown-toc/'>Table of contents generated with markdown-toc</a></i></small>


# 1. Dirty way to spin up cluster
## 1.1 Spin up pod like it's docker 
```
PS E:\git\k8s-notes> kubectl run db --image mongo
pod/db created
PS E:\git\k8s-notes> kubectl get pods
NAME   READY   STATUS    RESTARTS   AGE
db     1/1     Running   0          31s
```

## 1.2 Delete the pod
kubectl delete pod db

# 2. Pods
Once a *container* is ran, k8s ensures it is always running by restarting it if it crashes.

Not true if a *pod* crashed though.
```
apiVersion: v1
kind: Pod
metadata:
  name: db            # name of the pod
  labels:
    type: db          # identifier of the pod
    vendor: MongoLabs
spec:
  containers:
  - name: db          # name of the container
    image: mongo:3.3
    command: ["mongod"]
    args: ["--rest", "--httpinterface"]
```

Create it
```
kubectl create -f pod/db.yml
```
## 2.1 Get more details about the pod
```
kubectl get pods -o wide
kubectl get pods -o json
kubectl get pods -o yaml
kubectl describe pod db
kubectl describe -f pod/db.yml

// retrieve just the part of the json
kubectl get -f pod/go-demo-2.yml \
    -o jsonpath="{.spec.containers[*].name}"
```

## 2.2 (informational) sequence of events after applying the pod yaml
```
Kubernetes client (kubectl) sent a request to the API server requesting creation of a Pod defined in the pod/db.yml file.

Since the scheduler is watching the API server for new events, it detected that there is an unassigned Pod.

The scheduler decided which node to assign the Pod to and sent that information to the API server.

Kubelet is also watching the API server. It detected that the Pod was assigned to the node it is running on.

Kubelet sent a request to Docker requesting the creation of the containers that form the Pod. In our case, the Pod defines a single container based on the mongo image.

Finally, Kubelet sent a request to the API server notifying it that the Pod was created successfully.
```

## 2.3 Executing process inside a Pod's container
This executes a process inside the first container of the pod db. can use --container or -c to specify exactly which container to run on
```
// execute in first container of pod db
kubectl exec db ps aux

// go into the first container of pod db
kubectl exec -it db sh

// exec in a targeted container of pod db
kubectl exec -it -c db go-demo-2 ps aux
```

## 2.4 Get logs of a pod
output log of the only container in pod
```
kubectl logs db
```
output log of specific container
```
kubectl logs go-demo-2 -c db
```

## 2.5 Delete the pod
```
kubectl delete -f pod/db.yml
```

## 2.6 Communication within pod
All the processes (containers) inside a Pod share the same set of resources, and they can communicate with each other through localhost. One of those shared resources is storage. A volume (think of it as a directory with shareable data) defined in a Pod can be accessed by all the containers thus allowing them all to share the same data.

## 2.7 Health Monitor
*livenessProbe* can be used to confirm whether a container should be running. If the probe fails, Kubernetes will kill the container and apply restart policy which defaults to Always.

 *readinessProbe* is directly tied to Services.
 ```
 apiVersion: v1
kind: Pod
metadata:
  name: go-demo-2
  labels:
    type: stack
spec:
  containers:
  - name: db
    image: mongo:3.3
  - name: api
    image: vfarcic/go-demo-2
    env:
    - name: DB
      value: localhost
    livenessProbe:
      httpGet:
        path: /this/path/does/not/exist
        port: 8080
      initialDelaySeconds: 5
      timeoutSeconds: 2 # Defaults to 1
      periodSeconds: 5 # Defaults to 10
      failureThreshold: 1 # Defaults to 3
 ```