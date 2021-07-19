# Minikube argo setup

### Starting the cluster
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

login to argo cli and add argoproj repo 
get admin password
```
add argo repo to argo
```

Bootstrap argo apps from argo repo
```
helm template apps/ | kubectl apply -f -
```

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

Export config
```
kops export kubecfg k8s.cmh.securesea.io --admin
```

Deploy ingress-nginx
```
k create ns nginx
helm install ingress-nginx ingress-nginx/ingress-nginx
```

Core k8s components:

Core k8s components:
| Component | Deployed | Use |
--- |---|---
ingress-nginx | X | Backend ingress controller
cert-manager | | Dynamically generate ssl certificates
externaldns | | Dynamically modify route53 based on ingress controller
argo-cd | X | Manage application state

### Setting up kube2iam:
Workers need a role with the following policy:
```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Action": [
        "sts:AssumeRole"
      ],
      "Effect": "Allow",
      "Resource": "arn:aws:iam::1234567890:role/k8s-*"
    }
  ]
}
```

Allow masters/nodes.k8s.*.securesea.io instance groups to assume roles that start with k8-
```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "",
      "Effect": "Allow",
      "Principal": {
        "Service": "ec2.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    },
    {
      "Sid": "",
      "Effect": "Allow",
      "Principal": {
        "AWS": [
            "arn:aws:iam::011512970419:role/masters.k8s.cmh.securesea.io",
            "arn:aws:iam::011512970419:role/nodes.k8s.cmh.securesea.io"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

On cluster:
```
kubectl apply -f apps/templates/k2iam-sa.yaml
helm install --name iam \
    --namespace kube-system \
    -f charts/kube2iam/values-kube2iam.yaml \
    stable/kube2iam
```
Attach policies that that pods need to assume to the isntance group roles

to attach a policy to a pod the following annotation should be used:
```
iam.amazonaws.com/role: k8s-{{ role name }}
```

### Externaldns
Role required for pods
```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "route53:ChangeResourceRecordSets"
      ],
      "Resource": [
        "arn:aws:route53:::hostedzone/*"
      ]
    },
    {
      "Effect": "Allow",
      "Action": [
        "route53:ListHostedZones",
        "route53:ListResourceRecordSets"
      ],
      "Resource": [
        "*"
      ]
    }
  ]
}
```

Installing:
```
 helm install external-dns -f charts/external-dns/values.yaml bitnami/external-dns 
 ```
 This is how it should be done for core components, grabbing the official chart and using hosted values vaues

### Cert Manager
install cruds:
```
kubectl apply --validate=false -f https://raw.githubusercontent.com/jetstack/cert-manager/release-0.11/deploy/manifests/00-crds.yaml
```

create ns
```
kubectl create ns cert-manager
```

helm install \
  --name cert-manager \
  --namespace cert-manager \
  --version v0.11.1 \
  jetstack/cert-manager



Status:
- need to automate the deployment of the cluster and core components
- Use pulumi to verify aws dependencies are dpeloyed
- Setup cert-mangaer
- setup external-dns

kops deploys