# GKE Blue-Green Deployment Example

This project demonstrates a blue-green deployment pattern for a Python Flask application on Google Kubernetes Engine (GKE) using Helm and GitHub Actions. It is designed for **internal-only service exposure** and supports private Cloud DNS for friendly internal endpoints.

---

## Project Structure

```
GKE/
├── .github/workflows/
│   ├── Build-and-Publish-GAR.yml   # Build & publish Docker images to Artifact Registry
│   └── deploy.yml                  # Deploys blue/green workloads to GKE
├── hello-gke/
│   ├── app.py                      # Flask app source code
│   ├── Dockerfile                  # Docker build instructions
│   └── requirements.txt            # Python dependencies
├── helm-chart/
│   └── templates/
│       └── blue-green.yaml         # Helm template for blue/green deployments & services
└── README.md
```

---

## Application Overview

- **Flask App:** Minimal Python web server.
- **Docker:** Containerized using Ubuntu and Python 3.
- **Helm:** Manages Kubernetes deployments and services.
- **Blue-Green Pattern:** Two parallel deployments (`hello-gke-blue` and `hello-gke-green`), each with two replicas.

---

## Blue-Green Architecture (Internal-Only)

```
                +-------------------+         +-------------------+
                |   Internal LB     |         |   Internal LB     |
                | hello-gke-blue    |         | hello-gke-green   |
                +--------+----------+         +--------+----------+
                         |                             |
                +--------v----------+         +--------v----------+
                |  hello-gke-blue   |         |  hello-gke-green  |
                |   Deployment      |         |   Deployment      |
                +--------+----------+         +--------+----------+
                         |                             |
         +---------------+---------------+   +---------+---------+
         |                               |   |                   |
+--------v--------+           +----------v--+        +-----------v--+
| Pod v1.3 (rep1) |           | Pod v1.3    |        | Pod v1.0     |
+-----------------+           | (rep2)      |        | (rep1)       |
                             +-------------+        +--------------+
                                                 | Pod v1.0        |
                                                 | (rep2)          |
                                                 +-----------------+
```

- **Internal LoadBalancer:** Each service is provisioned with `cloud.google.com/load-balancer-type: "Internal"` and `type: LoadBalancer`, so it is only accessible within the VPC.
- **No public endpoints:** Services are not exposed to the internet.

---

## Internal-Only Service Manifest Example

```yaml
apiVersion: v1
kind: Service
metadata:
  name: hello-gke-blue
  annotations:
    cloud.google.com/load-balancer-type: "Internal"
spec:
  type: LoadBalancer
  selector:
    app: hello-gke-blue
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
```
*(Repeat for `hello-gke-green`.)*

---

## Private Cloud DNS Integration

To provide friendly DNS names for your internal endpoints:

1. **Deploy services with internal load balancers** (as above).
2. **Get internal IPs** with:
   ```sh
   kubectl get svc
   ```
3. **Create a private Cloud DNS zone** in GCP (e.g., `internal.example.com`), attached to your VPC.
4. **Add A records** for your services:
   - `blue.internal.example.com` → `<hello-gke-blue internal IP>`
   - `green.internal.example.com` → `<hello-gke-green internal IP>`
5. **Access from within the VPC** using these DNS names.

---

## Deployment Workflow

- **Build-and-Publish-GAR.yml:**  
  Builds and pushes Docker images to Artifact Registry (manual or on push).
- **deploy.yml:**  
  Deploys blue and green workloads to GKE using Helm (manual or on push).

---

## Requirements

- Google Cloud project with GKE (standard or Autopilot) and Artifact Registry enabled.
- Service account with:
  - Kubernetes Engine Admin
  - Artifact Registry Reader/Writer
- GitHub repository secrets for authentication.

### GitHub Actions Variables

Set these in **Settings > Variables > Actions**:
- `GKE_CLUSTER`: Name of your GKE cluster (e.g., `hello-standard`)
- `PROJECT_ID`: GCP Project ID (e.g., `algebraic-inn-256717`)
- `ZONE`: GCP zone (e.g., `us-central1`)
- `REPOSITORY`: Artifact Registry repo (e.g., `hello-gke-repo`)
- `IMAGE`: Docker image name (e.g., `hello-gke`)
- `IMAGE_TAG`: Tag to deploy (e.g., `HelloWorld_1.3`)

### GitHub Actions Secrets

- `GCP_SA_KEY`: JSON key for the GCP service account.

---

## How to Use

1. **Build and Publish:**  
   Trigger the "Build and Publish to GAR" workflow to build and push a new Docker image.
2. **Deploy:**  
   Trigger the "Deploy to GKE" workflow to deploy blue and green environments to your GKE cluster.
3. **Test Internally:**  
   Use the private Cloud DNS names (e.g., `blue.internal.example.com`) from within your VPC to access the services.

---

## Security and Cost Management

- **No public endpoints:** All services are internal-only.
- **DNS is private:** Only accessible within your VPC.
- **To save costs:** Delete unused clusters, scale node pools to zero, and clean up old images in Artifact Registry.

---

## Reference

- [GKE Internal Load Balancers](https://cloud.google.com/kubernetes-engine/docs/how-to/internal-load-balancing)
- [Google Cloud DNS Private Zones](https://cloud.google.com/dns/docs/zones/manage-private-zones)
- [Helm](https://helm.sh/)
- [GitHub Actions](https://docs.github.com/en/actions)

---

*This project is a reference implementation for secure, internal-only blue-green deployments on GKE with private DNS integration.*