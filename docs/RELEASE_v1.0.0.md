# fastapi-builder-nuitka v1.0.0 — Release Notes

**Date:** 2026-11-21  
**Commit:** (see `git tag -v v1.0.0`)  
**Branch:** master  
**Test baseline:** 441 passed, 0 failed, 0 skipped

---

## Release Summary

fastapi-builder-nuitka v1.0.0 is the first production release of the platform.

It delivers a complete, tested, Python-only pipeline for building, packaging, securing, and preparing FastAPI applications for deployment.  The core build pipeline, security scanning, FAPKG portable packaging, and resource management subsystem are all implemented and regression-tested.

The optional Rust resource engine (Phases O–Q) was benchmarked and retired in Phase S before this release. All safety protections are implemented in Python.

---

## Major Features

### Build Engine

- Nuitka standalone and onefile compilation
- Build profiles: `standard`, `development`, `low-memory`, `release`, `release-low-memory`
- Build resource preflight (RAM, swap, CPU, disk, compiler PIDs)
- Adaptive worker selection with swap-exhausted guard
- Cross-process project-scoped build locking
- Live build monitoring via `/proc` polling
- Build history and manifest generation with checksum verification
- Docker image building

### FAPKG Portable Packaging

- Pack Nuitka artifacts into cryptographically verified portable archives
- Inspect, verify, and unpack FAPKG packages
- Path traversal and symlink protection
- Integrity verification before extraction

### Security Engine

- Dependency auditing via `pip-audit`
- Vulnerability scanning via Trivy (optional; scanner availability reported)
- Python code scanning via Bandit
- Secret scanning via `detect-secrets`
- CycloneDX SBOM generation via `cyclonedx-bom`
- Artifact (onefile, FAPKG) security scanning
- Security report export (PDF, DOCX, XLSX, Markdown)

### Resource Management (Python-only)

| Module | Responsibility |
|---|---|
| `resources.py` | RAM, swap, CPU, load, disk, compiler PIDs |
| `monitor.py` | Active compiler phase detection (gcc, clang, LTO) |
| `policy.py` | Safe worker count with swap-exhausted guard |
| `lock.py` | Project-scoped build lock |

**Swap-exhausted guard (ported from Phase Q Rust benchmarks):**  
If `swap_available_gb < 0.05`, worker count is hard-capped to 1, regardless of available RAM.  This protects against OOM during LTO builds when no swap headroom remains.

### Deployment Engine

- Deployment planning and dry-run previews
- Docker, Systemd, and Kubernetes adapter interfaces
- Deployment history

---

## Compatibility Matrix

| Component | Status | Notes |
|---|---|---|
| Python 3.15.2 | **Tested** | Primary validated runtime |
| Python 3.11–3.14 | Partial | Code uses `X \| Y` syntax (3.10+); not regression-tested |
| Nuitka 4.3.5 | **Tested** | Required for native builds |
| Linux (Ubuntu 22.04) | **Tested** | Required for `/proc`-based resource monitoring |
| macOS | Not validated | Resource monitoring degrades gracefully; build execution untested |
| Windows | Not validated | Resource monitoring degrades gracefully; build execution untested |
| Docker Engine | Required for Docker builds | Any recent version |

---

## Security Notes

- No `shell=True` subprocess calls in production code.
- FAPKG: path traversal, symlink, and duplicate member rejection enforced.
- Build lock uses PID-verified JSON in `/tmp`; stale locks auto-recovered.
- Trivy and Bandit findings do not guarantee a binary is fully secure.
- Worker policy: 4.0 GB/worker threshold is conservative but **not validated under Nuitka LTO at > 1 worker**.

---

## Installation

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install fastapi-builder-nuitka==1.0.0
fastapi-builder --help
```

---

## CLI Reference (key commands)

```bash
# System resource check
fastapi-builder build resources --project <path>

# Build plan (no execution)
fastapi-builder build plan --project <path>

# Nuitka standalone build (conservative LTO profile)
fastapi-builder build nuitka --project <path> --profile release-low-memory

# Nuitka onefile build
fastapi-builder build onefile --project <path>

# Package into FAPKG
fastapi-builder build fapkg --project <path>

# Inspect / verify / unpack FAPKG
fastapi-builder fapkg inspect <file.fapkg>
fastapi-builder fapkg verify  <file.fapkg>
fastapi-builder fapkg unpack  <file.fapkg> --output <dir>

# Security scan
fastapi-builder security scan --project <path>

# SBOM generation
fastapi-builder security sbom --project <path>

# Deployment plan (dry-run)
fastapi-builder deploy plan --project <path> --adapter docker
```

---

## Known Limitations

1. **Linux-only resource monitoring.** `/proc`-based preflight, adaptive workers, and live monitoring are not available on Windows or macOS. Builds proceed with `advisory_jobs = 1` on non-Linux.

2. **Multi-worker LTO unvalidated.** The 4.0 GB/worker threshold has not been tested under real Nuitka LTO at `--jobs > 1`. All reference builds used `release-low-memory` (`--jobs=1`). Do not use multi-worker builds for LTO until this is validated.

3. **Deployment adapters are dry-run only.** Docker, Systemd, and Kubernetes adapters produce plans and previews. Actual deployment execution, health checks, and rollback are not implemented.

4. **Scanner dependencies are optional.** Trivy, Bandit, detect-secrets, and cyclonedx-bom are runtime dependencies. Missing scanners are reported as unavailable, not as scan errors.

---

## Verification Evidence

```
Tests:     441 passed, 0 failed, 0 skipped (Python 3.15.2)
CLI:       All command groups respond correctly to --help
Resources: Preflight returns valid JSON; swap-exhausted guard fires at swap=0.0
Policy:    select_safe_worker_count(7.25, 0.0, 20, 32) = 1 (guard active)
Config:    NuitkaConfig validates without resource_engine field
FAPKG:     inspect/verify/unpack commands functional
Security:  audit/code-scan/secret-scan commands functional
```

**Note:** Reference builds used `release-low-memory` profile. Native build was not performed at release time due to constrained swap on the reference machine.

---

## Files Changed Since v0.2.0

Phases O, P, Q, Q.1, S and release preparation.  Key additions/changes:

- `src/fastapi_builder/build/policy.py` — swap-exhausted guard, invariant docs
- `src/fastapi_builder/build/monitor.py` — compiler detection contract (gcc, cc1, clang, collect2, lto1, lto-wrapper)
- `src/fastapi_builder/build/resources.py` — compiler detection aligned to contract
- `src/fastapi_builder/build/nuitka.py` — direct resource imports, no Rust dispatcher
- `src/fastapi_builder/config/engine.py` — removed `resource_engine` field
- `tests/test_worker_policy_safety.py` — 56 safety tests (Python-only)
- `tests/test_compiler_detection_contract.py` — 57 contract tests
- `docs/architecture.md` — resource management module table
- `pyproject.toml` — version 1.0.0
- **Deleted:** `src/fastapi_builder/build/resource_engine.py`, `resources_rust.py`, `tests/test_resource_engine.py`, `tests/test_resources_rust.py`, `rust/` crate, `scripts/benchmark_resource_engines.py`
