# CLI Reference

All commands use the `fastapi-builder` entry point.

```
fastapi-builder [OPTIONS] COMMAND [ARGS]...
```

---

## Top-Level Options

| Option | Description |
|---|---|
| `--help` | Show help and exit |

---

## `discover`

Discover and inspect a FastAPI project.

```bash
fastapi-builder discover --project <path>
```

| Option | Short | Type | Required | Default | Description |
|---|---|---|---|---|---|
| `--project` | `-p` | path | No | `.` | Path to the FastAPI project |
| `--help` | | | | | Show help |

**Output:** JSON report including detected entrypoints, Python version, dependencies, static directories, template directories, environment files, dependency files, source files scanned, and warnings.

---

## `config`

Validate and display the project configuration.

```bash
fastapi-builder config --project <path>
```

| Option | Short | Type | Required | Default | Description |
|---|---|---|---|---|---|
| `--project` | `-p` | path | No | `.` | Path to the FastAPI project |
| `--help` | | | | | Show help |

**Output:** JSON representation of the validated `fastapi-builder.yaml` configuration, followed by `Configuration is valid.`

---

## `build` Commands

### `build plan`

Create a build plan without executing the build.

```bash
fastapi-builder build plan --project <path>
```

| Option | Short | Type | Required | Default | Description |
|---|---|---|---|---|---|
| `--project` | `-p` | path | No | `.` | Path to the FastAPI project |
| `--help` | | | | | Show help |

**Output:** JSON build plan with status, project name, version, build targets, and output directory. Does not start a build.

---

### `build nuitka`

Build a standalone application using Nuitka.

```bash
fastapi-builder build nuitka --project <path> [OPTIONS]
```

| Option | Short | Type | Required | Default | Description |
|---|---|---|---|---|---|
| `--project` | `-p` | path | No | `.` | Path to the FastAPI project |
| `--host` | | string | No | (from config) | Override runtime bind host |
| `--port` | | integer | No | (from config) | Override runtime port (1–65535) |
| `--profile` | | string | No | `standard` | Build profile: `standard`, `development`, `low-memory`, `release`, `release-low-memory` |
| `--monitor` | | bool | No | false | Enable live resource monitoring during build |
| `--monitor-interval` | | float | No | `5.0` | Seconds between monitor polls |
| `--auto-cancel` | | bool | No | false | Cancel build automatically on critical resource threshold (opt-in) |
| `--help` | | | | | Show help |

Produces a standalone `.dist/` directory under `artifacts/build/versions/`. Records a build manifest and history entry.

---

### `build onefile`

Build a self-contained onefile executable using Nuitka.

```bash
fastapi-builder build onefile --project <path> [OPTIONS]
```

Same options as `build nuitka`. Produces a single executable file.

---

### `build fapkg`

Package a Nuitka artifact into a portable FAPKG archive.

```bash
fastapi-builder build fapkg \
  --project <path> \
  --artifact <artifact-path> \
  [OPTIONS]
```

| Option | Short | Type | Required | Default | Description |
|---|---|---|---|---|---|
| `--project` | `-p` | path | No | `.` | FastAPI project directory |
| `--artifact` | `-a` | path | Yes | — | Nuitka artifact (`.dist/` directory or onefile binary) |
| `--output` | `-o` | path | No | auto | Destination `.fapkg` path |
| `--build-manifest` | | path | No | | Path to an existing build manifest JSON |
| `--sbom` | | path | No | | Path to a CycloneDX SBOM to embed |
| `--security-report` | | path | No | | Path to a security report JSON to embed |
| `--overwrite` | | bool | No | false | Replace an existing `.fapkg` at the output path |
| `--help` | | | | | Show help |

---

### `build docker`

Build a Docker image for the FastAPI project.

```bash
fastapi-builder build docker --project <path>
```

| Option | Short | Type | Required | Default | Description |
|---|---|---|---|---|---|
| `--project` | `-p` | path | No | `.` | Path to the FastAPI project |
| `--help` | | | | | Show help |

Requires Docker Engine to be running.

---

### `build resources`

Inspect system resources available for a build.

```bash
fastapi-builder build resources --project <path>
```

| Option | Short | Type | Required | Default | Description |
|---|---|---|---|---|---|
| `--project` | `-p` | path | No | `.` | Path to the FastAPI project |
| `--help` | | | | | Show help |

