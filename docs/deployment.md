# FastAPI Builder — Deployment Guide

## 1. Overview

The Deployment Engine prepares FastAPI projects for deployment through configurable adapters.

Its purpose is to validate deployment requests, create deployment plans, execute adapter previews, and maintain deployment history.

**Current status:** Deployment execution is dry-run only.

No containers, system services, or Kubernetes workloads are started by the current deployment adapters.

## 2. Supported Adapters

### Docker

Intended for deploying applications as Docker containers.

Current capabilities:

- Validate the Docker deployment request.
- Check build image metadata.
- Prepare a deployment preview.

Actual container creation and startup are not implemented.

### Systemd

Intended for managing FastAPI applications as Linux services.

Current capabilities:

- Validate the project path.
- Prepare a systemd service deployment preview.
- Generate the planned service name.

Service file installation and service startup are not implemented.

### Kubernetes

Intended for deploying FastAPI applications into Kubernetes clusters.

Current capabilities:

- Validate deployment requests.
- Check image metadata.
- Prepare deployment information such as namespace and deployment name.

Applying Kubernetes manifests to a cluster is not implemented.

## 3. Configuration

Select the deployment adapter in `fastapi-builder.yaml`:

```yaml
deployment:
  adapter: docker
```

Supported values:

- `docker`
- `systemd`
- `kubernetes`

The adapter specified in a CLI request must match the project's configured adapter.

## 4. Creating a Deployment Plan

Run:

```bash
fastapi-builder deploy plan \
  --project examples/inventory-api \
  --adapter docker
```

This command validates the project and creates a deployment plan.

The result includes:

- Deployment ID.
- Project name.
- Adapter.
- Environment.
- Deployment status.
- Target information.
- Port.
- Execution state.

A successful plan does not mean the application has been deployed.

## 5. Deployment Preview

Run:

```bash
fastapi-builder deploy preview \
  --project examples/inventory-api \
  --adapter docker
```

The preview invokes the selected adapter in dry-run mode.

A valid build manifest with image metadata may be required for Docker and Kubernetes previews.

Preview results are stored in deployment history.

## 6. Deployment History

Display deployment history:

```bash
fastapi-builder deploy history \
  --project examples/inventory-api
```

Deployment records are stored in:

```text
artifacts/deployments/
```

Each deployment record is stored as a JSON file identified by its deployment ID.

## 7. Deployment Environments

The Deployment Engine defines three environments:

- `local`
- `staging`
- `production`

Remote deployment planning requires a target host.

Production execution is currently blocked.

The environment model is available, but full remote deployment orchestration is not implemented.

## 8. Deployment Lifecycle

The intended deployment lifecycle is:

1. Validate the project.
2. Validate the deployment configuration.
3. Load build metadata.
4. Create a deployment plan.
5. Select the appropriate adapter.
6. Run the adapter preview.
7. Record deployment history.
8. In future versions, execute deployment and verify application health.

## 9. Current Limitations

The following capabilities remain future work:

- Real Docker container deployment.
- Systemd service installation and management.
- Kubernetes manifest generation and application.
- Remote staging deployment.
- Production deployment.
- Automated health checks.
- Deployment status polling.
- Rollback execution.

Do not interpret a `planned` status as a successfully deployed application.

## 10. Safety

The current Deployment Engine is designed to prepare and preview deployments without changing running infrastructure.

Real deployment execution must be implemented with explicit safeguards, health verification, and rollback handling before production use.
