# Project structure contract

This release pipeline assumes the following project structure and conventions.

## Command-line tools

Each CWL CommandLineTool MUST satisfy all of the following:

* The tool has a unique `id` value.
* The `id` MUST match the name of a tool directory exactly.
* The tool directory MUST contain a Dockerfile.

Example:

```
command-line-tools/
├── crop/
│   └── Dockerfile
├── norm_diff/
│   └── Dockerfile
├── otsu/
│   └── Dockerfile
```

Corresponding CWL:

```yaml
class: CommandLineTool
id: crop
```

## CWL workflows

* CWL workflows MUST reference tools using id values that match tool directory names.
* Tool discovery is not implicit.
* Tools not listed in the release input (tools) are ignored.

## Container image naming

Container images are published using the following convention:

`<registry>/<image-prefix>/<tool-id>:<release-tag>`

Example:

`ghcr.io/eoap/advanced-tooling/crop:0.3.1`

## DockerRequirement normalization

At release time:

* All CWL workflows are normalized to include a `DockerRequirement` in `requirements[]`.
* `hints.DockerRequirement` is ignored for released artifacts.
* All dockerPull values reference immutable image digests.

This ensures deterministic execution.

## Failure semantics

The release pipeline MUST fail if any of the following occur:

* A tool listed in the release input has no corresponding Dockerfile.
* A Dockerfile exists for a tool not referenced in CWL.
* A CWL CommandLineTool cannot be resolved to a container image.
* Digest injection cannot be completed for all tools.

Partial or best-effort releases are not allowed.