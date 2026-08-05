# Discord — Image Command

An authenticated POST webhook that normalizes a Discord image request, calls the
reusable **Discord — Generate Image Capability** workflow, and returns the generated file
as a binary attachment response.

## Request contract

Send JSON with `prompt` and optional `aspect`, `requestedBy`, and
`requestId`. The response includes the generated file and an
`X-Image-Request-Id` header.

## Setup

1. Import the [Generate Image Capability](../generate-image-capability/workflow.json)
   and configure its OpenRouter credential.
2. Import [workflow.json](./workflow.json).
3. In **Call 'Capability - Generate Image'**, replace
   `YOUR_GENERATE_IMAGE_WORKFLOW_ID` by selecting the imported capability.
4. In **Webhook**, replace
   `your-discord-image-command-webhook-path` with a unique path and attach an
   HTTP Header Auth credential shared with the Discord-side caller.
5. Validate the caller's payload size, set rate limits and timeouts, and test the
   complete binary response before publishing.

The checked-in export has no production webhook ID, path, header secret, or
linked instance workflow ID.
