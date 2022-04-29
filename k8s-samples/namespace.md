
```
apiVersion: v1
kind: Namespace
metadata:
  name: my-namespace
```

```
kubectl apply -f namespace.yaml
# namespace/my-namespace created

$ kubectl get namespaces
# NAME            STATUS
# default         Active
# docker          Active
# ingress-nginx   Active
# kube-public     Active
# kube-system     Active
# my-namespace    Active
```

```
kubectl get pods -n ingress-nginx
# NAME                                        READY   STATUS
# nginx-ingress-controller-57548b96c8-mwkrp   1/1     Running
```

```
kubectl get pods --all-namespaces
kubectl get all --all-namespaces
```

apps go into default namespace
```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx-container
    image: nginx
```

explicitly put in a namespace
```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  namespace: my-namespace
spec:
  containers:
  - name: nginx-container
    image: nginx
```

or do it after the fact
```
kubectl apply -f nginx-pod.yaml -n my-namespace
# pod/nginx created
```

DNS will assume current namespace if namespace-name is ommited.
otherwise it will use <service-name>.<namespace-name>.
```
# pods.yaml
apiVersion: v1
kind: Pod
metadata:
  namespace: default
  name: hellok8s
  labels:
    app: hellok8s
spec:
  containers:
  - name: hellok8s-container
    image: brianstorti/hellok8s:v4

---

apiVersion: v1
kind: Pod
metadata:
  namespace: my-namespace
  name: hellok8s
  labels:
    app: hellok8s
spec:
  containers:
  - name: hellok8s-container
    image: brianstorti/hellok8s:v3
```
```
kubectl apply -f service.yaml -n default
# service/hellok8s-svc created

kubectl apply -f service.yaml -n my-namespace
# service/hellok8s-svc created
```


if we access it from a pod where it isnt the default
```
kubectl exec -it hellok8s -n my-namespace -- sh

# Now inside the container

curl hellok8s-svc:4567
# [v3] Hello, Kubernetes, from hellok8s!

curl hellok8s-svc.default:4567
# [v4] Hello, Kubernetes (from hellok8s)

curl hellok8s-svc.my-namespace:4567
# [v3] Hello, Kubernetes, from hellok8s!
```