# AI-Powered Resume Builder with n8n

An automated resume-generation workflow built with n8n, Firecrawl, Notion MCP, OpenRouter, Google Drive, and Slack.

The workflow accepts a public job-posting URL, validates and analyzes the listing, retrieves verified professional information from Notion, generates a tailored ATS-friendly resume, converts it to PDF, and returns the completed file through either an n8n form or Slack.

## Overview

This project demonstrates how n8n can orchestrate a multi-step AI workflow while keeping validation, routing, document generation, and delivery visible within a low-code interface.

The workflow supports two entry points:

1. An authenticated n8n form where a job-posting URL can be submitted.
2. A Slack app mention containing a job-posting URL.

Both entry points are normalized into the same internal structure before continuing through the shared resume-generation pipeline.

## Workflow

```text
n8n Form ──→ Normalize Form Input ───────────────┐
                                                 │
Slack Mention → Extract URL → Validate URL ──────┤
                                                 ↓
                                      Scrape Job Posting
                                                 ↓
                                  Basic Content Validation
                                                 ↓
                                    AI Job Validation
                                                 ↓
                              Retrieve Verified Notion Data
                                                 ↓
                                  Generate Tailored Resume
                                                 ↓
                              Create Google Drive Document
                                                 ↓
                                      Export Resume PDF
                                                 ↓
                           Route Output by Original Trigger
                              ├── Form → Browser download
                              └── Slack → File upload
```

## Features

* Dual input through an n8n form or Slack app mention
* URL extraction from Slack message text
* Public job-posting scraping with Firecrawl
* Deterministic checks for missing or insufficient scraped content
* AI-based validation of the job posting
* Prompt-injection boundary around scraped webpage content
* Structured AI output using JSON Schema
* Retrieval of verified candidate information through Notion MCP
* ATS-focused resume tailoring
* Guardrails against invented skills, dates, metrics, and experience
* Automatic Google Docs creation
* PDF export through Google Drive
* Source-aware delivery to either the browser or Slack
* Explicit error branches for invalid input and unusable job pages

## Technology Stack

| Component         | Purpose                                        |
| ----------------- | ---------------------------------------------- |
| n8n               | Workflow orchestration                         |
| Firecrawl         | Job-posting scraping and Markdown extraction   |
| OpenRouter        | Language-model access                          |
| DeepSeek V4 Flash | Job validation and resume generation           |
| Notion MCP        | Retrieval of verified professional information |
| Google Drive API  | Google Docs creation and PDF export            |
| Slack             | Job submission and PDF delivery                |
| n8n Forms         | Browser-based submission and download          |

## Workflow Stages

### 1. Input

The workflow can begin from either an n8n form or a Slack app mention.

#### Form input

The form requires a job-posting URL:

```text
https://company.example/careers/job-title
```

The form branch normalizes the submitted value into:

```json
{
  "url": "https://company.example/careers/job-title",
  "source": "form"
}
```

#### Slack input

Mention the configured Slack app and include a public URL:

```text
@ResumeBot https://company.example/careers/job-title
```

The workflow extracts the first HTTP or HTTPS URL found in the message.

The Slack branch normalizes the result into:

```json
{
  "url": "https://company.example/careers/job-title",
  "source": "slack"
}
```

If no URL is found, the workflow stops before invoking Firecrawl or an AI model.

### 2. Job-posting scraping

The normalized URL is sent to Firecrawl.

Firecrawl returns the page as Markdown, along with available page metadata.

The workflow performs initial deterministic checks before spending tokens on AI validation:

* Markdown must exist.
* Content must exceed a minimum length.
* Content must contain common job-description terminology.

These checks are not intended to conclusively identify a job posting. They prevent obviously empty or unusable scrape results from continuing.

### 3. AI job validation

A dedicated LLM chain determines whether the scraped content represents a genuine and sufficiently complete job posting.

The validator rejects content such as:

* Access-denied pages
* Login pages
* CAPTCHA or bot-verification pages
* Expired listing notices
* Generic career homepages
* Job-search result pages
* Pages without enough role information to tailor a resume

The validator returns structured output:

```json
{
  "status": "success",
  "company_name": "Example Company",
  "job_title": "Systems Administrator",
  "validation_message": ""
}
```

A failed result uses:

```json
{
  "status": "validation_failed",
  "company_name": "",
  "job_title": "",
  "validation_message": "The page does not contain a complete job description."
}
```

