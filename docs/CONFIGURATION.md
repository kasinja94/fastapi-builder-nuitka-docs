# Configuration Reference

FastAPI Builder projects are configured with a `fastapi-builder.yaml` file in the project root.

---

## Minimal Configuration

```yaml
project:
  name: my-api
  version: 1.0.0

runtime:
  python: "3.11"
  entrypoint: app.main:app
```

---

## Full Example

```yaml
project:
  name: my-api
  version: 1.0.0

runtime:
  python: "3.15"
  entrypoint: app.main:app
  host: "127.0.0.1"
  port: 8000

build:
  nuitka:
    enabled: true
    onefile: false
    profile: release-low-memory
  docker:
    enabled: true
  fapkg:
    enabled: false
    include_runtime: true
    platforms: []

security:
  dependency_audit: true
  vulnerability_scan: true
  code_scan: true
  secret_scan: true
  generate_sbom: true
  docker_image_scan: true

deployment:
  adapter: docker
```

---

## `project`

| Field | Type | Required | Default | Description |
|---|---|---|---|---|
| `name` | string | Yes | — | Project name |
| `version` | string | No | `0.1.0` | Project version |

---

## `runtime`

| Field | Type | Required | Default | Description |
|---|---|---|---|---|
| `python` | string | Yes | — | Python version string (e.g. `"3.11"`, `"3.15"`) |
| `entrypoint` | string | Yes | — | FastAPI application entrypoint in `module:attribute` form (e.g. `app.main:app`) |
| `host` | string | No | `127.0.0.1` | Bind host |
| `port` | integer | No | `8000` | Bind port (1–65535) |

---

## `build.nuitka`

| Field | Type | Required | Default | Description |
|---|---|---|---|---|
| `enabled` | boolean | No | `true` | Enable Nuitka standalone builds |
| `onefile` | boolean | No | `false` | Use onefile mode |
| `profile` | string | No | `standard` | Build profile (see below) |

### Build Profiles

| Profile | LTO | `--jobs` | `--low-memory` | Recommended for |
|---|---|---|---|---|
| `standard` | No | adaptive | No | General development |
| `development` | No | 2 | No | Quick iteration |
| `low-memory` | No | 1 | Yes | RAM-constrained machines |
| `release` | Yes | adaptive | No | Production builds with ample RAM |
| `release-low-memory` | Yes | 1 | Yes | Production builds, RAM-constrained |

**Adaptive `--jobs`:** The build engine reads available RAM and swap before starting the build and computes a safe worker count using `policy.py`. If swap is below 0.05 GB, `--jobs=1` is forced regardless of available RAM.

> **Note:** Multi-worker LTO builds (`release` profile with more than one job) have not been validated. All reference builds used `release-low-memory`. Do not use `release` profile with high parallelism until multi-worker LTO has been validated on your hardware.

---

## `build.docker`

| Field | Type | Required | Default | Description |
|---|---|---|---|---|
| `enabled` | boolean | No | `true` | Enable Docker image builds |

---

## `build.fapkg`

| Field | Type | Required | Default | Description |
|---|---|---|---|---|
| `enabled` | boolean | No | `false` | Enable FAPKG packaging |
| `include_runtime` | boolean | No | `true` | Include runtime metadata in the package |
| `platforms` | list of strings | No | `[]` | Target platform identifiers |

Supported platform identifiers: `linux-x86_64`, `linux-arm64`, `windows-x86_64`, `windows-arm64`, `macos-x86_64`, `macos-arm64`.

> **Note:** Nuitka cannot cross-compile. A build for `linux-arm64` must run on a Linux ARM64 host.

---

## `security`

| Field | Type | Required | Default | Description |
|---|---|---|---|---|
| `dependency_audit` | boolean | No | `true` | Audit declared dependencies with `pip-audit` |
| `vulnerability_scan` | boolean | No | `true` | Scan for vulnerabilities (Trivy, if available) |
| `code_scan` | boolean | No | `true` | Scan Python source with Bandit |
| `secret_scan` | boolean | No | `true` | Scan for secrets with `detect-secrets` |
| `generate_sbom` | boolean | No | `true` | Generate CycloneDX SBOM |
| `docker_image_scan` | boolean | No | `true` | Scan Docker image with Trivy (requires Docker and Trivy) |

If a scanner is not installed, FastAPI Builder reports it as unavailable rather than failing.

---

## `deployment`

| Field | Type | Required | Default | Description |
|---|---|---|---|---|
| `adapter` | string | No | `docker` | Deployment adapter: `docker`, `systemd`, or `kubernetes` |

> **Note:** Deployment adapters currently support dry-run planning and previews only. Actual deployment execution is not implemented.

The adapter value in the configuration must match the adapter passed to `fastapi-builder deploy` commands.
