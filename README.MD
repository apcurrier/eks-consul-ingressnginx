# Consul on EKS using OSS nginx as ingress (transparent mode)

## requirements
- an AWS account and a region that support EKS
- environment variables to access AWS account locally
- `eksctl` installed locally
- `kubectl` installed locally
- `helm` installed locally

## Installation

Deploy an EKS cluster:

```bash
eksctl create cluster --name=<cluster name> --region=<region> --nodes=3 --version 1.21
```

Import kubectl config

```bash
aws eks update-kubeconfig --region <region> --name <cluster name>
```

Deploy Consul

```bash
helm repo add hashicorp https://helm.releases.hashicorp.com
helm install consul hashicorp/consul --values consul-values.yaml --create-namespace --namespace consul
```

Add deny all intention
```bash
kubectl apply -f denyall.yaml
```

Deploy nginx ingress
```bash
git clone https://github.com/nginxinc/kubernetes-ingress.git --branch v2.4.1
helm repo add nginx-stable https://helm.nginx.com/stable
helm upgrade --install nginx-ingress nginx-stable/nginx-ingress --namespace=nginx-ingress --create-namespace --values ../../../eks-consul-ingressnginx/nginxingress-values.yaml
```

Add configuration for Dialed Directly

```bash
kubectl apply -f sd-direct.yaml
```

Add configuration for ingress config to static-server. If there are issues with validating the ingress object see https://github.com/kubernetes/ingress-nginx/issues/5968 for workarounds. 
```bash
kubectl apply -f ingress.yaml
```

deploy static-server
```bash
kubectl apply -f static-server.yaml
```

add intention to static server from ingress
```bash
kubectl apply -f allow-static-server.yaml
```

To get the ingress endpoint provisioned in ELB:
```bash
kubectl get service -n nginx-ingress
```

Ensure you get back a `hello world` response:

```bash
STATIC_SERVER_LB=$(kubectl get service -n nginx-ingress nginx-ingress-nginx-ingress -o jsonpath="{.status.loadBalancer.ingress[*].hostname}")
curl $STATIC_SERVER_LB
```
