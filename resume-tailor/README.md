# AI-Powered Resume Builder with n8n

An automated resume-generation workflow built with n8n, Firecrawl, Notion MCP, OpenRouter, Google Drive, and Slack.

The workflow accepts a public job-posting URL, validates and analyzes the listing, retrieves verified professional information from Notion, generates a tailored ATS-friendly resume, converts it to PDF, and returns the completed file through either an n8n form or Slack.

## Overview

This project demonstrates how n8n can orchestrate a multi-stage AI workflow while keeping input validation, retrieval, document generation, error handling, and delivery visible within a low-code interface.

The workflow supports two entry points:

1. An authenticated n8n form where a job-posting URL can be submitted.
2. A Slack app mention containing a job-posting URL.

Both entry points are normalized into the same internal structure before entering the shared resume-generation pipeline.

Responses are routed back through the same source that initiated the workflow:

* Form requests receive browser-based success or error responses.
* Slack requests receive file uploads or channel notifications.

## Workflow

```text
n8n Form → Normalize Form URL ────────────────────┐
                                                  │
Slack Mention → Extract URL → URL Present? ───────┤
                              │                   │
                              └─ No → Slack Error │
                                                  ↓
                                       Scrape Job Posting
                                                  ↓
                                    Content Available?
                                      │           │
                                      │           └─ No
                                      │               ↓
                                      │       Route Error by Source
                                      │       ├─ Form response
                                      │       └─ Slack notification
                                      ↓
                                   AI Job Validation
                                      │
                              Valid Job Posting?
                                │             │
                                │             └─ No
                                │                 ↓
                                │         Route Error by Source
                                │         ├─ Form response
                                │         └─ Slack notification
                                ↓
                         Retrieve Verified Notion Data
                                ↓
                           Generate Tailored Resume
                                ↓
                      Create Google Drive Document
                                ↓
                           Export Resume as PDF
                                ↓
                         Route Success by Source
                         ├─ Form → File download
                         └─ Slack → File upload
```

## Features

* Dual input through an n8n form or Slack app mention
* URL extraction from Slack message text
* Missing-URL validation before scraping
* Public job-posting scraping through Firecrawl
* Deterministic checks for empty or insufficient scraped content
* AI-based job-posting validation
* Prompt-injection boundary around scraped webpage content
* Structured AI output using JSON Schema
* Retrieval of verified candidate information through Notion MCP
* ATS-focused resume tailoring
* Guardrails against invented skills, dates, metrics, and experience
* Automatic Google Docs creation
* PDF export through Google Drive
* Source-aware success delivery
* Source-aware validation and scraping error responses
* Slack notifications for failed Slack submissions
* Browser completion responses for failed form submissions

## Technology Stack

| Component         | Purpose                                               |
| ----------------- | ----------------------------------------------------- |
| n8n               | Workflow orchestration and routing                    |
| Firecrawl         | Job-posting scraping and Markdown extraction          |
| OpenRouter        | Language-model access                                 |
| DeepSeek V4 Flash | Job validation and resume generation                  |
| Notion MCP        | Retrieval of verified professional information        |
| Google Drive API  | Google Docs creation and PDF export                   |
| Slack             | Job submission, error notifications, and PDF delivery |
| n8n Forms         | Browser-based submission, feedback, and PDF download  |

## Workflow Stages

### 1. Input

The workflow can begin through either an n8n form or a Slack app mention.

### Form input

The form requires a job-posting URL:

```text
https://company.example/careers/job-title
```

The form submission is normalized into:

```json
{
  "url": "https://company.example/careers/job-title",
  "source": "form"
}
```

The form field is required, so blank submissions are rejected before entering the workflow.

### Slack input

Mention the configured Slack app and include a public job URL:

```text
@ResumeBot https://company.example/careers/job-title
```

The workflow extracts the first HTTP or HTTPS URL found in the Slack message.

The Slack submission is normalized into:

```json
{
  "url": "https://company.example/careers/job-title",
  "source": "slack"
}
```

If no valid URL is found, the workflow sends a Slack notification:

```text
No valid HTTP or HTTPS URL was found in the Slack message.
```

The request does not continue to Firecrawl or either AI node.

## 2. Job-posting scraping

The normalized URL is passed to Firecrawl.

Firecrawl returns the page as Markdown along with available metadata, including the resolved page URL.

The workflow performs an initial deterministic content check before spending tokens on AI validation.

The scraped result must:

