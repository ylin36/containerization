```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: hello-ingress
  annotations:
    # We are defining this annotation to prevent nginx
    # from redirecting requests to `https` for now
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
spec:
  rules:
    - http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: nginx-svc
                port:
                  number: 1234
```

```
kubectl apply -f ingress.yaml
# ingress.extensions/hello-ingress created

# Please enter the following command if you get an error while applying the ingress 
# and then reapply the ingress using the above command
# If not, you can skip this command and move ahead
kubectl delete -A ValidatingWebhookConfiguration ingress-nginx-admission

kubectl get ingress
# NAME            HOSTS   ADDRESS     PORTS   AGE
# hello-ingress   *       localhost   80      1m
```

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: hello-ingress
  annotations:
    # We are defining this annotation to prevent nginx
    # from redirecting requests to `https` for now
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
spec:
  rules:
    - http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: nginx-svc
                port:
                  number: 1234
          - path: /hello
            pathType: Prefix
            backend:
              service:
                name: hellok8s-svc
                port:
                  number: 4567
```

```
kubectl apply -f ingress-updated.yaml
# ingress.extensions/hello-ingress configured

# Inorder to run the ingress on the platform, you need to run the following port-forward command:
# You don't require this command if you're working locally
nohup kubectl port-forward --address 0.0.0.0 -n ingress-nginx service/ingress-nginx-controller 80:80 > /dev/null 2>&1 &

curl http://localhost/hello
# [v3] Hello, Kubernetes, from hellok8s-7f4c57d446-qth54!

curl http://localhost
# (nginx welcome page)
```

serving services in diff hosts
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: hello-ingress
  annotations:
    # We are defining this annotation to prevent nginx
    # from redirecting requests to `https` for now
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
spec:
  rules:
    - host: nginx.local.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: nginx-svc
                port:
                  number: 1234

    - host: hello.local.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: hellok8s-svc
                port:
                  number: 4567
```

for curl debugging, manipulate header for localhost.
for browser, change host file
```
curl --header 'Host: hello.local.com' localhost
# [v3] Hello, Kubernetes, from hellok8s-7f4c57d446-cjznh!

curl --header 'Host: nginx.local.com' localhost
# nginx welcome page
```