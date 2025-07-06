         ___        ______     ____ _                 _  ___  
    
# Kubernetes Deployment on EC2 using KIND

This project demonstrates how to deploy WebApp and ySQL database on a single-node Kubernetes cluster (using KIND) hosted inside an Amazon EC2 instance. The application containers are pulled from AWS ECR (Elastic Container Registry).

##Prerequisites

- AWS EC2 instance (Amazon Linux 2)
- Docker and KIND installed
- AWS CLI configured with appropriate credentials
- ECR repository access
- Kubernetes manifests (Pods, ReplicaSets, Deployments, Services)

## Setup and Deployment Steps

### 1. SSH into EC2 Instance

chmod 400 assignment2.pem
ssh -i "assignment2.pem" ec2-user@<public_ip>

### 2. Create KIND Cluster

kind create cluster --name clo835-cluster --config kind-config.yaml

### 3. Configure AWS Credentials

mkdir -p ~/.aws
nano ~/.aws/credentials

Add  AWS credentials
[default]
aws_access_key_id=YOUR_KEY
aws_secret_access_key=YOUR_SECRET


### 4. Login to Amazon ECR

aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin <AWS_AccountID>.dkr.ecr.us-east-1.amazonaws.com

### 5. Pull Docker Images from ECR

docker pull <AWS_AccountID>.dkr.ecr.us-east-1.amazonaws.com/webapp:latest
docker pull <AWS_AccountID>.dkr.ecr.us-east-1.amazonaws.com/mysql:latest

### 6. Create Image Pull Secret in Kubernetes

kubectl delete secret regcred --ignore-not-found

kubectl create secret docker-registry regcred \
  --docker-server=<AWS_AccountID>.dkr.ecr.us-east-1.amazonaws.com \
  --docker-username=AWS \
  --docker-password=$(aws ecr get-login-password --region us-east-1)

## Kubernetes Deployment

### 1. Verify Cluster

kubectl get nodes
kubectl get pods -A
kubectl cluster-info


### 2. Deploy MySQL and WebApp Pods

kubectl apply -f pod/pod_mysql.yaml
kubectl apply -f pod/pod_webapp.yaml

### 3. Port Forward and Test WebApp

kubectl port-forward webapp-pod 8080:8080
curl http://localhost:8080


### 4. Check WebApp Logs

kubectl logs webapp-pod

### 5. Delete Pods and Apply ReplicaSets

kubectl delete pod webapp-pod mysql-pod --ignore-not-found
kubectl apply -f replicaset/rs_mysql.yaml
kubectl apply -f replicaset/rs_webapp.yaml
kubectl get pods --show-labels

### 6. Apply Deployments

kubectl apply -f deployment/deployment_mysql.yaml
kubectl apply -f deployment/deployment_webapp.yaml

kubectl get deployments
kubectl get rs
kubectl get pods --show-labels


### 7. Expose WebApp via NodePort

kubectl apply -f service/service_webapp.yaml
kubectl get svc

### 8. Test Application via Public IP


curl http://localhost:30000
# Or access from browser:
http://<ec2-publicip>:30000

##  Update WebApp Image

1. Edit deployment:
nano deployment/deployment_webapp.yaml

2. Update image tag:
<AWS_AccountID>.dkr.ecr.us-east-1.amazonaws.com/webapp:v2

3. Apply changes:

kubectl apply -f deployment/deployment_webapp.yaml


4. Check pods:
kubectl get pods -l app=employees
kubectl describe pod <pod-name>

##  Clean Up Resources

kubectl delete deployments --all
kubectl delete services --all
kubectl delete pods --all
kubectl delete replicasets --all

