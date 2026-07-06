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
