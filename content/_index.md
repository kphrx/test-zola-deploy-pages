+++
title = "About this"
+++

This is a test of using [`actions/deploy-pages`](https://github.com/actions/deploy-pages) to deploy the static site engine "[Zola](https://www.getzola.org/)".  
`actions/deploy-pages` is a beta version, and the API used is not yet documented, it is not suitable for production use.

<!-- more -->

# Requirements for `actions/deploy-pages@v1-beta`
- Add write permission to `permissions.id-token` for the OIDC token.
- Deploying Pages of course requires write permission to `permissions.pages`.
- Add source branch to the protection rule of `github-pages` environment.
- The artifact you upload **must** be in `artifact.tar`. ~~You can also create and upload an `artifact.tar` by [`alehechka/upload-tartifact`](https://github.com/alehechka/upload-tartifact),  but unlike `path` in [`action/upload-artifact`](https://github.com/action/upload-artifact), globbing does not work, so you need to set it with the option of `tar`.~~ See *[actions/upload-artifact #Maintaining file permissions and case sensitive files](https://github.com/actions/upload-artifact#maintaining-file-permissions-and-case-sensitive-files)* for instructions on how to create a tar and upload it.

# Example workflow file

```yaml
# ...

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: 'Checkout'
        uses: actions/checkout@master
      - name: 'Build'
        uses: shalzz/zola-deploy-action@master
        env:
          BUILD_DIR: .
          BUILD_ONLY: true
      - name: Tar files
        run: tar -cvf artifact.tar -C public .
      - name: Upload Artifact
        uses: actions/upload-artifact@v2
        with:
          name: github-pages
          path: artifact.tar

  deploy:
    needs: build
    runs-on: ubuntu-latest
    permissions:
      id-token: 'write'
      pages: 'write'
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    steps:
      - name: 'Deploy'
        id: deployment
        uses: actions/deploy-pages@v1-beta
```
