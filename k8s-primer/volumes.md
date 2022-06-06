# 1. Volume use cases
Kubernetes Volumes solve the need to preserve the state across container crashes. In essence, Volumes are references to files and directories made accessible to containers that form a Pod. The significant difference between different types of Kubernetes Volumes is in the way these files and directories are created.

While the primary use-case for Volumes is the preservation of state, there are quite a few others. For example, we might use Volumes to access Docker’s socket running on a host. Or we might use them to access configuration residing in a file on the host file system.

## 1.1 minikube example
We’ll need the volume/prometheus-conf.yml file inside the soon-to-be-created Minikube VM. When it starts, it will copy all the files from ~/.minikube/files on your host, into the /files directory in the VM.
```
cp volume/prometheus-conf.yml \
    ~/.minikube/files

minikube start
minikube addons enable ingress
kubectl config current-context
```

## 1.2 Creating a Pod with Docker Image#
Instead of executing the docker image build command, we could create a Pod based on the docker image.
Kubernetes will make sure that the Pod is scheduled somewhere inside the cluster, thus distributing resource usage much better.

## 1.2.1 Pod with hostpath
```
apiVersion: v1
kind: Pod
metadata:
  name: docker
spec:
  containers:
  - name: docker
    image: docker:17.11
    command: ["sleep"]
    args: ["100000"]       # let us play around with stuff
    volumeMounts:            
    - mountPath: /var/run/docker.sock         # path inside container     
      name: docker-socket                     # mount volume called docker-socket
  volumes:
  - name: docker-socket
    hostPath:                                # hostPath is ok, We’re not using it to preserve state, 
      path: /var/run/docker.sock            # path to mount from host
      type: Socket                          # we want to mount a socket
```

## 1.2.2 volume types
Other volume types:

The *Directory* type will mount a directory from the host. It must exist on the given path. If it doesn’t, we might switch to *DirectoryOrCreate* type which serves the same purpose. The difference is that DirectoryOrCreate will create the directory if it does not exist on the host.

The *File* and *FileOrCreate* are similar to their Directory equivalents. The only difference is that this time we’d mount a file, instead of a directory.

other supported types are *Socket*, *CharDevice*, and *BlockDevice*. They should be self-explanatory

## 1.2.3 creating
```
kubectl create \
  -f volume/docker.yml

kubectl exec -it docker \
    -- docker image ls \
    --format "{{.Repository}}"

k8s.gcr.io/kube-proxy
k8s.gcr.io/kube-controller-manager
k8s.gcr.io/kube-scheduler
k8s.gcr.io/kube-apiserver
quay.io/kubernetes-ingress-controller/nginx-ingress-controller
k8s.gcr.io/kube-addon-manager
k8s.gcr.io/coredns
k8s.gcr.io/kubernetes-dashboard-amd64
k8s.gcr.io/etcd
k8s.gcr.io/k8s-dns-sidecar-amd64
k8s.gcr.io/k8s-dns-kube-dns-amd64
k8s.gcr.io/k8s-dns-dnsmasq-nanny-amd64
k8s.gcr.io/pause
docker
gcr.io/k8s-minikube/storage-provisioner
gcr.io/google_containers/defaultbackend
```

Even though we executed the docker command inside a container, the output clearly shows the images from the host. We proved that mounting the Docker socket (/var/run/docker.sock) as a Volume allows communication between Docker client inside the container, and Docker server running on the host.

## 1.2.4 playing with docker
```
kubectl exec -it docker sh

// alpine uses apk to install git
apk add -U git                
git clone \
    https://github.com/vfarcic/go-demo-2.git
cd go-demo-2

cat Dockerfile

FROM golang:1.9 AS build
ADD . /src
WORKDIR /src
RUN go get -d -v -t
RUN go test --cover -v ./... --run UnitTest
RUN go build -v -o go-demo


FROM alpine:3.4
MAINTAINER      Viktor Farcic <viktor@farcic.com>

RUN mkdir /lib64 && ln -s /lib/libc.musl-x86_64.so.1 /lib64/ld-linux-x86-64.so.2

EXPOSE 8080
ENV DB db
CMD ["go-demo"]
HEALTHCHECK --interval=10s CMD wget -qO- localhost:8080/demo/hello

COPY --from=build /src/go-demo /usr/local/bin/go-demo
RUN chmod +x /usr/local/bin/go-demo
```

