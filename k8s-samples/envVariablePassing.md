directly in deployment
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
            value: "It works!"
```


from config map
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
              configMapKeyRef:
                name: hellok8s-config
                key: MESSAGE
```

can use another variable name if we want
```
env:
  - name: VAR1
    valueFrom:
      configMapKeyRef:
        name: hellok8s-config
        key: VAR1

  - name: VAR2
    valueFrom:
      configMapKeyRef:
        name: hellok8s-config
        key: VAR2

  - name: VAR3
    valueFrom:
      configMapKeyRef:
        name: hellok8s-config
        key: VAR3
# ...
```

we can get all variables from a config map and inject them in at once.
we are forced to use the name defined in config map now instead of being
able to choose a var name with this approach
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
        envFrom:
          - configMapRef:
              name: hellok8s-config
```

to avoid conflicts from multiple config map refs, use a prefix  to all variable names imported from each config map
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hellok8s
spec:
  replicas: 1
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
        envFrom:
        - configMapRef:
            name: hellok8s-config
          prefix: CONFIG_
```

we can also create a volume called config and use config map as a source to inject it. this exposes it as files that are mounted in the container. It's useful when we store things like config files in a configmap instead of simple strings. useful for (nginx.conf)
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
       - name: config
         configMap:
           name: hellok8s-config
      containers:
      - image: brianstorti/hellok8s:v4
        name: hellok8s-container
        volumeMounts:
        - name: config
          mountPath: /config
```

nginx example
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-conf
data:
  nginx.conf: |
    user nginx;
    worker_processes  1;
    events {
      worker_connections  10240;
    }
    http {
      server {
          listen       80;
          server_name  localhost;
          location / {
            root   /usr/share/nginx/html; #Change this line
            index  index.html index.htm;
        }
      }
    }
```