# 1. Configuration ways
## 1.1 Environment variables vs Files (Yaml)
fit well into distributed systems; easy to define, portable; ideal choice for configuration mechanism of new applications.

in some cases, the configuration might be too complex for environment variables. In such situations, we might need to fall back to files (hopefully YAML).

## 1.2 ConfigMap
ConfigMap allows us to “inject” configuration into containers. The source of the configs can be files, directories, or literal values. The destination can be files or *environment variables*.

```
global:
  scrape_interval:     15s

scrape_configs:
  - job_name: prometheus
    metrics_path: /prometheus/metrics
    static_configs:
      - targets:
        - localhost:9090


kubectl create cm my-config \
    --from-file=cm/prometheus-conf.yml
```

### 1.2.1 Mounting config map AS A FILE
ConfigMap is useless by itself. It is yet another Volume which, like all the others, needs a mount.

```
apiVersion: v1
kind: Pod
metadata:
  name: alpine
spec:
  containers:
  - name: alpine
    image: alpine
    command: ["sleep"]
    args: ["100000"]
    volumeMounts:
    - name: config-vol
      mountPath: /etc/config
  volumes:
  - name: config-vol          # name of the cm from earlier
    configMap:
      name: my-config
```

### 1.2.2 Creating ConfigMap from multiple files
the sample file below for promethus-conf.yml
```
global:
  scrape_interval:     15s

scrape_configs:
  - job_name: prometheus
    metrics_path: /prometheus/metrics
    static_configs:
      - targets:
        - localhost:9090

```
```
kubectl create cm my-config \
    --from-file=cm/prometheus-conf.yml \
    --from-file=cm/prometheus.yml

kubectl create -f cm/alpine.yml

#Run the following command separately
kubectl exec -it alpine -- \
    ls /etc/config

// output is
something  weather
```

### 1.2.2 Injecting config from key-value literals
The --from-literal argument is useful when we’re in need to set a relatively small set of configuration entries in different clusters. It makes more sense to specify only the things that change, than all the configuration options.


```
kubectl create cm my-config --from-literal=something=else --from-literal=weather=sunny

kubectl get cm my-config -o yaml

apiVersion: v1
data:
  something: else                    # these got created from literal
  weather: sunny
kind: ConfigMap
metadata:
  creationTimestamp: "2022-06-05T20:49:57Z"
  managedFields:
  - apiVersion: v1
    fieldsType: FieldsV1
    fieldsV1:
      f:data:
        .: {}
        f:something: {}
        f:weather: {}
    manager: kubectl-create
    operation: Update
    time: "2022-06-05T20:49:57Z"
  name: my-config
  namespace: default
  resourceVersion: "13180"
  uid: 01b99b9b-9d11-4251-b95e-c3b3de2b196e
```

### 1.2.3 Injecting conf from environment files
Contents of cm/my-env-file.yml (just a key-value mapping. this isn't a configmap yaml)
```
something=else
weather=sunny
```
```
kubectl create cm my-config \
    --from-env-file=cm/my-env-file.yml
```

# 3. Using ConfigMap as Environment variable for pods
# 3.1 configMapKefRef Way
With env.valueFrom.configMapKeyRef syntax, we need to specify each ConfigMap key separately. That gives us control over the scope and the relation with the names of container variables.

```
cat cm/my-env-file.yml

something=else
weather=sunny

kubectl create cm my-config \
    --from-env-file=cm/my-env-file.yml

kubectl get cm my-config -o yaml

apiVersion: v1
data:
  something: else
  weather: sunny
kind: ConfigMap
```
```
apiVersion: v1
kind: Pod
metadata:
  name: alpine-env
spec:
  containers:
  - name: alpine
    image: alpine
    command: ["sleep"]
    args: ["100000"]
    env:
    - name: something
      valueFrom:
        configMapKeyRef:
          name: my-config
          key: something
    - name: weather
      valueFrom:
        configMapKeyRef:
          name: my-config
          key: weather
...
```

# 3.2 configMapRef Way
The envFrom.configMapRef converts all ConfigMap’s data into environment variables. That is often a better and simpler option if you don’t need to use different names between ConfigMap and environment variable keys. The syntax is short, and we don’t need to worry whether we forgot to include one of the ConfigMap’s keys.
```
apiVersion: v1
kind: Pod
metadata:
  name: alpine-env
spec:
  containers:
  - name: alpine
    image: alpine
    command: ["sleep"]
    args: ["100000"]
    envFrom:
    - configMapRef:
        name: my-config
```

# 4. Defining configmap as yaml and using it
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: prometheus
spec:
  ...
  template:
    ...
    spec:
      containers:
        ...
        volumeMounts:
        - mountPath: /etc/prometheus
          name: prom-conf
      volumes:
      - name: prom-conf
        configMap:
          name: prom-conf
...
apiVersion: v1
kind: ConfigMap
metadata:
  name: prom-conf
data:
  prometheus.yml: |  # this config map has 1 key. prometheus. once mounted, filename = name of key (prometheus.yml)
    global:
      scrape_interval:     15s

    scrape_configs:
      - job_name: prometheus
        metrics_path: /prometheus/metrics
        static_configs:
          - targets:
            - localhost:9090
```

When working with a large value, we can start with the pipe sign *|*. Kubernetes will interpret the value as “everything that follows, as long as it is indented.