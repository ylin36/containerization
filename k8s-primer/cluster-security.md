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
