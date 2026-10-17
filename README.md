# Hendy Server Management Panel Plus v2.0.0

Cloudflare-native control panel for viewing Cloudflare Workers, D1 databases, and KV namespaces. The dashboard is served by a Cloudflare Worker and deployed from GitHub Actions. It does not require a self-managed VPS, Redis, Express server, Docker daemon, or Docker socket.

> **Scope:** this version is a read-only Cloudflare account inventory. It is not a general-purpose Docker manager and cannot administer arbitrary containers on third-party Linux servers. No resource creation, deletion, SQL execution, KV value access, or Worker mutation API is exposed from the dashboard.

## Architecture

- **GitHub** — source control, pull requests, dependency updates, CI and deployment automation.
- **Cloudflare Workers + Static Assets** — frontend and server-side API gateway.
- **Cloudflare Access** — identity gate; the Worker independently validates the signed Access JWT and its configured issuer/audience on every non-health request.
- **Cloudflare API** — read-only inventory of Workers scripts, D1 databases and KV namespaces using a separate runtime token.
- **Workers Logs / Observability** — operational request records without logging the access token or API token.

## First deployment

### 1. Create a Cloudflare Access application

1. In Cloudflare One, create an Access self-hosted application for the custom hostname that will serve this Worker. Use an allow policy restricted to named users or your identity-provider group; do not use an unrestricted `Everyone` policy.
2. Attach a custom domain to the Worker and protect that hostname with the Access application.
3. Copy the exact **Team domain / issuer URL** (for example `https://your-team.cloudflareaccess.com`) and the application's **Audience (AUD) tag**.
4. The Worker returns `401` unless `Cf-Access-Jwt-Assertion` has a valid RS256 signature, issuer, audience, expiry and not-before claims. Requests to the `workers.dev` hostname are also blocked unless they contain a valid token; use the Access-protected custom hostname for normal access.

### 2. Create two different Cloudflare API tokens

**Runtime read-only token** — scoped to the target account with only the read permissions needed to inventory:

- Workers Scripts: Read
- D1: Read
- Workers KV Storage: Read

The exact permission labels can differ slightly in the Cloudflare dashboard. Do not grant edit, delete, account-wide administrator, or global API-key access to this runtime token.

**Deploy token** — separate from the runtime token; scoped to the target account and only the permissions required to upload/deploy the Worker and its assets. Never place this token inside Worker runtime secrets.

### 3. Configure GitHub

In the repository, create the GitHub environment `cloudflare-production`. Add these environment secrets:

| Secret | Purpose |
|---|---|
| `CLOUDFLARE_API_TOKEN` | Deployment-only API token used by Wrangler in GitHub Actions |
| `CLOUDFLARE_ACCOUNT_ID` | The 32-character Cloudflare account ID |
| `CF_API_READ_TOKEN` | Separate read-only token consumed by the deployed Worker |
| `ACCESS_ISSUER` | Exact HTTPS issuer/team domain for Cloudflare Access |
| `ACCESS_AUD` | Access application's Audience (AUD) tag |

Require reviewers for `cloudflare-production` if your GitHub plan supports environment protection. Protect `main` with pull requests and passing checks. Do not commit any token, `.env` file, or account secrets.

### 4. Push and deploy

```bash
npm ci
npm run check
npm run deploy:dry-run
git add .
git commit -m "release: Hendy Server Management Panel Plus v2.0.0"
git push origin main
```

A push to `main`, a `v*` tag, or a manual run triggers `.github/workflows/deploy-cloudflare.yml`. The workflow installs from the lockfile, runs tests, audits dependencies, performs a Wrangler dry-run, then deploys the Worker and sets the runtime secrets. A failed validation blocks deployment.

For a release tag after `main` is verified:

```bash
git tag -a v2.0.0 -m "Hendy Server Management Panel Plus v2.0.0"
git push origin v2.0.0
```

Do not create the release tag until the staging and acceptance checks pass.

## Local development

Requires Node.js 22 or newer.

```bash
npm ci
npm run check
npm run dev
```

A local Worker needs the same Access/JWT settings and a reachable Cloudflare API runtime token to return live inventory. Keep local secrets in `.dev.vars` (gitignored) and never include them in an issue or screenshot:

```dotenv
CF_ACCOUNT_ID=replace-with-32-character-account-id
CF_API_READ_TOKEN=replace-with-read-only-token
ACCESS_ISSUER=https://your-team.cloudflareaccess.com
ACCESS_AUD=replace-with-access-audience-tag
```

Local tests do not require Cloudflare credentials and use synthetic JWTs plus mocked API responses.

## API endpoints

- `GET /healthz` — public liveness endpoint, no account or secret details.
- `GET /api/overview` — Access-protected inventory for Worker scripts, D1 databases and KV namespaces.
- Every other `/api/*` endpoint is closed by default. The API accepts `GET`/`HEAD` only; mutations are not implemented.

## Production hardening checklist

- [ ] Attach the production custom domain and verify the Access application is enforced.
- [ ] Verify an unauthenticated request through both the custom domain and `workers.dev` gets `401` or the Access login flow, never inventory data.
- [ ] Verify allowed users can see inventory and unallowed users cannot access any static asset or API endpoint.
- [ ] Confirm the runtime token is read-only and a different token is used for deployment.
- [ ] Confirm token rotation works: update GitHub secrets and redeploy; do not paste values into source.
- [ ] Add a Cloudflare WAF rate-limiting rule for the dashboard hostname/API and monitor Workers Observability. The in-code JWT check is not a substitute for WAF rate limits.
- [ ] Enable GitHub branch protection and environment approval on production deployments.
- [ ] Review `npm audit`, GitHub Actions run logs, and the release checklist before publishing a tag.
- [ ] Verify the production Worker URL and `/healthz` after each deployment.

## Limitations

This build intentionally avoids arbitrary Docker commands and write-capable Cloudflare API operations. It can inventory account resources only. Cloudflare Containers lifecycle operations, general Docker host management, logs/exec for arbitrary containers, and other write actions require a separately designed and explicitly authorized API surface; they are not silently emulated here.
