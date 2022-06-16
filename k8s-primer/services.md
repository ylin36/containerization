- [1. Exposing service imperively [Example exposing NodePort]](#1-exposing-service-imperively--example-exposing-nodeport-)
- [2. Create service declaratively](#2-create-service-declaratively)
- [3. Request forwarding](#3-request-forwarding)
- [4. Container Readiness used with services](#4-container-readiness-used-with-services)
- [5. Service types](#5-service-types)
- [6. Service Discovery](#6-service-discovery)

<small><i><a href='http://ecotrust-canada.github.io/markdown-toc/'>Table of contents generated with markdown-toc</a></i></small>


# 1. Exposing service imperively [Example exposing NodePort]

sample RS
```
apiVersion:  apps/v1
kind: ReplicaSet
metadata:
  name: go-demo-2
spec:
  replicas: 2
  selector:
    matchLabels:
      type: backend
      service: go-demo-2
  template:
    metadata:
      labels:
        type: backend
        service: go-demo-2
        db: mongo
        language: go
    spec:
      containers:
      - name: db
        image: mongo:3.3
        command: ["mongod"]
        args: ["--rest", "--httpinterface"]
        ports:
        - containerPort: 28017
          protocol: TCP
      - name: api
        image: vfarcic/go-demo-2
        env:
        - name: DB
          value: localhost
        livenessProbe:
          httpGet:
            path: /demo/hello
            port: 8080
```

```
kubectl expose rs go-demo-2 --name=go-demo-2-svc --target-port=28017 --type=NodePort
```
As a result, the target port will be exposed on every node of the cluster to the outside world, and it will be routed to one of the Pods controlled by the ReplicaSet.

```
kubectl describe svc go-demo-2-svc
```
```
Name:                    go-demo-2-svc
Namespace:               default
Labels:                  db=mongo
                         language=go
                         service=go-demo-2
                         type=backend
Annotations:             <none>
Selector:                service=go-demo-2,type=backend
Type:                    NodePort
IP:                      10.0.0.194
Port:                    <unset>  28017/TCP
TargetPort:              28017/TCP
NodePort:                 <unset>  31879/TCP
Endpoints:               172.17.0.4:28017,172.17.0.5:28017
Session Affinity:        None
External Traffic Policy: Cluster
Events:                  <none>
```

```
Kubernetes client (kubectl) sent a request to the API server requesting the creation of the Service based on Pods created through the go-demo-2 ReplicaSet.

Endpoint controller is watching the API server for new service events. It detected that there is a new Service object.

Endpoint controller created endpoint objects with the same name as the Service, and it used Service selector to identify endpoints (in this case the IP and the port of go-demo-2 Pods).

kube-proxy is watching for service and endpoint objects. It detected that there is a new Service and a new endpoint object.

kube-proxy added iptables rules which capture traffic to the Service port and redirect it to endpoints. For each endpoint object, it adds iptables rule which selects a Pod.

The kube-dns add-on is watching for Service. It detected that there is a new service.

The kube-dns added db's record to the dns server (skydns).
```

This site can now be access by minikube's node ip and port.
```
PORT=$(kubectl get svc go-demo-2-svc -o jsonpath="{.spec.ports[0].nodePort}") IP=$(minikube ip) echo "http://$IP:$PORT"

http://172.26.49.2:30029/
```
# 2. Create service declaratively

```
apiVersion: v1
kind: Service
metadata:
  name: go-demo-2
spec:
  type: NodePort
  ports:
  - port: 28017
    nodePort: 30001
    protocol: TCP
  selector:
    type: backend
    service: go-demo-2
```
```
kubectl create -f svc/go-demo-2-svc.yml
kubectl get -f svc/go-demo-2-svc.yml

NAME      TYPE     CLUSTER-IP EXTERNAL-IP PORT(S)         AGE
go-demo-2 NodePort 10.0.0.129 <none>      28017:30001/TCP 10m

echo "http://$IP:30001"
```

# 3. Request forwarding
Each Pod has a unique IP that is included in the algorithm used when forwarding requests. Actually, it’s not much of an algorithm. Requests will be sent to those Pods randomly. (round robin if number of replicas don't change. Current solution is based off ip tables)

# 4. Container Readiness used with services
```
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: go-demo-2-api
spec:
  replicas: 3
  selector:
    matchLabels:
      type: api
      service: go-demo-2
  template:
    metadata:
      labels:
        type: api
        service: go-demo-2
        language: go
    spec:
      containers:
      - name: api
        image: vfarcic/go-demo-2
        env:
        - name: DB
          value: go-demo-2-db
        readinessProbe:
          httpGet:
            path: /demo/hello
            port: 8080
          periodSeconds: 1
        livenessProbe:
          httpGet:
            path: /demo/hello
            port: 8080 
```

The *readinessProbe* should be used as an indication that the service is ready to serve requests. When combined with Services construct, only containers with the readinessProbe state set to Success will receive requests.

While *livenessProbe* is used to determine whether a Pod is alive or it should be replaced by a new one, the readinessProbe is used by the iptables.

# 5. Service types
Services are built on top of each other. Each subsequent type would create one of type before
ClusterIp (Default, not exposed to a node's port for external)

NodePort (ClusterIp is created, also expose a mapping to a node's port)

LoadBalancer

# 6. Service Discovery
two principal modes:

1) Environment variables

2) DNS
```
Kubernetes converts Service names into DNSes and adds them to the DNS server. It is a cluster add-on that is already set up by Minikube.
```

```
When the api container go-demo-2 tries to connect with the go-demo-2-db Service, it looks at the nameserver configured in /etc/resolv.conf. kubelet configured the nameserver with the kube-dns Service IP (10.96.0.10) during the Pod scheduling process.

The container queries the DNS server listening to port 53. go-demo-2-db DNS gets resolved to the service IP 10.0.0.19. This DNS record was added by kube-dns during the service creation process.

The container uses the service IP which forwards requests through the iptables rules. They were added by kube-proxy during Service and Endpoint creation process.

Since we only have one replica of the go-demo-2-db Pod, iptables forwards requests to just one endpoint. If we had multiple replicas, iptables would act as a load balancer and forward requests randomly among Endpoints of the Service.
```