# Release status — v2.0.0 Cloudflare-native rebuild

## Current artifact status

This source tree is a Cloudflare-native rebuild; the previous Express + Redis + Docker socket server is not part of this release package. The panel inventories Cloudflare Workers scripts, D1 databases and KV namespaces through a server-side read-only API token.

### Automated source checks

Run the following in CI or locally after installing from the lockfile:

- `npm ci`
- `npm run check`
- `npm audit --audit-level=high`
- `npm run deploy:dry-run` (requires reaching the npm registry to download the pinned Wrangler CLI)

The tests exercise Access JWT validation, rejection of unauthenticated requests, read-only method enforcement, fixed Cloudflare API endpoints, resource projection and static-asset access controls with mocked Cloudflare API responses.

### Must verify before public production release

- [ ] CI and Wrangler dry-run succeed using the committed lockfile.
- [ ] Cloudflare Access app is configured with a restrictive allow policy.
- [ ] Custom domain is attached to the Worker and the Access app covers the exact hostname.
- [ ] Runtime API token is read-only and separate from the deployment token.
- [ ] Browser UI shows only allowed account resources; forbidden user and direct `workers.dev` access reveal no private data.
- [ ] WAF rate limiting and Worker Observability are enabled.
- [ ] GitHub `main` protection and production environment approvals are enabled.
- [ ] Post-deploy smoke test succeeds and rollback procedure is known.

## Release gate

Do not mark this release production-ready until the target Cloudflare account has passed the checks above. The dry-run could not complete in this build environment because downloading Wrangler from the npm registry timed out. Passing local unit tests does not prove real Cloudflare Access, API permissions, account routing, production quotas, or live deployment. The new dashboard is read-only and does not manage arbitrary Docker hosts or expose resource mutation operations.

### Verification in this authoring environment (2026-10-17)

- `npm ci --ignore-scripts`: passed.
- `npm run check`: passed; 7/7 unit tests.
- `npm audit --audit-level=high`: passed with 0 issues reported for the repository's npm dependency tree (the Worker itself declares no runtime npm dependencies).
- Wrangler `deploy --dry-run`: attempted but timed out while obtaining the pinned Wrangler CLI from npm; this remains unverified here and must pass on GitHub Actions.
- Live Cloudflare Access/API/hostname deployment: not run because no account credentials were provided to this build environment.
