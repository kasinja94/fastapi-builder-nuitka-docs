# FastAPI Builder — Usage Guide

## 1. Requirements

Before using FastAPI Builder, ensure you have:

- Python 3.11 or later.
- pip.
- Docker Engine (for Docker builds).
- Git (recommended).

## 2. Installation

Create a virtual environment:

```bash
python3 -m venv .venv
source .venv/bin/activate
```

Install from PyPI:

```bash
pip install fastapi-builder-nuitka
```

Verify installation:

```bash
fastapi-builder --help
```

## 3. Project Configuration

Create a `fastapi-builder.yaml` file in your FastAPI project directory.

Configure:

- Project name and version.
- Python runtime.
- Application entrypoint.
- Build options.
- Security options.
- Deployment adapter.

See `architecture.md` for a complete configuration example.

## 4. Build Commands

### Create a build plan

```bash
fastapi-builder build plan --project examples/inventory-api
```

### Build a Docker image

```bash
fastapi-builder build docker --project examples/inventory-api
```

This builds a Docker image. It does not automatically start a container.

### Build with Nuitka

```bash
fastapi-builder build nuitka --project examples/inventory-api
```

### View build history

```bash
fastapi-builder build history --project examples/inventory-api
```

### Verify a build

```bash
fastapi-builder build verify --help
```

Use the options displayed by this command to verify a build manifest.

## 5. Security

Security settings are configured in `fastapi-builder.yaml`.

Available settings include:

- Dependency auditing.
- Vulnerability scanning.
- SBOM generation.

Security scanning requires the relevant tools and artifacts to be available.

## 6. Deployment Commands

### Create a deployment plan

```bash
fastapi-builder deploy plan \
  --project examples/inventory-api \
  --adapter docker
```

### Preview deployment

```bash
fastapi-builder deploy preview \
  --project examples/inventory-api \
  --adapter docker
```

The preview runs in dry-run mode and records its result in deployment history.

Docker and Kubernetes previews may require a valid build manifest containing image metadata.

### View deployment history

```bash
fastapi-builder deploy history \
  --project examples/inventory-api
```

## 7. Supported Deployment Adapters

| Adapter | Current capability |
|---|---|
| Docker | Dry-run planning |
| Systemd | Dry-run planning |
| Kubernetes | Dry-run planning |

Actual deployment execution is not yet enabled.

The selected adapter must match the adapter configured in the project configuration.

## 8. Troubleshooting

### Command not found

Activate the virtual environment and reinstall the package:

```bash
source .venv/bin/activate
pip install fastapi-builder-nuitka
```

### Adapter mismatch

Ensure the adapter passed to the CLI matches:

```yaml
deployment:
  adapter: docker
```

### Missing Docker image metadata

Build the Docker image and use the corresponding build ID when creating a deployment preview.

### Deployment is only planned

This is expected. Current deployment adapters operate in dry-run mode.

## 10. Safety Notes

- Build commands can create local artifacts and Docker images.
- Deployment planning does not start containers or services.
- Do not use production deployment options until real execution and rollback capabilities have been implemented and tested.
