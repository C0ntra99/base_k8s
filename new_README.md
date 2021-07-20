# Base Kubernetes Clusters
---
### Descriptions
This repo is used to deploy the base kubernetes clusters for Secure Sea. 


### Core Components

__Kube2iam__
Gives pods ability to assume aws iam roles. 

__externaldns__
Syncs AWS Route53records with ingress resources

__cert manager__ 
Provides ssl services to all external facing ingress resources. 

__Argo CD__
Manages applications running on the cluster. 


## Deployment workflow

### Initial Deployment
- Deploy Kops cluster
    - kops create -f clusters/cmh/k8s.cmh.securesea.io
- Prerequisite cloud dependent setup
    - AWS iam roles and policies for kube2iam daemonset
    - AWS iam roles and policies for application pods 
- Deploy Core Stack
    - Install Kube2iam
    - Install ingress-nginx
    - Install externaldns
    - Install cert-manager
    - Install ArgoCD
- ArgoCD configuration takes over management of core stack against base_k8s repo
    - Monitors Core Stack for future changes
- Applications will have seperate Repos
    - Repos will deploy to cluster
    - ArgoCD will manage CI/CD for application


### Updates to the Core Cluster (kops config)
- Gitlab CI/CD will deploy updates 

### Updates to the Core Application Stack
_kube2iam, ingress-nginx, cert-manager, externaldns, ArgoCD_
- ArgoCD Monitors the Core stack repo (base_k8s) and will auto deploy new configuration
    - If error occurs Argo will rollback to previous config

### Updates to Application
- ArgoCD Monitors the Application repo and will auto deploy new applicaiton code
- Includes Testing, Scanning, Staging
