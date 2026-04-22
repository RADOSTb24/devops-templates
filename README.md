# Cloud-Native CI/CD Template

A universal and parameterized CI/CD pipeline for web services (frontend + backend) using GitHub Actions, Docker, Kubernetes, and Helm.

---

## 🚀 Overview

This project provides a reusable DevOps template for automating the full release lifecycle of web services, including:

- Build and test
- Containerization (Docker)
- Deployment to Kubernetes (Helm)
- Deployment via SSH (Docker)
- Rollback and uninstall scenarios
- **Optional PostgreSQL integration**

The main goal is to **standardize the release process** and eliminate duplication across multiple projects.

---

## 🧠 Key Features

- Reusable GitHub Actions workflows
- Parameterized CI/CD pipeline
- Kubernetes deployment via Helm
- [Cluster management via Cloudfleet](https://cloudfleet.ai/docs/introduction/getting-started/)
- SSH deployment for VM environments
- Docker-based build pipeline
- Rollback support (Helm + Docker)
- Environment-based configuration (dev/prod)
- Multi-layered Helm values support
- **PostgreSQL support for stateful applications**

---

## 🏗 Architecture

### Template repository (`cloud-native-cicd-template`)

Contains:
- reusable GitHub Actions workflows
- universal Helm chart
- **PostgreSQL Helm templates**

### Service repositories

Contain:
- application source code
- Dockerfile
- project-specific Helm values
- lightweight workflow wrappers

---

## 🔄 CI/CD Flow

1. Code push (`main` / `develop`)
2. Build Docker image
3. Run optional security scan (non-blocking by default)
4. Push to container registry
5. Deploy:
    - Kubernetes (Helm)
    - or SSH (Docker run)
6. Verify deployment
7. Optional rollback / uninstall

---

## 🔐 Security scanning (DevSecOps)

This template can run a **security scanning stage** before deployment.

- **Default behavior:** scans run, but the pipeline **does not fail** (useful for gradual adoption).
- To turn scans into a gate, set `security_fail_on_findings: true` in your service workflow wrapper.

### Supported scanners

The implementation is **stack-agnostic** (works for Node.js, Java, Go, Python, etc.) because it scans:

- the repository filesystem (dependencies + IaC/Kubernetes misconfigurations + secret patterns)
- the built container image

By default the template uses **Trivy**. You can optionally enable **OSV-Scanner**.

### Security inputs

These parameters are inputs of `.github/workflows/deploy-web-service.yml`:

- `security_enabled` (bool, default `true`) — enable/disable the security stage
- `security_fail_on_findings` (bool, default `false`) — fail pipeline and block deploy on findings
- `security_tools` (string, default `trivy`) — comma-separated list: `trivy`, `osv`
- `trivy_version` (string, default `v0.69.3`) — pinned Trivy version
- `trivy_severity` (string, default `CRITICAL,HIGH`) — severities to report
- `trivy_ignore_unfixed` (bool, default `true`) — ignore unfixed vulnerabilities

> Note: Trivy tooling had a supply-chain incident in March 2026. Use pinned versions and prefer immutable references in production pipelines.

### Example (non-blocking)

```yaml
with:
  security_enabled: true
  security_fail_on_findings: false
  security_tools: trivy,osv
```

### Example (enforced gate)

```yaml
with:
  security_enabled: true
  security_fail_on_findings: true
  security_tools: trivy
  trivy_severity: CRITICAL,HIGH
```

---

## 🛠 Enabling PostgreSQL

To include PostgreSQL in your project:

1. Update your `values.yaml` file to enable PostgreSQL:

    ```yaml
    postgresql:
      enabled: true
      username: your-username
      password: your-password
      database: your-database
    ```

2. Ensure the `postgresql` templates are included in your Helm chart:

    ```yaml
    # Example: templates/postgresql-statefulset.yaml
    apiVersion: apps/v1
    kind: StatefulSet
    metadata:
      name: postgresql
    spec:
      ...
    ```

3. Deploy your application with the updated Helm chart:

    ```bash
    helm upgrade --install your-release-name ./helm/web-service -f values.yaml
    ```

4. Verify that the PostgreSQL StatefulSet and Service are running in your Kubernetes cluster.

### Persistent volumes for PostgreSQL

You can set persistent volumes with Cloudfleet on Hetzner or with Longhorn on any Kubernetes cluster to ensure data durability for your PostgreSQL database.
Here is an [istruction](https://cloudfleet.ai/tutorials/cloud/use-persistent-volumes-with-cloudfleet-on-hetzner/) of how to set up a persistent volume claim:

---

## 🔧 Using BUILD_ARGS During Build

When building Docker images for the frontend, you can use the `BUILD_ARGS` environment variable to pass custom parameters. This allows you to flexibly configure the build process according to your requirements.

### Example Usage

1. Add the required build arguments to the `BUILD_ARGS` variable in your CI/CD pipeline or locally:

    ```bash
    export BUILD_ARGS="API_URL=https://api.example.com NODE_ENV=production"
    ```

2. Ensure your `Dockerfile` supports these arguments. For example:

    ```dockerfile
    ARG API_URL
    ARG NODE_ENV

    ENV REACT_APP_API_URL=$API_URL
    ENV NODE_ENV=$NODE_ENV

    RUN npm run build
    ```

3. During the image build process, the arguments will be passed to the container:

    ```bash
    docker build --build-arg API_URL=https://api.example.com --build-arg NODE_ENV=production -t frontend:latest .
    ```

4. In your CI/CD pipeline, use secrets or environment variables to pass values to `BUILD_ARGS`:

    ```yaml
    build-args: |
      ${{ secrets.BUILD_ARGS }}
    ```

---

## 📁 Repository Structure

```text
.github/workflows/
  deploy-web-service.yml
  rollback-web-service.yml
  uninstall-web-service.yml

helm/web-service/
  Chart.yaml
  values.yaml
  templates/
    _helpers.tpl
    configmap.yaml
    deployment.yaml
    hpa.yaml
    ingress.yaml
    secret.yaml
    service.yaml
    postgresql-configmap.yaml
    postgresql-secret.yaml
    postgresql-service.yaml
    postgresql-statefulset.yaml

example/
  backend-service/
  frontend-service/
```
