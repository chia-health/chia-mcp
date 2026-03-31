# Installing Chia Health MCP Server

Chia Health MCP Server is a **remote server** — no local installation or build steps needed.

## Configuration

Add to your MCP client configuration:

```json
{
  "mcpServers": {
    "chia-health": {
      "url": "https://mcp.chia.health/"
    }
  }
}
```

## Verification

After adding, test with these public tools (no auth required):

- `medications.categories` — returns medication categories (Weight Loss, Peptides, Longevity, etc.)
- `medications.list` — returns available medications with pricing
- `medications.availability` — check if a medication ships to a given US state
- `eligibility.check` — pre-screen patient eligibility (age, state, BMI)

## Authentication

Discovery tools (browse medications, pricing, eligibility, intake questions) are **public** — no API key needed. The patient can browse freely.

When the patient is ready to proceed with their medical intake, identity verification starts:

1. `auth.start(email, phone, name)` — sends a 6-digit OTP to the patient's email, returns a `session_id`
2. `auth.verify_otp(session_id, code)` — verifies the code, returns a guest-scoped bearer token

The guest token enables intake submission, consent signing, ordering, and checkout. After payment, `auth.check_payment` upgrades the token to full scope for portal access (care plan, messaging, refills).

## What This Server Does

Chia Health MCP Server provides 34 tools for patient workflow integration with a licensed US telehealth platform. Available treatments include GLP-1 medications (semaglutide, tirzepatide), peptide therapies (sermorelin, NAD+, glutathione), and longevity programs.

Patients can browse medications, check eligibility, complete intake questionnaires, sign consent documents, place orders, pay via Stripe, and manage their treatment — all through conversational AI.

All prescriptions are evaluated by licensed US healthcare providers and delivered from FDA-regulated pharmacies across 50 states + DC.
