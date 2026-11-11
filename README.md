# iberico-lab/iberico-lab

Shared GitHub Actions for the Iberico repos. Reference by major tag (`@v1`) or, for stricter pinning, by commit SHA.

| Name | Kind | Use |
|---|---|---|
| `actions/registry-login` | composite | Install regctl, log in to `registry.mediumrare.ai`, enable 50 MB chunked uploads (Cloudflare's 100 MB request cap). |
| `actions/oci-publish` | composite | `regctl image import` an OCI archive, add the other tags as server-side copies, with retries. |

Each repo keeps its own build step: build to an OCI archive (`--output type=oci,dest=...`), then publish it.

```yaml
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

Credentials are never stored here. Every caller passes its own.

## Conventions

- Third-party actions are pinned by commit SHA with the tag in a trailing comment. Dependabot keeps them current.
- `v1` is a moving tag; breaking changes get `v2`. Move it with `git tag -f v1 && git push -f origin v1` after a release.
