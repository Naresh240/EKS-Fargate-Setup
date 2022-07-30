# EKS-Fargate-Setup

## EKS Fargate Cluster Setup
```bash
eksctl create cluster --name eksdemo --region us-east-1 --fargate
```

## OIDC Provider Creation
```bash
eksctl utils associate-iam-oidc-provider --region=us-east-1 --cluster=eksdemo --approve
```

## Create Role to Access ECR from EKS Fargate Cluster
```bash
eksctl create iamserviceaccount \
    --attach-policy-arn arn:aws:iam::aws:policy/AmazonEC2ContainerRegistryPowerUser \
    --cluster eksdemo \
    --name ecr-role \
    --namespace default \
    --override-existing-serviceaccounts \
    --region us-east-1 \
    --approve	
```

## Build and Push Image to ECR
```bash
docker pull nginx
# Login to ECR
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 179746471541.dkr.ecr.us-east-1.amazonaws.com
docker tag nginx:latest 179746471541.dkr.ecr.us-east-1.amazonaws.com/nginx:latest
docker push 179746471541.dkr.ecr.us-east-1.amazonaws.com/nginx:latest
```
## Deploy application with below
```bash
cat << EOF > deployment.yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: 179746471541.dkr.ecr.us-east-1.amazonaws.com/nginx:latest
        ports:
        - containerPort: 80
EOF       
		
kubectl apply -f deployment.yaml
```