The workflow only continues when `status` equals `success`.

### 4. Candidate-data retrieval

The resume agent uses Notion MCP to retrieve verified information about the candidate.

The Notion workspace acts as the factual source of truth for:

* Employment history
* Job titles and employment dates
* Technical experience
* Systems administration
* IT support and troubleshooting
* Hardware and networking work
* Automation projects
* Homelab infrastructure
* Software and platforms
* Education
* Certifications

The agent is instructed to search again with narrower queries when the initial retrieval is incomplete.

Information that cannot be verified in Notion is omitted from the resume.

### 5. Resume generation

The resume agent compares the validated job description with verified candidate information and produces a targeted Markdown resume.

The agent prioritizes:

1. Relevant professional experience
2. Relevant technical skills
3. Certifications and education
4. Relevant projects and infrastructure work
5. Less relevant employment needed to preserve an accurate chronology

The output is designed to remain compatible with applicant tracking systems.

Formatting avoids:

* Tables
* Columns
* Icons
* Decorative graphics
* Keyword stuffing
* Unsupported claims
* First-person language

### 6. Accuracy guardrails

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
* Software proficiency

The agent must not silently transform:

* “Used” into “administered”
* “Helped” into “led”
* “Familiar with” into “expert”
* “Tested” into “deployed in production”
* Personal projects into professional experience
* Homelab work into enterprise infrastructure
* Technology exposure into years of experience

Quantified outcomes may only be included when the exact metric exists in the retrieved source material.

### 7. Structured resume output

The resume-generation agent returns:

```json
{
  "company_name": "Example Company",
  "job_title": "Systems Administrator",
  "resume_markdown": "# Christopher Nelson\n..."
}
```

The Structured Output Parser rejects additional properties and helps keep downstream document creation predictable.

### 8. Google Drive document generation

The generated Markdown is uploaded through the Google Drive API as a Google Docs document.

The document is placed in a company-specific folder under a configured parent resume folder.

Google Drive then exports the document as a PDF.

### 9. Output delivery

The completed PDF is routed according to the trigger that started the workflow.

#### Form execution

The PDF is returned as a downloadable binary file through the n8n form completion page.

#### Slack execution

The PDF is uploaded to a configured Slack channel.

## Required Credentials

The workflow requires the following n8n credentials:

* Firecrawl API
* OpenRouter API
* Notion MCP OAuth
* Google Drive OAuth2
* Slack API

Credentials are not included in the public workflow export.

## Prerequisites

Before importing the workflow, prepare:

1. An n8n instance with access to community nodes.
2. A Firecrawl account and API key.
3. An OpenRouter account and API key.
4. A Notion workspace containing structured professional information.
5. A Notion MCP connection with access to the required pages or databases.
6. A Google Drive folder where generated resumes will be stored.
7. A Slack app configured for app mentions and file uploads.
8. OAuth credentials for Google Drive and Slack inside n8n.

## Notion knowledge-base recommendations

The quality and reliability of the generated resume depend heavily on the structure of the Notion workspace.

Recommended categories include:

* Contact information
* Employment history
* Technical skills
* Projects
* Infrastructure and homelab work
* Education
* Certifications
* Accomplishments and verified metrics

Each employment record should clearly include:

* Employer
* Exact job title
* Start date
* End date
* Location
* Responsibilities
* Technologies used
* Verified accomplishments
* Verified metrics

Personal projects should be clearly distinguished from paid employment.

## Slack configuration

The Slack Trigger listens for `app_mention` events in a configured channel.

Example usage:

```text
@ResumeBot https://company.example/jobs/12345
```

The Slack app requires appropriate permissions for:

* Reading app mentions
* Reading messages in the configured channel
* Uploading files
* Posting messages, if reply functionality is added

The current implementation uploads the generated PDF to a configured channel.

A future version may preserve the original channel and thread timestamp so that the completed resume can be returned directly in the originating Slack thread.

## Configuration

After importing the workflow, update the following values.

### Google Drive

Replace:

```text
YOUR_GOOGLE_DRIVE_PARENT_FOLDER_ID
```

with the folder ID where company-specific resume folders should be created.

### Slack

Replace:

```text
YOUR_SLACK_CHANNEL_ID
```

with the target Slack channel ID.

Reconnect:

* Slack Trigger credentials
* Slack file-upload credentials

### Notion

Reconnect the Notion MCP node and confirm it has access only to the pages or databases required for resume generation.

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

