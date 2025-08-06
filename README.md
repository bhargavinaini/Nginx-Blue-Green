# Nginx-Blue-Green Deployment

This repository demonstrates a **Blue-Green Deployment** strategy using **NGINX** and **Helm** in a Kubernetes environment. The setup ensures zero-downtime deployments by maintaining two identical production environments, allowing seamless switching between them.

## 🧪 Architecture Overview

- **Blue Environment**: Represents the current live production version.
- **Green Environment**: Hosts the new version to be deployed.
- **NGINX Ingress Controller**: Routes traffic between the Blue and Green environments.
- **Helm**: Manages the deployment and lifecycle of both environments.

## 🚀 Prerequisites

Ensure the following tools are installed:

- [Helm](https://helm.sh/docs/intro/install/)
- [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/)
- [NGINX Ingress Controller](https://kubernetes.github.io/ingress-nginx/deploy/)
- [Minikube](https://minikube.sigs.k8s.io/docs/start/) 

## 📁 Repository Structure

Nginx-Blue-Green/

├──Jenkins/

│  ├── JenkinsFile

│  ├── Jenkins_deployment.yaml

├──my-app/

│  ├── Chart.yaml

│  ├── values.yaml

│  ├── templates/

│       ├── configmap.yaml

│       ├── deployment.yaml

│       ├── service.yaml

│       └── ingress.yaml

└── README.md


- **my-app/**: Contains the Helm chart for deploying the application.
- **Jenkins/**: Contains Jenkins CI/CD pipeline for automating deployments and Jenkins deployment file.

## 🛠️ Setup Instructions

### 1. Clone the Repository

```bash
git clone https://github.com/bhargavinaini/Nginx-Blue-Green.git
cd Nginx-Blue-Green
```

### 2. Deploy Blue Environment
```bash
helm install nginx-blue ./my-app \
  --set app.color=blue \
  --set app.tag=1.0
```
### 3. Deploy Green Environment
``` bash
helm install nginx-green ./my-app \
  --set app.color=green \
  --set app.tag=1.1
```
### 4. Configure NGINX Ingress
Create an ingress resource to manage traffic routing:
``` bash
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nginx-traffic-router
  annotations:
    nginx.ingress.kubernetes.io/canary: "true"
    nginx.ingress.kubernetes.io/canary-weight: "0"
spec:
  rules:
  - host: nginx.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: nginx-blue-svc
            port:
              number: 80
      - path: /
        pathType: Prefix
        backend:
          service:
            name: nginx-green-svc
            port:
              number: 80
```

Apply the ingress resource:

``` bash
kubectl apply -f nginx-traffic-router.yaml
```

### 5. Shift Traffic to Green Environment using Pipeline 

To gradually shift traffic to the Green environment, update the percentage parameter while triggering Jenkins pipeline

## 🧪 CI/CD Pipeline
The repository includes a Jenkinsfile that automates the deployment process:

Checkout: Retrieves the latest code.
Deploy Blue: Deploys the Blue environment. or 
Deploy Green: Deploys the Green environment.
Shift Traffic: Gradually shifts traffic to the Green environment.

## Rollback to Blue Environment
If issues arise with the Green environment, you can revert traffic to the Blue environment:

```bash
kubectl annotate ingress nginx-traffic-router \
  nginx.ingress.kubernetes.io/canary-weight="0" --overwrite
```
This command stops traffic to the Green environment and directs all traffic to the Blue environment.