```
docker system prune -f  # f means force, don't prompt
docker image ls \
    --format "{{.Repository}}"

The docker system prune command removes all unused resources. At least, all those created and unused by Docker. We confirmed that by executing docker image ls again. This time, we can see the <none> image is gone.
```


exit the container and delete it...
```
exit

kubectl delete \
    -f volume/docker.yml
```

## 1.3 Prometheus example

```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: prometheus
  annotations:
    kubernetes.io/ingress.class: "nginx"
    ingress.kubernetes.io/ssl-redirect: "false"
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
spec:
  rules:
  - http:
      paths:
      - path: /prometheus
        backend:
          serviceName: prometheus
          servicePort: 9090

---

apiVersion: apps/v1
kind: Deployment
metadata:
  name: prometheus
spec:
  selector:
    matchLabels:
      type: monitor
      service: prometheus
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        type: monitor
        service: prometheus
    spec:
      containers:
      - name: prometheus
        image: prom/prometheus:v2.0.0
        command:
        - /bin/prometheus
        args:
        - "--config.file=/etc/prometheus/prometheus.yml"
        - "--storage.tsdb.path=/prometheus"
        - "--web.console.libraries=/usr/share"
        - "--web.external-url=http://192.168.99.100/prometheus"

---

apiVersion: v1
kind: Service
metadata:
  name: prometheus
spec:
  ports:
  - port: 9090
  selector:
    type: monitor
    service: prometheus
```

rometheus needs a full external-url if we want to change the base path. At the moment, it’s set to the IP of our Minikube VM. In your case, that IP might be different. We’ll fix that by adding a bit of sed “magic” that will make sure the IP matches that of your Minikube VM.

We output the contents of the volume/prometheus.yml file, we used sed to replace the hard-coded IP with the actual value of your Minikube instance, and we passed the result to kubectl create.
```
cat volume/prometheus.yml | sed -e \
    "s/192.168.99.100/$(minikube ip)/g" \
    | kubectl create -f - \
    --record --save-config

kubectl rollout status deploy prometheus
```

### 1.3.1 don't use hostPath for config
A hostPath Volume maps a directory from a host to where the Pod is running. Using it to “inject” configuration files into containers would mean that we’d have to make sure that the file is present on every node of the cluster.

## 1.4 volume type: gitRepo
gitRepo type defines the Git repository and the directory

# 2. Deploying Jenkins (no persistence)
```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: jenkins
  annotations:
    kubernetes.io/ingress.class: "nginx"
    ingress.kubernetes.io/ssl-redirect: "false"
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
spec:
  rules:
  - http:
      paths:
      - path: /jenkins
        backend:
          serviceName: jenkins
          servicePort: 8080

---

apiVersion: apps/v1
kind: Deployment
metadata:
  name: jenkins
spec:
  selector:
    matchLabels:
      type: master
      service: jenkins
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        type: master
        service: jenkins
    spec:
      containers:
      - name: jenkins
        image: vfarcic/jenkins
        env:
        - name: JENKINS_OPTS
          value: --prefix=/jenkins

---

apiVersion: v1
kind: Service
metadata:
  name: jenkins
spec:
  ports:
  - port: 8080
  selector:
    type: master
    service: jenkins
```

```
kubectl create \
    -f volume/jenkins.yml \
    --record --save-config

kubectl rollout status deploy jenkins

open "http://$(minikube ip)/jenkins"
```

## 2.1 Deploying Jenkins with emptyDir
```
...
kind: Deployment
...
spec:
  ...
  template:
    ...
    spec:
      containers:
        ...
        volumeMounts:
        - mountPath: /var/jenkins_home
          name: jenkins-home
      volumes:
      - emptyDir: {}
        name: jenkins-home
...
```

An *emptyDir* Volume is created when a Pod is assigned to a node. It will exist for as long as the Pod continues running on that server.