* Contain a non-empty Markdown value
* Contain more than the configured minimum number of characters

These checks are intentionally simple. Their purpose is to reject empty or obviously unusable scrape results, not to determine whether the page is truly a job posting.

If the check fails, the workflow routes the error according to the original source.

### Form failure response

```text
No usable content was received from the job-posting scraper.
```

### Slack failure notification

```text
The URL was found, but no usable job-posting content could be extracted. The page may be blocked, expired, or require authentication.
```

## 3. AI job validation

A dedicated LLM chain determines whether the scraped content represents a genuine and sufficiently complete job posting.

A usable posting should identify a specific role and provide meaningful information about areas such as:

* Responsibilities
* Requirements
* Qualifications
* Technologies
* Expected experience

The validator rejects content such as:

* Access-denied pages
* Login pages
* CAPTCHA or bot-verification pages
* Expired listing notices
* Generic career homepages
* Job-search results
* Pages without enough role information to tailor a resume

The validator treats all scraped webpage content as untrusted reference material and is instructed not to follow commands embedded in the page.

### Successful validation output

```json
{
  "status": "success",
  "company_name": "Example Company",
  "job_title": "Systems Administrator",
  "validation_message": ""
}
```

### Failed validation output

```json
{
  "status": "validation_failed",
  "company_name": "",
  "job_title": "",
  "validation_message": "The page does not contain a complete job description."
}
```

The workflow only continues when `status` equals `success`.

If validation fails, the validator’s explanation is returned through the original request source.

### Form validation failure

The form completion page displays the validation message.

### Slack validation failure

The workflow posts the validation message to the configured Slack channel.

## 4. Candidate-data retrieval

The resume agent uses Notion MCP to retrieve verified information about the candidate.

The Notion workspace acts as the factual source of truth for:

* Employment history
* Exact job titles and dates
* IT support and troubleshooting experience
* Systems administration
* Hardware deployment and configuration
* Networking and infrastructure
* Automation and workflow development
* Homelab and self-hosted systems
* Local AI and GPU deployment
* Software platforms and technical tools
* Projects
* Education
* Certifications

The agent is instructed to retry searches using narrower or differently phrased queries when the first result is incomplete.

Information that cannot be verified in Notion must not be included.

## 5. Resume generation

The resume agent compares the validated job description with verified candidate information and creates a targeted Markdown resume.

The agent prioritizes:

1. Relevant professional experience
2. Relevant technical skills
3. Relevant certifications and education
4. Relevant projects and infrastructure work
5. Less relevant employment needed to preserve an accurate chronology

The generated resume is designed to remain compatible with applicant tracking systems.

Formatting avoids:

* Tables
* Columns
* Icons
* Decorative graphics
* Keyword stuffing
* Unsupported claims
* First-person language
* Unnecessary commentary

## 6. Accuracy guardrails

The resume agent is explicitly prohibited from inventing or inferring:

* Employment experience
* Technical skills
* Employment dates
* Certifications
* Education
* Metrics
* Team sizes
* Project scale
* Years of experience
* Leadership responsibilities
* Enterprise production experience
* Security clearances
* Software proficiency

The agent must not silently transform:

* “Used” into “administered”
* “Helped” into “led”
* “Familiar with” into “expert”
* “Tested” into “deployed in production”
* Personal projects into professional employment
* Homelab work into enterprise infrastructure
* Technology exposure into years of experience

Quantified outcomes may only be used when the exact number exists in the retrieved source material.

## 7. Resume structure and length controls

The agent produces a standard single-column Markdown resume with the following sections:

```text
Christopher Nelson
Contact Information

Professional Summary
Technical Skills
Professional Experience
Projects & Infrastructure
Education & Certifications
```

The prompt applies the following length controls:

* Professional summary: no more than three concise sentences
* Most relevant position: no more than five bullets
* Other positions: no more than three bullets each
* Projects and infrastructure: no more than three bullets total
* Education and certifications: no more than seven entries

Lower-value information should be omitted before the resume is allowed to exceed the intended two-page layout.

## 8. Structured resume output

The resume agent returns a predictable structured result:

```json
{
  "company_name": "Example Company",
  "job_title": "Systems Administrator",
  "resume_markdown": "# Christopher Nelson\n..."
}
```

The Structured Output Parser requires exactly these properties and rejects additional output.

This allows downstream Google Drive nodes to reference the employer, title, and resume content reliably.

## 9. Google Drive document generation

The workflow creates a company-specific folder inside a configured parent Google Drive folder.

