# Example Projects

This directory contains two minimal demonstration services that illustrate how to use the **cloud-native-cicd-template** in a real-world setup.

The examples are intentionally generic and do not include any real project names, domains, credentials, or infrastructure-specific values.

---

## 📁 Contents

```text
example/
  backend-service/
  frontend-service/
```

### backend-service

A minimal backend service example including:

- Dockerfile
- Helm values files
- Lightweight workflow wrappers:
    - deploy
    - rollback
    - uninstall

The deploy workflow demonstrates `container_network` and `container_volumes` usage for SSH deployment mode.

### frontend-service

A minimal frontend service example including:

- Dockerfile
- Simple static build flow
- Helm values files
- Lightweight workflow wrappers:
    - deploy
    - rollback
    - uninstall

### PostgreSQL Integration Example

Both backend and frontend services can be extended to include PostgreSQL integration. The `helm/web-service/templates` directory contains PostgreSQL templates that can be enabled by updating the `values.yaml` file in the `deploy/` directory of each service.

---

## 🎯 Purpose

The goal of these example services is to demonstrate:

- How to connect a service repository to reusable workflows
- How to structure project-specific Helm values
- How to separate:
    - Template logic
    - Service-specific configuration
- **How to enable PostgreSQL for stateful applications**

These examples are **not production-ready applications**.  
They serve as **reference implementations** for integration with the CI/CD template.

---

## ⚙️ How the Example Works

Each example service contains:

### 1. Application Code

A minimal service implementation along with a Dockerfile.

### 2. Deployment Configuration

A `deploy/` directory with layered Helm values:

- `values.yaml` — base configuration
- `values-dev.yaml` — development overrides
- `values-prod.yaml` — production overrides

### 3. Lightweight Workflows

A `.github/workflows/` directory that calls reusable workflows from the template repository.

### 4. Optional PostgreSQL Configuration

To enable PostgreSQL, update the `values.yaml` file as follows:

```yaml
postgresql:
  enabled: true
  username: your-username
  password: your-password
  database: your-database
```

---

## 🔗 Expected Integration Model

### Template Repository

Contains:

- Reusable GitHub Actions workflows
- Universal Helm chart

### Service Repository

Contains:

- Application source code
- Dockerfile
- Project-specific values files
- Minimal workflow wrappers

---

## 🔧 Usage

### Connect Reusable Workflow

```yaml
jobs:
  deploy:
    uses: RADOSTb24/cloud-native-cicd-template/.github/workflows/deploy-web-service.yml@v1
```

---

## ⚙️ Main Parameters

| Parameter | Description |
|----------|------------|
| component_name | Service name |
| deploy_mode | ssh / kubernetes |
| image_name | Docker image |
| release_name | Helm release |
| namespace_prod | Production namespace |
| namespace_dev | Development namespace |
| values_files | Helm values files |
| build_args | Docker build arguments |
| security_enabled | Enable security scanning stage |
| security_fail_on_findings | Fail pipeline on findings (blocks deploy) |
| security_tools | Enabled tools (e.g., `trivy,osv`) |
| trivy_version | Trivy version (pinned) |
| trivy_severity | Severities to report |
| trivy_ignore_unfixed | Ignore unfixed vulnerabilities |
| container_network | Docker network name for SSH deploy (e.g., `shared-net`) |
| container_volumes | Volume mounts for SSH deploy (e.g., `-v /host:/container`) |

---

## 🔐 Required Secrets

### Container Registry

- GHCR_USERNAME
- GHCR_PAT

### Kubernetes Access

- CLOUDFLEET_ACCESS_TOKEN_ID
- CLOUDFLEET_ACCESS_TOKEN_SECRET
- CLOUDFLEET_ORGANIZATION_ID
- CLOUDFLEET_CLUSTER_ID

### Frontend Build-Time Secrets (optional)

Secrets required for building the frontend can now be passed directly through `BUILD_ARGS`. This simplifies the process and ensures that sensitive data is handled securely during the build process.

#### Example Usage

To pass secrets as build arguments, you can define them in your CI/CD pipeline or directly in your build command. Below is an example of how to configure `BUILD_ARGS` in a YAML file:

```yaml
build_args:
  NEXT_PUBLIC_GOOGLE_CLIENT_ID: your-client-id
  NEXT_PUBLIC_GOOGLE_MAPS_API_KEY: your-api-key
```

These arguments will be passed to the Docker build process and mapped to environment variables inside the container. Ensure that the `Dockerfile` includes the following instructions to handle these arguments:

```dockerfile
ARG NEXT_PUBLIC_GOOGLE_CLIENT_ID="1234567890.apps.googleusercontent.com"
ARG NEXT_PUBLIC_GOOGLE_MAPS_API_KEY="AIzaSyD-EXAMPLEKEY"
ENV NEXT_PUBLIC_GOOGLE_CLIENT_ID=$NEXT_PUBLIC_GOOGLE_CLIENT_ID
ENV NEXT_PUBLIC_GOOGLE_MAPS_API_KEY=$NEXT_PUBLIC_GOOGLE_MAPS_API_KEY
```

This setup ensures that the secrets are available as environment variables during runtime.

---

## 🔄 Deployment Modes

### Kubernetes

- Helm-based deployment
- Namespace separation
- Rollout verification

### SSH

- Docker-based deployment
- Container restart strategy
- Optional network attachment via `container_network`
- Optional volume mounts via `container_volumes`

```yaml
with:
  deploy_mode: ssh
  container_network: shared-net
  container_volumes: "-v /var/log/my-service:/logs"
```

---

## 🔁 Rollback

### Kubernetes

```bash
helm rollback <release>
```

### SSH

- Redeploy container using a stable tag

---

## ❌ Uninstall

### Kubernetes

```bash
helm uninstall <release>
```

### SSH

```bash
docker rm -f <container>
```

---

## ⚠️ Required Adjustments Before Real Usage

Before using this template, replace:

- Image names
- Namespaces
- Hostnames
- API URLs
- Secrets

You must also configure the required GitHub secrets in your repository.

---

## 🧩 Configuration Strategy

### Backend

- Runtime configuration via Helm values

### Frontend

- Build-time configuration via Docker build arguments

---

## 🧠 DevOps Standardization Approach

This project introduces:

- Reusable CI/CD pipelines
- Unified deployment logic
- Clear separation of concerns:
    - Template → pipeline logic
    - Project → configuration

---

## 🔧 How to Adapt for Your Project

1. Replace the example application code with your own
2. Keep workflow wrappers and update:
    - Image names
    - Release names
    - Namespaces
    - Values file paths
3. Adjust Helm values files
4. Configure repository secrets
5. Run pipeline on `develop` and `main`

---

## 📌 Benefits

- Eliminates CI/CD duplication
- Simplifies onboarding
- Improves release reliability
- Enables scalable DevOps practices

---

## 📚 Recommended Usage

Use this directory as a **reference implementation** when onboarding a new project to the template-based DevOps infrastructure.
