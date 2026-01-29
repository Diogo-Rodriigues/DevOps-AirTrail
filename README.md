# AirTrail Operations

This repository contains the infrastructure automation code for the AirTrail application, designed to deploy, configure, and monitor the application on Google Kubernetes Engine (GKE) using Ansible.

## 📖 About the Project

The goal of this project is to automate the lifecycle of the AirTrail web application—a flight tracking and history service—on the cloud. 

**Context & Objectives:**
- **Zero-Touch Automation**: Installs and configures the entire stack (infrastructure, database, application, monitoring) with minimal manual intervention.
- **Resilience**: Ensures critical user data survives application restarts or failures.
- **Scalability**: Automatically adjusts resources based on traffic load.
- **Monitoring**: Provides real-time visibility into system health and performance.

## ✨ Key Features

- **Automated Infrastructure**: Provisioning of GKE clusters and networking via Ansible.
- **Containerized Architecture**: Separation of concerns with distinct Pods for the Application and Database layers.
- **High Availability & Scaling**:
  - **Horizontal Pod Autoscaling (HPA)**: Automatically scales the application based on CPU usage.
  - **Load Balancing**: External traffic management via Google Cloud LoadBalancer.
- **Data Persistence**: Uses PersistentVolumeClaims (PVC) to ensure PostgreSQL data persists across Pod restarts.
- **Security**:
  - Database isolated via internal ClusterIP.
  - Secrets management (DB credentials, keys) using Ansible Vault.
- **Dynamic Configuration**: Automatic detection of external LoadBalancer IPs to configure the application's CORS and security settings (`ORIGIN`) on the fly.
- **Integrated Monitoring**: Custom Google Cloud Monitoring dashboards deployed automatically to track CPU, memory, and scaling events.

## 🛠 Tech Stack

- **Automation:** Ansible
- **Cloud Provider:** Google Cloud Platform (GKE - Google Kubernetes Engine)
- **Container Orchestration:** Kubernetes
- **Database:** PostgreSQL
- **Application:** AirTrail (Node.js/Web)
- **Scripting:** Python, Bash, YAML

## ✅ Prerequisites

Before running the automation, ensure you have the following:

1.  **Google Cloud Platform (GCP) Account**: A valid project with billing enabled.
2.  **Tools Installed**:
    -   [Ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html)
    -   [Google Cloud SDK (`gcloud`)](https://cloud.google.com/sdk/docs/install)
    -   [`kubectl`](https://kubernetes.io/docs/tasks/tools/)
    -   Python 3 & `pip`
3.  **Authentication**:
    -   GCP Service Account JSON key.
    -   SSH keys generated for the Ansible user.
4.  **Configuration**:
    -   Update `inventory/gcp.yml` with your specific GCP project ID, region, and zone.
    -   Ensure `requests` and `google-auth` python libraries are installed.

## 🚀 Installation & Usage

All commands should be executed from the root of this directory (`DevOps-AirTrail`).

### 1. Infrastructure Management

**Create Cluster:**
Provisions the VPC network and a 2-node GKE cluster.
```bash
ansible-playbook -i inventory/gcp.yml gke-cluster-create.yml -e 'ansible_python_interpreter=/home/vagrant/.checkpoints/bin/python3'
```

**Destroy cluster:**
Deletes all physical resources in GCP to prevent ongoing costs.
```bash
ansible-playbook -i inventory/gcp.yml gke-cluster-destroy.yml -e 'ansible_python_interpreter=/home/vagrant/.checkpoints/bin/python3'
```

### 2. Application Deployment

**Deploy AirTrail:**
Deploys PostgreSQL, the AirTrail app, services, and HPA.
*Note: Requires Ansible Vault password to decrypt secrets like database credentials and API keys.*
```bash
ansible-playbook -i inventory/gcp.yml -i inventory/vault.yml airtrail-deploy.yml --ask-vault-pass
```

**Undeploy AirTrail:**
Removes the application and services from the cluster.
```bash
# To also delete persistent data (PVC), add: -e delete_data=true
ansible-playbook -i inventory/gcp.yml airtrail-undeploy.yml
```
### Automated Testing & Monitoring
Validate performance and observe system health using Google Cloud's native tools.

**Monitoring Deploy:**
Creates custom dashboards in Google Cloud Monitoring to observe CPU spikes and HPA scaling events.

Before deploying monitoring, ensure the necessary GCP APIs are enabled:
```bash
gcloud services enable monitoring --project=<project_id>
gcloud services enable compute.googleapis.com
gcloud services enable monitoring.googleapis.com
gcloud services enable cloudbuild.googleapis.com
```

```bash
ansible-playbook -i inventory/gcp.yml monitoring-deploy.yml
```

**Undeploy Monitoring:**
Removes the custom dashboards.
```bash
ansible-playbook -i inventory/gcp.yml monitoring-undeploy.yml
```

### 4. Testing

**Run Tests:**
Executes connectivity and functionality tests against the deployed application.
```bash
ansible-playbook -i inventory/gcp.yml -i inventory/vault.yml test-all.yml --ask-vault-pass
```

## 📂 Project Structure

A brief overview of the key files and directories:

### Directories
- **`roles/`**: Contains the modular Ansible logic.
  - **`airtrail/`**: Manages the deployment of the Node.js application (Deployment, Service, HPA).
  - **`postgres/`**: Manages the database deployment (StatefulSet/Deployment, PVC, Service).
  - **`monitoring/`**: Handles the creation of Google Cloud Monitoring dashboards.
  - **`gke_cluster_create/`**: Tasks for provisioning the GKE cluster.
  - **`gke_cluster_destroy/`**: Tasks for dismantling the cluster.
  - **`test_airtrail/`**: Contains verification tasks.
- **`inventory/`**: Configuration files.
  - **`gcp.yml`**: Define hosts (localhost) and global GCP variables (project ID, zone, cluster name).
  - **`vault.yml`**: Encrypted sensitive data (DB passwords, API keys).

### Playbooks (Root)
- **`gke-cluster-create.yml`**: Entry point for infrastructure provisioning.
- **`gke-cluster-destroy.yml`**: Entry point for infrastructure destruction.
- **`airtrail-deploy.yml`**: Main playbook; coordinates the `postgres` and `airtrail` roles to deploy the stack.
- **`airtrail-undeploy.yml`**: Removes the application components.
- **`monitoring-deploy.yml`**: Deploys the monitoring dashboards.
- **`monitoring-undeploy.yml`**: Removes the monitoring dashboards.
- **`test-all.yml`**: Runs the validation suite.
