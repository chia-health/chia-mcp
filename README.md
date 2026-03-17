# Chia Health MCP Server

[![License](https://img.shields.io/badge/License-Apache_2.0-blue.svg)](https://github.com/chia-health/chia-mcp/blob/main/LICENSE)
[![MCP](https://img.shields.io/badge/MCP-streamable--http-green.svg)](https://mcp.chia.health/)
[![HIPAA](https://img.shields.io/badge/HIPAA-compliant-brightgreen.svg)](#hipaa-compliance)
[![Tools](https://img.shields.io/badge/Tools-30-orange.svg)](#tool-catalog)
[![Stripe ACP](https://img.shields.io/badge/Stripe-ACP-blueviolet.svg)](#stripe-acp-integration)

MCP (Model Context Protocol) server for the **[Chia Health](https://chia.health)** telehealth prescription platform. Enables AI assistants (ChatGPT, Claude, Gemini, Copilot, and custom agents) to help patients browse medications, complete medical intake, sign consent documents, place orders, pay via Stripe ACP, and manage their treatment — all through natural conversation.

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
doctormcp_list_categories     → medication categories
doctormcp_list_medications    → all medications with pricing
doctormcp_check_availability  → check if a medication ships to your state
```

### What Can Your AI Assistant Do?

| Category | Tools | Auth |
|----------|-------|------|
| **Discovery** — browse medications, pricing, availability | 5 | No |
| **Qualification** — eligibility checks, intake questionnaires | 4 | Partial |
| **Consent** — present and sign consent documents | 4 | Yes |
| **Ordering** — place orders, upload ID verification | 4 | Yes |
| **Checkout** — Stripe ACP payments or payment links | 5 | Yes |
| **Patient Portal** — log weight, message provider, refills | 6 | Yes |
| **Provider** — answer follow-up questions from your provider | 2 | Yes |

> **Important:** All prescriptions are evaluated and approved by licensed US healthcare providers. doctormcp facilitates the patient workflow — it does not make clinical decisions.

### Authentication

Discovery tools are **public**. Patient-specific tools require a **bearer token** issued during the intake flow. See the [Tool Catalog](#tool-catalog) for per-tool auth requirements.

## Tool Catalog

### Discovery (public, no auth)

| Tool | Description |
|---|---|
| `doctormcp_list_medications` | List all medications with categories, forms, and pricing |
| `doctormcp_get_medication_details` | Detailed info for a specific medication (plans, pricing, what's included) |
| `doctormcp_check_availability` | Check if a medication ships to a given state |
| `doctormcp_get_pricing` | Price breakdown for a specific medication/form/plan combo |
| `doctormcp_list_categories` | List medication categories (Weight Loss, Peptides, Anti-Aging, etc.) |

### Qualification (public + auth)

| Tool | Auth | Description |
|---|---|---|
| `doctormcp_check_eligibility` | No | Pre-screen age, state, BMI, and medical conditions |
| `doctormcp_get_intake_questions` | No | Get the structured intake questionnaire for a medication |
| `doctormcp_submit_intake` | Yes | Submit completed intake for provider review |
| `doctormcp_get_intake_status` | Yes | Check intake review status (under review, approved, denied) |

### Consent (auth required)

| Tool | Description |
|---|---|
| `doctormcp_get_required_consents` | List all 5 consent documents needed for an intake |
| `doctormcp_get_consent_text` | Get full verbatim text of a consent document |
| `doctormcp_submit_consent` | Record patient's consent confirmation with audit trail |
| `doctormcp_get_consent_status` | Check which consents are complete/pending |

### Ordering (auth required)

| Tool | Description |
|---|---|
| `doctormcp_create_order` | Create a medication order (requires all consents complete) |
| `doctormcp_get_order_status` | Check order status and tracking info |
| `doctormcp_get_required_documents` | List required ID documents for an order |
| `doctormcp_upload_document` | Upload photo ID or selfie for identity verification |

### Checkout / Stripe ACP (auth required)

| Tool | Description |
|---|---|
| `doctormcp_create_checkout` | Create checkout session; returns ACP details + fallback `payment_url` |
| `doctormcp_update_checkout` | Update a pending checkout (promo codes, shipping) |
| `doctormcp_complete_checkout` | Complete payment with a Stripe Shared Payment Token (ACP path) |
| `doctormcp_get_checkout_status` | Poll payment status after sending a payment link (fallback path) |
| `doctormcp_cancel_checkout` | Cancel an in-progress checkout |

### Patient Portal (auth required)

| Tool | Description |
|---|---|
| `doctormcp_log_weight` | Log weight for progress tracking |
| `doctormcp_log_side_effects` | Report side effects (severe = auto-flagged for provider) |
| `doctormcp_message_provider` | Send a message to the healthcare provider |
| `doctormcp_get_care_plan` | Get current medication, dosing, and weight progress |
| `doctormcp_request_refill` | Request a medication refill |
| `doctormcp_contact_support` | Create a customer support ticket |

### Provider (auth required)

| Tool | Description |
|---|---|
| `doctormcp_get_provider_questions` | Get follow-up questions from the provider |
| `doctormcp_submit_provider_response` | Submit answers to provider questions |

## Example Agent Flows

### 1. Discovery Flow (browse medications)

```
User: "What weight loss medications do you offer?"

Agent calls: doctormcp_list_medications
→ Returns categories with semaglutide, tirzepatide, etc.

Agent calls: doctormcp_get_medication_details(medication="semaglutide")
→ Returns plans (1-month $349, 4-month $299/mo, 6-month $249/mo)

Agent calls: doctormcp_check_availability(medication="semaglutide", state="TX")
→ { "available": true }

Agent calls: doctormcp_check_eligibility(age=35, state="TX", bmi=31.2)
→ { "eligible": true, "available_medications": [...] }
```

### 2. Full Ordering Flow (intake → consent → order → checkout → payment)

```
1. doctormcp_get_intake_questions(medication="semaglutide")
   → Structured questionnaire (demographics, vitals, medical history, etc.)

2. doctormcp_submit_intake(patient_email, patient_name, answers, bearer_token)
   → { "intake_id": "42", "next_step": "get_required_consents" }

3. doctormcp_get_required_consents(intake_id="42", bearer_token)
   → 5 consent documents (telehealth, treatment, pharmacy, HIPAA, AI disclosure)

4. For each consent:
   a. doctormcp_get_consent_text(consent_id, bearer_token)
      → Full text the agent MUST present verbatim
   b. Patient confirms: "I agree"
   c. doctormcp_submit_consent(intake_id, consent_id, "I agree", bearer_token)

5. doctormcp_create_order(intake_id, medication, form, plan_months, shipping_address, bearer_token)
   → { "order_id": "99", "total": "1096.00", "next_step": "create_checkout" }

6. doctormcp_create_checkout(order_id="99", bearer_token)
   → { "checkout_id": "7", "total": "$1,096.00", "payment_url": "https://checkout.stripe.com/..." }

7a. (ACP path) doctormcp_complete_checkout(checkout_id="7", shared_payment_token="spt_...", bearer_token)
    → { "payment_status": "success", "confirmation_number": "CHIA-000099" }

7b. (Fallback) Present payment_url to patient, then poll:
    doctormcp_get_checkout_status(checkout_id="7", bearer_token)
    → { "payment_status": "paid", "confirmation_number": "CHIA-000099" }
```

### 3. Patient Portal Flow (log weight, check care plan)

```
1. doctormcp_log_weight(patient_id, weight_lbs=195.5, date="2026-06-15", bearer_token)
   → { "recorded": true }

2. doctormcp_get_care_plan(patient_id, bearer_token)
   → Current medication, phase, dosing schedule, recent weights

3. doctormcp_log_side_effects(patient_id, effects=["nausea"], severity="mild", bearer_token)
   → { "recorded": true, "flagged_for_review": false }

4. doctormcp_message_provider(patient_id, message="Nausea improving", bearer_token)
   → { "sent": true, "estimated_response_time": "24-48 hours" }
```

## Stripe ACP Integration

doctormcp uses [Stripe's Agentic Commerce Protocol (ACP)](https://docs.stripe.com/acp) for payment processing:

1. **Order Creation** — `create_order` calculates the total and creates a pre-payment order record.
2. **Checkout Initiation** — `create_checkout` creates a Stripe `PaymentIntent` (for ACP) and a Stripe `Checkout Session` (for fallback). Returns line items, totals, and a `payment_url`.
3. **Payment Completion** — Two paths:
   - **ACP**: `complete_checkout` accepts a **Shared Payment Token (SPT)** from the AI platform and confirms the `PaymentIntent`. Instant, in-conversation payment.
   - **Fallback**: The agent presents the `payment_url` (Stripe-hosted checkout page) to the patient. After the patient pays in their browser, the agent calls `get_checkout_status` which detects the payment and triggers fulfillment.
4. **Post-Payment** — On success (either path), a `Subscription` and `Enrollment` are created automatically, triggering the pharmacy fulfillment pipeline.

Checkout sessions expire after 30 minutes and can be canceled at any time before completion.

## HIPAA Compliance

- **Audit Logging** — Every access to protected health information (PHI) is logged with actor identity, action type, resource, IP address, and timestamp. Logs are retained for 10 years.
- **Consent Records** — All patient consent confirmations include verbatim confirmation text, method (AI agent conversational), platform, session ID, and IP address. Consent records are immutable.
- **Input Sanitization** — All inputs are validated and sanitized. Control characters are stripped, lengths enforced, and domain-specific formats (email, state, phone, ZIP) are validated.
- **Bearer Token Auth** — Patient-scoped API tokens with SHA-256 hashing. Tokens expire after a configurable TTL. Deactivated and expired tokens are rejected.
- **Rate Limiting** — Redis-backed token-bucket rate limiting per user and endpoint category (public: 100/min, authenticated: 30/min, consent: 10/min, checkout: 5/min).
- **Minimal Data Exposure** — Tools return only the data needed for the current step. Sensitive fields (payment details, full SSN) are never returned.

## Support

- **Issues:** [github.com/chia-health/chia-mcp/issues](https://github.com/chia-health/chia-mcp/issues)
- **Website:** [chia.health](https://chia.health)
- **Email:** engineering@chia.health

## License

Apache License 2.0 — see [LICENSE](LICENSE) for details.

Copyright 2026 Chia Health, Inc.
