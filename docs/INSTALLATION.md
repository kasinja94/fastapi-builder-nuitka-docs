# Installation

## System Requirements

| Requirement | Minimum | Notes |
|---|---|---|
| Python | 3.11 | 3.15.2 is the primary validated runtime |
| Operating system | Linux | Resource monitoring requires `/proc`; other platforms degrade gracefully |
| Nuitka | 4.3.5 | Required for native compilation builds only |
| Docker Engine | Any recent version | Required for Docker image builds only |
| pip | Any current version | Included with Python |
| Git | Any | Recommended |

**Platform note:** FastAPI Builder is developed and tested on Linux (Ubuntu, kernel 6.8). Resource preflight, adaptive worker selection, and live build monitoring all require Linux `/proc`. On macOS or Windows these features are skipped and builds proceed with a single worker. See [PLATFORM_SUPPORT.md](PLATFORM_SUPPORT.md) for details.

---

## Install from PyPI

Install with pip:

```bash
pip install fastapi-builder-nuitka
```

Verify the installation:

```bash
fastapi-builder --version
fastapi-builder --help
```

Using a virtual environment is strongly recommended:

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install fastapi-builder-nuitka
```

---

---

## Optional Runtime Dependencies

The following tools are used by specific security commands. FastAPI Builder installs the Python bindings for these automatically; the system-level tools must be installed separately where noted.

| Tool | Used by | Notes |
|---|---|---|
| `pip-audit` | `security audit`, `security vulnerability-scan`, `security sbom` | Installed automatically as a Python dependency |
| `bandit` | `security code-scan`, `security scan` | Installed automatically |
| `detect-secrets` | `security secret-scan`, `security scan` | Installed automatically |
| `cyclonedx-bom` | `security sbom`, `security scan` | Installed automatically |
| Trivy | `security vulnerability-scan`, `security scan`, `security scan-onefile`, `security scan-fapkg` | System binary; install from [trivy.dev](https://trivy.dev) |
| Syft | `security artifact-sbom` | System binary; optional |
| Docker Engine | `build docker`, `security vulnerability-scan` (image scan) | System binary |
| Nuitka | `build nuitka`, `build onefile` | Installed automatically as a Python dependency |

If a scanner is not available, FastAPI Builder reports it as unavailable rather than failing.

---

## Verifying the Installation

```bash
# Check the CLI is accessible
fastapi-builder --help

# Check resource availability (requires Linux)
fastapi-builder build resources --project <path-to-project>

# Check configuration loading
fastapi-builder config --project <path-to-project>
```

---

## Upgrading

```bash
pip install --upgrade fastapi-builder-nuitka
```

---

## Uninstalling

```bash
pip uninstall fastapi-builder-nuitka
```