Reconnect the Firecrawl credential and confirm the selected scrape operation returns Markdown.

## Importing the workflow

1. Download or clone this repository.
2. Open the n8n workflow editor.
3. Select **Import from File**.
4. Import the sanitized workflow JSON.
5. Reconnect each required credential.
6. Replace placeholder folder and channel IDs.
7. Confirm that the Notion MCP node can access the intended data.
8. Test the form branch with a public job posting.
9. Test the Slack branch with an app mention containing a URL.
10. Activate the workflow after both paths complete successfully.

## Repository structure

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

Before committing an n8n workflow export, remove or replace:

* Credential IDs
* Personal credential names
* Slack channel IDs
* Google Drive folder IDs
* Webhook IDs
* n8n instance IDs
* Pinned execution data
* Real generated resumes
* Personal test URLs
* Private Notion identifiers
* Any access tokens or API keys

The public workflow should contain clearly labeled placeholders instead.

Example:

```json
{
  "credentials": {
    "slackApi": {
      "id": "REPLACE_WITH_SLACK_CREDENTIAL_ID",
      "name": "Slack Credential"
    }
  }
}
```

Never commit active secrets to the repository.

## Known limitations

### Resume layout

The current workflow imports generated Markdown into Google Docs and exports the result as PDF.

This provides a simple and reliable document pipeline, but offers limited control over:

* Typography
* Margins
* Page breaks
* Bullet spacing
* Heading spacing
* Consistent two-page layout

A future version may populate a fixed Google Docs or DOCX template instead of importing free-form Markdown.

### Job-page noise

Some career websites return navigation, testimonials, benefits, location marketing, and repeated page content alongside the actual listing.

The prompt instructs the model to ignore unrelated content, but large pages may still increase token usage.

Potential future improvements include:

* Firecrawl main-content extraction
* CSS selector filtering
* AI-generated cleaned job descriptions
* Maximum input-length controls

### Slack URL extraction

The current Slack implementation uses the first HTTP or HTTPS URL found in message text.

It does not currently support:

* Multiple job URLs in one message
* Interactive selection between links
* Slack modal submission
* Direct messages
* Replies in the original thread
* Per-user job history

### Folder duplication

The current workflow creates a company folder for each execution.

Repeated applications to the same employer may create folders with identical names.

A future version may search for and reuse an existing company folder or create a unique application folder based on employer, role, and date.

### Human review

The generated resume should be reviewed before submission.

The workflow includes factual guardrails, but AI-generated wording may still require adjustments for tone, relevance, page length, or interpretation.

## Potential improvements

* Populate a fixed resume template
* Save the original job description beside the resume
* Generate a matching cover letter
* Generate recruiter outreach messages
* Reply in the original Slack thread
* Allow pasted job descriptions when scraping fails
* Reuse existing Google Drive company folders
* Save job application records to Notion or Supabase
* Track application status
* Add duplicate-job detection
* Add a human approval step before PDF generation
* Add automated factual validation against retrieved source records
* Generate both PDF and DOCX output
* Add tests using known valid and invalid job pages

## Security considerations

The scraped job posting is treated as untrusted content.

The resume agent is instructed not to follow commands or prompts embedded inside the webpage and not to allow webpage content to control Notion tool usage.

Additional production safeguards may include:

* Restricting Notion MCP access to resume-related databases
* Validating URLs before scraping
* Blocking private network and localhost URLs
* Restricting accepted domains
* Limiting maximum scraped content size
* Reviewing community nodes before installation
* Using least-privilege OAuth scopes
* Rotating API credentials
* Keeping n8n and installed nodes updated

## Why this project exists

Tailoring a resume for each job is valuable but repetitive.

This workflow automates the mechanical parts of the process while preserving several important boundaries:

* Professional information comes from a controlled source of truth.
* Job postings are validated before generation.
* Unsupported skills and accomplishments are excluded.
* Workflow decisions remain visible within n8n.
* The final resume remains available for human review.

The goal is not to autonomously apply for jobs. The goal is to produce a well-targeted, factually grounded first draft faster and more consistently.

## License

Choose a license appropriate for the way you want others to use the workflow.

For permissive reuse, consider the MIT License.

## Disclaimer

This project generates draft resume content using AI.

Users are responsible for reviewing all generated documents for factual accuracy, suitability, and compliance with applicable employment and privacy requirements before submitting them to an employer.
