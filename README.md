# GitHub Actions CI/CD Platform

A production-style GitHub Actions CI pipeline for building, securing, and publishing a Dockerized NGINX application to GitHub Container Registry (GHCR).

The pipeline demonstrates Docker Buildx, image tagging, Trivy vulnerability scanning, GitHub Code Scanning integration, SBOM generation, and controlled image publishing.

## Architecture

```text
Developer
   |
   | git push / pull request
   v
GitHub Repository
   |
   v
GitHub Actions CI Pipeline
   |
   +--> Checkout source
   |
   +--> Extract Docker metadata
   |       |
   |       +--> latest
   |       +--> sha-<commit>
   |
   +--> Build Docker image
   |       |
   |       +--> load image into runner
   |
   +--> Trivy vulnerability scan
   |       |
   |       +--> HIGH / CRITICAL findings
   |       +--> SARIF report
   |                 |
   |                 v
   |          GitHub Code Scanning
   |
   +--> Generate SBOM
   |       |
   |       +--> CycloneDX JSON
   |                 |
   |                 v
   |          GitHub Actions Artifact
   |
   +--> Push Docker image
           |
           v
     GitHub Container Registry
       +--> :latest
       +--> :sha-<commit>
```

## What This Project Demonstrates

- GitHub Actions CI/CD fundamentals
- Docker Buildx image builds
- OCI image metadata and tagging
- GitHub Container Registry (GHCR)
- Container vulnerability scanning with Trivy
- GitHub Code Scanning using SARIF
- SBOM generation using CycloneDX
- Security checks before the final image push
- Reproducible image identification using commit SHA tags

## Repository Flow

```text
Code change
   -> GitHub Actions
   -> Build image locally
   -> Trivy scan
   -> Upload SARIF to GitHub Security
   -> Generate SBOM
   -> Upload SBOM artifact
   -> Push image to GHCR
```

The image is built with `load: true`, so the runner can scan the locally built image before it is pushed to GHCR.

## Prerequisites

1. A GitHub repository with Actions enabled.
2. GitHub Container Registry access.
3. A Dockerfile in `docker/Dockerfile`.
4. Application files under `app/`.
5. GitHub Actions permissions allowing package publishing and security-event uploads.

The workflow uses these permissions:

```yaml
permissions:
  contents: read
  packages: write
  security-events: write
```

## How to Use

### 1. Clone the repository

```bash
git clone <repository-url>
cd github-actions-cicd-platform
```

### 2. Make a change

Update the application or Docker configuration and commit the change.

```bash
git add .
git commit -m "Update application"
git push origin main
```

A pull request targeting `main` also triggers the workflow.

### 3. Monitor the workflow

Open:

```text
GitHub Repository
  -> Actions
  -> CI Pipeline
```

The workflow builds the image, scans it, generates the SBOM, and then publishes the image.

## Docker Image Tags

Each successful build publishes two tags:

```text
ghcr.io/<owner>/github-actions-cicd-platform:latest
ghcr.io/<owner>/github-actions-cicd-platform:sha-<commit>
```

`latest` is convenient for the current build, while the SHA tag provides an immutable reference to a specific commit.

## Security Results

### Trivy Vulnerability Scan

Trivy checks the Docker image for HIGH and CRITICAL vulnerabilities.

The current development configuration uses:

```yaml
exit-code: 0
ignore-unfixed: true
severity: CRITICAL,HIGH
```

This means findings are reported without currently blocking the workflow.

The scan is exported as:

```text
trivy-results.sarif
```

and uploaded to:

```text
GitHub Repository
  -> Security
  -> Code scanning
  -> Alerts
```

### SBOM

The pipeline generates a CycloneDX SBOM:

```text
sbom.cyclonedx.json
```

It contains an inventory of software components detected in the container image.

The SBOM is uploaded as a GitHub Actions artifact and can be downloaded from:

```text
GitHub Repository
  -> Actions
  -> CI Pipeline run
  -> Summary
  -> Artifacts
```

## Expected Results

A successful run should produce:

```text
CI Pipeline                 SUCCESS
Docker Build                SUCCESS
Trivy Scan                  SUCCESS
SARIF Upload                SUCCESS
SBOM Generation             SUCCESS
SBOM Artifact               AVAILABLE
GHCR Image Push             SUCCESS
```

The GitHub Security dashboard should show Trivy findings when vulnerabilities are detected, and the Actions run should contain a downloadable SBOM artifact.

## Security Gate

The pipeline is currently configured with:

```yaml
exit-code: 0
```

This allows the portfolio pipeline to complete while vulnerabilities are being reviewed.

For a stricter production security gate, this can be changed to:

```yaml
exit-code: 1
```

With that configuration, configured HIGH/CRITICAL findings can fail the scan and prevent the image from reaching the final push step.

## Key Files

```text
.
├── .github/
│   └── workflows/
│       └── ci.yml
├── app/
│   ├── index.html
│   └── nginx.conf
├── docker/
│   └── Dockerfile
└── README.md
```

## Interview Talking Points

This project can be discussed as a small DevSecOps pipeline:

> Build the container with Docker Buildx, generate deterministic image tags, scan the image with Trivy before publishing, send SARIF findings to GitHub Code Scanning, generate a CycloneDX SBOM, retain the SBOM as a workflow artifact, and publish the validated image to GHCR.
