# Changelog

All notable changes to this project are documented in this file.

The format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).
Versions follow [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

---

## [1.0.2] — 2026-11-22

### Fixed

- **FAPKG standalone archive path consistency** — `pack_fapkg()` previously stripped the top-level artifact directory name from archive member paths (storing files as `app/<file>` instead of `app/<dir-name>/<file>`), while `MANIFEST.json` `artifact.entry` pointed to `app/<dir-name>` — a path that did not exist in the archive. Both helpers are now consistent: standalone files are stored under `app/<directory-name>/` and `artifact.entry` equals `app/<directory-name>`.
- `_detect_artifact_entry()` simplified: returns `app/<artifact.name>` unconditionally, removing the fragile executable-search heuristic that only worked when the directory was named `<stem>.dist` and contained an executable with the same stem.
- **FAPKG CLI documentation corrected** — `fapkg inspect` and `fapkg verify` accept a positional path argument, not `--package`. `fapkg unpack` uses `--output` for the target directory, not `--dest`. `security scan-fapkg` also uses a positional argument.
- **Build profile documentation corrected** — Profiles are `standard`, `development`, `low-memory`, `release`, `release-low-memory`. Removed non-existent `debug` profile reference.
- **Removed false `fapkg launch` CLI claim** — No `launch` command is exposed in the CLI. The Python function `generate_launcher_script()` exists as a utility but is not a CLI command.
- **`generate_launcher_script()` standalone support** — Added optional `dist_directory` parameter. Standalone artifact launchers now correctly reference `app/<dist_directory>/<executable_name>` instead of the flat onefile path `app/<executable_name>`. Backward compatible: calling with `dist_directory=None` (the default) preserves onefile behavior.
- Stale test count in README corrected.
- Trailing whitespace in `pyproject.toml` version field removed.

### Changed

- Updated supported versions in `SECURITY.md` to reflect 1.0.2 as the currently maintained release.

### Compatibility

Archives created by versions 1.0.0 and 1.0.1 may have `artifact.entry` pointing to either a file inside `app/` or a path that does not exist as an archive member. These archives remain fully verifiable and extractable: `verify_fapkg` and `unpack_fapkg` use `checksums.sha256` for integrity and extract all members — neither function uses `artifact.entry` for extraction decisions.

---

## [1.0.1] — 2026-11-22

### Added

- **`artifact_type` manifest field** — FAPKG `MANIFEST.json` now includes an explicit `artifact_type` field set to `"standalone"` (Nuitka standalone `.dist/` directory build) or `"onefile"` (Nuitka self-contained executable build). The field is optional and defaults to `null` when reading archives created by earlier versions of FastAPI Builder, preserving backward compatibility.
- `inspect_fapkg()` and `fastapi-builder fapkg inspect` now return `artifact_type` in their output.

### Changed

- `FAPKG_FORMAT.md` rewritten to accurately reflect the implemented manifest schema:
  - Corrects field names (`schema_version`, `package_format`, `build_type`, `artifact.kind`, `artifact.entry`, `artifact_type`).
  - Corrects the `checksums.sha256` scope description: checksums cover all archive members (including `meta/` and `MANIFEST.json`), not only `app/` files.
  - Corrects the Python runtime claim: Nuitka standalone and onefile builds **bundle** the Python runtime. The target host does not require a Python installation.
  - Adds separate layout diagrams for standalone and onefile distributions.
  - Adds recommended Linux deployment layout and runtime requirement notes.
- `docs/architecture.md` updated with high-level architecture and build lifecycle Mermaid diagrams.

### Notes

- Existing `.fapkg` archives remain valid. `verify_fapkg`, `inspect_fapkg`, and `unpack_fapkg` accept archives without `artifact_type` in their manifest.
- The `build_type` field (`"nuitka"` / `"nuitka-onefile"`) is preserved unchanged.

---

## [1.0.0] — 2026-11-21

First production release.

### Added

- **Build engine:** Nuitka standalone and onefile compilation with five build profiles (`standard`, `development`, `low-memory`, `release`, `release-low-memory`).
- **Resource preflight:** Point-in-time snapshot of RAM, swap, CPU, disk, and active compiler PIDs before each build.
- **Adaptive worker selection:** Computes a safe `--jobs` count from available memory and CPU. Swap-exhausted guard: `swap_available_gb < 0.05` forces `--jobs=1` regardless of RAM.
- **Live build monitoring:** Polls `/proc` during Nuitka builds to track compiler phase (gcc, cc1, clang, LTO) and peak resource usage.
- **Cross-process build locking:** Project-scoped JSON lock in `/tmp` prevents concurrent Nuitka builds on the same project; stale locks are auto-recovered by PID check.
- **FAPKG portable packaging:** Pack, inspect, verify, and unpack Nuitka artifacts as cryptographically verified zip-based archives; path traversal, symlink attack, and duplicate-member rejection enforced.
- **Security engine:** Dependency auditing (`pip-audit`), vulnerability scanning (Trivy), Python code scanning (Bandit), secret scanning (`detect-secrets`), CycloneDX SBOM generation, Docker image scanning, artifact scanning, artifact SBOM generation.
- **Security report export:** PDF, DOCX, XLSX, HTML, Markdown, and JSON formats.
- **Deployment engine:** Dry-run deployment planning and preview for Docker, Systemd, and Kubernetes adapters; deployment history.
- **Build history and manifest verification:** SHA-256 artifact checksums stored in JSON manifests; `build verify` command confirms integrity.
- **Platform matrix:** Defined target identifiers (`linux-x86_64`, `linux-arm64`, `windows-x86_64`, `windows-arm64`, `macos-x86_64`, `macos-arm64`); native host detection.
- **MIT License.**

### Notes

- Multi-worker Nuitka LTO builds have not been validated. All reference builds used `release-low-memory` (`--jobs=1`).
- Deployment adapters are dry-run only. Actual deployment execution is not yet implemented.
- Resource monitoring requires Linux `/proc`. Functionality degrades gracefully on other platforms.

---

## [0.2.0] — 2026-11-21

### Added

- Native build pipeline and resource management subsystem.
- Build profiles and adaptive worker selection (initial implementation).
- FAPKG packaging: pack, inspect, verify, unpack.
- Security engine integrations: pip-audit, Trivy, Bandit, detect-secrets, cyclonedx-bom.
- Security report export (PDF, DOCX, XLSX, HTML, Markdown, JSON).
- Build history and manifest verification.
- Deployment planning and dry-run preview for Docker, Systemd, Kubernetes.
- Platform matrix and host-target detection.

---

## [0.1.0] — 2026-11-21

Initial release.

### Added

- Modular CLI built with Typer.
- FastAPI project discovery.
- YAML-based configuration loading and validation (Pydantic).
- Build planning (no execution).
- Nuitka standalone build integration.
- Docker image build integration.
- Security engine scaffold.
- Deployment planning scaffold.
