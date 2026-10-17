# Changelog

## 2.0.0 — Cloudflare-native release candidate rebuild

- Replaced the Express/Redis/Docker-socket architecture with a Cloudflare Worker and Static Assets.
- Added Cloudflare Access JWT cryptographic validation using the Access JWKS endpoint.
- Added a server-side read-only inventory for Workers scripts, D1 databases, and KV namespaces.
- Restricted API operations to fixed GET endpoints; removed arbitrary host/container mutations.
- Added security headers, no-store behavior, outbound API timeouts, API pagination limits, and sanitized errors.
- Added GitHub Actions CI/deploy workflows, dependency audit, Wrangler dry-run, and Dependabot configuration.
- Removed VPS-specific Docker Compose, Dockerfile, Caddy and Express backend files from the Cloudflare-only release.

This release requires real account acceptance testing before it is labeled production-ready.
