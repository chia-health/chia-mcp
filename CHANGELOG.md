# Changelog

All notable changes to doctormcp will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.1.0] - 2026-03-31

### Added
- 4 new auth tools: `auth.start`, `auth.verify_otp`, `auth.resend_otp`, `auth.check_payment` (34 tools total)
- OTP-first authentication flow — email ownership verified before any patient data access
- Guest/full token scopes — guest for intake→checkout, full for portal (after payment)
- `auth.check_payment` tool — polls for active enrollment, upgrades token scope in-place (no second OTP)
- Authenticated magic-link checkout URLs — auto-authenticate patients in browser, also sent via email+SMS
- Browse-first UX — discovery, eligibility, and intake question preview all public with no auth required
- Auth rate limiting: 5 requests/min per IP for auth tools, 3/min for OTP resend

### Changed
- `checkout.create` now returns a magic-link URL (e.g. `https://chia.health/checkout/aBc-_xyz`) instead of Stripe-hosted checkout page
- Token issuance requires OTP verification (previously tokens were pre-issued without email proof)
- Portal tools (`portal.*`) now enforce `scope=full` — guest tokens are rejected with guidance to call `auth.check_payment`
- Updated server instructions and get-started prompt for browse-first phasing

### Security
- Fixed account impersonation vulnerability: `auth.start` previously issued tokens for any email without verification
- OTP codes: 6-digit, cryptographically random, SHA-256 hashed, 5-minute expiry, 3 attempts max
- Session IDs stored in Redis with 10-minute TTL — opaque, cannot access any data
- Race-safe OTP verification via `select_for_update` + `transaction.atomic`

## [1.0.0] - 2026-03-17

### Added
- 30 MCP tools across 7 categories: Discovery, Qualification, Consent, Ordering, Checkout, Patient Portal, Provider
- Streamable HTTP transport via FastMCP + Starlette at `https://mcp.chia.health/`
- Bearer token authentication with SHA-256 hashing and configurable TTL
- HIPAA audit logging with 10-year retention (`HIPAAAuditLog` model)
- 5 consent documents with verbatim presentation and audit trail
- Redis-backed token-bucket rate limiting (public: 100/min, authenticated: 30/min, consent: 10/min, checkout: 5/min)
- Stripe Agentic Commerce Protocol (ACP) integration for in-conversation payments with fallback payment links
- Patient data isolation (token-scoped access — patient A cannot access patient B's data)
- Input validation and sanitization for all user-provided data
- Health check endpoint (`GET /health`) and server metadata endpoint (`GET /server.json`)
- DNS rebinding protection via configurable allowed hosts
- Multi-stage Dockerfile with non-root user (`appuser`)
- PostHog analytics integration (optional, non-fatal)
- Full test suite covering services, auth, validation, tools, checkout, audit, and isolation
