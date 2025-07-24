# Simple Python Web Application with Manual ArgoCD Deployment

This repository contains a basic Python web application built with Flask. It is designed to be deployed to a Google Kubernetes Engine (GKE) cluster using a manual workflow with Docker, Google Artifact Registry, and ArgoCD.

## 📝 Overview

This project demonstrates a fully manual process for deploying a containerized web service. The workflow involves manually building a Docker image, pushing it to a registry, updating Kubernetes configuration files, and using the ArgoCD web UI to deploy the application.

This approach is useful for understanding the fundamental steps of a deployment before moving to a more automated GitOps model.

## 📂 Repository Structure

*   **`app.py`**: The main application file containing the Flask web server.
*   **`requirements.txt`**: A list of Python dependencies (`Flask`, `gunicorn`).
*   **`Dockerfile`**: Defines the steps to build a Docker image for the application.
*   **`k8s/`**: Contains the Kubernetes configuration manifests.
    *   **`deployment.yaml`**: Deploys the application as a Kubernetes `Deployment`.
    *   **`service.yaml`**: Exposes the application as a Kubernetes `Service`.

## 🚀 Local Development and Testing

Before deploying, you can run the application on your local machine.

### Prerequisites

*   Python 3
*   Docker

### Running Locally

1.  **Clone the repository:**
    ```bash
    git clone https://github.com/aashishrrjp/python-app.git
    cd python-app
    ```
2.  **Install dependencies:**
    ```bash
    pip install -r requirements.txt
    ```
3.  **Run the application:**
    ```bash
    python app.py
    ```
    The application will be available at `http://127.0.0.1:5000`.

### 🐳 Running with Docker

1.  **Build the Docker image:**
    ```bash
    docker build -t python-app .
    ```
2.  **Run the container:**
    ```bash
    docker run -p 5000:5000 python-app
    ```
    The application will be available at `http://localhost:5000`.

## ☸️ Manual Deployment to GKE with ArgoCD

This workflow outlines the manual steps to deploy the application to a GKE cluster.

### Step 1: Build and Push the Docker Image

After making changes to the application, you must build a new Docker image and push it to Google Artifact Registry.

1.  **Authenticate Docker with gcloud:**
    ```bash
    gcloud auth configure-docker us-central1-docker.pkg.dev
    ```
2.  **Build and Push the Image:**
    Replace `your-gcp-project` and `your-repo-name` with your details.
    ```bash
    # Define a unique tag for the new image
    export IMAGE_TAG=v1.0.1
    export IMAGE_URI="us-central1-docker.pkg.dev/your-gcp-project/your-repo-name/python-app:$IMAGE_TAG"

    # Build the image
    docker build -t $IMAGE_URI .

    # Push the image
    docker push $IMAGE_URI
    ```

### Step 2: Update the Kubernetes Deployment

Manually edit the `k8s/deployment.yaml` file to update the image reference to the new URI and tag you just pushed.

```yaml
# k8s/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: python-app
spec:
  replicas: 2
  template:
    spec:
      containers:
      - name: python-app
        # Update this line with the new image URI
        image: us-central1-docker.pkg.dev/your-gcp-project/your-repo-name/python-app:v1.0.1
        ports:
        - containerPort: 5000
