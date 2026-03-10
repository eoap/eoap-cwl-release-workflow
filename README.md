# eoap-cwl-release-workflow

Reusable GitHub Actions workflow for releasing EO Application Package CWL workflows and their container images.

## Intent

This repository provides a CI template that standardizes how CWL-based EO applications are validated, containerized, and published.

The workflow is designed to:

* validate CWL definitions before any container build starts
* build tool images with Kaniko
* scan images and generate an SBOM
* rewrite CWL `dockerPull` references to immutable image digests
* publish the resulting CWL files as OCI artifacts

The goal is reproducible EOAP releases where workflow definitions and container references stay aligned for a given release tag.

## Repository contents

### Reusable workflow

[`release-cwl.yml`](/data/work/github/eoap/eoap-cwl-release-workflow/.github/workflows/release-cwl.yml) is the main reusable GitHub Actions workflow exposed by this repository.

It expects the caller repository to provide:

* the list of tool IDs to build
* the container registry and image prefix
* the CWL workflow directory
* the tool container directory
* the workflow entrypoint used by additional validation tooling

At runtime it uses the prebuilt CI image `ghcr.io/terradue/ci-eoap-container:1.0.0` for all steps except the Kaniko image build step itself.

### Template release workflow

[`release-template.yml`](/data/work/github/eoap/eoap-cwl-release-workflow/.github/workflows/release-template.yml) packages this repository as a versioned release artifact.

For `v*` tags, it creates:

* a `.tar.gz` archive
* a `.zip` archive
* a SHA-256 checksum file

These assets contain the reusable workflow plus the accompanying documentation so downstream users can vendor or inspect the template as a release artifact.

## Release behavior

The reusable release workflow follows three stages:

1. Resolve the release tag from the Git ref.
2. Validate the CWL with `cwltool`, `transpiler-mate codemeta`, and `cwl2ogc`.
3. Build and publish tool images, inject image digests into CWL, and publish the CWL as OCI artifacts.

This ensures the pipeline fails early when the CWL is invalid and only publishes artifacts derived from validated sources.

## Contracts

The repository includes supporting documentation for consumers of the template:

* [`docs/release-contract.md`](/data/work/github/eoap/eoap-cwl-release-workflow/docs/release-contract.md)
* [`docs/project-structure-contract.md`](/data/work/github/eoap/eoap-cwl-release-workflow/docs/project-structure-contract.md)

These documents describe the release guarantees, expected repository layout, and assumptions the workflow makes about CWL tool IDs and Dockerfiles.

## Typical use

A caller repository references the reusable workflow from `.github/workflows/release-cwl.yml` and supplies the required `workflow_call` inputs and secrets.

This repository itself can then be versioned independently, allowing projects to pin a specific workflow release instead of tracking the default branch.
