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
    
## Consuming CWL OCI artifacts

CWL workflows are distributed as OCI artifacts and can be retrieved using oras.

### Authentication

Access to the GitHub Container Registry requires authentication:

```console
echo "$GITHUB_TOKEN" | oras login ghcr.io \
  -u <github-username> \
  --password-stdin
```

The token must have at least read:packages permissions.

### Pulling a CWL workflow

To retrieve a specific released workflow:

```console
oras pull \
  ghcr.io/eoap/advanced-tooling/app-water-bodies-cloud-native-geoparquet:0.1.2
```

This command downloads the CWL file into the current directory:

`app-water-bodies-cloud-native-geoparquet.cwl`

The pulled CWL:

* has been validated with cwltool
* references container images by immutable digest
* corresponds exactly to release 0.1.2

### Pulling from a rolling channel

To retrieve the latest development version:

```console
oras pull \
  ghcr.io/eoap/advanced-tooling/app-water-bodies-cloud-native-geoparquet:latest-dev
```

Or the latest stable version:

```
oras pull \
  ghcr.io/eoap/advanced-tooling/app-water-bodies-cloud-native-geoparquet:latest
```

These tags are mutable and may change over time.

### Inspecting the artifact (optional)

To inspect the OCI manifest before pulling:

```console
oras manifest fetch \
  ghcr.io/eoap/advanced-tooling/app-water-bodies-cloud-native-geoparquet:0.1.2
```

This displays the artifact type and referenced layers, for example:

```json
{
  "artifactType": "application/cwl",
  "layers": [
    {
      "mediaType": "application/cwl",
      "digest": "sha256:…"
    }
  ]
}
```

### Discovering related artifacts (optional)

If additional artifacts (e.g. SBOMs or provenance) are attached to the same reference, they can be listed with:

```console
oras discover \
  ghcr.io/eoap/advanced-tooling/app-water-bodies-cloud-native-geoparquet:0.1.2
```
    
### Executing the pulled workflow

Once retrieved, the workflow can be executed directly:

```console
cwltool app-water-bodies-cloud-native-geoparquet.cwl
```

No additional repository checkout is required.

## Notes on stability

Consumers are encouraged to use semantic version tags (X.Y.Z) for reproducible executions.

The `latest` and `latest-dev` tags are provided for convenience and integration testing only.