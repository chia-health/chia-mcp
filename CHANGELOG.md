# Changelog

All notable changes to doctormcp will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

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
