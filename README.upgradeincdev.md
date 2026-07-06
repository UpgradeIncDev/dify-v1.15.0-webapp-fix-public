# dify-v1.15.0-webapp-fix

This repository is **not affiliated with LangGenius**. It is a mirror of
[langgenius/dify](https://github.com/langgenius/dify) at tag `1.15.0`, with one isolated,
minimal patch applied on top — nothing else from upstream `main` is included.

## What's different from upstream v1.15.0

Dify v1.15.0 has a regression: public WebApp chat links (`/chat/{code}`) redirect anonymous
visitors to the console sign-in page, because the shared `useTimestamp()` hook always fetches
`/console/api/account/profile`, which requires an authenticated console session that anonymous
WebApp visitors don't have.

Upstream fixed this on `main` in
[langgenius/dify@52c106b](https://github.com/langgenius/dify/commit/52c106b5320fd866c42ab0849b3ffa56a0166a29)
(PR [#37915](https://github.com/langgenius/dify/pull/37915)), merged 2026-06-26 — one day after the
`1.15.0` tag. As of this writing, that fix has **not** been included in any tagged Dify release
(no `1.15.1` exists yet).

This repository backports **only** that fix on top of `1.15.0`, isolated to the 5 files it actually
touches:

- `web/hooks/use-timestamp.ts`
- `web/hooks/use-timestamp.spec.ts`
- `web/app/components/base/chat/chat/hooks.ts`
- `web/app/components/base/chat/chat-with-history/chat-wrapper.tsx`
- `web/app/components/base/chat/embedded-chatbot/chat-wrapper.tsx`

No other upstream changes (the ~500+ commits on `main` since `1.15.0`) are included.

## Deploying this fix to an existing Dify v1.15.0 docker-compose environment

The standard Dify `docker/docker-compose.yaml` runs the frontend from the pre-built
`langgenius/dify-web:1.15.0` image (no `build:` section), so pulling this repo alone does not
change a running deployment. The fix only touches the `web` (frontend) container — `api`, `worker`,
`db`, `redis`, etc. are unaffected and do not need to be rebuilt or restarted.

### 1. Build the patched `web` image

From the root of this repo (build context is the repo root, not `web/`):

```bash
docker build --platform linux/amd64 -f web/Dockerfile -t dify-web:1.15.0-webapp-fix .
```

Build natively on the target architecture if possible — this is a Next.js production build and
can be memory/CPU-intensive. Building directly on a small instance (e.g. `t3.medium`) alongside
the already-running stack risks starving the other containers; prefer building elsewhere and
transferring the image.

### 2. Get the image onto the Docker host

Pick whichever transfer method fits your environment — none of the following is required by this
repo itself:

- **Container registry** (ECR, Docker Hub, etc.): tag and push, then `docker pull` on the host.
- **No registry**: `docker save dify-web:1.15.0-webapp-fix | gzip > dify-web-1.15.0-webapp-fix.tar.gz`,
  transfer the file to the host by whatever means you have (e.g. via an S3 object + a presigned
  URL, `scp`, SSM), then `gunzip -c dify-web-1.15.0-webapp-fix.tar.gz | docker load` on the host.

### 3. Point `docker-compose` at the new image

Change the `web` service's `image:` in your `docker-compose.yaml` (or add a
`docker-compose.override.yaml` so you don't have to hand-edit the tracked file) from
`langgenius/dify-web:1.15.0` to `dify-web:1.15.0-webapp-fix`.

### 4. Restart only the `web` service

```bash
docker compose up -d web
```

This recreates just the `web` container (brief downtime for the frontend only) — `api`, `db`,
`redis`, and other services keep running throughout.

## References

- Upstream repository: https://github.com/langgenius/dify
- Regression reports: [#38043](https://github.com/langgenius/dify/issues/38043),
  [#38111](https://github.com/langgenius/dify/issues/38111)
- Upstream fix: [PR #37915](https://github.com/langgenius/dify/pull/37915)
- Tag [`1.15.0-webapp-fix`](../../releases/tag/1.15.0-webapp-fix) in this repository points to the
  patched commit; tag `1.15.0` (inherited from upstream) points to the original, unpatched release.

## License

Dify is licensed under a modified Apache License 2.0 (see `LICENSE`). This mirror does not remove
or modify any LOGO or copyright notices and does not operate a multi-tenant hosted service, per the
license's additional conditions.
