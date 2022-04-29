```
kubectl apply -f \
https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v0.44.0/\
deploy/static/provider/cloud/deploy.yaml
```

confirm its running
```
kubectl get pods --all-namespaces \
  -l app.kubernetes.io/name=ingress-nginx

# NAMESPACE       NAME                     READY   STATUS
# ingress-nginx   nginx-ingress-contr...   1/1     Running
```