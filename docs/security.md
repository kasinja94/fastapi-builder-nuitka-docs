# FastAPI Builder — Security Guide

## 1. Overview

The Security Engine provides security checks for FastAPI projects and their build artifacts.

Its goal is to help identify vulnerable dependencies, detect security issues, and generate Software Bill of Materials (SBOM) reports.

Security checks are part of the application's build and release workflow.

## 2. Security Configuration

Security options are configured in `fastapi-builder.yaml`.

Example:

```yaml
security:
  dependency_audit: true
  vulnerability_scan: true
  generate_sbom: true
```

Available options:

- `dependency_audit`: Enables dependency auditing.
- `vulnerability_scan`: Enables vulnerability scanning.
- `generate_sbom`: Enables SBOM generation.

## 3. Dependency Auditing

Dependency auditing examines project dependencies for known security vulnerabilities.

The project dependencies are declared in:

```text
requirements.txt
```

Keep dependencies updated and review security reports before releasing an application.

## 4. Vulnerability Scanning

Vulnerability scanning checks available project or build artifacts for known security issues.

The project supports Trivy integration for Docker image scanning.

Depending on the scan configuration, checks can include:

- Vulnerable software packages.
- Exposed secrets.
- Security misconfigurations.
- Image configuration issues.

A successful scan does not guarantee that an application is completely secure.

## 5. Software Bill of Materials (SBOM)

An SBOM describes the software components included in an application or build artifact.

It helps developers:

- Identify application dependencies.
- Track third-party components.
- Review dependency risks.
- Support software supply-chain security.

SBOM generation depends on the configured security tools and supported artifact formats.

## 6. Recommended Security Workflow

Before releasing an application:

1. Review `requirements.txt`.
2. Run dependency auditing.
3. Build the application.
4. Scan the resulting artifacts where supported.
5. Generate an SBOM.
6. Review and address reported vulnerabilities.
7. Verify the build manifest.
8. Prepare a deployment plan.

## 7. Security Best Practices

- Do not commit passwords, API keys, or private credentials.
- Keep dependencies updated.
- Use trusted package sources.
- Run security scans regularly.
- Review vulnerability reports instead of ignoring them.
- Use non-root users in application containers.
- Verify build artifacts before release.
- Avoid deploying artifacts with unresolved critical security issues.

## 8. Current Limitations

Security scanning depends on external tools, their databases, and the available build artifacts.

A scan may produce false positives or miss vulnerabilities.

Security reports require human review.

The Security Engine does not replace secure application development, code review, or production monitoring.

## 9. Security Status

FastAPI Builder includes security engine integrations for dependency auditing, vulnerability scanning, and SBOM generation.

The exact checks available depend on the project configuration and installed security tools.
