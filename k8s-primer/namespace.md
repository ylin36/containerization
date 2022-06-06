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
all the kubectl commands will be executed within the context of the testing Namespace. That is, until we change the context again, or use the --namespace argument.
```
kubectl config use-context testing
```

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