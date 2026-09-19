# Go Web App – GitHub Actions CI + Docker + EKS + Argo CD

A containerized Go web application deployed on **Amazon EKS** using **Docker, Kubernetes, GitHub Actions, Docker Hub, NGINX Ingress, and Argo CD**.

This project demonstrates a complete DevOps/GitOps workflow:

```text
Developer
    │
    ▼
GitHub Repository
    │
    ├──────────────► GitHub Actions
    │                    │
    │                    ├── Checkout
    │                    ├── Setup Go
    │                    ├── Build
    │                    ├── Test
    │                    ├── Docker Build
    │                    └── Docker Push
    │                            │
    │                            ▼
    │                       Docker Hub
    │
    └──────────────► Kubernetes Manifests
                         │
                         ▼
                       Argo CD
                         │
                         ▼
                    Amazon EKS
                         │
                  ┌──────┴──────┐
                  │             │
             Deployment      Service
                  │             │
                  └──────┬──────┘
                         │
                         ▼
                    NGINX Ingress
                         │
                         ▼
                  AWS Load Balancer
                         │
                         ▼
                   Go Web Application
```

---

# 1. Project Overview

The application is a simple Go web application containing:

* Home
* Courses
* About
* Contact

The application listens on:

```text
8080
```

The application is packaged as a Docker image and deployed to Amazon EKS.

GitHub Actions is used for **Continuous Integration (CI)**.

Argo CD is used for **Continuous Delivery (CD) / GitOps**.

---

# 2. DevOps Workflow

The complete workflow implemented in this project is:

```text
Code Change
    │
    ▼
Git Push
    │
    ▼
GitHub
    │
    ▼
GitHub Actions
    │
    ├── Checkout source code
    ├── Setup Go
    ├── Build application
    ├── Run tests
    ├── Build Docker image
    ├── Login to Docker Hub
    └── Push Docker image
             │
             ▼
        Docker Hub
             │
             │
             ▼
       Kubernetes Manifest
       image updated
             │
             ▼
          GitHub
             │
             ▼
          Argo CD
             │
             ▼
        Amazon EKS
             │
             ▼
      Kubernetes Deployment
             │
             ▼
          Go Pod
```

---

# 3. Continuous Integration – GitHub Actions

GitHub Actions was used to automate the CI process.

The CI pipeline performs:

1. Checkout source code
2. Set up Go
3. Build the Go application
4. Run tests
5. Build Docker image
6. Login to Docker Hub
7. Push Docker image

The workflow is stored under:

```text
.github/workflows/
```

Example:

```text
.github/
└── workflows/
    └── ci.yml
```

---

# 4. GitHub Actions Workflow

Example CI workflow:

```yaml
name: Go Web App CI

on:
  push:
    branches:
      - main

  pull_request:
    branches:
      - main

jobs:

  build:

    runs-on: ubuntu-latest

    steps:

      - name: Checkout source code
        uses: actions/checkout@v4

      - name: Setup Go
        uses: actions/setup-go@v5
        with:
          go-version: '1.22'

      - name: Build Go application
        run: go build -v ./...

      - name: Run tests
        run: go test -v ./...

      - name: Login to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}

      - name: Build Docker image
        run: |
          docker build \
            -t balajmk/go-web-app:${{ github.sha }} \
            .

      - name: Push Docker image
        run: |
          docker push \
            balajmk/go-web-app:${{ github.sha }}
```

---

# 5. GitHub Actions Pipeline Stages

## Stage 1 – Checkout

GitHub Actions checks out the repository:

```yaml
- uses: actions/checkout@v4
```

This downloads the source code into the GitHub Actions runner.

---

## Stage 2 – Setup Go

The Go environment is configured:

```yaml
- uses: actions/setup-go@v5
  with:
    go-version: '1.22'
```

---

## Stage 3 – Build

The application is compiled:

```bash
go build -v ./...
```

This ensures the application can successfully compile.

---

## Stage 4 – Test

Tests are executed:

```bash
go test -v ./...
```

If the tests fail, the GitHub Actions workflow stops.

---

## Stage 5 – Docker Build

The Docker image is created:

```bash
docker build -t balajmk/go-web-app:${{ github.sha }} .
```

Using:

```text
${{ github.sha }}
```

creates a unique image tag for each commit.

