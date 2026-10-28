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

- `medications_categories` — returns medication categories (Weight Loss, Peptides, Longevity, etc.)
- `medications_list` — returns available medications with pricing
- `medications_availability` — check if a medication ships to a given US state

## Authentication

Discovery tools (browse medications, pricing, eligibility) are **public** — no API key needed.

Patient-specific tools (intake submission, consent signing, ordering, portal) require a **bearer token** obtained during the intake flow.

## What This Server Does

Chia Health MCP Server provides 34 tools for patient workflow integration with a licensed US telehealth platform. Available treatments include GLP-1 medications (semaglutide, tirzepatide), peptide therapies (sermorelin, NAD+, glutathione), and longevity programs.

Patients can browse medications, check eligibility, complete intake questionnaires, sign consent documents, place orders, pay via Stripe, and manage their treatment — all through conversational AI.

All prescriptions are evaluated by licensed US healthcare providers and delivered from US-licensed 503A pharmacies that source their ingredients solely from FDA-regulated sources, across 50 states + DC. Chia Health is LegitScript-certified.
