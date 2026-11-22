# Quickstart

This guide walks through the most common workflow: discovering a project, checking resources, compiling with Nuitka, packaging as FAPKG, and running a security scan.

---

## Prerequisites

- fastapi-builder-nuitka installed (see [INSTALLATION.md](INSTALLATION.md)).
- A FastAPI project with a `fastapi-builder.yaml` configuration file.
- Nuitka 4.3.5 or later (for native builds).
- Linux (for resource-aware builds).

---

## Step 1 — Create a Configuration File

In your FastAPI project directory, create `fastapi-builder.yaml`:

```yaml
project:
  name: my-api
  version: 1.0.0

runtime:
  python: "3.11"
  entrypoint: app.main:app
  host: "127.0.0.1"
  port: 8000

build:
  nuitka:
    enabled: true
  docker:
    enabled: false

security:
  dependency_audit: true
  vulnerability_scan: true
  code_scan: true
  secret_scan: true
  generate_sbom: true

deployment:
  adapter: docker
```

See [CONFIGURATION.md](CONFIGURATION.md) for all available options.

---

## Step 2 — Discover the Project

```bash
fastapi-builder discover --project /path/to/my-api
```

The output reports detected entrypoints, dependencies, and static directories.

---

## Step 3 — Check System Resources

```bash
fastapi-builder build resources --project /path/to/my-api
```

This prints available RAM, swap, CPU count, disk space, and any active compiler processes. The build engine uses this snapshot automatically before starting a build.

**If swap is 0 GB:** The swap-exhausted guard will force `--jobs=1` during the build. This is intentional and safe.

---

## Step 4 — Create a Build Plan

```bash
fastapi-builder build plan --project /path/to/my-api
```

This validates the configuration and shows what would be built, without executing anything.

---

## Step 5 — Compile with Nuitka

For a standard build:

```bash
fastapi-builder build nuitka --project /path/to/my-api
```

For a conservative LTO build (recommended when RAM is limited):

```bash
fastapi-builder build nuitka \
  --project /path/to/my-api \
  --profile release-low-memory
```

With live resource monitoring:

```bash
fastapi-builder build nuitka \
  --project /path/to/my-api \
  --monitor
```

The build artifact is saved under `artifacts/build/versions/` in the project directory.

**Build profiles:**

| Profile | LTO | Jobs | Use when |
|---|---|---|---|
| `standard` | No | adaptive | General development |
| `development` | No | 2 | Quick iteration |
| `low-memory` | No | 1 | RAM-constrained machine |
| `release` | Yes | adaptive | Production (high RAM) |
| `release-low-memory` | Yes | 1 | Production (RAM-constrained) — recommended |

> **Important:** Multi-worker LTO builds have not been validated. When using `release` profile, monitor resource usage carefully. Using `release-low-memory` is safer until multi-worker LTO is validated.

---

## Step 6 — View Build History

```bash
fastapi-builder build history --project /path/to/my-api
```

---

## Step 7 — Run a Security Scan

```bash
fastapi-builder security scan --project /path/to/my-api
```

Export a report:

```bash
fastapi-builder security scan \
  --project /path/to/my-api \
  --report-format pdf \
  --output security-report.pdf
```

---

## Step 8 — Package as FAPKG

Package the standalone artifact:

```bash
fastapi-builder build fapkg \
  --project /path/to/my-api \
  --artifact artifacts/build/versions/<build-id>/my_api.dist
```

The `.fapkg` file is saved under `artifacts/packages/`.

Verify the package:

```bash
fastapi-builder fapkg verify my_api-1.0.0.fapkg
```

Inspect metadata without extracting:

```bash
fastapi-builder fapkg inspect my_api-1.0.0.fapkg
```

---

## Step 9 — Plan a Deployment

```bash
fastapi-builder deploy plan \
  --project /path/to/my-api \
  --adapter docker
```

This creates a dry-run deployment plan. No containers or services are started.

See [Deployment](deployment.md) for details on deployment adapters and their current status.

---

## What is Not Covered Here

- Actual deployment execution (not yet implemented).
- Multi-worker LTO builds (not validated).
- Windows and macOS builds (resource monitoring not available; compilation untested).

For full CLI reference, see [CLI_REFERENCE.md](CLI_REFERENCE.md).
