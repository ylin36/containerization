define how much cpu and memory it will use. k8s will only schedule this pod on nodes that have enough capacity to fulfill the req
```
apiVersion: v1
kind: Pod
metadata:
  name: busybox
spec:
  containers:
  - name: busybox-container
    image: busybox
    command: ["dd", "if=/dev/zero", "of=/dev/null"]
    resources:
      requests:
        cpu: 500m
        memory: 5Mi
```

limiting container
```
apiVersion: v1
kind: Pod
metadata:
  name: limited-busybox
spec:
  containers:
  - name: limited-busybox-container
    image: busybox
    command: ["dd", "if=/dev/zero", "of=/dev/null"]
    resources:
      requests:
        cpu: 500m
        memory: 10Mi
      limits:
        cpu: 500m
        memory: 10Mi
```

using limitRange
```
apiVersion: v1
kind: LimitRange
metadata:
  name: memory-limit-range
spec:
  limits:
  - default:
      memory: 500Mi
      cpu: 200m
    defaultRequest:
      memory: 100Mi
      cpu: 100m
    type: Container
```