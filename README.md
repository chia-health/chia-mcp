# Chia Health MCP Server

[![License](https://img.shields.io/badge/License-Apache_2.0-blue.svg)](https://github.com/chia-health/chia-mcp/blob/main/LICENSE)
[![MCP](https://img.shields.io/badge/MCP-streamable--http-green.svg)](https://mcp.chia.health/)
[![HIPAA](https://img.shields.io/badge/HIPAA-compliant-brightgreen.svg)](#hipaa-compliance)
[![Tools](https://img.shields.io/badge/Tools-34-orange.svg)](#tool-catalog)
[![Stripe ACP](https://img.shields.io/badge/Stripe-ACP-blueviolet.svg)](#stripe-acp-integration)

MCP (Model Context Protocol) server for the **[Chia Health](https://chia.health)** telehealth prescription platform. Enables AI assistants (ChatGPT, Claude, Gemini, OpenClaw, Copilot, and custom agents) to help patients browse medications, complete medical intake, sign consent documents, place orders, pay, and manage their treatment — all through natural conversation.

Available treatments include GLP-1 medications (semaglutide, tirzepatide including tablets), peptide therapies (sermorelin, NAD+, glutathione), and longevity programs. All prescriptions are evaluated by licensed US healthcare providers and delivered from FDA-regulated 503A compounding pharmacies across all 50 US states + DC.

## Getting Started

doctormcp is a **remote MCP server** — connect over the network, no local installation required.

**Server URL:** `https://mcp.chia.health/`
**Transport:** Streamable HTTP
**Metadata:** `https://mcp.chia.health/server.json`

<details>
<summary><strong>Claude Desktop</strong></summary>

Add to `~/Library/Application Support/Claude/claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "chia-health": {
      "url": "https://mcp.chia.health/"
    }
  }
}
```

</details>

<details>
<summary><strong>Cursor</strong></summary>

Add to `.cursor/mcp.json` in your project or `~/.cursor/mcp.json` globally:

```json
{
  "mcpServers": {
    "chia-health": {
      "url": "https://mcp.chia.health/"
    }
  }
}
```

</details>

<details>
<summary><strong>Cline / VS Code</strong></summary>

Add to your Cline MCP settings:

```json
{
  "mcpServers": {
    "chia-health": {
      "url": "https://mcp.chia.health/"
    }
  }
}
```

</details>

<details>
<summary><strong>Windsurf</strong></summary>

Add to `~/.codeium/windsurf/mcp_config.json`:

```json
{
  "mcpServers": {
    "chia-health": {
      "serverUrl": "https://mcp.chia.health/"
    }
  }
}
```

</details>

<details>
<summary><strong>Other MCP Clients</strong></summary>

Any MCP client that supports streamable HTTP transport can connect:

- **Server URL:** `https://mcp.chia.health/`
- **Transport:** Streamable HTTP
- **Server metadata:** `https://mcp.chia.health/server.json`

</details>

### Verify Connection

Once connected, your AI assistant can immediately call these public tools (no auth required):

```
medications.categories     → medication categories
medications.list           → all medications with pricing
medications.availability   → check if a medication ships to your state
eligibility.check          → pre-screen age, state, BMI
```

### What Can Your AI Assistant Do?

| Category | Tools | Auth |
|----------|-------|------|
| **Auth** — OTP verification, session management, payment detection | 4 | No* |
| **Discovery** — browse medications, pricing, availability | 5 | No |
| **Qualification** — eligibility checks, intake questionnaires | 4 | Partial |
| **Consent** — present and sign consent documents | 4 | Guest |
| **Ordering** — place orders, upload ID verification | 4 | Guest |
| **Checkout** — Stripe ACP payments or authenticated payment links | 5 | Guest |
| **Patient Portal** — log weight, message provider, refills | 6 | Full |
| **Provider** — answer follow-up questions from your provider | 2 | Guest |

\* Auth tools use `session_id` (no token) except `auth.check_payment` which uses a bearer token.

> **Important:** All prescriptions are evaluated and approved by licensed US healthcare providers. doctormcp facilitates the patient workflow — it does not make clinical decisions.

### Authentication

Patients can **browse freely without authentication** — discovery, eligibility, and intake question preview are all public.

When the patient is ready to proceed with their medical intake, they verify their email:

1. `auth.start(email, phone, name)` → sends OTP to email, returns `session_id`
2. `auth.verify_otp(session_id, code)` → returns guest-scoped bearer token

The guest token enables intake, consent, ordering, and checkout. After payment, `auth.check_payment` upgrades the token to full scope for portal access (care plan, messaging, refills).

## Tool Catalog

### Auth

| Tool | Auth | Description |
|---|---|---|
| `auth.start` | No | Send OTP to patient's email, get `session_id` |
| `auth.verify_otp` | No | Verify code, get guest-scoped bearer token |
| `auth.resend_otp` | No | Resend OTP if expired or not received |
| `auth.check_payment` | Guest | Poll for payment, upgrade token to full scope |

### Discovery (public, no auth)

| Tool | Description |
|---|---|
| `medications.list` | List all medications with categories, forms, and pricing |
| `medications.details` | Detailed info for a specific medication (plans, pricing, what's included) |
| `medications.availability` | Check if a medication ships to a given state |
| `medications.pricing` | Price breakdown for a specific medication/form/plan combo |
| `medications.categories` | List medication categories (Weight Loss, Peptides, Anti-Aging, etc.) |

### Qualification (public + guest)

| Tool | Auth | Description |
|---|---|---|
| `eligibility.check` | No | Pre-screen age, state, BMI, and medical conditions |
| `intake.questions` | No | Get the structured intake questionnaire for a medication |
| `intake.submit` | Guest | Submit completed intake for provider review |
| `intake.status` | Guest | Check intake review status (under review, approved, denied) |

### Consent (guest token required)

| Tool | Description |
|---|---|
| `consent.list` | List all 5 consent documents needed for an intake |
| `consent.text` | Get full verbatim text of a consent document |
| `consent.submit` | Record patient's consent confirmation with audit trail |
| `consent.status` | Check which consents are complete/pending |

### Ordering (guest token required)

| Tool | Description |
|---|---|
| `order.create` | Create a medication order (requires all consents complete) |
| `order.status` | Check order status and tracking info |
| `order.documents` | List required ID documents for an order |
| `order.upload` | Upload photo ID or selfie for identity verification |

### Checkout (guest token required)

| Tool | Description |
|---|---|
| `checkout.create` | Create checkout; returns authenticated payment link (magic-link URL) |
| `checkout.update` | Update a pending checkout (promo codes, shipping) |
| `checkout.complete` | Complete payment with a Stripe Shared Payment Token (ACP path) |
| `checkout.status` | Poll payment status after sending a payment link (fallback path) |
| `checkout.cancel` | Cancel an in-progress checkout |

### Patient Portal (full token required — after payment)

| Tool | Description |
|---|---|
| `portal.log_weight` | Log weight for progress tracking |
| `portal.log_side_effects` | Report side effects (severe = auto-flagged for provider) |
| `portal.message` | Send a message to the healthcare provider |
| `portal.care_plan` | Get current medication, dosing, and weight progress |
| `portal.refill` | Request a medication refill |
| `portal.support` | Create a customer support ticket |

### Provider (guest token required)

| Tool | Description |
|---|---|
| `provider.questions` | Get follow-up questions from the provider |
| `provider.respond` | Submit answers to provider questions |

## Example Agent Flows

### 1. Browse Medications (no auth)

```
User: "What weight loss medications do you offer?"

Agent calls: medications.list
→ Returns categories with semaglutide, tirzepatide, etc.

Agent calls: medications.details(medication="semaglutide-injectable")
→ Returns plans (1-month $349, 4-month $299/mo, 6-month $249/mo)

Agent calls: medications.availability(medication="semaglutide-injectable", state="TX")
→ { "available": true }

Agent calls: eligibility.check(age=35, state="TX", bmi=31.2)
→ { "eligible": true, "available_medications": [...] }
```

### 2. Full Ordering Flow (auth → intake → consent → order → pay)

```
== VERIFY IDENTITY ==

1. auth.start(email="patient@example.com", phone="5551234567", first_name="Jane")
   → { "session_id": "abc123...", "otp_sent": true }

2. auth.verify_otp(session_id="abc123...", code="847293")
   → { "guest_token": "mcp_...", "scope": "guest" }

== MEDICAL INTAKE ==

3. intake.questions(medication="semaglutide-injectable")
   → Structured questionnaire (demographics, vitals, medical history, etc.)
   → Agent asks patient each question conversationally

4. intake.submit(patient_email, patient_name, answers, bearer_token)
   → { "intake_id": "42", "next_step": "get_required_consents" }

5. consent.list(intake_id="42", bearer_token)
   → 5 consent documents (telehealth, treatment, pharmacy, HIPAA, AI disclosure)

6. For each consent:
   a. consent.text(consent_id, bearer_token)
      → Full text the agent MUST present verbatim
   b. Patient confirms: "I agree"
   c. consent.submit(intake_id, consent_id, "I agree", bearer_token)

== ORDER & PAY ==

7. order.create(intake_id, medication, form, plan_months, shipping_address, bearer_token)
   → { "order_id": "99", "total": "1079.39", "next_step": "create_checkout" }

8. checkout.create(order_id="99", bearer_token)
   → { "checkout_id": "7", "payment_url": "https://chia.health/checkout/aBc-_xyz" }
   → Agent shares link with patient; also sent via email+SMS

9a. (ACP path) checkout.complete(checkout_id="7", shared_payment_token="spt_...", bearer_token)
    → { "payment_status": "success", "confirmation_number": "CHIA-000099" }

9b. (Fallback) Patient opens payment_url in browser, pays on Chia checkout page

10. auth.check_payment(bearer_token)
    → { "paid": true, "scope": "full" }  // token upgraded, portal unlocked
```

### 3. Patient Portal (full token, after payment)

```
1. portal.log_weight(patient_id, weight_lbs=195.5, date="2026-06-15", bearer_token)
   → { "recorded": true }

2. portal.care_plan(patient_id, bearer_token)
   → Current medication, phase, dosing schedule, recent weights

3. portal.log_side_effects(patient_id, effects=["nausea"], severity="mild", bearer_token)
   → { "recorded": true, "flagged_for_review": false }

4. portal.message(patient_id, message="Nausea improving", bearer_token)
   → { "sent": true, "estimated_response_time": "24-48 hours" }
```

## Stripe ACP Integration

doctormcp uses [Stripe's Agentic Commerce Protocol (ACP)](https://docs.stripe.com/acp) for payment processing:

1. **Order Creation** — `order.create` calculates the total and creates a pre-payment order record.
2. **Checkout Initiation** — `checkout.create` creates a Stripe `PaymentIntent` (for ACP) and generates an authenticated payment link. The link auto-authenticates patients in the browser and lands on the Chia Health checkout page. Also sent via email and SMS.
3. **Payment Completion** — Two paths:
   - **ACP**: `checkout.complete` accepts a **Shared Payment Token (SPT)** from the AI platform and confirms the `PaymentIntent`. Instant, in-conversation payment.
   - **Fallback**: The agent shares the `payment_url` (authenticated magic-link) with the patient. The patient opens it in their browser, auto-authenticates, and pays on the Chia Health checkout page. The agent polls `auth.check_payment` to detect completion.
4. **Post-Payment** — On success (either path), a `Subscription` and `Enrollment` are created automatically. `auth.check_payment` upgrades the agent's token to full scope for portal access.

## HIPAA Compliance

- **Audit Logging** — Every access to protected health information (PHI) is logged with actor identity, action type, resource, IP address, and timestamp. Logs are retained for 10 years.
- **Consent Records** — All patient consent confirmations include verbatim confirmation text, method (AI agent conversational), platform, session ID, and IP address. Consent records are immutable.
- **Input Sanitization** — All inputs are validated and sanitized. Control characters are stripped, lengths enforced, and domain-specific formats (email, state, phone, ZIP) are validated.
- **OTP-First Auth** — Email ownership verified via 6-digit OTP before any patient data access. SHA-256 hashed tokens with scoped access (guest/full). No token issued without email verification.
- **Rate Limiting** — Redis-backed token-bucket rate limiting per user and endpoint category (public: 100/min, auth: 5/min, authenticated: 30/min, consent: 10/min, checkout: 5/min).
- **Minimal Data Exposure** — Tools return only the data needed for the current step. Sensitive fields (payment details, full SSN) are never returned.

## Support

- **Issues:** [github.com/chia-health/chia-mcp/issues](https://github.com/chia-health/chia-mcp/issues)
- **Website:** [chia.health](https://chia.health)
- **Email:** engineering@chia.health

## License

Apache License 2.0 — see [LICENSE](LICENSE) for details.

Copyright 2026 Chia Health, Inc.
