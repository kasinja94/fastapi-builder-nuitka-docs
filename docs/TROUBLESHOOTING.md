# Troubleshooting

---

## Installation Issues

### `fastapi-builder: command not found`

The package is not installed or the virtual environment is not active.

```bash
source .venv/bin/activate
pip install fastapi-builder-nuitka
fastapi-builder --help
```

If upgrading:

```bash
pip install fastapi-builder-nuitka
```

---

### `ModuleNotFoundError: No module named 'fastapi_builder'`

The package was not installed in the current Python environment.

```bash
pip install fastapi-builder-nuitka
```

---

## Configuration Issues

### `Configuration file not found`

No `fastapi-builder.yaml` exists in the project directory.

Create one (see [CONFIGURATION.md](CONFIGURATION.md)) or pass the correct `--project` path:

```bash
fastapi-builder config --project /path/to/your-project
```

---

### `Configuration validation failed`

The `fastapi-builder.yaml` file has invalid fields or values.

Common causes:

- `entrypoint` is missing (required field).
- `adapter` value is not `docker`, `systemd`, or `kubernetes`.
- `profile` value is not one of: `standard`, `development`, `low-memory`, `release`, `release-low-memory`.
- `port` is out of range (must be 1–65535).

Run `fastapi-builder config --project <path>` to see the validation error.

---

### Adapter mismatch error

The `--adapter` passed to a deploy command does not match the `deployment.adapter` in `fastapi-builder.yaml`.

Fix: either update the YAML or pass the matching adapter:

```bash
fastapi-builder deploy plan --project <path> --adapter docker
```

---

## Build Issues

### Build fails with `swap exhausted` or `advisory_jobs=1`

This is expected behavior, not an error. When `swap_available_gb < 0.05`, the swap-exhausted guard forces `--jobs=1` to prevent out-of-memory conditions during LTO.

The build will proceed with a single worker. If you need multiple workers, ensure swap space is available.

---

### `Build lock is held by another process`

Another `fastapi-builder build nuitka` or `build onefile` command is running against the same project.

If no build is running, the lock may be stale. The lock is auto-recovered if the PID no longer exists. Check:

```bash
fastapi-builder build lock-status --project <path>
```

If the lock is stale (PID does not match a running process), it will be removed automatically on the next build attempt.

---

### `Nuitka not found` or `nuitka: command not found`

Nuitka is a Python package that installs a command-line entry point. Ensure it is installed in the same environment:

```bash
pip install "nuitka>=4.3.5"
```

If using a virtual environment, confirm the environment is active.

---

### Nuitka build fails with `linker error` or `undefined reference`

This is a Nuitka or compiler issue, not a FastAPI Builder issue. Common causes:

- Missing system libraries required by your FastAPI application.
- Incompatible Nuitka and Python version combination.
- Insufficient disk space in the build output directory.

Check the full Nuitka output. The build log is printed to stdout during the build.

---

### Build fails on Python 3.15 with Nuitka 4.3.5

Nuitka 4.3.5 has partial Python 3.15 support. Some Python 3.15 standard library modules may not compile correctly. This is a Nuitka upstream limitation.

If you encounter this:

- Try the `low-memory` or `release-low-memory` profile.
- Try with Python 3.13 or earlier.
- Check Nuitka release notes for 3.15 support status.

---

## Resource Monitoring Issues

### `ram_available_gb: null` / `swap_available_gb: null`

Resource monitoring requires Linux `/proc`. On macOS or Windows these values are `null`.

The build engine falls back to `advisory_jobs = 1` when resource info is unavailable. Builds proceed normally.

---

### `active_compiler_pids` is always empty

Compiler PID detection scans `/proc/<pid>/cmdline` for gcc, g++, cc1, cc1plus, clang, clang++, collect2, lto1, and lto-wrapper. If no compiler is currently running, the list is empty.

This is correct behavior when sampled between compilations.

---

## Security Scan Issues

### Scanner reported as `unavailable`

FastAPI Builder reports scanners that are not installed as unavailable rather than failing. Install the missing tool:

| Scanner | Installation |
|---|---|
| Trivy | See [trivy.dev](https://trivy.dev) |
| Syft | See [github.com/anchore/syft](https://github.com/anchore/syft) |
| pip-audit | `pip install pip-audit` |
| bandit | `pip install bandit` |
| detect-secrets | `pip install detect-secrets` |
| cyclonedx-bom | `pip install cyclonedx-bom` |

---

### SBOM generation fails

Ensure `cyclonedx-bom` is installed:

```bash
pip install cyclonedx-bom
```

---

## Deployment Issues

### `Docker image reference is missing`

The `deploy preview` command requires a build manifest containing an image reference. Build the Docker image first with `build docker`, then use the resulting build ID.

---

### Deployment status is `planned` but nothing was deployed

This is expected. All deployment adapters are dry-run only. No containers, services, or workloads are started. See [deployment.md](deployment.md) for the current status.

---

## FAPKG Issues

### `Rejected: path traversal`

The FAPKG archive contains a member with an unsafe path (e.g. `../../etc/passwd`). Do not unpack archives from untrusted sources.

---

### Checksum verification failed

The archive contents do not match the checksums recorded at pack time. The archive may have been corrupted or tampered with. Do not use it.

---

### `fapkg verify` passes but `fapkg unpack` fails

Verify only checks archive integrity. Unpack additionally applies path safety checks and may refuse individual members. If this happens with a legitimately-produced archive, open a bug report.

---

## Reporting a Bug

If none of the above resolves your issue, open an issue at [github.com/kasinja94/fastapi-builder-nuitka-docs/issues](https://github.com/kasinja94/fastapi-builder-nuitka-docs/issues) with:

- Python version (`python3 --version`).
- fastapi-builder-nuitka version (`pip show fastapi-builder-nuitka`).
- Nuitka version if applicable (`nuitka --version`).
- Operating system and kernel version.
- Exact command and complete output.
- Steps to reproduce.