**Output:** JSON snapshot including `ram_total_gb`, `ram_available_gb`, `ram_used_gb`, `swap_total_gb`, `swap_available_gb`, `cpu_count`, `load_avg_1m`, `load_avg_5m`, `active_compiler_pids`, `disk_free_gb`.

Requires Linux `/proc`. Values are `null` on unsupported platforms.

---

### `build lock-status`

Show the build lock status for a project.

```bash
fastapi-builder build lock-status --project <path>
```

| Option | Short | Type | Required | Default | Description |
|---|---|---|---|---|---|
| `--project` | `-p` | path | No | `.` | Path to the FastAPI project |
| `--help` | | | | | Show help |

---

### `build history`

Display the build history for a project.

```bash
fastapi-builder build history --project <path>
```

| Option | Short | Type | Required | Default | Description |
|---|---|---|---|---|---|
| `--project` | `-p` | path | No | `.` | Path to the FastAPI project |
| `--help` | | | | | Show help |

**Output:** JSON list of build manifests from `artifacts/build/versions/`.

---

### `build verify`

Verify a build manifest and artifact integrity.

```bash
fastapi-builder build verify \
  --project <path> \
  --manifest <manifest-path>
```

| Option | Short | Type | Required | Default | Description |
|---|---|---|---|---|---|
| `--project` | `-p` | path | Yes | — | Path to the project directory |
| `--manifest` | `-m` | path | Yes | — | Path to the build manifest JSON |
| `--help` | | | | | Show help |

---

## `fapkg` Commands

### `fapkg inspect`

Read package metadata without extracting the archive.

```bash
fastapi-builder fapkg inspect <file.fapkg>
```

| Argument/Option | Short | Type | Required | Default | Description |
|---|---|---|---|---|---|
| `package` | | path | Yes | — | Path to the `.fapkg` archive (positional) |
| `--help` | | | | | Show help |

---

### `fapkg verify`

Verify the integrity of a FAPKG archive.

```bash
fastapi-builder fapkg verify <file.fapkg>
```

| Argument/Option | Short | Type | Required | Default | Description |
|---|---|---|---|---|---|
| `package` | | path | Yes | — | Path to the `.fapkg` archive (positional) |
| `--help` | | | | | Show help |

Checks archive integrity, checksums, and manifest consistency. Does not extract files.

---

### `fapkg unpack`

Verify and extract a FAPKG archive to a directory.

```bash
fastapi-builder fapkg unpack <package> --output <dir> [OPTIONS]
```

| Argument/Option | Short | Type | Required | Default | Description |
|---|---|---|---|---|---|
| `package` | | path | Yes | — | Path to the `.fapkg` archive (positional) |
| `--output` | `-o` | path | Yes | — | Target directory for extraction |
| `--platform` | | string | No | | Required platform (e.g. `linux-x86_64`) for compatibility check |
| `--overwrite` | | bool | No | false | Overwrite existing files in the output directory |
| `--help` | | | | | Show help |

Verifies integrity before extracting. Rejects path traversal, symlinks, and duplicate archive members.

---

## `security` Commands

### `security audit`

List dependencies declared by the project.

```bash
fastapi-builder security audit --project <path>
```

| Option | Short | Type | Required | Default | Description |
|---|---|---|---|---|---|
| `--project` | `-p` | path | No | `.` | Path to the FastAPI project |
| `--help` | | | | | Show help |

---

### `security vulnerability-scan`

Scan project dependencies for known vulnerabilities.

```bash
fastapi-builder security vulnerability-scan --project <path>
```

| Option | Short | Type | Required | Default | Description |
|---|---|---|---|---|---|
| `--project` | `-p` | path | No | `.` | Path to the FastAPI project |
| `--help` | | | | | Show help |

Requires `pip-audit`. Trivy is used for Docker image scanning if available.

---

### `security code-scan`

Scan Python source code using Bandit.

```bash
fastapi-builder security code-scan --project <path>
```

| Option | Short | Type | Required | Default | Description |
|---|---|---|---|---|---|
| `--project` | `-p` | path | No | `.` | Path to the FastAPI project |
| `--help` | | | | | Show help |

Requires Bandit.

---

### `security secret-scan`

Scan the project for exposed secrets.

```bash
fastapi-builder security secret-scan --project <path>
```

| Option | Short | Type | Required | Default | Description |
|---|---|---|---|---|---|
| `--project` | `-p` | path | No | `.` | Path to the FastAPI project |
| `--help` | | | | | Show help |

