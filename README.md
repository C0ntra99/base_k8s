# Minikube argo setup
Install Argo
```
helm install argo-cd chargs/argo-cd/ --set namespaceOverride=argocd
```

Apply ingress controller
```
kubectl apply -f argo_ingress.yaml
```
```
echo {ingress loadbalncer IP} argocd.questz.biz >> /etc/hosts
```