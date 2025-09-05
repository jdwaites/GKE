
# GKE Hello World Example

This project demonstrates how to build, containerize, and deploy a simple Python Flask web application to Google Kubernetes Engine (GKE) using GitHub Actions, Docker, and Helm.

## Project Structure

```
GKE/
├── .github/workflows/deploy-to-gke.yml   # GitHub Actions workflow for CI/CD
├── helm-chart/                           # Helm chart for Kubernetes deployment
│   ├── Chart.yaml                        # Helm chart metadata
│   ├── values.yaml                       # Helm chart default values
│   └── templates/                        # Helm templates (deployment, service, helpers)
├── hello-gke/
│   ├── app.py                            # Flask web app ("Hello from GKE!")
│   ├── Dockerfile                        # Docker build instructions
│   └── requirements.txt                  # Python dependencies
└── README.md                             # Project documentation
```

## Application

- **Flask App**: A minimal Python web server that responds with "Hello from GKE!".
- **Docker**: The app is containerized using a Dockerfile based on Ubuntu and Python 3.
- **Kubernetes**: The app is deployed to GKE using Helm as a Deployment with two replicas and exposed via a LoadBalancer Service.

## Deployment Workflow

- **GitHub Actions**: Automates deployment to GKE using Helm.
	- Authenticates to Google Cloud using a service account.
	- (Optional) Builds and pushes the Docker image to Google Artifact Registry (steps are commented out by default).
	- Deploys the app to GKE using Helm, pulling the image from Artifact Registry.
	- You can trigger the workflow manually from the Actions tab to deploy a specific image tag without building/pushing.

## Key Files

- `hello-gke/app.py`: Flask app code.
- `hello-gke/Dockerfile`: Container build instructions.
- `hello-gke/requirements.txt`: Python dependencies (`flask`).
- `helm-chart/`: Directory containing the Helm chart for deployment.
- `.github/workflows/deploy-to-gke.yml`: CI/CD pipeline.

## How It Works

1. **Push to GitHub** or **Run workflow manually**: Triggers the workflow.
2. **(Optional) Build & Push Image**: If enabled, Docker image is built and pushed to Artifact Registry.
3. **Deploy to GKE with Helm**: Helm deploys the app using the specified image tag from Artifact Registry.

## Accessing the App

After deployment, the app is available at the external IP of the LoadBalancer service.  
Use `kubectl get services` to find the IP.

## Helm-based Deployment

This project now uses Helm for all Kubernetes deployments. The workflow is set up to only deploy with Helm by default. If you want to build and push a new Docker image, uncomment the relevant steps in `.github/workflows/deploy-to-gke.yml`.

To deploy a specific image tag, trigger the workflow manually from the GitHub Actions tab and set the appropriate GitHub Actions variables as needed.


## Requirements

- Google Cloud project with GKE and Artifact Registry enabled.
- Service account with permissions for GKE and Artifact Registry.
- GitHub repository secrets for authentication.
- **GitHub Actions Variables**: The workflow and Helm deployment reference several variables that must be set in your repository's GitHub Actions variables (Settings > Variables > Actions):
	- `GKE_CLUSTER`: Name of your GKE cluster (e.g., `hello-autopilot`)
	- `DEPLOYMENT_NAME`: Name of your Kubernetes deployment (e.g., `gke-test`)
	- `REPOSITORY`: Artifact Registry repository name (e.g., `hello-gke-repo`)
	- `IMAGE`: Docker image name (e.g., `hello-gke`)
	- `IMAGE_REPOSITORY`: Full Artifact Registry image path (e.g., `us-central1-docker.pkg.dev/YOUR_PROJECT/YOUR_REPO/YOUR_IMAGE`)
	- `IMAGE_TAG`: Tag of the image to deploy (e.g., `latest` or a specific SHA)

You can update these variables in the GitHub UI without changing the workflow file. The workflow will echo these values at runtime for debugging.
