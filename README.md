# iberico-lab/iberico-lab

Shared GitHub Actions for the Iberico repos. Reference by major tag (`@v1`).

| Name | Kind | Use |
|---|---|---|
| `actions/registry-login` | composite | Install regctl, log in to `registry.mediumrare.ai`, enable 50 MB chunked uploads (Cloudflare's 100 MB request cap). |
| `actions/oci-publish` | composite | `regctl image import` an OCI archive, add the other tags as server-side copies, with retries. |
| `.github/workflows/docker-publish.yml` | reusable workflow | Build one image with buildx and publish it. Wraps the two actions above. |

```yaml
# Simple case: one image, one Dockerfile
jobs:
  image:
    uses: iberico-lab/iberico-lab/.github/workflows/docker-publish.yml@v1
    with:
      image: asset-api
      tags: ${{ github.ref_name }},latest
      build-args: |
        VERSION=${{ github.ref_name }}
    secrets: inherit

# Custom build (bake, reuse of a previous release): compose the actions
steps:
  - uses: iberico-lab/iberico-lab/actions/registry-login@v1
    with:
      username: ${{ secrets.REGISTRY_USERNAME }}
      password: ${{ secrets.REGISTRY_PASSWORD }}
  # ... build to an OCI archive ...
  - uses: iberico-lab/iberico-lab/actions/oci-publish@v1
    with:
      image: registry.mediumrare.ai/iberico-lab/asset-api
      tags: v1.2.3,latest
      archive: oci/asset-api.tar
```

## Conventions

- Third-party actions are pinned by commit SHA with the tag in a trailing comment. Dependabot keeps them current.
- `v1` is a moving tag; breaking changes get `v2`. Move it with `git tag -f v1 && git push -f origin v1` after a release.
- Internal references (`iberico-lab/iberico-lab/...@v1`) must be bumped together when the major version changes.

## Access

Actions access is set to "organization", so any `iberico-lab` repo can call these workflows even while this repo is private. A repo outside the org cannot; move it into the org or make this repo public.