For example:

```text
balajmk/go-web-app:abc123456789
```

This is preferable to relying only on:

```text
latest
```

because every deployment can be traced back to a specific Git commit.

---

# 6. Docker Hub Authentication

Docker Hub credentials are stored as GitHub repository secrets.

Example secrets:

```text
DOCKERHUB_USERNAME
DOCKERHUB_TOKEN
```

The workflow uses:

```yaml
- name: Login to Docker Hub
  uses: docker/login-action@v3
  with:
    username: ${{ secrets.DOCKERHUB_USERNAME }}
    password: ${{ secrets.DOCKERHUB_TOKEN }}
```

The credentials are **not hard-coded inside the workflow**.

---

# 7. Docker Image Push

After building the image, GitHub Actions pushes it to Docker Hub:

```bash
docker push balajmk/go-web-app:${{ github.sha }}
```

The image is stored in:

```text
balajmk/go-web-app
```

---

# 8. CI/CD Separation

The project follows a separation between **CI** and **CD**.

### CI – GitHub Actions

GitHub Actions handles:

```text
Source Code
     ↓
Build
     ↓
Test
     ↓
Docker Build
     ↓
Docker Push
```

### CD – Argo CD

Argo CD handles:

```text
Git Repository
     ↓
Detect Change
     ↓
Sync
     ↓
Amazon EKS
```

This separation follows the GitOps model.

---

# 9. GitOps Deployment Model

The desired Kubernetes state is maintained in Git.

For example:

```yaml
image: balajmk/go-web-app:<VERSION>
```

Argo CD continuously compares:

```text
Git Repository
      │
      │ Desired State
      ▼
   Argo CD
      │
      │
      ▼
Kubernetes Cluster
      │
      │ Live State
      ▼
   Comparison
```

If the Git configuration differs from the Kubernetes cluster, Argo CD identifies the application as:

```text
OutOfSync
```

After synchronization:

```text
Synced
```

---

# 10. Complete CI/CD Architecture

The final architecture is:

```text
                         Developer
                             │
                             ▼
                       GitHub Repository
                             │
                ┌────────────┴─────────────┐
                │                          │
                ▼                          ▼
         GitHub Actions               Kubernetes
              CI                      Manifests
                │                          │
        ┌───────┼────────┐                 │
        │       │        │                 │
      Build   Test    Docker Build         │
                         │                 │
                         ▼                 │
                    Docker Hub             │
                         │                 │
                         │                 │
                         └────────┐   ┌────┘
                                  │   │
                                  ▼   ▼
                                GitHub
                                  │
                                  ▼
                               Argo CD
                                  │
                                  ▼
                             Amazon EKS
                                  │
                     ┌────────────┼────────────┐
                     │            │            │
                     ▼            ▼            ▼
                Deployment     Service      Ingress
                     │            │            │
                     ▼            │            ▼
                  Go Pod ◄────────┘      AWS Load Balancer
                     │                         │
                     └────────────┬────────────┘
                                  ▼
                           Go Web Application
```

---

# 11. End-to-End Flow

A typical application change now follows this process:

### Step 1

Developer modifies `main.go`.

### Step 2

Commit and push:

```bash
git add .
git commit -m "Update Go web application"
git push origin main
```

### Step 3

GitHub Actions starts automatically.

```text
Git Push
   ↓
GitHub Actions
```

### Step 4

CI performs:

```text
Checkout
   ↓
Setup Go
   ↓
Build
   ↓
Test
   ↓
Docker Build
   ↓
Docker Login
   ↓
Docker Push
```

### Step 5

New Docker image is available in Docker Hub.

```text
balajmk/go-web-app:<commit-sha>
```

### Step 6

The Kubernetes deployment manifest is updated with the new image version.

### Step 7

The manifest change is pushed to GitHub.

### Step 8

Argo CD detects the Git change.

```text
GitHub
   ↓
Argo CD
   ↓
EKS
```

### Step 9

Argo CD synchronizes the Kubernetes Deployment.

### Step 10

Kubernetes performs a rolling update.

```text
Old Pod
   ↓
New Pod
   ↓
Application Updated
```

---

# 12. GitHub Actions Node Version Notice

During the GitHub Actions workflow, a notice was encountered regarding Node.js versions used by GitHub Actions:

