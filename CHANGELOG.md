# Changelog

All notable changes to this template are documented here. Format based on
[Keep a Changelog](https://keepachangelog.com/en/1.1.0/); versions follow
[Semantic Versioning](https://semver.org/). Release notes for tagged
versions are also generated on the GitHub Releases page by semantic-release.

## [Unreleased]

## [1.0.0] - 2026-11-06

First public release.

### Added
- Monorepo (pnpm workspaces): Hono + Prisma 7 + Postgres API, Next.js +
  shadcn/ui + Clerk web, shared zod types.
- Layered API (route → service → repository) with the `Result<T>` pattern
  and OpenAPI/Scalar docs generated from zod.
- Redis L1/L2 caching, rate limiting, structured logging (pino).
- GlitchTip/Sentry error monitoring with PII scrubbing (source maps off by
  default, no `SENTRY_AUTH_TOKEN` required).
- Reusable web UI kit: pagination, tables, forms, image uploader, error
  boundaries.
- CI (lint, check, typecheck, tests, build) that needs no secrets; opt-in
  release workflows (`ENABLE_RELEASES`) deploying via Docker Hub + Dokploy.
- MIT license, code of conduct, security policy, issue and PR templates.
- Docs: README, architecture, contributing guide, secrets reference,
  GitHub setup guide.

[Unreleased]: https://github.com/morvenlabs/web-project-template/compare/1.0.0...HEAD
[1.0.0]: https://github.com/morvenlabs/web-project-template/releases/tag/1.0.0
