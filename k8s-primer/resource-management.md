# 1. Enable metrics server
```
minikube addons enable metrics-server
```

# 2. Specify limit and requests
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: go-demo-2-db
spec:
  ...
  template:
    ...
    spec:
      containers:
      - name: db
        image: mongo:3.3
        resources:  # specify resource limit here
          limits:
            memory: 200Mi   
            cpu: 0.5                # or can be specified as 500m
          requests:
            memory: 100Mi   # specified as K,M,G,T,P,E or power of 2 equivalent Ki,Mi...
            cpu: 0.3
```

## 2.1 Limit (upper bound)
amount of resources that a container should not pass. 

define limits as upper boundaries which, when reached, indicate that something went wrong, as well as a way to guard our resources from being overtaken by a single rogue container due to memory leaks or similar problems.

* if container is restartable, k8s will restart a container that exceeds its memory limit. 
* if container is NOT restartable, it might terminate it. terminated container will be recreated if it belongs to a Pod
* Unlike memory, CPU limits never result in termination or restarts. a container will not be allowed to consume more than the CPU limit for an extended period.

## 2.2 Requests
Expected resource utilization. used by Kubernetes to *decide where to place Pods depending on actual resource utilization of the nodes that form the cluster.*

If a container exceeds its memory requests, the *Pod* it resides in might be *evicted if a node runs out of memory*. Such eviction usually results in the Pod being scheduled on a different node, as long as there is one with enough available memory.

If a Pod cannot be scheduled to any of the nodes due to lack of available resources, it enters the pending state waiting until resources on one of the nodes are freed, or a new node is added to the cluster.

* memory requests are associated with containers. Pod requested memory = sum of container requested memory. During the scheduling process, Kubernetes sums the requests of a Pod and looks for a node that has enough available memory and CPU. If Pod’s request cannot be satisfied, it is placed in the pending state in the hope that resources will be freed on one of the nodes, or that a new server will be added to the cluster. 
  
# 3. Viewing nodes usage conditions
```
kubectl describe nodes

kubectl top pods
```

# 4. Quality of Service Contracts
Kubernetes will decide to deploy a Pod, whenever it is possible, inside one of the nodes that has enough available memory.

When *memory* requests are defined, Pods will get the memory they requested. If the memory usage of one of the containers exceeds the requested amount, or if some other Pod needs that memory, the Pod hosting it might be killed. Please note that we wrote that a Pod might be killed. Whether that will happen depends on the requests from other Pods and the available memory in the cluster. On the other hand, containers that exceed their memory limits are always killed (unless it was a temporary situation).

*CPU* requests and limits work a bit differently. Containers that exceed specified CPU resources are not killed. Instead, they are throttled.

## 4.1 Guaranteed QoS
* Both memory and CPU limits must be set

* Memory and CPU requests = memory and cpu limits, or requests can be left empt (defaults to limit)

## 4.2 Burstable QoS
* do not meet req of guaranteed QoS
* have at least one container with memory or CPU requests defined.
* guaranteed minimal (requested) memory usage
* more likely to be killed than guaranteed QoS Pods

## 4.3 Best effort QoS
* pods that do not qualify as Guaranteed or Burstable. they consist of containers that have none of the resources defined. 
* use any available memory they need
* lowest priority and first to be evicted if memory is required for others.

# 5. Defining Default Requests and Limits
In case owners of ns in clusters forget to assign limits, it prevents giving more resources than intended
```
apiVersion: v1
kind: LimitRange
metadata:
  name: limit-range
spec:
  limits:
  - default:
      memory: 50Mi
      cpu: 0.2
    defaultRequest:
      memory: 30Mi
      cpu: 0.05
    max:
      memory: 80Mi
      cpu: 0.5
    min:
      memory: 10Mi
      cpu: 0.01
    type: Container
```
The default limit and defaultRequest entries will be applied to the containers that do not specify resources.

When a container does have the resources defined, they will be evaluated against LimitRange thresholds specified as max and min.

# 6. Resource Quota (hard boundry)
define the total amount of compute resources (memory and CPU) that can be spent in a *Namespace*.  

```
apiVersion: v1
kind: Namespace
metadata:
  name: dev

---

apiVersion: v1
kind: ResourceQuota
metadata:
  name: dev
  namespace: dev
spec:
  hard:
    requests.cpu: 0.8
    requests.memory: 500Mi
    limits.cpu: 1
    limits.memory: 1Gi
    pods: 10
    services.nodeports: "0"
```

resource|desc
|-|-|
cpu	|Across all pods in a non-terminal state, the sum of CPU requests cannot exceed this value.
limits.cpu	|Across all pods in a non-terminal state, the sum of CPU limits cannot exceed this value.
limits.memory	|Across all pods in a non-terminal state, the sum of memory limits cannot exceed this value.
memory	|Across all pods in a non-terminal state, the sum of memory requests cannot exceed this value.
requests.cpu	|Across all pods in a non-terminal state, the sum of CPU requests cannot exceed this value.
requests.memory	|Across all pods in a non-terminal state, the sum of memory requests cannot exceed this value.

storage resource|desc
|-|-|
requests.storage	|Across all persistent volume claims, the sum of storage requests cannot exceed this value.
persistentvolumeclaims	|The total number of persistent volume claims that can exist in the namespace.
[PREFIX]/requests.storage	|Across all persistent volume claims associated with the storage-class-name, the sum of storage requests cannot exceed this value.
[PREFIX]/persistentvolumeclaims	|Across all persistent volume claims associated with the storage-class-name, the total number of persistent volume claims that can exist in the namespace.
requests.ephemeral-storage	|Across all pods in the namespace, the sum of local ephemeral storage requests cannot exceed this value.
limits.ephemeral-storage	|Across all pods in the namespace, the sum of local ephemeral storage limits cannot exceed this value.

|object count quota|desc|
-|-
configmaps	|The total number of config maps that can exist in the namespace.
persistentvolumeclaims	|The total number of persistent volume claims that can exist in the namespace.
pods	|The total number of pods in a non-terminal state that can exist in the namespace. A pod is in a terminal state if status.phase in (Failed, Succeeded) is true.
replicationcontrollers	|The total number of replication controllers that can exist in the namespace.
resourcequotas	|The total number of resource quotas that can exist in the namespace.
services	|The total number of services that can exist in the namespace.
services.loadbalancers	|The total number of services of type load balancer that can exist in the namespace.
services.nodeports	|The total number of services of type node port that can exist in the namespace.
secrets	|The total number of secrets that can exist in the namespace.