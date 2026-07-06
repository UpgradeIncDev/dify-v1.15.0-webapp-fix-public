# dify-v1.15.0-webapp-fix

本リポジトリは [langgenius/dify](https://github.com/langgenius/dify)（LangGenius社）の公式リポジトリではありません。v1.15.0タグを完全な履歴込みでミラーし、その上に特定の修正のみを追加したものです。

## v1.15との違い

Dify v1.15.0には、認証不要で公開しているWebAppのチャットリンク（`/chat/{code}`）にアクセスした際、匿名ユーザーがログイン画面へリダイレクトされてしまう回帰バグがあります。原因は、共通フックの`useTimestamp()`が常に`/console/api/account/profile`（コンソールへの認証済みログインが必要なAPI）を呼び出してしまうためです。

この修正は本家の`main`ブランチには[langgenius/dify@52c106b](https://github.com/langgenius/dify/commit/52c106b5320fd866c42ab0849b3ffa56a0166a29)（PR [#37915](https://github.com/langgenius/dify/pull/37915)）として取り込まれていますが、マージ日は`1.15.0`タグのリリース翌日であり、本書作成時点でタグ付きリリース（v1.15.1等）には未収録です。

本リポジトリでは、この巨大な同期PR（321ファイル）から**該当バグに関係する5ファイルのみ**を抽出し、`1.15.0`タグの上に適用しています。他の変更（`main`のそれ以降の500以上のコミット）は一切含みません。

- タグ `1.15.0`：本家v1.15.0と完全に同一
- タグ `1.15.0-webapp-fix`：上記の修正を追加したもの
- 差分は[こちら（GitHub Compare）](https://github.com/UpgradeIncDev/dify-v1.15.0-webapp-fix/compare/1.15.0...1.15.0-webapp-fix)で確認できます
- 変更ファイル（計5件）:
  - `web/hooks/use-timestamp.ts`
  - `web/hooks/use-timestamp.spec.ts`
  - `web/app/components/base/chat/chat/hooks.ts`
  - `web/app/components/base/chat/chat-with-history/chat-wrapper.tsx`
  - `web/app/components/base/chat/embedded-chatbot/chat-wrapper.tsx`

## EC2インスタンスにおける適用方法

Difyの標準的な`docker/docker-compose.yaml`は、フロントエンドを本家のビルド済みイメージ`langgenius/dify-web:1.15.0`からそのまま起動しています（`build:`定義なし）。今回の修正はフロントエンド（`web`コンテナ）のみに関係するため、`api`・`worker`・`db`・`redis`等の再起動は不要です。

### 1. `web`イメージをビルドする

本リポジトリのルートで（ビルドコンテキストは`web/`ではなくリポジトリルート）:

```bash
docker build --platform linux/amd64 -f web/Dockerfile -t dify-web:1.15.0-webapp-fix .
```

Next.jsの本番ビルドはメモリ・CPUを多く消費します。`t3.medium`のような小規模インスタンス上で、稼働中の他コンテナと同時にビルドを行うと、リソース不足で他サービスに影響する恐れがあるため、EC2上で直接ビルドせず、別環境でビルドしてイメージを転送することを推奨します。

### 2. ビルドしたイメージをEC2に転送する

レジストリ（ECR等）を経由してもよいですが、以下のようにレジストリなしで転送することも可能です。

```bash
# ビルドした側
docker save dify-web:1.15.0-webapp-fix | gzip > dify-web-1.15.0-webapp-fix.tar.gz
# S3にアップロードし、署名付きURLを発行するなどしてEC2側へ転送

# EC2側
gunzip -c dify-web-1.15.0-webapp-fix.tar.gz | docker load
```

### 3. docker-compose.yamlの`web`イメージ参照を変更する

`docker-compose.yaml`（または上書き用の`docker-compose.override.yaml`）内の`web`サービスの`image:`を、`langgenius/dify-web:1.15.0`から`dify-web:1.15.0-webapp-fix`に変更します。

### 4. `web`サービスのみ再作成する

```bash
docker compose up -d web
```

`web`コンテナのみが再作成されます（フロントエンドのみ数十秒〜数分のダウンタイム）。`api`・`db`・`redis`等は無停止のまま稼働し続けます。

### 5. 本家パッチリリースへの切り替え

次期パッチリリース（v1.15.1等、本修正を含むもの）が公開され次第、`image:`を本家の正式イメージに戻すことを推奨します。
