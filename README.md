# Minikube argo setup

### Starting the cluster
Install ingress-nginx
```
minikube addons enable ingress
```

Install helm deps
```
helm dep build charts/
```

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

Bootstrap argo to manage argo
### How applications get installed
Applications are bootstrapped using ArgoCD's application CRDs and then managed via a helm chart. 

Application CRD definitions are stored in `apps/templates/` which define and application to deploy with argo. 