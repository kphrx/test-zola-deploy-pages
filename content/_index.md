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
- The artifact you upload **must** be in `artifact.tar`. You can also create and upload an `artifact.tar` by `alehechka/upload-tartifact`,  but unlike `path` in `action/upload-artifact`, globbing does not work, so you need to set it with the option of `tar`.

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
      - uses: alehechka/upload-tartifact@v1
        with:
          name: github-pages
          path: -C public .

  deploy:
    needs: build
    runs-on: ubuntu-latest
    environment:
      name: github-pages
      url: ${{ steps.pages.outputs.result }}
    permissions:
      id-token: 'write'
      pages: 'write'
    steps:
      - name: 'Pages Info'
        id: pages
        uses: actions/github-script@v5
        with:
          result-encoding: string
          script: |
            const pages = await github.rest.repos.getPages({
              owner: context.repo.owner,
              repo: context.repo.repo,
            });
            return pages.data.html_url;
      - name: 'Deploy'
        uses: actions/deploy-pages@v1-beta
```
