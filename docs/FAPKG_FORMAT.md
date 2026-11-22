# FAPKG Format

FAPKG (FastAPI Package) is a zip-based portable archive format for distributing Nuitka-compiled FastAPI application artifacts.

---

## Purpose

A `.fapkg` file packages a Nuitka build artifact — either a **standalone** `.dist/` directory build or a **onefile** self-contained executable — into a single verifiable, portable archive that can be distributed, inspected, and unpacked without running the original build toolchain.

---

## File Extension

`.fapkg`

---

## Artifact Types

FAPKG supports two Nuitka artifact types:

### Standalone distribution

A Nuitka standalone build produces a directory (conventionally named `<app>.dist/`) containing:

- The compiled application executable.
- The Python runtime (bundled by Nuitka — no separate Python installation required on the target host).
- Compiled extension modules and pure-Python dependencies.

Inside the FAPKG archive the entire artifact directory is stored under `app/`, preserving its name:

```
app/
  <artifact-directory>/       — the artifact directory tree (e.g. myapp.dist/)
    <application-name>        — the main executable
    lib/
    ...
```

The `artifact.entry` manifest field points to `app/<artifact-directory>` (the directory itself). The `artifact.kind` field is `"directory"` and `artifact_type` is `"standalone"`.

### Onefile distribution

A Nuitka onefile build produces a single self-extracting executable that bundles the Python runtime and all compiled dependencies.

Inside the FAPKG archive the executable is stored under `app/`:

```
app/
  <application-executable>    — the self-contained onefile binary
```

The `artifact.kind` manifest field is `"file"` and `artifact_type` is `"onefile"`.

### Runtime requirements

Nuitka standalone and onefile artifacts include the Python runtime. The target host does **not** require Python to be installed.

The target host may still require:

- Operating-system shared libraries (`glibc`, `libssl`, etc., depending on what the application uses).
- External services the application connects to (databases, message brokers, caches).
- Configuration files and environment variables the application reads at runtime.

---

## Archive Structure

A FAPKG archive is a standard ZIP file (ZIP64-compatible) with this internal layout:

```
MANIFEST.json               — Package manifest (metadata and artifact description)
checksums.sha256            — SHA-256 checksums for every member except itself
app/
  <compiled artifact>       — Standalone directory tree or onefile executable
meta/                       — Optional metadata files
  build-manifest.json       — Nuitka build manifest (optional)
  sbom.cdx.json             — CycloneDX SBOM (optional)
  security-report.json      — Security scan report (optional)
```

---

## `MANIFEST.json`

The package manifest is a JSON object. All fields except those marked optional are required.

| Field | Type | Description |
|---|---|---|
| `schema_version` | string | Always `"1.0"` |
| `package_format` | string | Always `"fapkg"` |
| `build_id` | string | Unique identifier for this build |
| `created_at` | string | ISO 8601 UTC timestamp |
| `project.name` | string | Project name |
| `project.version` | string | Project version |
| `platform.os` | string | Build OS — e.g. `"linux"` |
| `platform.arch` | string | Build architecture — e.g. `"x86_64"` |
| `build_type` | string | Nuitka build type: `"nuitka"` (standalone) or `"nuitka-onefile"` |
| `artifact_type` | string \| null | Human-readable classification: `"standalone"` or `"onefile"`. Optional; `null` in archives created before this field was added. |
| `artifact.kind` | string | `"directory"` (standalone) or `"file"` (onefile) |
| `artifact.entry` | string | Archive path to the artifact root: `app/<directory-name>` for standalone (a directory prefix), `app/<executable>` for onefile (an exact member name) |
| `runtime.entrypoint` | string | Application entrypoint — e.g. `"app.main:app"` |
| `runtime.python_version` | string | Python version used to build (optional) |
| `runtime.host` | string | Bind host (optional) |
| `runtime.port` | integer | Bind port (optional) |
| `checksum_algorithm` | string | Always `"sha256"` |
| `compatibility.os` | string | Required OS for extraction compatibility check |
| `compatibility.arch` | string | Required architecture for extraction compatibility check |
| `sbom_reference` | string \| null | Archive path to embedded SBOM, if present — e.g. `"meta/sbom.cdx.json"` |
| `security_report_reference` | string \| null | Archive path to embedded security report, if present |

### Example `MANIFEST.json`

