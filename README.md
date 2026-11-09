# Rental-log (Monorepo)

```text
apps/
 ├─ web    (React/Vite)            # main app; still serves /admin for compatibility
 ├─ api    (Cloudflare Hono worker + Express/Node fallback)
 └─ admin  (standalone Vite admin; currently reuses web admin modules)

packages/ ui, database, auth, validation
infra/    wrangler.toml, migrations/, schema.sql, github-actions/, supabase-legacy/
```

## Key commands
- `npm install`
- `npm run dev` / `npm run dev:web` — web
- `npm run dev -w @rental-log/admin` — admin
- `npm run dev -w @rental-log/api` — Workers (wrangler); `npm run dev:api:node` — Express fallback
- `npm run lint` — web + api type checks
- `npm test -w @rental-log/web`; `npx tsx apps/api/src/worker.test.ts`; `npx tsx apps/api/src/routes.test.ts`
- `npm run build` (web), `npm run build -w @rental-log/admin`, `npm run build:api` / `npm run deploy:api`

## Cloudflare deployment
This project is configured for Cloudflare Pages + Workers, not Render.

### Front-end apps
- `apps/web` deploys to a Cloudflare Pages project
- `apps/admin` deploys to a second Cloudflare Pages project
- both apps use Vite and require the build output directory `dist`

### API
- `apps/api` deploys to a Cloudflare Worker using `apps/api/wrangler.toml`
- route base: `/api/v1`
- runtime: Workers + Hono

### Required repo variables/secrets
Add the following to GitHub Actions: 
- `CLOUDFLARE_API_TOKEN`
- `CLOUDFLARE_ACCOUNT_ID`
- `CF_PAGES_WEB_PROJECT`
- `CF_PAGES_ADMIN_PROJECT`
- `VITE_API_BASE_URL` = `/api/v1`
- `VITE_WEB_URL` = `/`

### Local cloudflare checks
```bash
npm install
npm run build -w @rental-log/web
npm run build -w @rental-log/admin
npx --yes wrangler@4 deploy --dry-run --config apps/api/wrangler.toml
```

### Worker + Pages wiring
- The web/admin front ends must point at the Worker URL or the same-origin `/api/v1` route.
- In Cloudflare, create the Pages projects and add the worker route for `/api/*` to the Worker.
- The repo includes SPA fallback redirect files in `apps/web/public/_redirects` and `apps/admin/public/_redirects` so client-side routes keep working on Pages.

## Status
Workers/Hono has core routes migrated (health/config/blog/team/listings/auth/profile/portfolio/distributions subset) with 501 fallback for unmigrated legacy endpoints; Express `apps/api/src/routes.ts` remains the Node fallback for durable ledger/email flows until fully migrated.
