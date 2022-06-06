# 1. Intro
A Secret is a relatively small amount of sensitive data. Some of the typical candidates for Secrets would be passwords, tokens, and SSH keys.

k8s secrets are like configmaps with very little syntax difference

# 2. Built in secrets
k8s has built in secrets.

default-token-l9fhk Secret was created automatically by Kubernetes. It contains credentials that can be used to access the API.

Kubernetes automatically modifies the Pods to use this Secret. Unless we tweak Service Accounts, every Pod we create will have this Secret. Let’s confirm that is indeed true.
```
kubectl get secrets

NAME                TYPE                                DATA AGE
default-token-l9fhk kubernetes.io/service-account-token 3    32m
```

# 3. Decoding secrets
```
kubectl create secret \
    generic my-creds \
    --from-literal=username=jdoe \
    --from-literal=password=incognito

kubectl get secret my-creds -o json

{
    "apiVersion": "v1",
    "data": {
        "password": "aW5jb2duaXRv",
        "username": "amRvZQ=="
    },
    "kind": "Secret",
    "metadata": {
        ...
    },
    "type": "Opaque"
}
```
```
kubectl get secret my-creds \
    -o jsonpath="{.data.username}" \
    | base64 --decode
```

# 4. Mounting secrets
```
kubectl create secret \
    generic my-creds \
    --from-literal=username=jdoe \
    --from-literal=password=incognito
```
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: jenkins
spec:
  ...
  template:
    ...
    spec:
      containers:
      - name: jenkins
        image: vfarcic/jenkins
        env:
        - name: JENKINS_OPTS
          value: --prefix=/jenkins
        volumeMounts:
        - name: jenkins-home
          mountPath: /var/jenkins_home
        - name: jenkins-creds
          mountPath: /etc/secrets
      volumes:
      - name: jenkins-home
        emptyDir: {}
      - name: jenkins-creds
        secret:
          secretName: my-creds  # secret created in # 3.
          defaultMode: 0444       # read permission for everyone
          items:
          - key: username
            path: jenkins-user # image explicitly requires files jenkins-user. default would have been file username
          - key: password
            path: jenkins-pass
...
```

# 5. Only diff from ConfigMap (Secrets are in Temp file storage)
Secrets creates files in a tmpfs (temporary file storage).

Secrets are constructed as in-memory files, thus leaving no trace on the host’s files system. it's not not enough to call Secrets secure, but it is a step towards it. 

it needs to combine with Authorization Policies to make the passwords, keys, tokens, and other never-to-be-seen-by-publicly types of data secure. 

Even then, we might want to turn our attention towards third-party Secret managers like HashiCorp Vault.

# 6. Insecurities of secrets
Secrets can give us a false sense of security.

Almost everything Kubernetes needs is stored in etcd. That includes Secrets. The problem is that they are stored as plain text. Anyone with access to etcd has access to Kubernetes Secrets. We can limit the access to etcd

Restricting the access to etcd still leaves the Secrets vulnerable to who has access to the file system. That, in a way, diminishes the advantage of storing Secrets in containers in tmpfs. There’s not much benefit of having them in tmpfs used by containers, if those same Secrets are stored on disk by etcd.

When multiple replicas of etcd are running, data is synchronized between them. By default, etcd communication between replicas is not secured. Anyone sniffing that communication could get a hold of our secrets.

## 6.1 How to secure?
* Secure the communication between etcd instances with SSL/TLS.

* Limit the access to etcd and wipe the disk or partitions that were used by it.

* Do not define Secrets in YAML files stored in a repository. Create Secrets through ad-hoc kubectl create secret commands. If possible, delete commands history afterward.

* Make sure that the applications using Secrets do not accidentally output them to logs or transmit them to other applications.

* Create policies that allow only trusted users to retrieve secrets. However, you should be aware that even with proper policies in place, any user with permissions to run a Pod could mount a Secret and read it.