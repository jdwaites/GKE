
# GKE Hello World Example

This project demonstrates how to build, containerize, and deploy a simple Python Flask web application to Google Kubernetes Engine (GKE) using GitHub Actions, Docker, and Kustomize.

## Project Structure

```
GKE/
├── .github/workflows/deploy-to-gke.yml   # GitHub Actions workflow for CI/CD
├── deployment.yaml                       # Kubernetes Deployment manifest
├── service.yaml                          # Kubernetes Service (LoadBalancer) manifest
├── kustomization.yaml                    # Kustomize configuration
├── hello-gke/
│   ├── app.py                            # Flask web app ("Hello from GKE!")
│   ├── Dockerfile                        # Docker build instructions
│   └── requirements.txt                  # Python dependencies
└── README.md                             # Project documentation
```

## Application

- **Flask App**: A minimal Python web server that responds with "Hello from GKE!".
- **Docker**: The app is containerized using a Dockerfile based on Ubuntu and Python 3.
- **Kubernetes**: The app is deployed to GKE as a Deployment with two replicas and exposed via a LoadBalancer Service.

## Deployment Workflow

- **GitHub Actions**: Automates build, push, and deployment to GKE.
	- Authenticates to Google Cloud using a service account.
	- Builds and pushes the Docker image to Google Artifact Registry.
	- Uses Kustomize to update the image tag in manifests.
	- Applies manifests to GKE and checks rollout status.

## Key Files

- `hello-gke/app.py`: Flask app code.
- `hello-gke/Dockerfile`: Container build instructions.
- `hello-gke/requirements.txt`: Python dependencies (`flask`).
- `deployment.yaml`: Defines the Kubernetes Deployment.
- `service.yaml`: Exposes the app via a LoadBalancer.
- `kustomization.yaml`: Kustomize config for image substitution.
- `.github/workflows/deploy-to-gke.yml`: CI/CD pipeline.

## How It Works

1. **Push to GitHub**: Triggers the workflow.
2. **Build & Push Image**: Docker image is built and pushed to Artifact Registry.
3. **Update Manifests**: Kustomize updates the image tag.
4. **Deploy to GKE**: Manifests are applied; app is exposed via an external IP.

## Accessing the App

After deployment, the app is available at the external IP of the LoadBalancer service.  
Use `kubectl get services` to find the IP.

## Requirements

- Google Cloud project with GKE and Artifact Registry enabled.
- Service account with permissions for GKE and Artifact Registry.
- GitHub repository secrets for authentication.
