Used to pass credentials and private keys.

distributed only to nodes that are running pods that needs them
never written to physical storage
in master node, they are encrypted
get base64 encoded when reading secret (mainly for storing binary)
```
# hellok8s-secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: hellok8s-secret
data:
  SECRET_MESSAGE: "SXQgd29ya3Mgd2l0aCBhIFNlY3JldAo="
```

instead of using configMapKeyRef, we are using secretKeyRef.
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hellok8s
spec:
  replicas: 2
  selector:
    matchLabels:
      app: hellok8s
  template:
    metadata:
      labels:
        app: hellok8s
    spec:
      containers:
      - image: brianstorti/hellok8s:v4
        name: hellok8s-container
        env:
          - name: MESSAGE
            valueFrom:
              secretKeyRef:
                name: hellok8s-secret
                key: SECRET_MESSAGE
```

can use stringData to pass in raw string instead of using data and base64.
this will auto convert to base64
```
apiVersion: v1
kind: Secret
metadata:
  name: hellok8s-secret
stringData:
  SECRET_MESSAGE: "It works with a Secret"
```

```
kubectl get secret hellok8s-secret -o yaml
# apiVersion: v1
# kind: Secret
# data:
#   SECRET_MESSAGE: SXQgd29ya3Mgd2l0aCBhIFNlY3JldAo=
# ...
```

like config map, we can also mount secrets as files instead of env variable.
but secrets in env variables have unintended consequences. (apps sometimes like to dump all env variables when they crash)
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hellok8s
spec:
  replicas: 2
  selector:
    matchLabels:
      app: hellok8s
  template:
    metadata:
      labels:
        app: hellok8s
    spec:
      volumes:
        - name: secrets
          secret:
            secretName: hellok8s-secret
      containers:
      - image: brianstorti/hellok8s:v4
        name: hellok8s-container
        volumeMounts:
          - name: secrets
            mountPath: /secrets
```
