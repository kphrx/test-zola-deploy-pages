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
- アップロードするartifactは`artifact.tar`でなければならない。~~[`alehechka/upload-tartifact`](https://github.com/alehechka/upload-tartifact)で`artifact.tar`を作れるが[`action/upload-artifact`](https://github.com/action/upload-artifact)と違い`path`はglobが機能しないので`tar`のオプションを設定する。~~ *[actions/upload-artifact #Maintaining file permissions and case sensitive files](https://github.com/actions/upload-artifact#maintaining-file-permissions-and-case-sensitive-files)* でtarを作ってアップロードする方法が書かれているので参考にする

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