Requires `detect-secrets`.

---

### `security scan`

Run all configured security scanners and optionally export a report.

```bash
fastapi-builder security scan --project <path> [OPTIONS]
```

| Option | Short | Type | Required | Default | Description |
|---|---|---|---|---|---|
| `--project` | `-p` | path | No | `.` | Path to the FastAPI project |
| `--report-format` | | string | No | `none` | Output format: `none`, `json`, `html`, `markdown`, `pdf`, `docx`, `xlsx`, or `all` |
| `--output` | `-o` | path | No | | Output file path, or directory when using `all` |
| `--help` | | | | | Show help |

Scanners that are not available are reported as unavailable, not as errors.

---

### `security sbom`

Generate a CycloneDX SBOM for the project.

```bash
fastapi-builder security sbom --project <path>
```

| Option | Short | Type | Required | Default | Description |
|---|---|---|---|---|---|
| `--project` | `-p` | path | No | `.` | Path to the FastAPI project |
| `--help` | | | | | Show help |

Requires `cyclonedx-bom`.

---

### `security scan-onefile`

Scan a Nuitka onefile or standalone artifact for vulnerabilities.

```bash
fastapi-builder security scan-onefile <artifact>
```

| Argument/Option | Type | Required | Description |
|---|---|---|---|
| `artifact` | path | Yes | Nuitka artifact — a onefile binary or standalone `.dist/` directory (positional) |
| `--help` | | | Show help |

Requires Trivy. For compiled onefile binaries, static scanning cannot enumerate all bundled Python packages; this limitation is noted in the output.

---

### `security scan-fapkg`

Verify and scan a FAPKG archive for vulnerabilities.

```bash
fastapi-builder security scan-fapkg <file.fapkg>
```

| Argument/Option | Type | Required | Description |
|---|---|---|---|
| `package` | path | Yes | Path to the `.fapkg` archive (positional) |
| `--help` | | | Show help |

---

### `security artifact-sbom`

Generate a CycloneDX SBOM for a bundled artifact.

```bash
fastapi-builder security artifact-sbom <artifact> [OPTIONS]
```

| Argument/Option | Short | Type | Required | Default | Description |
|---|---|---|---|---|---|
| `artifact` | | path | Yes | — | Nuitka artifact (positional) |
| `--project` | `-p` | path | No | | Project directory (reads name and version from config) |
| `--name` | `-n` | string | No | `unknown` | Project name |
| `--version` | `-V` | string | No | `0.0.0` | Project version |
| `--requirements` | `-r` | path | No | | Path to `requirements.txt` for pip-audit enumeration |
| `--output` | `-o` | path | No | auto | Destination CycloneDX JSON file |
| `--help` | | | | | Show help |

Requires at least one detection tool (pip-audit or Syft).

---

## `deploy` Commands

> **Note:** All deployment commands are dry-run only. No containers, services, or Kubernetes workloads are started.

### `deploy plan`

Create a deployment plan without executing it.

```bash
fastapi-builder deploy plan \
  --project <path> \
  --adapter <adapter> \
  [OPTIONS]
```

| Option | Short | Type | Required | Default | Description |
|---|---|---|---|---|---|
| `--project` | `-p` | path | Yes | — | Path to the FastAPI project |
| `--adapter` | | string | Yes | — | Deployment adapter: `docker`, `systemd`, or `kubernetes` |
| `--environment` | | string | No | `local` | Deployment environment |
| `--build-id` | | string | No | | Specific build ID to deploy |
| `--target-host` | | string | No | | Remote target host |
| `--port` | | integer | No | | Override port |
| `--help` | | | | | Show help |

The `--adapter` value must match the adapter in `fastapi-builder.yaml`.

---

### `deploy preview`

Preview a deployment and save its result to history.

```bash
fastapi-builder deploy preview \
  --project <path> \
  --adapter <adapter> \
  [OPTIONS]
```

Same options as `deploy plan`. Invokes the adapter in dry-run mode. For Docker and Kubernetes adapters, a valid build manifest with image metadata is required.

---

### `deploy history`

Show deployment history.

```bash
fastapi-builder deploy history --project <path>
```

| Option | Short | Type | Required | Default | Description |
|---|---|---|---|---|---|
| `--project` | `-p` | path | No | `.` | Path to the FastAPI project |
| `--help` | | | | | Show help |

Deployment records are stored in `artifacts/deployments/`.
