# Chia Health MCP Server

[![License](https://img.shields.io/badge/License-Apache_2.0-blue.svg)](https://github.com/chia-health/chia-mcp/blob/main/LICENSE)
[![MCP](https://img.shields.io/badge/MCP-streamable--http-green.svg)](https://mcp.chia.health/)
[![HIPAA](https://img.shields.io/badge/HIPAA-compliant-brightgreen.svg)](#hipaa-compliance)
[![Tools](https://img.shields.io/badge/Tools-34-orange.svg)](#tool-catalog)
[![Stripe ACP](https://img.shields.io/badge/Stripe-ACP-blueviolet.svg)](#stripe-acp-integration)

The official MCP (Model Context Protocol) server for **[Chia](https://chia.health)** (Chia Health, Inc.), a US telehealth prescription platform. It lets AI assistants (ChatGPT, Claude, Gemini, Copilot and custom agents) help patients browse treatments, check eligibility, complete a medical intake, sign consent documents, place an order, pay, and manage their treatment, all in conversation.

Treatments include GLP-1 medications (semaglutide, tirzepatide, including tablets), peptide therapies (sermorelin, NAD+, glutathione) and longevity programs. Every prescription is evaluated by a licensed US provider and delivered from US-licensed 503A compounding pharmacies that source their ingredients solely from FDA-regulated sources, to all 50 states plus DC. Chia Health is LegitScript-certified, and patient support is available 24/7.

**Prices:** current plans and prices are on [chia.health](https://chia.health), and the `medications_list`, `medications_details` and `medications_pricing` tools always return the live prices. This README does not list prices.

## Getting Started

Chia is a **remote MCP server**: connect over the network, no local installation required.

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

Once connected, your AI assistant can call these public tools right away (no auth required):

```
medications_categories     → treatment categories
medications_list           → all medications with current pricing
medications_availability   → check if a medication ships to your state
eligibility_check          → pre-screen age, state, BMI
```

### What Can Your AI Assistant Do?

| Category | Tools | Auth |
|----------|-------|------|
| **Auth**: email verification code, session, payment detection | 4 | No* |
| **Discovery**: browse medications, pricing, availability | 5 | No |
| **Qualification**: eligibility checks, intake questionnaires | 4 | Partial |
| **Consent**: present and sign consent documents | 4 | Guest |
| **Ordering**: place orders, upload ID verification | 4 | Guest |
| **Checkout**: Stripe ACP payments or authenticated payment links | 5 | Guest |
| **Patient Portal**: log weight, message the care team, refills | 6 | Full |
| **Provider**: answer follow-up questions from your provider | 2 | Guest |

\* Auth tools use a `session_id` (no token), except `auth_check_payment`, which uses a bearer token.

> **Important:** Every prescription is evaluated and approved by a licensed US provider. This server handles the patient workflow; it does not make clinical decisions.

### Authentication

Patients can **browse freely without signing in**: discovery, eligibility and the intake question preview are all public.

When the patient is ready to start their medical intake, they verify their email:

1. `auth_start(email, phone, first_name)` → sends a 6-digit code to their email, returns a `session_id`
2. `auth_verify_otp(session_id, code)` → returns a guest-scoped bearer token

The guest token covers intake, consent, ordering and checkout. After payment, `auth_check_payment` upgrades the token to full scope for portal access (care plan, messaging, refills).

## Tool Catalog

### Auth

| Tool | Auth | Description |
|---|---|---|
| `auth_start` | No | Send a verification code to the patient's email, get a `session_id` |
| `auth_verify_otp` | No | Verify the code, get a guest-scoped bearer token |
| `auth_resend_otp` | No | Resend the code if it expired or did not arrive |
| `auth_check_payment` | Guest | Poll for payment, upgrade the token to full scope |

### Discovery (public, no auth)

| Tool | Description |
|---|---|
| `medications_list` | List all medications with categories, forms and current pricing |
| `medications_details` | Detailed info for one medication (plans, pricing, what's included) |
| `medications_availability` | Check if a medication ships to a given state |
| `medications_pricing` | Price breakdown for a medication, form and plan length |
| `medications_categories` | List treatment categories (Weight Loss, Peptides, Longevity, etc.) |

### Qualification (public + guest)

| Tool | Auth | Description |
|---|---|---|
| `eligibility_check` | No | Pre-screen age, state, BMI and medical conditions |
| `intake_questions` | No | Get the structured intake questionnaire for a medication |
| `intake_submit` | Guest | Submit the completed intake for provider review |
| `intake_status` | Guest | Check intake review status (under review, approved, denied) |

### Consent (guest token required)

| Tool | Description |
|---|---|
| `consent_list` | List the consent documents needed for an intake |
| `consent_text` | Get the full verbatim text of a consent document |
| `consent_submit` | Record the patient's consent with an audit trail |
| `consent_status` | Check which consents are complete or pending |

### Ordering (guest token required)

| Tool | Description |
|---|---|
| `order_create` | Create a medication order (requires all consents complete) |
| `order_status` | Check order status and tracking |
| `order_documents` | List required ID documents for an order |
| `order_upload` | Upload a photo ID or selfie for identity verification |

### Checkout (guest token required)

| Tool | Description |
|---|---|
| `checkout_create` | Create a checkout; returns an authenticated payment link |
| `checkout_update` | Update a pending checkout (promo code, shipping) |
| `checkout_complete` | Complete payment with a Stripe Shared Payment Token (ACP path) |
| `checkout_status` | Poll payment status after sending a payment link (fallback path) |
| `checkout_cancel` | Cancel an in-progress checkout |

### Patient Portal (full token required, after payment)

| Tool | Description |
|---|---|
| `portal_log_weight` | Log weight for progress tracking |
| `portal_log_side_effects` | Report side effects (severe ones are flagged for the provider) |
| `portal_message` | Send a message to the care team |
| `portal_care_plan` | Get current medication, dosing and weight progress |
| `portal_refill` | Request a refill |
| `portal_support` | Open a support ticket |

### Provider (guest token required)

| Tool | Description |
|---|---|
| `provider_questions` | Get follow-up questions from the provider |
| `provider_respond` | Submit answers to the provider's questions |

## Example Agent Flows

### 1. Browse Medications (no auth)

```
User: "What weight loss medications do you offer?"

Agent calls: medications_list
→ Categories with semaglutide, tirzepatide, etc., and current prices

Agent calls: medications_details(medication="semaglutide-injectable")
→ Available plans with their current prices

Agent calls: medications_availability(medication="semaglutide-injectable", state="TX")
→ { "available": true }

Agent calls: eligibility_check(age=35, state="TX", bmi=31.2)
→ { "eligible": true, "available_medications": [...] }
```

### 2. Full Ordering Flow (verify → intake → consent → order → pay)

```
== VERIFY IDENTITY ==

1. auth_start(email="patient@example.com", phone="5551234567", first_name="Jane")
   → { "session_id": "abc123...", "otp_sent": true }

2. auth_verify_otp(session_id="abc123...", code="847293")
   → { "guest_token": "mcp_...", "scope": "guest" }

== MEDICAL INTAKE ==

3. intake_questions(medication="semaglutide-injectable")
   → Structured questionnaire (demographics, vitals, medical history, etc.)
   → Agent asks one section at a time

4. intake_submit(patient_email, patient_name, answers, bearer_token)
   → { "intake_id": "42", "next_step": "get_required_consents" }

5. consent_list(intake_id="42", bearer_token)
   → Consent documents (telehealth, treatment, pharmacy, HIPAA, AI disclosure)

6. For each consent:
   a. consent_text(consent_id, bearer_token)
      → Full text the agent MUST present verbatim
   b. Patient confirms: "I agree"
   c. consent_submit(intake_id, consent_id, "I agree", bearer_token)

== ORDER & PAY ==

7. order_create(intake_id, medication, form, plan_months, shipping_address, bearer_token)
   → { "order_id": "99", "total": "<current price>", "next_step": "create_checkout" }

8. checkout_create(order_id="99", bearer_token)
   → { "checkout_id": "7", "payment_url": "https://chia.health/..." }
   → Agent shares the link with the patient; it is also sent by email and SMS

9a. (ACP path) checkout_complete(checkout_id="7", shared_payment_token="spt_...", bearer_token)
    → { "payment_status": "success", "confirmation_number": "CHIA-000099" }

9b. (Fallback) Patient opens payment_url in a browser and pays on the Chia checkout page

10. auth_check_payment(bearer_token)
    → { "paid": true, "scope": "full" }  // token upgraded, portal unlocked
```

### 3. Patient Portal (full token, after payment)

```
1. portal_log_weight(patient_id, weight_lbs=195.5, date="2026-06-15", bearer_token)
   → { "recorded": true }

2. portal_care_plan(patient_id, bearer_token)
   → Current medication, phase, dosing schedule, recent weights

3. portal_log_side_effects(patient_id, effects=["nausea"], severity="mild", bearer_token)
   → { "recorded": true, "flagged_for_review": false }

4. portal_message(patient_id, message="Nausea improving", bearer_token)
   → { "sent": true }
```

## Stripe ACP Integration

Chia supports [Stripe's Agentic Commerce Protocol (ACP)](https://docs.stripe.com/acp):

1. **Order creation**: `order_create` calculates the total from current prices and creates a pre-payment order.
2. **Checkout**: `checkout_create` creates a Stripe `PaymentIntent` (for ACP) and an authenticated payment link that signs the patient in and opens the Chia checkout page. The link is also sent by email and SMS.
3. **Payment**, two paths:
   - **ACP**: `checkout_complete` accepts a **Shared Payment Token (SPT)** from the AI platform and confirms the `PaymentIntent`. Instant, in-conversation payment.
   - **Fallback**: the agent shares the `payment_url`, the patient pays in their browser, and the agent polls `auth_check_payment` to detect completion.
4. **After payment**: the subscription and treatment enrollment are created automatically, and `auth_check_payment` upgrades the agent's token to full scope for portal access.

## HIPAA Compliance

- **Audit logging**: every access to protected health information (PHI) is logged with actor, action, resource, IP address and timestamp. Logs are retained for 10 years.
- **Consent records**: every consent confirmation stores the verbatim confirmation text, method (AI agent conversation), platform, session ID and IP address. Consent records cannot be deleted.
- **Input validation**: all inputs are validated and sanitized; control characters are stripped, lengths enforced, and email, state, phone and ZIP formats checked.
- **Email verification first**: email ownership is verified with a 6-digit code before any patient data is accessed. Tokens are SHA-256 hashed and scoped (guest / full).
- **Rate limiting**: per-user and per-category limits (public: 100/min, sign-in: 5/min, authenticated: 30/min, consent: 10/min, checkout: 5/min).
- **Minimal data exposure**: tools return only what the current step needs. Payment details and full SSNs are never returned.

## Support

- **Issues:** [github.com/chia-health/chia-mcp/issues](https://github.com/chia-health/chia-mcp/issues)
- **Website:** [chia.health](https://chia.health)
- **Email:** engineering@chia.health

## License

Apache License 2.0. See [LICENSE](LICENSE) for details.

Copyright 2026 Chia Health, Inc.
