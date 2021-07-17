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
helm install argo-cd chargs/argo-cd/
```

Apply ingress controller
```
kubectl apply -f argo_ingress.yaml
```
```
echo {ingress loadbalncer IP} argocd.questz.biz >> /etc/hosts
```

login to argo cli and add argoproj repo 
get admin password
```
add argo repo to argo
```

Bootstrap argo apps
```
ad```

## SSL stuff
isntall CRDs
```
kubectl apply -f https://github.com/jetstack/cert-manager/releases/download/v1.4.0/cert-manager.yaml
```
Create issuer
```
kubectl --namespace cert-manager apply -f letsencrypt-issuer.yaml
```

### How applications get installed
Applications are bootstrapped using ArgoCD's application CRDs and then managed via a helm chart. 

Application CRD definitions are stored in `apps/templates/` which define and application to deploy with argo. 

# Kops Setup

Pre requisists
- Setup route53 
- setup kops user
- configure terminal context for kops user
    - export KOPS_STATE_STORE=s3://cmh-ss-state-stor
    - export AWS_ACCESS_KEY
    - export AWS_SECRET_KEY 
  
Create the cluster
```
kops create -f cluster/*/k8s.*.securesea.io
```

Define ssh public key (ideally create this on deployment)
```
kops create secret --name k8s.cmh.securesea.io sshpublickey admin -i ~/.ssh/id_rsa.pub
```

Deploy cluster 
```
kops update cluster --name k8s.cmh.securesea.io --yes 
```