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
3. Push to container registry
4. Deploy:
    - Kubernetes (Helm)
    - or SSH (Docker run)
5. Verify deployment
6. Optional rollback / uninstall

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
