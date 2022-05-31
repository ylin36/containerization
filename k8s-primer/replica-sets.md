# 1. ReplicaSets Purpose
Ensure the specified number of replicas of a service are always running

# 2. Definiting replica set yaml
```
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: go-demo-2
spec:
  replicas: 2                 
  selector:
    matchLabels:
      type: backend               # rules to apply to which pods that has metadata that match them
      service: go-demo-2
  template:                       # template for the pod
    metadata:
      labels:                     # meta data that will get matched against
        type: backend
        service: go-demo-2
        db: mongo
        language: go
    spec:
      containers:
      - name: db                      #container 1 name
        image: mongo:3.3
      - name: api                     #container 2 name
        image: vfarcic/go-demo-2
        env:
        - name: DB
          value: localhost
        livenessProbe:
          httpGet:
            path: /demo/hello
            port: 8080
```

# 3. Delete replica set
## 3.1 Delete replica set and its associated pods
```
kubectl delete -f rs/go-demo-2.yml
```

## 3.2 Delete replica set and not its associated pods
```
kubectl delete -f rs/go-demo-2.yml --cascade=false
```

## 3.3 Reattach the pods with the same rs

The --save-config is used so that if yamls update later, in 3.4 it will take that as an update.

if we used apply instead of create, apply would have automatically saved the config
```
kubectl create -f rs/go-demo-2.yml --save-config
```
## 3.4 Modifying pod label matches
If label matches change, and replicaset detects total replicas with matching labels didnt match what was specified in the replicaset yaml, it will delete or create to make it match.


