# FastAPI Builder

FastAPI Builder compiles [FastAPI](https://fastapi.tiangolo.com/) applications into native binaries using [Nuitka](https://nuitka.net/) and packages the result into portable, cryptographically verified archives.

---

## What It Does

1. **Compiles** your FastAPI application and its dependencies with Nuitka — the target host does not require Python.
2. **Packages** the binary into a `.fapkg` archive with SHA-256 checksums, project metadata, and optional SBOM.
3. **Audits** dependencies, scans code and secrets, generates SBOMs, and exports security reports.
4. **Plans** Docker, Systemd, and Kubernetes deployments (dry-run).

---

## Get Started

```bash
pip install fastapi-builder-nuitka
fastapi-builder --help
```

Then follow the [Installation](INSTALLATION.md) and [Quickstart](QUICKSTART.md) guides.

---

## Documentation

| Section | Description |
|---|---|
| [Installation](INSTALLATION.md) | System requirements, install from PyPI, optional dependencies |
| [Quickstart](QUICKSTART.md) | End-to-end workflow from config to packaged artifact |
| [Architecture](architecture.md) | Build engine overview and lifecycle diagrams |
| [Configuration](CONFIGURATION.md) | `fastapi-builder.yaml` reference |
| [CLI Reference](CLI_REFERENCE.md) | All commands and options |
| [FAPKG Format](FAPKG_FORMAT.md) | Archive layout, manifest schema, security notes |
| [Platform Support](PLATFORM_SUPPORT.md) | Supported OS, architecture, and tool requirements |
| [Security](security.md) | Security engine, scanning tools, and design notes |
| [Deployment](deployment.md) | Deployment adapters and dry-run planning |
| [Troubleshooting](TROUBLESHOOTING.md) | Common issues and solutions |
| [Roadmap](ROADMAP.md) | Planned features and known limitations |

---

## Local Preview

```bash
pip install -r requirements-docs.txt
mkdocs serve
```

Open [http://127.0.0.1:8000](http://127.0.0.1:8000) in a browser.