```json
{
  "schema_version": "1.0",
  "package_format": "fapkg",
  "build_id": "nuitka-inventory-api-1.0.0-20261121T071714Z",
  "created_at": "2026-11-21T07:17:14+00:00",
  "project": {
    "name": "inventory-api",
    "version": "1.0.0"
  },
  "platform": {
    "os": "linux",
    "arch": "x86_64"
  },
  "build_type": "nuitka",
  "artifact_type": "standalone",
  "runtime": {
    "entrypoint": "app.main:app",
    "python_version": "3.15.2",
    "host": "127.0.0.1",
    "port": 8765
  },
  "artifact": {
    "kind": "directory",
    "entry": "app/inventory_api_server.dist"
  },
  "checksum_algorithm": "sha256",
  "compatibility": {
    "os": "linux",
    "arch": "x86_64"
  },
  "sbom_reference": null,
  "security_report_reference": null
}
```

---

## `checksums.sha256`

A text file listing SHA-256 checksums for every member in the archive **except `checksums.sha256` itself**. This includes `app/` files, `meta/` files, and `MANIFEST.json`.

Format (two spaces between digest and path):

```
<64-hex-chars>  app/path/to/file
<64-hex-chars>  MANIFEST.json
<64-hex-chars>  meta/build-manifest.json
```

Checksums are verified before extraction by `fapkg verify` and `fapkg unpack`. Every member in the archive must be covered; uncovered members cause verification to fail.

---

## Security Protections

FAPKG extraction enforces the following checks before writing any file to disk:

1. **Path traversal rejection** — archive member names containing `..`, absolute paths, or backslash-based escape sequences are rejected.
2. **Symlink rejection** — archive members that have symlink mode bits set are rejected.
3. **Duplicate member rejection** — archives containing duplicate member names are rejected.
4. **Checksum verification** — every member (except `checksums.sha256` itself) is verified against `checksums.sha256` before extraction begins. Any mismatch or uncovered member fails extraction.
5. **Escape check** — each extracted path is resolved and confirmed to remain inside the target directory before being written.

These checks run unconditionally. There is no option to skip them.

---

## Commands

### Pack a standalone artifact

```bash
fastapi-builder build fapkg \
  --project <project-dir> \
  --artifact <nuitka-dist-dir> \
  [--output <file.fapkg>] \
  [--build-manifest <manifest.json>] \
  [--sbom <sbom.cdx.json>] \
  [--security-report <report.json>] \
  [--overwrite]
```

### Pack a onefile artifact

```bash
fastapi-builder build fapkg \
  --project <project-dir> \
  --artifact <nuitka-onefile-executable> \
  [--output <file.fapkg>]
```

The command detects the artifact type automatically: a file → onefile, a directory → standalone.

### Inspect

Read metadata without extracting or verifying checksums:

```bash
fastapi-builder fapkg inspect <file.fapkg>
```

### Verify

Verify all checksums without extracting:

```bash
fastapi-builder fapkg verify <file.fapkg>
```

### Unpack

Verify and extract to a target directory:

```bash
fastapi-builder fapkg unpack <file.fapkg> \
  --output <target-dir> \
  [--platform linux-x86_64] \
  [--overwrite]
```

The `--platform` option checks the archive's `compatibility` fields against the supplied target. If the platforms do not match, extraction is refused. Omitting `--platform` skips the compatibility check.

---

## Recommended Deployment Layout (Linux)

After unpacking a standalone artifact into `/opt/my-app`:

```
/opt/my-app/
  app/
    my_server.dist/         — artifact directory tree
      my_server             — main executable (chmod +x)
      lib/                  — compiled runtime and extensions
  MANIFEST.json
  checksums.sha256
  meta/
    build-manifest.json
```

Run the application:

```bash
/opt/my-app/app/my_server.dist/my_server
```

No Python installation is required on the host. External services (database, Redis, etc.) must be available separately.

---

## Compatibility

FAPKG archives are standard ZIP files and can be opened with any zip-capable tool for inspection. However, the security extraction protections are **only applied** when using `fastapi-builder fapkg unpack`. Extracting with a third-party zip tool bypasses these protections.

---

## Limitations

- FAPKG archives are platform-specific. A standalone or onefile artifact built on `linux-x86_64` will not run on `linux-arm64` or macOS without rebuilding.
- FAPKG does not cryptographically sign archives. For publisher authenticity verification, sign the `.fapkg` file separately (e.g. with GPG or sigstore).
- The `--platform` compatibility check in `fapkg unpack` reads the manifest but does not prevent extraction on a mismatched platform when `--platform` is omitted.
- Nuitka standalone and onefile binaries may still require specific versions of OS libraries (`glibc`, `libssl`, etc.) that differ across Linux distributions. Build on a compatible base image when deploying across distributions.
