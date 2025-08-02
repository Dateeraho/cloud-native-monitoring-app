

# 🌩️ Cloud Native Resource Monitoring Python App on Kubernetes

This project demonstrates how to build a **Flask-based resource monitoring app** in Python and deploy it using **Docker**, **AWS ECR**, and **AWS EKS** with Kubernetes. The app monitors system metrics like CPU and memory usage and is accessible via a web interface.

---

## 🎯 Learning Outcomes

✅ Build a Monitoring App using **Flask + psutil**  
✅ Run a Python App locally  
✅ Containerize a Python app using **Docker**  
✅ Push Docker images to **AWS ECR** using **Boto3**  
✅ Create and deploy to an **AWS EKS cluster**  
✅ Write Kubernetes deployments/services using Python SDK  
✅ Automate everything through code  

---

## 🧰 Prerequisites

Make sure you have the following before you start:

- ✅ AWS Account with programmatic access
- ✅ AWS CLI configured (`aws configure`)
- ✅ Python 3.x installed
- ✅ Docker installed and running
- ✅ Kubectl installed and configured
- ✅ Code editor (like VS Code)

---

## 🚀 Project Setup

### Part 1: Run the Flask App Locally

1. **Clone the repository**
   
   git clone https://github.com/Dateeraho/cloud-native-monitoring-app.git
   cd cloud-native-monitoring-app
Install required packages

pip install -r requirements.txt
Start the Flask App
python app.py
Access the app in your browser: http://localhost:5000

🐳 Part 2: Dockerize the Flask App
Dockerfile

Here's the Dockerfile used:

FROM python:3.9-slim-buster
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
ENV FLASK_RUN_HOST=0.0.0.0
EXPOSE 5000
CMD ["flask", "run"]

### Build Docker Image
docker build -t flask-monitor-app .
### Run the Docker Container
docker run -p 5000:5000 flask-monitor-app
Now your app is running inside a container on http://localhost:5000

☁️ Part 3: Push Docker Image to AWS ECR
Create ECR Repository using Boto3

import boto3
ecr = boto3.client('ecr')
response = ecr.create_repository(repositoryName='my-ecr-repo')
print(response['repository']['repositoryUri'])
Push Docker Image to ECR

Follow AWS ECR push instructions after creating the repo:

aws ecr get-login-password | docker login --username AWS --password-stdin <aws_account_id>.dkr.ecr.<region>.amazonaws.com
docker tag flask-monitor-app <repo_uri>:latest
docker push <repo_uri>:latest

☸️ Part 4: Deploy App to AWS EKS
Step 1: Create EKS Cluster + Node Group
You can do this from the AWS Console or use eksctl CLI for automation.

Step 2: Use Python to Create Deployment & Service
In eks.py:
from kubernetes import client, config

config.load_kube_config()
api_client = client.ApiClient()

# Deployment
deployment = client.V1Deployment(
    metadata=client.V1ObjectMeta(name="my-flask-app"),
    spec=client.V1DeploymentSpec(
        replicas=1,
        selector=client.V1LabelSelector(match_labels={"app": "my-flask-app"}),
        template=client.V1PodTemplateSpec(
            metadata=client.V1ObjectMeta(labels={"app": "my-flask-app"}),
            spec=client.V1PodSpec(
                containers=[client.V1Container(
                    name="flask-container",
                    image="YOUR_ECR_IMAGE_URI",
                    ports=[client.V1ContainerPort(container_port=5000)]
                )]
            )
        )
    )
)

apps_api = client.AppsV1Api(api_client)
apps_api.create_namespaced_deployment(namespace="default", body=deployment)

# Service
service = client.V1Service(
    metadata=client.V1ObjectMeta(name="my-flask-service"),
    spec=client.V1ServiceSpec(
        selector={"app": "my-flask-app"},
        ports=[client.V1ServicePort(port=5000)]
    )
)

core_api = client.CoreV1Api(api_client)
core_api.create_namespaced_service(namespace="default", body=service)
Replace "YOUR_ECR_IMAGE_URI" with your actual ECR URI.

## Run the script:

python eks.py
📊 Validate Kubernetes Deployment
kubectl get deployments
kubectl get services
kubectl get pods

### Expose the app locally:

kubectl port-forward service/my-flask-service 5000:5000
Now you can again access it on http://localhost:5000