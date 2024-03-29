- [1. Namespaces intro](#1-namespaces-intro)
  - [1.1 Kubectl get all](#11-kubectl-get-all)
  - [1.2 Virtual clusters](#12-virtual-clusters)
  - [1.3 Default namespace](#13-default-namespace)
- [1.4 Get existing namespace](#14-get-existing-namespace)
- [2. Creating namespace](#2-creating-namespace)
  - [2.1 Specify namespace](#21-specify-namespace)
  - [2.2 creating namespace](#22-creating-namespace)
- [3. Change context to new namespace](#3-change-context-to-new-namespace)
- [4. Communication between namespace](#4-communication-between-namespace)
- [5. Cascade delete Namespace (everything in it)](#5-cascade-delete-namespace-everything-in-it)
- [6. Creating resource in namespace](#6-creating-resource-in-namespace)
  - [6.1 Creating in current active namespace](#61-creating-in-current-active-namespace)
  - [6.2 Creating in specified namespace](#62-creating-in-specified-namespace)
  - [6.3 Declaritively definiting a fixed namespace in yaml](#63-declaritively-definiting-a-fixed-namespace-in-yaml)

<small><i><a href='http://ecotrust-canada.github.io/markdown-toc/'>Table of contents generated with markdown-toc</a></i></small>


# 1. Namespaces intro
Segments the cluster. Namespaces are virtual clusters.

## 1.1 Kubectl get all 
This only shows objects we created.

The objects created inside the kube-public Namespace are visible Throughout the cluster

```
kubectl get all
```

## 1.2 Virtual clusters
Kubernetes uses Namespaces to create virtual clusters. When we created the Minikube cluster, we got three Namespaces. In a way, each Namespace is a cluster within the cluster. They provide scope for names.

## 1.3 Default namespace
Commands will be sent to a default namespace unless otherwise specified

Namespaces are so much more than scopes for object names.

* allow us to split a cluster among different groups of users.

* each namespaces can have different permissions and resources quotas.

* if we combine Namespaces with other Kubernetes services and concepts, there are alot of possibilities

# 1.4 Get existing namespace
Namespaces that were setup automatically when minikube start
```
kubectl get ns

NAME              STATUS   AGE
default           Active   105m
kube-node-lease   Active   105m
kube-public       Active   105m
kube-system       Active   105m
```

# 2. Creating namespace
## 2.1 Specify namespace
```
kubectl --namespace kube-public get all
```

## 2.2 creating namespace
```
kubectl create ns testing
kubectl get ns

NAME              STATUS   AGE
default           Active   116m
kube-node-lease   Active   116m
kube-public       Active   116m
kube-system       Active   116m
testing           Active   4m30s
```

# 3. Change context to new namespace
```
kubectl config set-context testing \
    --namespace testing \
    --cluster minikube \
    --user minikube
```
See the new config context that's created
```
kubectl config view
```
```
apiVersion: v1
clusters:
- cluster:
    certificate-authority: C:\Users\Yang\.minikube\ca.crt
    server: https://172.26.58.131:8443
  name: minikube
contexts:
- context:
    cluster: minikube
    namespace: default
    user: minikube
  name: minikube
- context:
    cluster: minikube
    namespace: testing            # here
    user: minikube
  name: testing
current-context: minikube
kind: Config
preferences: {}
users:
- name: minikube
  user:
    client-certificate: C:\Users\Yang\.minikube\profiles\minikube\client.crt
    client-key: C:\Users\Yang\.minikube\profiles\minikube\client.key
```

Switch to newly created namespace
```
kubectl config use-context testing
```
all the kubectl commands will be executed within the context of the testing Namespace. That is, until we change the context again, or use the --namespace argument.


# 4. Communication between namespace

The primary objective behind the existence of the DNSes with the Namespace name is when we want to reach services running in a different Namespace.

Kube DNS used the DNS suffix testing to deduce that we want to reach the Service located in that Namespace.

* When we create a Service, it creates a few DNS entries. One of them corresponds to the name of the Service.

* So, the go-demo-2-api Service created a DNS based on that name.

* the full DNS entry is go-demo-2-api.svc.cluster.local. Both resolve to the same service go-demo-2-api which, in this case, runs in the default Namespace.

* The third DNS entry we got is in the format <service-name>.<namespace-name>.svc.cluster.local. In our case, that is go-demo-2-api.default.svc.cluster.local. Or, if we prefer a shorter version, we could use go-demo-2-api.default.

* In most cases, there is no good reason to use the <service-name>.<namespace-name> format when communicating with Services within the same Namespace.

# 5. Cascade delete Namespace (everything in it)
```
kubectl delete ns testing

kubectl -n testing get all

No resources found in testing namespace.
```

# 6. Creating resource in namespace

## 6.1 Creating in current active namespace
Apply yaml to current active namespace. if non is explicitly made active, it runs in default ns.

```
apiVersion: v1
kind: Pod
metadata:
  name: mypod
  labels:
    name: mypod
spec:
  containers:
  - name: mypod
    image: nginx
```
```
kubectl apply -f pod.yaml
```

## 6.2 Creating in specified namespace

specify the namespace

```
kubectl apply -f pod.yaml --namespace=test
```

## 6.3 Declaritively definiting a fixed namespace in yaml

specify a namespace in the YAML declaration, the resource will always be created in that namespace. If you try to use the “namespace” flag to set another namespace, the command will fail.
```
apiVersion: v1
kind: Pod
metadata:
  name: mypod
  namespace: test
  labels:
    name: mypod
spec:
  containers:
  - name: mypod
    image: nginx
```