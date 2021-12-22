+++
title = "About this"
+++

[`actions/deploy-pages`](https://github.com/actions/deploy-pages)を使って静的サイトエンジン「[Zola](https://www.getzola.org/)」をデプロイするテスト。  
`actions/deploy-pages`はベータ版であり、使用されているAPIもドキュメントにまだ無いため、本番環境で使用するものではない。

<!-- more -->

# `actions/deploy-pages@v1-beta`の要件
- OIDC tokenが必要。`permissions.id-token`の書き込み権限を追加する。
- Pagesをデプロイするので当然 `permissions.pages`の書き込み権限が必要。
- `github-pages` environmentのprotection ruleにソースブランチを追加する。
- アップロードするartifactは`artifact.tar`でなければならない。[`alehechka/upload-tartifact`](https://github.com/alehechka/upload-tartifact)で`artifact.tar`を作れるが[`action/upload-artifact`](https://github.com/action/upload-artifact)と違い`path`はglobが機能しないので`tar`のオプションを設定する。

# ワークフローファイルの例

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
