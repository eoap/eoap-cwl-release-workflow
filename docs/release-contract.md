# Release Contract

## Release channels

| Git reference | Release tag | Mutability |
|---------------|-------------|------------|
| develop       | latest-dev  | mutable    |
| main          | latest      | mutable    |
| vX.Y.Z        | X.Y.Z       | immutable  |


## Produced artifacts

For each release:

### Container images

`ghcr.io/eoap/advanced-tooling/<tool>:<release-tag>`

Guarantees:

* Built from repository state at trigger commit
* Tagged consistently across all tools
* Digest-stable after push

### CWL OCI artifacts

`ghcr.io/<owner>/<repo>/<workflow>:<release-tag>`

Artifact type:

`application/cwl`

Guarantees:

* CWL validated with cwltool
* All DockerRequirement.dockerPull values:
  * use immutable image digests
  * correspond exactly to images built in the same release
* CWL content is self-contained and reproducible

## Immutability guarantees

| Artifact    | Mutable? | Notes                  |
|-------------|----------|------------------------|
| latest-dev  | yes      | development channel    |
| latest      | yes      | rolling stable         |
| X.Y.Z       | no       | must never be overwritten |

Rule: Semantic version tags must be treated as immutable research artifacts.

## Reproducibility guarantees

Given:

* a CWL OCI artifact <workflow>:<tag>
* access to referenced container registry

Then:

* execution will use exact container images
* independent of tag drift
* independent of registry defaults

This satisfies:

* EO Application Package expectations
* long-term archival requirements
* scientific reproducibility

## Non-goals (explicit)

The release pipeline does not:

* guarantee numerical reproducibility of results
* manage runtime configuration or input datasets
* guarantee backward compatibility across CWL versions

Those are responsibility of:

* CWL authors
* workflow consumers
* higher-level packaging