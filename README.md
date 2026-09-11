# Rex BaaS Partner Documentation

Partner-facing API documentation for the Rex Banking-as-a-Service platform,
covering exactly the surface in `rexmobileapp`'s `routes/baas.php`
(`/baas/api/v1/*`). Built for [Mintlify](https://mintlify.com).

Never documents `routes/api.php` (the internal mobile app) or the
`/baas/api/v1/admin/*` group (Rex staff partner-review tooling) — see
`rexmobileapp`'s `docs/baas/api-docs-site-plan.md` for why.

## Structure

```
openapi.yaml           # source of truth for the API Reference tab — 54 operations, 8 tags
docs.json              # Mintlify navigation + theme config
introduction.mdx
authentication.mdx     # ⚠️ documents a real auth gap — read before publishing, see below
errors.mdx
rate-limits.mdx
webhooks/
  overview.mdx
  events.mdx
  signature-verification.mdx
guides/
  going-live-checklist.mdx
  sandbox-vs-live.mdx
  virtual-vs-merchant-accounts.mdx
```

## Branding

`favicon.svg` and the `colors` in `docs.json` are placeholders — swap them
for Rex's actual brand mark/palette before publishing.

## Local preview

```bash
npm install
npm run dev
```

Requires a free Mintlify account the first time `mintlify dev` runs
locally (device-code login in the terminal) — this is separate from
deploying the site, which needs the GitHub App connected (see below).

## Deploying

1. Push this repo to GitHub.
2. In the Mintlify dashboard, connect this repo — it installs a GitHub App
   and redeploys automatically on every push to the default branch. No CI
   config needed for the common case.
3. Point your domain (e.g. `developers.rexmfbank.com`) at it from the
   Mintlify dashboard's custom domain settings.

## Keeping this in sync with `rexmobileapp`

There is deliberately no automated spec generation — see
`docs/baas/api-docs-site-plan.md` §1 in `rexmobileapp` for why. The
workflow:

1. A PR in `rexmobileapp` changes a route/controller under
   `app/Http/Controllers/API/BaaS/V1/*` or `routes/baas.php`.
2. The same PR (or a same-day follow-up) updates the matching operation in
   this repo's `openapi.yaml`, and the relevant `.mdx` guide if the change
   affects one (a new webhook event → `docs/webhooks/events.mdx`, a new
   error code → the operation's own description, etc.).
3. Push. Mintlify redeploys automatically.

## ⚠️ Before you publish this to real partners

Two things this documentation surfaces that are worth resolving in
`rexmobileapp` first, not just documenting around:

1. **API keys aren't wired to anything.**
   `POST /client/onboarding/apikeys` issues a real-looking
   `sk_live_*`/`sk_sandbox_*` secret, but no middleware or guard anywhere in
   the app actually checks it — every endpoint, including every service
   call, only accepts the JWT bearer token from `POST client/onboarding/login`.
   `docs/authentication.mdx` documents this honestly with a callout rather
   than describing the key as a working credential, but a partner reading
   it will reasonably ask "why do you hand me a secret key that does
   nothing?" — worth fixing (wire the key into `BaasPartnerAuth`) or
   pulling the endpoint before this goes external.
2. **No refresh token, and a 2-hour session (`JWT_TTL`).** A
   server-to-server integration has no clean way to stay authenticated
   indefinitely without storing your partner's login password to
   auto-relogin. Documented in `docs/authentication.mdx`, but this is the
   kind of thing that usually becomes a support-ticket generator once
   partners actually build against it.

Everything else checked out as accurate and fully implemented while
writing this — in particular, webhook HMAC signing
(`app/Jobs/SendBaasWebhookJob.php`) is real and correctly documented in
`docs/webhooks/signature-verification.mdx`, not a stub.
