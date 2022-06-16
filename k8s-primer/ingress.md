- [1. Need for ingress](#1-need-for-ingress)
  - [1.2 SSL Certs](#12-ssl-certs)
- [2. Ingress controllers](#2-ingress-controllers)
  - [2.1 Minikube's option](#21-minikubes-option)
  - [2.2 yaml](#22-yaml)
  - [2.3 Creating second ingress resource](#23-creating-second-ingress-resource)
  - [2.4 Ingress domain on minikube](#24-ingress-domain-on-minikube)
  - [2.5 Non-matching requests](#25-non-matching-requests)

<small><i><a href='http://ecotrust-canada.github.io/markdown-toc/'>Table of contents generated with markdown-toc</a></i></small>


# 1. Need for ingress 
make all services accessible through standard HTTP (80) or HTTPS (443) ports. Kubernetes Services alone cannot get us there. We need more.

grant access to our services on predefined paths and domains. Our go-demo-2 service could be distinguished from others through the base path /demo

## 1.2 SSL Certs
we should be able to make some, if not all, applications (partly) secure by enabling HTTPS access. That means that we should have a place to store our SSL certificates. We could put them inside our applications, but that would only increase the operational complexity. Instead, we should aim towards SSL offloading somewhere between clients and the applications, and it should come as no surprise that Kubernetes has a solution for all these.

# 2. Ingress controllers
Kubernetes itself does not have a ready-to-go solution for this. Unlike other types of Controllers that are typically part of the kube-controller-manager binary, Ingress Controller needs to be installed separately.

## 2.1 Minikube's option
```
minikube addons list

- addon-manager: enabled
- dashboard: enabled
- default-storageclass: enabled
- efk: disabled
- freshpod: disabled
- gvisor: disabled
- heapster: disabled
- ingress: enabled
- logviewer: disabled
- metrics-server: disabled
- nvidia-driver-installer: disabled
- nvidia-gpu-device-plugin: disabled
- registry: disabled
- registry-creds: disabled
- storage-provisioner: enabled

minikube addons enable ingress
```

By default, the Ingress Controller is configured with only two endpoints.

to check the controller's health
```
IP=$(minikube ip)
curl -i "http://$IP/healthz"
```

The Ingress Controller has a default catch-all endpoint that is used when a request does not match any of the other criteria. Since we did not yet create any Ingress Resource, this endpoint should provide the same response to all requests except /healthz.

```
IP=$(minikube ip)
curl -i "http://$IP/something"
```

## 2.2 yaml
The annotations section allows us to provide additional information to the Ingress Controller. As you’ll see soon, Ingress API specification is concise and limited.

The specification API defines only the fields that are mandatory for all Ingress Controllers. All the additional info an Ingress Controller needs is specified through annotations. That way, the community behind the Controllers can progress at great speed, while still providing basic general compatibility and standards.
```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: go-demo-2
  annotations:
    kubernetes.io/ingress.class: "nginx"
    ingress.kubernetes.io/ssl-redirect: "false"
    nginx.ingress.kubernetes.io/ssl-redirect: "false"    # we dont have ssl cert here, so don't force redirect
spec:
  rules:
  - http:
      paths:
      - path: /demo
        backend:
          serviceName: go-demo-2-api
          servicePort: 8080


kubectl create \
    -f ingress/go-demo-2-ingress.yml

kubectl get \
    -f ingress/go-demo-2-ingress.yml

NAME      HOSTS ADDRESS        PORTS AGE
go-demo-2 *     192.168.99.100 80    29s
```

## 2.3 Creating second ingress resource
```
cat ingress/devops-toolkit.yml

apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: devops-toolkit
  annotations:
    kubernetes.io/ingress.class: "nginx"
    ingress.kubernetes.io/ssl-redirect: "false"
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
spec:
  rules:
  - http:
      paths:
      - path: /                         # this will serve all requests,
        backend:
          serviceName: devops-toolkit
          servicePort: 80
```
It will serve all requests. It would be a much better solution if we’d change it to a unique base path (e.g., /devops-toolkit) since that would provide a unique identifier.

However, this application does not have an option to define a base path, so an attempt to do so in Ingress would result in a failure to retrieve resources. We’d need to write rewrite rules instead. We could, for example, create a rule that rewrites path base /devops-toolkit to /.

For now, / as the base path should do.
...

## 2.4 Ingress domain on minikube
on minikube, you don't have the dns entry, so the ingress won't match the domain part

Since it’s not feasible to give you access to the DNS registry of devopstoolkitseries.com. So you cannot configure it with the IP of your Minikube cluster. Therefore, we won’t be able to test it by sending a request to devopstoolkitseries.com.

Fake it by adding domain to header
```
curl -I \
    -H "Host: devopstoolkitseries.com" \
    "http://$IP"
```

## 2.5 Non-matching requests
We can use the default backend as a substitute for the default 404 pages or for any other occasion that is not covered by other rules.

When an Ingress spec is without rules, it is considered a default backend.
```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: default
  annotations:
    kubernetes.io/ingress.class: "nginx"
    ingress.kubernetes.io/ssl-redirect: "false"
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
spec:
  backend:
    serviceName: devops-toolkit
    servicePort: 80
```