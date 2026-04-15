# Cloudflare Workers deployment checklist (Next.js SSR)

## 1) Install tooling

```bash
yarn add -D @opennextjs/cloudflare wrangler
```

## 2) Configure bindings

1. Create KV namespace for SSR cache:

```bash
wrangler kv namespace create NEXT_CACHE_KV
wrangler kv namespace create NEXT_CACHE_KV --preview
```

2. Create R2 bucket (optional but recommended):

```bash
wrangler r2 bucket create b2c-storefront-prod
wrangler r2 bucket create b2c-storefront-preview
```

3. Fill `wrangler.toml` IDs and bucket names.

## 3) Configure secrets

Use `.dev.vars.example` as template for local development.

For production, set secrets via Wrangler:

```bash
wrangler secret put MEDUSA_BACKEND_URL
wrangler secret put NEXT_PUBLIC_MEDUSA_PUBLISHABLE_KEY
wrangler secret put REVALIDATE_SECRET
wrangler secret put NEXT_PUBLIC_STRIPE_KEY
wrangler secret put NEXT_PUBLIC_ALGOLIA_ID
wrangler secret put NEXT_PUBLIC_ALGOLIA_SEARCH_KEY
wrangler secret put NEXT_PUBLIC_TALKJS_APP_ID
```

## 4) Build / preview / deploy

```bash
yarn cf:build
yarn cf:preview
yarn cf:deploy
```

Optional environment deploy:

```bash
CF_ENV=production yarn cf:deploy:env
```

`cf:deploy` always runs the build first and verifies `.open-next/worker.js` before calling Wrangler deploy.

---

## Troubleshooting runbook

### A. `Error: Cannot find module 'opennextjs-cloudflare'`
- Cause: dependency not installed.
- Fix: run `yarn add -D @opennextjs/cloudflare wrangler`.
- Then rerun `yarn cf:build` (which executes `npx opennextjs-cloudflare build`).

### B. Worker starts but pages 500 on SSR
- Confirm `compatibility_flags` includes `nodejs_compat` in `wrangler.toml`.
- Confirm required env vars are present (`MEDUSA_BACKEND_URL`, publishable key, etc.).
- Check logs:

```bash
wrangler tail
```

### C. KV binding errors (`NEXT_CACHE_KV is not defined`)
- Confirm `[[kv_namespaces]]` binding name is exactly `NEXT_CACHE_KV`.
- Confirm both production `id` and `preview_id` are filled.
- Re-run `wrangler dev` after updating `wrangler.toml`.

### D. R2 binding errors (`NEXT_R2 is not defined`)
- Confirm `[[r2_buckets]]` binding name is `NEXT_R2`.
- Ensure bucket names exist in your Cloudflare account.

### E. Font/network fetch failures during build
- This often comes from restricted outbound networking (e.g. `fonts.googleapis.com`).
- Retry in a network-allowed environment or self-host fonts.

### F. Auth redirect loop in middleware
- Validate `NEXT_PUBLIC_DEFAULT_REGION` and locale path behavior.
- Verify `_medusa_jwt` cookie exists and token is not expired.

### G. `wrangler dev` local env not loaded
- Ensure you copied `.dev.vars.example` to `.dev.vars`.
- Ensure variable names match exactly with app expectations.
