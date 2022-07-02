- [1. Need to Authentication and Authorization](#1-need-to-authentication-and-authorization)
  - [1.2 API](#12-api)
  - [1.3 Port](#13-port)
  - [1.4 API process](#14-api-process)
    - [1.4.1 Authentication](#141-authentication)
    - [1.4.2 Authorization](#142-authorization)
    - [1.4.3 Passing the admin control](#143-passing-the-admin-control)
      - [1.4.3.1 Why admin control](#1431-why-admin-control)
- [2. Authorization methods](#2-authorization-methods)
  - [2.1 Demo](#21-demo)
    - [2.1.1 start cluster and create some demo resources](#211-start-cluster-and-create-some-demo-resources)
    - [2.1.2 use openssl to generate a rsa private key](#212-use-openssl-to-generate-a-rsa-private-key)
    - [2.1.3 use private key to generate a csr](#213-use-private-key-to-generate-a-csr)
    - [2.1.4 use minikube cluster ca to sign the csr.](#214-use-minikube-cluster-ca-to-sign-the-csr)
    - [2.1.5 Authenticating as jdoe](#215-authenticating-as-jdoe)
      - [2.1.5.1 create new cluster connection against server called jdoe](#2151-create-new-cluster-connection-against-server-called-jdoe)
      - [2.1.5.2 set credentials](#2152-set-credentials)
      - [2.1.5.3 set context](#2153-set-context)
      - [2.1.5.4 authorization](#2154-authorization)
- [3. RBAC Authority](#3-rbac-authority)
  - [3.1 Rules](#31-rules)
  - [3.2 Roles](#32-roles)
  - [3.3 Subject](#33-subject)
  - [3.4 RoleBinding](#34-rolebinding)
- [4. Check if someone else' user account can do things](#4-check-if-someone-else-user-account-can-do-things)

<small><i><a href='http://ecotrust-canada.github.io/markdown-toc/'>Table of contents generated with markdown-toc</a></i></small>


# 1. Need to Authentication and Authorization
## 1.2 API
Every interaction with Kubernetes goes through its API and needs to be authorized. 

communication can be initiated through a user or a service account. 

All Kubernetes objects currently running inside our cluster are interacting with the API through service accounts.

Kubectl talks to the API

## 1.3 Port
```
kubectl config view -o jsonpath='{.clusters[?(@.name=="minikube")].cluster.server}'

https://172.26.60.223:8443
```

minikube cluster creates a cert, and it's currently the only way to access the cluster. you need ca cert to securely access minikube cluster
```
kubectl config view o jsonpath='{.clusters[?(@.name=="minikube")].cluster.certificate-authority}'

C:\Users\Yang\.minikube\ca.crt
```

## 1.4 API process
Requests to api goes through 3 stages

* Authentication
* Authorization
* Passing the admin control

### 1.4.1 Authentication
k8s uses client certificates, bearer tokens, an authenticating proxy, or HTTP basic auth to authenticate API requests through authentication plugins. 

the username is retrieved from the HTTP request. If the request cannot be authenticated, the operation is aborted with the status code 401.

### 1.4.2 Authorization
Once authenticated, authorization validates whether it is allowed to execute the specified action. 

The authorization can be performed through *ABAC*, *RBAC*, or *Webhook* modes.

### 1.4.3 Passing the admin control
https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/

once a request is authorized, it passes through admission controllers. They intercept requests to the API before the objects are persisted and can modify them.

The controllers consist of the list below, are compiled into the kube-apiserver binary, and may only be configured by the cluster administrator. In that list, there are two special controllers: MutatingAdmissionWebhook and ValidatingAdmissionWebhook. These execute the mutating and validating (respectively) admission control webhooks which are configured in the API.

Admission controllers may be "validating", "mutating", or both. Mutating controllers may modify related objects to the requests they admit; validating controllers may not.

Admission controllers limit requests to create, delete, modify objects or connect to proxy. They do not limit requests to read objects.

The admission control process proceeds in two phases. In the first phase, mutating admission controllers are run. In the second phase, validating admission controllers are run. Note again that some of the controllers are both.

If any of the controllers in either phase reject the request, the entire request is rejected immediately and an error is returned to the end-user.

Finally, in addition to sometimes mutating the object in question, admission controllers may sometimes have side effects, that is, mutate related resources as part of request processing. Incrementing quota usage is the canonical example of why this is necessary. Any such side-effect needs a corresponding reclamation or reconciliation process, as a given admission controller does not know for sure that a given request will pass all of the other admission controllers.

#### 1.4.3.1 Why admin control
Many advanced features in Kubernetes require an admission controller to be enabled in order to properly support the feature. As a result, a Kubernetes API server that is not properly configured with the right set of admission controllers is an incomplete server and will not support all the features you expect.

# 2. Authorization methods
Node: Node authorization grants permissions to kubelets based on the Pods they are scheduled to run.

ABAC: Attribute-based access control (ABAC) is based on attributes combined with policies and is considered deprecated in favor of RBAC.

Webhooks: Webhooks are used for event notifications through HTTP POST requests.

RBAC (the go to): Role-based access control (RBAC) grants (or denies) access to resources based on roles of individual users or groups.

## 2.1 Demo
### 2.1.1 start cluster and create some demo resources
```
minikube start

kubectl create \
    -f auth/go-demo-2.yml \
    --record --save-config
```
### 2.1.2 use openssl to generate a rsa private key
```
openssl version

mkdir keys
openssl genrsa -out keys/jdoe.key 2048
```
### 2.1.3 use private key to generate a csr
windows bash use this
```
openssl req -new -key keys/jdoe.key -out keys/jdoe.csr -subj "//CN=jdoe\O=devs"
```
linux use
```
openssl req -new -key keys/jdoe.key -out keys/jdoe.csr -subj "/CN=jdoe/O=devs"
``` 
### 2.1.4 use minikube cluster ca to sign the csr.

confirm the cluster ca is all there.
```
ls -1 ~/.minikube/ca.*

----this should show----
/c/Users/Yang/.minikube/ca.crt
/c/Users/Yang/.minikube/ca.key
/c/Users/Yang/.minikube/ca.pem
```
sign the csr. for this demo, make the cert valid for a year
```
openssl x509 -req \
    -in keys/jdoe.csr \
    -CA ~/.minikube/ca.crt \
    -CAkey ~/.minikube/ca.key \
    -CAcreateserial \
    -out keys/jdoe.crt \
    -days 365
```
to simply things, copy the cluster ca to the keys folder, too
```
cp ~/.minikube/ca.crt keys/ca.crt
```
get address of the server
```
SERVER=$(kubectl config view \
    -o jsonpath='{.clusters[?(@.name=="minikube")].cluster.server}')

echo $SERVER
```

### 2.1.5 Authenticating as jdoe
This is kind of like opposite of https.

you give the crt to k8s cluster. they check that it was signed by the CA, and validate it. Take the public key and encrypt messages.

private key at jdoe will decrypt it.

....


copy keys folder to the same partition as the .kube folder before running below

#### 2.1.5.1 create new cluster connection against server called jdoe
I think using the ca cert here makes it so you are trusting the server for ssl calls.
```
kubectl config set-cluster jdoe     --certificate-authority     c:/Users/Yang/keys/ca.crt     --server $SERVER
```
#### 2.1.5.2 set credentials
```
kubectl config set-credentials jdoe --client-certificate c:/Users/Yang/keys/jdoe.crt --client-key c:/Users/Yang/keys/jdoe.key
```

#### 2.1.5.3 set context
set context to use cluster jdoe and authenticate with user jdoe
```
kubectl config set-context jdoe \
    --cluster jdoe \
    --user jdoe
```
use context
```
kubectl config use-context jdoe
```

#### 2.1.5.4 authorization
without authorization, it will fail
```
kubectl get pods

Error from server (Forbidden): pods is forbidden: User "jdoe" cannot list resource "pods" in API group "" in the namespace "default"
```
# 3. RBAC Authority

## 3.1 Rules
A Rule is a set of operations (verbs), resources, and API groups. 

permissions defined are additive. some rules cannot be excluded

Verbs describe activities that can be performed on resources which belong to different API Groups.
rule|desc
|-|-|
get	|Retrieves information about a specific object
list	|Retrieves information about a collection of objects
create	|Creates a specific object
update	|Updates a specific object
patch	|Patches a specific object
watch|Watches for changes to an object
proxy	|Proxies requests
redirect	|Redirects requests
delete	|Deletes a specific object
deletecollection	|Deletes a collection of objects

## 3.2 Roles
There are by default some pre-defined cluster roles

A Role is a collection of Rules. It defines one or more Rules that can be bound to a user or a group of users.

## 3.3 Subject
Subjects define entities that are executing operations. A Subject can be a User, a Group, or a Service Account.

A User is a person or a process residing outside a cluster. A Service Account is used for processes running inside Pods that want to use the API. Since this chapter focuses on human authentication, we won’t explore them right now. Finally, Groups are collections of Users or Service Accounts. Some Groups are created by default (e.g., cluster-admin).

## 3.4 RoleBinding
bind Subjects to Roles.

# 4. Check if someone else' user account can do things
```
kubectl auth can-i get pods --as otheruser
```