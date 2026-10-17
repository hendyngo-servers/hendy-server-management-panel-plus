# Cloudflare Worker implementation

The Worker serves the static dashboard and implements a small read-only API gateway to the Cloudflare API. It does not proxy arbitrary URLs, accept upstream origins from a request, connect to a VPS, use Docker socket, or depend on Express/Redis/Socket.IO.

## Environment secrets

- `CF_ACCOUNT_ID` — Cloudflare account ID (32 hex characters).
- `CF_API_READ_TOKEN` — separate account-scoped runtime token with Workers Scripts Read, D1 Read and Workers KV Storage Read.
- `ACCESS_ISSUER` — HTTPS Cloudflare Access team issuer URL.
- `ACCESS_AUD` — Audience (AUD) identifier for the Access application protecting the dashboard.

Set the values in GitHub environment secrets; deployment workflow uploads them to the Worker as runtime secrets. Do not use the deployment token as the runtime API token.

## Routes

- `GET /healthz` — public liveness response only.
- `GET /api/overview` — requires a valid `Cf-Access-Jwt-Assertion`; collects a projected inventory of Workers scripts, D1 databases and KV namespaces.
- Other `/api/*` routes return `404`. Mutating HTTP methods return `405`.
- Static assets are gated by Access JWT validation too. Use the custom hostname protected by Cloudflare Access; direct access through `workers.dev` must not return the dashboard unless a valid JWT is present.

## Authentication safety

The Worker validates RS256 signatures with the configured Access JWKS, then checks `iss`, `aud`, `exp`, `nbf`, and `iat`. The JWKS URL is derived only from trusted Worker configuration. Raw JWTs, API tokens, and provider error details are never returned to the browser or application logs.

## Verification

`node --test` uses generated RSA keys and mocked Cloudflare endpoints to test authentication, resource projection, method restrictions and sanitized provider errors without requiring real account credentials. A live deployment still requires testing the actual Access application, token scopes, custom hostname, and WAF configuration.