The generated Markdown is uploaded through the Google Drive API as a Google Docs document.

The document is named using:

```text
Company Name - Job Title - YYYY-MM-DD
```

Example:

```text
Example Company - Systems Administrator - 2026-07-16
```

The document is then downloaded from Google Drive as a PDF.

## 10. Output delivery

The workflow determines which trigger initiated the execution and returns the completed PDF through the same source.

### Form execution

The PDF is returned as a downloadable binary file through the n8n form completion page.

### Slack execution

The PDF is uploaded to the configured Slack channel.

## Error handling

The workflow uses user-facing responses instead of terminating failed branches with generic workflow errors.

### Missing Slack URL

```text
Slack mention
→ Extract URL
→ URL not found
→ Post Slack notification
```

### Scrape failure

```text
Firecrawl result
→ Content check fails
→ Determine original source
├─ Form → Display browser error
└─ Slack → Post Slack notification
```

### AI validation failure

```text
AI validator
→ status = validation_failed
→ Determine original source
├─ Form → Display validation message
└─ Slack → Post validation message
```

This keeps expected input and content failures visible to the user without relying on generic n8n execution errors.

## Required credentials

The workflow requires the following n8n credentials:

* Firecrawl API
* OpenRouter API
* Notion MCP OAuth
* Google Drive OAuth2
* Slack API

Secret credential values are stored by n8n and are not included in the workflow export.

## Prerequisites

Before importing the workflow, prepare:

1. An n8n instance with community-node support
2. A Firecrawl account and API key
3. An OpenRouter account and API key
4. A Notion workspace containing structured professional information
5. A Notion MCP connection with access to the required pages or databases
6. A Google Drive folder for generated resumes
7. A Slack app configured for app mentions, messages, and file uploads
8. OAuth credentials for Google Drive and Slack inside n8n

## Notion knowledge-base recommendations

The quality and factual accuracy of the generated resume depend heavily on the structure of the Notion workspace.

Recommended categories include:

* Candidate contact details
* Employment history
* Technical skills
* Projects
* Infrastructure and homelab work
* Education
* Certifications
* Accomplishments
* Verified metrics

Each employment record should clearly include:

* Employer
* Exact job title
* Start and end dates
* Location
* Responsibilities
* Technologies used
* Verified accomplishments
* Verified metrics

Personal projects should be clearly distinguished from paid employment.

## Slack configuration

The Slack Trigger listens for `app_mention` events in a configured channel.

Example:

```text
@ResumeBot https://company.example/jobs/12345
```

The Slack app requires permissions appropriate for:

* Receiving app mentions
* Reading mention text
* Posting channel messages
* Uploading files

The current implementation posts responses and uploads PDFs to a configured Slack channel.

A future version may preserve the original channel and thread timestamp so responses can be returned directly to the initiating thread.

## Configuration

After importing the workflow, update the following values.

### Google Drive

Replace:

```text
YOUR_GOOGLE_DRIVE_PARENT_FOLDER_ID
```

with the folder ID where generated company folders should be created.

Reconnect the Google Drive OAuth2 credential.

### Slack

Replace:

```text
YOUR_SLACK_CHANNEL_ID
```

with the intended Slack channel ID.

Reconnect the Slack credential on:

* Slack Trigger
* URL error notification
* Scrape error notification
* Validation error notification
* Resume file upload

### Notion

Reconnect the Notion MCP node and confirm it has access only to the candidate information required for resume generation.

### OpenRouter

Reconnect the OpenRouter credential.

The workflow currently uses:

```text
deepseek/deepseek-v4-flash
```

Another tool-capable model may be substituted, but it should reliably support:

* Structured output
* Tool use
* Long job-description inputs
* Multi-step retrieval
* Instruction following

### Firecrawl

Reconnect the Firecrawl credential and confirm the scrape operation returns Markdown and metadata.

## Importing the workflow

1. Download or clone this repository.
2. Open the n8n workflow editor.
3. Select **Import from File**.
4. Import the sanitized workflow JSON.
5. Reconnect all required credentials.
6. Replace placeholder Google Drive and Slack IDs.
7. Confirm the Notion MCP node can access the intended data.
8. Test a valid URL through the form.
9. Test a valid URL through Slack.
10. Test a Slack mention with no URL.
11. Test a blocked or inaccessible page.
12. Test a page that is not a job posting.
13. Activate the workflow after all success and error paths work correctly.

## Suggested repository structure

