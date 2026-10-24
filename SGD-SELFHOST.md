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

1. `deploy/alchemy/alchemy.run.ts`: `SELFHOST_DOMAIN` + `SELFHOST_ZONE_ID` serve the app on a custom hostname via a Worker route, point the Access gate at it, and turn off the workers.dev URL. It uses a route, not alchemy's `domain`, because alchemy only checks the first page of zones and SGD has 139. The route relies on a proxied DNS record `openseo.sgd.com.au AAAA 100::` (created 25 Oct 2026).
2. `deploy/.env.selfhost.example`: documents `SELFHOST_DOMAIN`.
3. `.claude-plugin/marketplace.json` + `plugins/openseo/.mcp.json`: the Claude plugin is republished as marketplace `sgd-openseo`, with its MCP pointed at `https://openseo.sgd.com.au/mcp`. Install with `claude plugin marketplace add seriouslygooddesign/open-seo` then `claude plugin install openseo@sgd-openseo`.
4. This file and the update workflow (which also pushes the upstream merge to `main` when it can, so plugin skills stay current).

New SGD features: build them on a branch, merge to `main`, and the workflow deploys them.

## Common tasks

**Add a teammate:** add their email to `ACCESS_ALLOWED_EMAILS` in `.env.selfhost`, update the `ENV_SELFHOST` secret (`gh secret set ENV_SELFHOST < .env.selfhost`), then run the workflow (Actions tab, "Run workflow").

**Update now:** Actions tab, "SGD self-host update", "Run workflow".

**Deploy from a laptop:** `pnpm install && pnpm deploy:selfhost --yes` (needs `.env.selfhost` and a one-off `pnpm alchemy login deploy/alchemy/alchemy.run.ts --configure`, then `pnpm alchemy cloudflare bootstrap`). Plain `pnpm alchemy login` from the upstream docs fails: it looks for `alchemy.run.ts` in the repo root.

**Upstream conflict:** `git fetch upstream && git merge upstream/main`, resolve, push to `main`.

**Logs:** Cloudflare dashboard, Workers, `open-seo-selfhost`, Logs. Or `pnpm exec wrangler tail open-seo-selfhost`.

## MCP login (Managed OAuth)

Claude signs in to `/mcp` through Cloudflare Access Managed OAuth, enabled on the `open-seo selfhost` Access app (dynamic client registration, localhost and loopback allowed). Turned on via the API on 25 Oct 2026. Deploys leave it alone unless the app's domain or name changes; if MCP login breaks after such a change, re-enable it in Zero Trust, Access, Applications, Edit, Additional settings, OAuth.

## Cloudflare API token for GitHub Actions

One token serves both local deploys (`alchemy login`, choose "API Token") and GitHub Actions.

Account permissions: Workers Scripts Edit, Workers KV Storage Edit, D1 Edit, Workers R2 Storage Edit, Secrets Store Edit, Account Settings Read, Access: Apps and Policies Edit, Access: Organizations, Identity Providers, and Groups Edit, Browser Rendering Edit.
Zone permissions (sgd.com.au): Zone Read, Workers Routes Edit, DNS Edit.
User permissions: User Details Read, Memberships Read.
