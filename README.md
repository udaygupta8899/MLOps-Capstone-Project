<h1 align="center">🚀 Capstone MLOps Project: Scalable ML Deployment with CI/CD, EKS, and Monitoring 🌐</h1>

<p align="center">
  <img src="https://img.shields.io/badge/MLFlow-Tracking-blue" />
  <img src="https://img.shields.io/badge/DVC-Versioning-purple" />
  <img src="https://img.shields.io/badge/Docker-Containerization-green" />
  <img src="https://img.shields.io/badge/Kubernetes-EKS-orange" />
  <img src="https://img.shields.io/badge/Monitoring-Prometheus%20%26%20Grafana-red" />
</p>

---

## 📌 Project Overview

This capstone project is a full-fledged **MLOps pipeline** built for robust Machine Learning lifecycle management—from data versioning, model training, and experiment tracking to deployment using **Docker & Kubernetes on AWS EKS**, with **CI/CD via GitHub Actions**, and **real-time monitoring via Prometheus & Grafana**.

The project repository is hosted on GitHub: [MLOps-Capstone-Project](https://github.com/udaygupta8899/MLOps-Capstone-Project)

---

## 🗂 Project Structure

Below is an overview of the project structure that organizes the code, pipelines, and deployment configurations:

MLOps-Capstone-Project/
├── .github/
│   └── workflows/
│       └── ci.yaml           # GitHub Actions CI/CD pipeline configuration
├── data/                     # Datasets and raw data
├── dvc.yaml                  # DVC pipeline definitions
├── params.yaml               # Hyperparameters and configuration parameters
├── dockerfile                # Docker build file for containerizing the Flask app
├── flask_app/                # Flask application for serving the model
│   ├── app.py                # Main Flask app entry point
│   ├── requirements.txt      # Python dependencies specific to the Flask app
│   └── ...                   # Other Flask-related modules and files
├── src/                      # Source code for ML pipeline
│   ├── logger/               # Custom logging utilities
│   ├── data_ingestion.py     # Data ingestion module
│   ├── data_preprocessing.py # Data cleaning and preprocessing
│   ├── feature_engineering.py# Feature extraction and transformation
│   ├── model_building.py     # Model training scripts
│   ├── model_evaluation.py   # Model evaluation and metrics
│   └── register_model.py     # Model registration and tracking
├── tests/                    # Test suites for CI and model validation
├── scripts/                  # Helper scripts for deployment and CI tasks
└── README.md                 # This documentation file


This structure ensures a clear separation of concerns—from model development to deployment and monitoring.

---

## 🧱 Project Setup and Installation

### Initial Setup

```bash
# 1. Create GitHub repo & clone it locally
git clone https://github.com/udaygupta8899/MLOps-Capstone-Project.git
cd MLOps-Capstone-Project

# 2. Create and activate a conda virtual environment
conda create -n atlas python=3.10
conda activate atlas

# 3. Install Cookiecutter and scaffold the project
pip install cookiecutter
cookiecutter -c v1 https://github.com/drivendata/cookiecutter-data-science

# 4. Rename directory if necessary (src.models -> src.model) and push changes
git add .
git commit -m "Initial project structure setup"
git push
```

---

## 🔍 Experiment Tracking with MLFlow on DagsHub

1. Connect your GitHub repo to [DagsHub](https://dagshub.com/dashboard) and follow the connection steps.
2. Install the necessary packages:
   ```bash
   pip install dagshub mlflow
   ```
3. Use the provided experiment tracking URL and code snippet from DagsHub to log experiments.

---

## 🧪 DVC - Data & Pipeline Versioning

Initialize and configure DVC:

```bash
# Initialize DVC
dvc init

# Create a local folder for temporary S3 storage
mkdir local_s3
dvc remote add -d mylocal local_s3

# Add pipeline configuration files
# (Ensure dvc.yaml and params.yaml are properly configured)
dvc repro
dvc status

# Commit the changes
git add .
git commit -m "Add DVC pipeline configuration"
git push
```

For S3 remote storage:

```bash
pip install dvc[s3] awscli
aws configure
dvc remote add -d myremote s3://<bucket-name>
```

---

## ⚙️ Flask App and Docker Setup

1. Inside the `flask_app/` directory, create your Flask application files.
2. Install Flask:
   ```bash
   pip install flask
   ```
3. Freeze your requirements:
   ```bash
   pip freeze > requirements.txt
   ```
4. Build and run your Docker image:
   ```bash
   docker build -t capstone-app:latest .
   docker run -p 8888:5000 -e CAPSTONE_TEST=<your-dagshub-token> capstone-app:latest
   ```

---

## 🔁 CI/CD with GitHub Actions

1. Create a CI/CD workflow file at `.github/workflows/ci.yaml`.
2. Generate and add your DAGsHub token as a GitHub secret.
3. Configure the workflow to include:
   - Code linting and testing
   - Docker image building
   - DVC pipeline execution
   - Pushing Docker image to AWS ECR

---

## ☁️ AWS & Kubernetes (EKS) Deployment

### Pre-requisites
- **AWS CLI**: Verify installation and update PATH if necessary.
- **kubectl**: Install and verify the client.
- **eksctl**: Install for managing EKS clusters.

### Create an EKS Cluster

```bash
eksctl create cluster \
  --name flask-app-cluster \
  --region us-east-1 \
  --nodegroup-name flask-app-nodes \
  --node-type t3.small \
  --nodes 1 \
  --nodes-min 1 \
  --nodes-max 1 \
  --managed
```

Verify cluster configuration:

```bash
kubectl get nodes
kubectl get pods
kubectl get svc
```

Access your Flask app via:
```bash
curl http://<external-ip>:5000
```

---

## 📊 Monitoring with Prometheus & Grafana

### Prometheus Setup (Port: 9090)

1. Download and extract Prometheus:
   ```bash
   wget https://github.com/prometheus/prometheus/releases/download/v2.46.0/prometheus-2.46.0.linux-amd64.tar.gz
   tar -xvzf prometheus-2.46.0.linux-amd64.tar.gz
   sudo mv prometheus-2.46.0.linux-amd64 /etc/prometheus
   sudo mv /etc/prometheus/prometheus /usr/local/bin/
   ```
2. Configure Prometheus in `/etc/prometheus/prometheus.yml`:
   ```yaml
   global:
     scrape_interval: 15s

   scrape_configs:
     - job_name: "flask-app"
       static_configs:
         - targets: ["<EKS External IP>:5000"]
   ```
3. Run Prometheus:
   ```bash
   /usr/local/bin/prometheus --config.file=/etc/prometheus/prometheus.yml
   ```

### Grafana Setup (Port: 3000)

1. Download and install Grafana:
   ```bash
   wget https://dl.grafana.com/oss/release/grafana_10.1.5_amd64.deb
   sudo apt install ./grafana_10.1.5_amd64.deb -y
   sudo systemctl start grafana-server
   sudo systemctl enable grafana-server
   ```
2. Access Grafana at: `http://<grafana-ec2-ip>:3000`  
   Default credentials: **admin/admin**
3. Add Prometheus as a data source and start building your dashboards.

---

## 🧹 AWS Resource Cleanup

To ensure proper resource management, execute these commands:

```bash
# Delete Kubernetes resources
kubectl delete deployment flask-app
kubectl delete service flask-app-service
kubectl delete secret capstone-secret

# Delete EKS Cluster
eksctl delete cluster --name flask-app-cluster --region us-east-1

# Optionally clean up ECR, S3, and related CloudFormation stacks via the AWS Console
```

---

## 🧠 CloudFormation & EKS

**AWS CloudFormation** automates the provisioning of AWS infrastructure as code.  
When running `eksctl`, CloudFormation templates are generated behind the scenes to create:
- **EKS Control Plane Stack**
- **Node Group Stack**

These stacks are managed as a unit to simplify resource management.

---

## 🧾 Concepts Glossary

| Term                | Description                                                                   |
|---------------------|-------------------------------------------------------------------------------|
| **DVC**             | Data version control & ML pipeline management                                 |
| **MLFlow**          | Experiment tracking and model management platform                             |
| **Docker**          | Containerization tool used to package the Flask app and its dependencies        |
| **Kubernetes (EKS)**| Managed cluster service for deploying, scaling, and operating containerized apps|
| **CI/CD**           | Continuous Integration and Continuous Deployment pipeline                     |
| **Prometheus**      | Monitoring system and time-series database for collecting metrics              |
| **Grafana**         | Visualization tool for displaying metrics from Prometheus                     |
| **PVC**             | Persistent Volume Claim for Kubernetes storage                                |
| **Fleet Requests**  | Internal AWS requests for provisioning EC2 instances via Auto Scaling Groups     |

---

## 🧠 Final Thoughts

This project demonstrates end-to-end **MLOps** by combining industry-standard tools and workflows for reproducible ML development, deployment, and monitoring. Whether you’re refining data science models or scaling to enterprise-grade infrastructures, this setup offers a comprehensive blueprint for success.

---

<h3 align="center">⚡ Built with ❤️</h3>
