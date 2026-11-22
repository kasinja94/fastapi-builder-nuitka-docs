# Roadmap

This document describes known gaps and planned future work for fastapi-builder-nuitka.

Items are grouped by area and priority. This is not a commitment or release schedule.

---

## Deployment Execution (High Priority)

The deployment engine currently supports dry-run planning and previews only.

Planned work:

- **Docker deployment execution:** Create and start containers from built images. Manage container lifecycle (start, stop, remove).
- **Docker health verification:** Poll container health status after deployment. Report readiness.
- **Systemd service installation:** Generate and install systemd unit files. Start and enable services.
- **Kubernetes manifest generation:** Produce Deployment and Service manifests. Apply manifests to a cluster using `kubectl` or the Kubernetes API.
- **Rollback execution:** Track the previous deployment state and restore it on failure.
- **Remote staging deployment:** Execute deployments against remote hosts, not just local environments.

---

## Resource Monitoring

- **Multi-worker LTO validation:** The 4.0 GB/worker threshold in `policy.py` has not been tested under real Nuitka LTO with more than one worker. Validate the threshold with profiled builds and adjust if necessary.
- **macOS support:** Implement resource snapshot collection for macOS using `sysctl` and `psutil` as alternatives to `/proc`.
- **Windows support:** Implement resource snapshot collection for Windows.

---

## Build Engine

- **Build cancellation:** The `--auto-cancel` flag is present but the cancellation logic is opt-in and conservative. Improve the threshold heuristics.
- **Build notifications:** Emit structured events (JSON-lines or similar) during builds for integration with external build dashboards.
- **Incremental builds:** Leverage Nuitka's caching for faster rebuilds when only application code changes.

---

## Security Engine

- **SBOM signing:** Add an optional step to sign the generated CycloneDX SBOM (e.g. with sigstore or GPG).
- **FAPKG signature verification:** Add optional GPG/sigstore signature verification for `.fapkg` files at unpack time.
- **Policy enforcement:** Allow configuring which scanner findings should block a build.

---

## Platform

- **Python 3.14 classifier:** Add Python 3.14 to the classifier list once compatibility is verified.
- **ARM64 Linux testing:** Confirm builds and resource monitoring work on ARM64 Linux.
- **Windows CI:** Add Windows to the CI matrix once Nuitka builds are validated on Windows.
- **macOS CI:** Add macOS to the CI matrix once resource monitoring degrades are validated.

---

## Documentation

- **Example project update:** Expand the `examples/inventory-api` example to include a complete build-to-deploy walkthrough.
- **Multi-worker LTO guide:** Once multi-worker LTO is validated, document the recommended profile and resource thresholds.

---

## Developer Experience

- **`__version__` attribute:** Export the package version from `fastapi_builder.__init__` so `fastapi_builder.__version__` works programmatically.
- **Structured output option:** Add `--output-format json` to commands that currently only print human-readable text.
- **Exit codes:** Document and stabilize exit code conventions across all commands.

---

## Not Planned

The following are explicitly out of scope for this project:

- Becoming a FastAPI application scaffold generator (project creation from templates).
- Supporting non-FastAPI Python web frameworks.
- Reintroducing a Rust resource engine. The Python-only implementation is the permanent architecture.
- Cloud provider integrations (AWS, GCP, Azure) as first-class adapters.