```text
Node 20 is being deprecated.
This workflow is running with Node 24 by default.
```

This is related to the Node.js runtime used internally by GitHub Actions actions and is separate from the Go application's runtime.

The workflow should use current versions of official actions such as:

```yaml
actions/checkout@v4
actions/setup-go@v5
docker/login-action@v3
```

Action versions should be reviewed periodically as GitHub updates its runner and action runtimes.

---

# 13. GitHub Actions Secrets

The following credentials should be configured under:

```text
GitHub Repository
→ Settings
→ Secrets and variables
→ Actions
```

Example:

```text
DOCKERHUB_USERNAME
DOCKERHUB_TOKEN
```

Never place credentials directly in:

```text
workflow YAML
Dockerfile
Kubernetes manifests
Git repository
README.md
```

---

# 14. Recommended Repository Structure

A clean structure for the final project is:

```text
go-web-app/
│
├── .github/
│   └── workflows/
│       └── ci.yml
│
├── k8s/
│   ├── deployment.yaml
│   ├── service.yaml
│   └── ingress.yaml
│
├── main.go
├── Dockerfile
├── go.mod
├── go.sum
└── README.md
```

This separates:

```text
Application Code
       │
       ├── main.go
       ├── go.mod
       └── Dockerfile
       
CI
       │
       └── .github/workflows/ci.yml

Kubernetes
       │
       └── k8s/
           
Documentation
       │
       └── README.md
```

---

# 15. Technologies Used

| Technology        | Role                    |
| ----------------- | ----------------------- |
| Go                | Application             |
| Git               | Version control         |
| GitHub            | Source repository       |
| GitHub Actions    | CI                      |
| Docker            | Containerization        |
| Docker Hub        | Container registry      |
| Kubernetes        | Container orchestration |
| Amazon EKS        | Managed Kubernetes      |
| NGINX Ingress     | HTTP routing            |
| AWS Load Balancer | External access         |
| Argo CD           | GitOps CD               |
| kubectl           | Kubernetes CLI          |
| AWS CLI           | AWS management          |

---

# 16. Skills Demonstrated

This project demonstrates practical experience with:

### CI/CD

* GitHub Actions
* Automated builds
* Automated testing
* Docker image creation
* Docker Hub publishing
* Git-based deployment

### Containerization

* Dockerfile
* Multi-stage Docker builds
* Distroless container images
* Image tagging
* Docker Hub

### Kubernetes

* Pods
* Deployments
* Services
* ClusterIP
* Ingress
* Namespaces
* Rolling updates
* Kubernetes troubleshooting

### AWS

* Amazon EKS
* AWS Load Balancer
* AWS CLI
* Kubernetes on AWS
* `ap-south-1` region

### GitOps

* Argo CD
* Desired state
* Live state
* Application synchronization
* Git as the source of truth

---

# 17. Final DevOps Pipeline

The final implementation can be summarized as:

```text
                    CI                         CD
                    │                          │
                    ▼                          ▼

Developer
    │
    ▼
GitHub
    │
    ▼
GitHub Actions
    │
    ├── Build Go
    ├── Test Go
    ├── Build Docker
    └── Push Docker Image
              │
              ▼
         Docker Hub
              │
              │
              ▼
      Kubernetes Manifests
              │
              ▼
           GitHub
              │
              ▼
           Argo CD
              │
              ▼
          Amazon EKS
              │
              ▼
       Kubernetes Deployment
              │
              ▼
          Go Application
              │
              ▼
       NGINX Ingress
              │
              ▼
      AWS Load Balancer
              │
              ▼
           End User
```

---

# 18. Project Outcome

The project evolved from a locally developed Go application into a Kubernetes-based cloud deployment with an automated CI/CD and GitOps workflow.

The final architecture combines:

```text
Go
+
Docker
+
GitHub
+
GitHub Actions
+
Docker Hub
+
Kubernetes
+
Amazon EKS
+
NGINX Ingress
+
Argo CD
```

This provides hands-on experience with a complete modern DevOps deployment lifecycle:

```text
Code
 ↓
Version Control
 ↓
Continuous Integration
 ↓
Container Build
 ↓
Container Registry
 ↓
GitOps
 ↓
Continuous Delivery
 ↓
Kubernetes
 ↓
Cloud Infrastructure
 ↓
Application
```
