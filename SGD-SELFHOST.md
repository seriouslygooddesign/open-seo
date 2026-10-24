# OpenSEO at SGD

SGD's self-hosted OpenSEO, live at **https://openseo.sgd.com.au**.
Fork of [every-app/open-seo](https://github.com/every-app/open-seo), deployed to SGD's Cloudflare account.

## How it works

- **Hosting:** Cloudflare Workers + D1 + KV + R2 on the SGD account, via OpenSEO's own `pnpm deploy:selfhost`.
- **Login:** Cloudflare Access on `openseo.sgd.com.au`. Only emails in `ACCESS_ALLOWED_EMAILS` get in (one-time code by email). The app also verifies the Access token itself.
- **Data:** SGD's DataForSEO account (`DATAFORSEO_API_KEY`, base64 of `login:password`). Billed by DataForSEO directly.
- **Updates:** `.github/workflows/sgd-selfhost-update.yml` runs every Monday 3am Melbourne, on demand, and on every push to `main`. It merges upstream `main` on top of ours, builds, deploys, and checks the site is live behind Access. A conflict or failed build stops before deploying. Each deploy is logged on the `deploy-log` branch.

## SGD changes to upstream code

Keep these small so upstream merges stay clean.

1. `deploy/alchemy/alchemy.run.ts`: `SELFHOST_DOMAIN` env var serves the app on a custom hostname, points the Access gate at it, and turns off the workers.dev URL.
2. `deploy/.env.selfhost.example`: documents `SELFHOST_DOMAIN`.
3. This file and the update workflow.

New SGD features: build them on a branch, merge to `main`, and the workflow deploys them.

## Common tasks

**Add a teammate:** add their email to `ACCESS_ALLOWED_EMAILS` in `.env.selfhost`, update the `ENV_SELFHOST` secret (`gh secret set ENV_SELFHOST < .env.selfhost`), then run the workflow (Actions tab, "Run workflow").

**Update now:** Actions tab, "SGD self-host update", "Run workflow".

**Deploy from a laptop:** `pnpm install && pnpm deploy:selfhost --yes` (needs `.env.selfhost` and `pnpm alchemy login`).

**Upstream conflict:** `git fetch upstream && git merge upstream/main`, resolve, push to `main`.

**Logs:** Cloudflare dashboard, Workers, `open-seo-selfhost`, Logs. Or `pnpm exec wrangler tail open-seo-selfhost`.

## Cloudflare API token for GitHub Actions

Account permissions: Workers Scripts Edit, Workers KV Storage Edit, D1 Edit, Workers R2 Storage Edit, Secrets Store Read, Account Settings Read, Access: Apps and Policies Edit, Access: Organizations, Identity Providers, and Groups Read, Browser Rendering Edit.
Zone permissions (sgd.com.au): Zone Read, Workers Routes Edit, DNS Edit.
