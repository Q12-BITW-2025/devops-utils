````markdown
# DEVOPS README

This README provides step-by-step instructions for IT professionals to deploy and manage the q12-odigos setup on AWS EKS, including Docker image management, Kubernetes commands, and Odigos installation.

---

## Prerequisites

- **AWS CLI** installed and configured
- **kubectl** installed
- **Docker** installed and running
- **Helm** installed
- (Optional) **eksctl** for IAM OIDC association

---

## 1. AWS Access

1. Open the AWS console:
   ```bash
   AWS LINK: https://bit.ly/35KqkKb
````

2. Authenticate with your sandbox credentials:

   ```bash
   pcl aws --sandbox-user --domain naeast --sid <SID>
   ```

### If using PowerShell

```powershell
$Env:AWS_PROFILE='adfs'
```

### If using a Unix shell

```bash
export AWS_PROFILE='adfs'
```

---

## 2. Configure kubeconfig for EKS

Point `kubectl` to your EKS cluster:

```bash
aws eks --region <AWS_REGION> update-kubeconfig --name <EKS_CLUSTER_NAME>
```

---

## 3. Docker & Amazon ECR

### 3.1 Create an ECR repository

```bash
aws ecr create-repository \
  --repository-name <ECR_REPO_NAME> \
  --region <AWS_REGION>
```

### 3.2 Authenticate Docker to ECR

```bash
aws ecr get-login-password --region <AWS_REGION> \
  | docker login --username AWS --password-stdin <ACCOUNT_ID>.dkr.ecr.<AWS_REGION>.amazonaws.com
```

### 3.3 Tag and push your image

```bash
# Tag local image for ECR
docker tag <docker-image-name>:<version> \
  <ACCOUNT_ID>.dkr.ecr.<AWS_REGION>.amazonaws.com/<docker-image-name>:<version>

# Push to ECR
docker push \
  <ACCOUNT_ID>.dkr.ecr.<AWS_REGION>.amazonaws.com/<docker-image-name>:<version>
```

### 3.4 Retrieve repository URI

```bash
aws ecr describe-repositories \
  --repository-names <ECR_REPO_NAME> \
  --region <AWS_REGION> \
  --query 'repositories[0].repositoryUri' \
  --output text
```

---

## 4. Kubernetes Operations

### General kubectl commands

```bash
kubectl get nodes
kubectl config current-context
kubectl describe node <NODE_NAME> | grep -E "Capacity:|Allocatable:|pods"
```

### Managing Pods

```bash
# Force-delete a pod
kubectl delete pod <POD_NAME> --grace-period=0 --force

# Tail logs of a container
kubectl logs -f <POD_NAME> -c <CONTAINER_NAME>

# Describe a pod
kubectl describe pod <POD_NAME> -n <NAMESPACE>
```

### Interactive debug pod

```bash
kubectl run curl-debug --rm -i --tty \
  --image=curlimages/curl --restart=Never -- sh

#Then you can run curl commands to validate HTTP/HTTPS connectivity
```

---

## 5. Verifying Odigos Components

```bash
# List all Odigos pods
iemkubectl get pods -n odigos-system

# List DaemonSets
iemkubectl get daemonset -n odigos-system

# Check HostPath & privileged settings (Linux)
kubectl describe daemonset odiglet -n odigos-system \
  | grep -E 'HostPath|privileged'

# Check HostPath & privileged settings (Windows)
kubectl describe daemonset odiglet -n odigos-system -n odigos-system \
  | Select-String -Pattern 'HostPath','privileged'
```

---

## 6. Scaling Node Group Dynamically

```bash
aws eks update-nodegroup-config \
  --cluster-name <EKS_CLUSTER_NAME> \
  --nodegroup-name <NODE_GROUP_NAME> \
  --scaling-config minSize=3,maxSize=3,desiredSize=3
```

---

## 7. Testing with Public APIs

* Postman Collection: [https://www.postman.com/cs-demo/public-rest-apis/collection/tfzpqfc/public-rest-api](https://www.postman.com/cs-demo/public-rest-apis/collection/tfzpqfc/public-rest-api)
* Example API:

  ```bash
  curl https://pokeapi.co/api/v2/pokemon/charizard/
  ```

---

## 8. Associate IAM OIDC Provider (for IRSA)

```bash
eksctl utils associate-iam-oidc-provider \
  --region <AWS_REGION> \
  --cluster <EKS_CLUSTER_NAME> \
  --approve
```

---

## 9. Installing Odigos via Helm

```bash
helm repo add odigos https://odigos-io.github.io/odigos/
helm repo update
helm upgrade --install odigos odigos/odigos \
  --namespace odigos-system --create-namespace

helm install odigos-operator odigos/odigos-operator \
  --namespace odigos-system \
  --set rbac.create=true \
  --set args="--log-level=info"
```

---

## 10. Odigos Demo Deployment

```bash
# Jaeger backend
kubectl apply -f https://raw.githubusercontent.com/odigos-io/simple-demo/main/kubernetes/jaeger.yaml

# Create demo namespace and app\kubectl create namespace odigos-demo
kubectl apply -n odigos-demo \
  -f https://raw.githubusercontent.com/odigos-io/simple-demo/main/kubernetes/deployment.yaml
```

---

## 11. Building & Pushing Docker Locally

```bash
# Build locally
docker build -t <ECR_REPO_NAME>:latest .

# Tag for ECR
docker tag <ECR_REPO_NAME>:latest 512979937293.dkr.ecr.<AWS_REGION>.amazonaws.com/<ECR_REPO_NAME>:latest

# Push to ECR
docker push 512979937293.dkr.ecr.<AWS_REGION>.amazonaws.com/<ECR_REPO_NAME>:latest
```

---

*For more details or troubleshooting, refer to the official AWS, Kubernetes, and Odigos documentation.*

```
```
