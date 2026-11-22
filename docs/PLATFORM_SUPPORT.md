# Platform Support

---

## Summary

| Feature | Linux (x86_64) | Linux (ARM64) | macOS | Windows |
|---|---|---|---|---|
| Resource preflight | Validated | Expected to work | Graceful degradation | Graceful degradation |
| Adaptive worker selection | Validated | Expected to work | Falls back to 1 job | Falls back to 1 job |
| Live build monitoring | Validated | Expected to work | Skipped | Skipped |
| Build locking | Validated | Expected to work | Untested | Untested |
| Nuitka compilation | Validated | Untested | Untested | Untested |
| FAPKG packaging | Validated | Untested | Untested | Untested |
| Security scanning | Validated | Untested | Untested | Untested |

**Primary validated platform:** Linux x86_64, Ubuntu 22.04 / kernel 6.8.

---

## Linux

Linux is the only fully validated platform.

Resource monitoring (`resources.py`, `monitor.py`, `policy.py`) reads `/proc/meminfo` and `/proc/<pid>/cmdline`. These files are only available on Linux.

**Validated configuration:**

| Item | Value |
|---|---|
| Kernel | 6.8.0-149-generic |
| Distribution | Ubuntu 22.04 (compatible) |
| Architecture | x86_64 |
| Python | 3.15.2 (PyEnv) |
| Nuitka | 4.3.5 |
| CPU | 20 logical cores |
| RAM | 31 GB |

ARM64 Linux is expected to work because the code reads `/proc` using the same kernel interface, but it has not been tested.

---

## macOS

macOS is not validated.

**What works:** Python code that does not rely on `/proc` — configuration loading, build planning, FAPKG packaging, security scanning, deployment planning, CLI interface.

**What degrades gracefully:**

- `build resources`: Returns `null` values for all `/proc`-derived fields (`ram_available_gb`, `swap_available_gb`, `cpu_count`, etc.) and an empty `active_compiler_pids` list.
- `build nuitka` / `build onefile`: Proceeds with `advisory_jobs = 1` (the conservative fallback when resource info is unavailable). The swap-exhausted guard does not fire because swap info is not available.
- Live monitoring: Compiler PID detection returns empty results.
- Build locking: Uses filesystem-based JSON lock; expected to work but not tested.

**Nuitka on macOS:** Not tested. Nuitka may require Xcode command-line tools and behaves differently from Linux builds. Build profiles and their flags were defined and tested on Linux only.

---

## Windows

Windows is not validated.

**What works:** Configuration loading, build planning, security scanning, deployment planning, CLI interface.

**What degrades gracefully:** Same as macOS. Resource monitoring returns null/empty values; `advisory_jobs = 1` is used.

**Nuitka on Windows:** Not tested. Nuitka on Windows requires the MSVC toolchain or MinGW. Build profiles and flags were defined and tested on Linux only.

**Path handling:** The codebase uses `pathlib.Path` throughout, which handles Windows paths. However, path handling for FAPKG archives and artifact resolution has not been tested on Windows.

---

## Cross-Compilation

Nuitka **cannot cross-compile**. Building a `linux-arm64` artifact requires running the build on a Linux ARM64 host. Building a `windows-x86_64` artifact requires running on a Windows x86_64 host.

FastAPI Builder validates platform target identifiers in configuration (`build.fapkg.platforms`) but does not prevent building for a non-matching target — the actual build result depends on the Nuitka invocation.

---

## Python Version Compatibility

| Python version | Status |
|---|---|
| 3.15.2 | Primary validated runtime; full regression test suite passes |
| 3.11, 3.12, 3.13 | Declared in classifiers; `requires-python = ">=3.11"`; not independently regression-tested |
| 3.10 and earlier | Not supported (code uses `X \| Y` union syntax requiring Python 3.10+; minimum declared as 3.11) |

The project uses `X | Y` union type syntax throughout (Python 3.10+). `from __future__ import annotations` is not used universally, so runtime behavior is not tested below 3.11.

---

## Nuitka Compatibility

| Nuitka version | Status |
|---|---|
| 4.3.5 | Validated |
| Other versions | Not tested |

Nuitka 4.3.5 has partial Python 3.15 support. Builds were completed successfully on Python 3.15.2 with Nuitka 4.3.5 during development.