```text
.
├── README.md
├── workflow/
│   └── resume-builder.workflow.json
├── examples/
│   ├── sample-job-posting.md
│   └── sample-output-redacted.pdf
├── docs/
│   ├── workflow-overview.png
│   ├── notion-data-model.md
│   └── slack-setup.md
└── LICENSE
```

The `examples` and `docs` directories are optional but useful for presenting the project publicly.

## Public-export checklist

Before committing the n8n workflow export, remove or replace:

* Credential IDs
* Personal credential names
* Slack channel IDs
* Cached Slack channel names
* Google Drive folder IDs
* Cached Google Drive URLs
* Webhook IDs
* n8n instance IDs
* Pinned execution data
* Real generated resumes
* Private job URLs
* Private Notion identifiers
* API keys, tokens, or secrets

The public workflow should contain clearly labeled placeholders.

Example:

```json
{
  "credentials": {
    "slackApi": {
      "id": "YOUR_SLACK_CREDENTIAL_ID",
      "name": "Slack Credential"
    }
  }
}
```

Never commit active secrets to the repository.

## Known limitations

### Resume layout

The current workflow imports generated Markdown into Google Docs and exports it as PDF.

This provides a straightforward document pipeline but offers limited control over:

* Typography
* Margins
* Page breaks
* Bullet spacing
* Heading spacing
* Consistent two-page rendering

A future version may populate a fixed Google Docs or DOCX resume template.

### Job-page noise

Some career websites return navigation, testimonials, benefits, location content, and repeated page elements alongside the actual listing.

The resume prompt instructs the model to ignore unrelated content, but noisy pages may still increase token use.

Potential improvements include:

* Firecrawl main-content extraction
* CSS selector filtering
* Cleaned job-description output from the validator
* Maximum scraped-input length

### Slack URL extraction

The Slack branch uses the first HTTP or HTTPS URL found in the message.

It does not currently support:

* Multiple job URLs in one mention
* Interactive URL selection
* Slack modal submission
* Direct-message workflows
* Returning responses to the original thread
* Per-user job history

### Slack routing

Slack responses currently use a configured channel rather than dynamically using the channel and thread from the triggering message.

### Folder duplication

The current workflow creates a company folder for each execution.

Repeated applications to the same employer may create duplicate folders with the same name.

A future version may search for and reuse existing folders.

### Human review

Every generated resume should be reviewed before submission.

The workflow includes factual guardrails, but generated wording may still require adjustments for:

* Relevance
* Tone
* Interpretation
* Page length
* Formatting
* Employer-specific preferences

## Potential improvements

* Populate a fixed resume template
* Return Slack responses in the original thread
* Preserve the originating Slack channel dynamically
* Save the original job description with the resume
* Generate a matching cover letter
* Generate recruiter outreach messages
* Support pasted job descriptions when scraping fails
* Reuse existing Google Drive folders
* Record applications in Notion or Supabase
* Add application-status tracking
* Detect duplicate job postings
* Add a human approval step before PDF generation
* Validate generated claims against retrieved source records
* Generate both PDF and DOCX output
* Add reusable workflow tests for success and failure branches

## Security considerations

The scraped job posting is treated as untrusted content.

The resume agent is instructed not to follow prompts or commands embedded inside the webpage and not to allow webpage content to control Notion tool usage.

Additional production safeguards may include:

* Restricting Notion MCP access to resume-specific databases
* Validating submitted URL protocols
* Blocking private-network and localhost addresses
* Restricting accepted domains
* Limiting scraped input size
* Reviewing community nodes before installation
* Using least-privilege OAuth scopes
* Rotating API credentials
* Keeping n8n and installed nodes updated

## Why this project exists

Tailoring a resume for each job is valuable but repetitive.

This workflow automates the mechanical parts of the process while preserving several important boundaries:

* Candidate information comes from a controlled source of truth.
* Job postings are checked before resume generation.
* Unsupported skills and accomplishments are excluded.
* Expected failures receive useful source-specific responses.
* Workflow decisions remain visible inside n8n.
* The final document remains available for human review.

The goal is not to autonomously apply for jobs. The goal is to produce a targeted, factually grounded resume draft faster and more consistently.

## License

Choose a license appropriate for the way others may use the workflow.

For permissive reuse, consider the MIT License.

## Disclaimer

This project generates draft resume content using AI.

Users are responsible for reviewing all generated documents for factual accuracy, suitability, formatting, and compliance with applicable employment and privacy requirements before submitting them to an employer.
