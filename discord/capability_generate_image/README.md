# Generate Image Capability

A reusable n8n sub-workflow that accepts an image prompt, normalizes the requested
aspect, calls OpenRouter's image API, converts the base64 response to binary, and
returns normalized image metadata with the binary file.

## Inputs and output

The **When Executed by Another Workflow** trigger accepts `prompt`, `aspect`
(`square`, `landscape`, or `portrait`), `requestedBy`, and `requestId`.
The workflow currently requests `bytedance-seed/seedream-4.5` at 2K resolution.
It returns the prompt, request ID, model, filename, MIME type, aspect metadata,
and binary image data.

## Setup

1. Import [workflow.json](./workflow.json).
2. Create an HTTP Header Auth credential containing the OpenRouter authorization
   header and attach it to **OpenRouter Image Request**.
3. Review the model, resolution, cost, content policy, and timeout behavior for
   your account.
4. Keep the workflow callable only by trusted same-owner workflows.
5. Test it from a separate caller workflow with a harmless prompt before using
   it from Discord.

The export contains no OpenRouter key or credential binding. Add input length
limits and application-level moderation appropriate to the server that invokes
it.
