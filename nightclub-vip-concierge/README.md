# Nightclub VIP Concierge AI Agent (n8n + Google Workspace)

An autonomous, production-grade conversational AI agent built in **n8n** utilizing **LangChain**, **OpenRouter**, and native **Google Workspace / Sheets** APIs. This agent functions as a 24/7 digital concierge for a high-end nightclub, handling real-time inventory checks, resource calendar availability, automated inventory stock reduction, and direct event booking without human intervention.

---

## Architecture Overview

* **Orchestration:** n8n LangChain Agent utilizing OpenWeight LLMs (Qwen3.6-35ba3b via OpenRouter).
* **State Management:** Window Buffer Memory bound to the chat session ID to maintain step-by-step context.
* **Data Sources:** * **Google Sheets:** Live inventory tracking with read/write operations for stock levels.
  * **Google Calendar (Resources):** Individual VIP table calendars (`@resource.calendar.google.com`) for collision detection and booking.
  * **Google Calendar (Events):** Schedule discovery for club marketing and DJ sets.

---

## How It Hooks into Google Workspace & Services

1. **OAuth2 Credential Consolidation:** * A single, secure OAuth2 connection manages both master event creation and resource queries, bypassing service account limitations for calendar objects.
2. **Resource Calendar Abstraction:**
   * Each VIP table is provisioned as an independent Google Workspace Resource. The agent dynamically queries individual table IDs using n8n's `$fromAI()` binding.
3. **Native Collision Handling:**
   * Bookings are created on the master calendar with the specific table's resource email injected as an attendee. Google's native resource handling automatically accepts the invite and blocks out availability for the 8 PM–2 AM operating block.
4. **Inventory Ledger Synchronization:**
   * Separate read and write Google Sheets tool nodes allow the agent to verify stock and autonomously decrement bottle counts by 1 upon customer confirmation.

---

## Operational State Machine (System Prompt Flow)

The agent operates under a strict sequential state machine enforced via system prompt engineering:
1. **Scope & Date:** Collects requested date and party size (4 or 6 guests).
2. **Policy Briefing:** Informs the guest that VIP blocks run 8 PM–2 AM with a mandatory 10 PM arrival deadline.
3. **Availability Validation:** Dynamically queries table resource availability.
4. **Inventory Verification & Deduction:** Pulls live bottle pricing and reduces stock on confirmation.
5. **CRM Collection:** Gathers guest Full Name, Email, and Phone Number.
6. **Autonomic Booking:** Injects parameters via `$fromAI()` to write the calendar event and invite the table resource.
7. **Handoff:** Confirms reservation details and prompts backend admin notification for invoicing.

---

## Setup & Deployment Instructions

1. **Import the Workflow:** Import the provided JSON workflow into your n8n instance.
2. **Configure Credentials:**
   * Attach your **OpenRouter** API credentials to the language model node.
   * Authenticate your **Google Calendar OAuth2** credentials for calendar tools.
   * Attach your **Google Service Account** credentials to the Google Sheets inventory nodes.
3. **Map Resource IDs:** Update the System Message resource directory with your specific `@resource.calendar.google.com` table addresses.
4. **Activate Webhook:** Enable the chat trigger node to expose the active endpoint for your fr
