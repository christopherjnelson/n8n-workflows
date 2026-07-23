# AI-Powered Resume Builder with n8n

An automated resume-generation workflow built with n8n, Firecrawl, Notion, Google Gemini, and Google Drive.

The workflow accepts a public job-posting URL, validates and analyzes the listing, retrieves verified professional information from Notion, generates a tailored ATS-friendly resume, converts it to PDF, and returns the completed file through an n8n form.

## Overview

This project demonstrates how n8n can orchestrate a multi-stage AI workflow while keeping input validation, retrieval, document generation, error handling, and delivery visible within a low-code interface.

The workflow is triggered through an authenticated n8n form where a job-posting URL is submitted.

Responses are returned as browser-based success or error responses on the form completion page.

## Workflow

```text
n8n Form -> Normalize Form URL -> URL Present?
                                |
                                +-> No -> Form Error
                                   |
                                   v
                            Scrape Job Posting
                                   |
                                   v
                         Content Available?
                                   |
                                   +-> No -> Form Error
                                      |
                                      v
                              AI Job Validation
                                   |
                            Valid Job Posting?
                                   |
                                   +-> No -> Form Error
                                      |
                                      v
                         Retrieve Verified Notion Data
                                   |
                                   v
                           Generate Tailored Resume
                                   |
                                   v
                      Create Google Drive Document
                                   |
                                   v
                           Export Resume as PDF
                                   |
                                   v
                         Return File to Form
```

## Features

* Single input through an authenticated n8n form
* Missing-URL validation before scraping
* Public job-posting scraping through Firecrawl
* Deterministic checks for empty or insufficient scraped content
* AI-based job-posting validation
* Prompt-injection boundary around scraped webpage content
* Structured AI output using JSON Schema
* Retrieval of verified candidate information through individual Notion tool nodes (one per database)
* ATS-focused resume tailoring
* Guardrails against invented skills, dates, metrics, and experience
* Automatic Google Docs creation
* PDF export through Google Drive
* Browser completion responses for both success and error cases

## Technology Stack

| Component        | Purpose                                              |
| ---------------- | ---------------------------------------------------- |
| n8n              | Workflow orchestration and routing                   |
| Firecrawl        | Job-posting scraping and Markdown extraction         |
| Google Gemini    | Language-model access                                |
| Gemini Flash 3.6 | Job validation and resume generation                 |
| Notion           | Retrieval of verified professional information       |
| Google Drive API | Google Docs creation and PDF export                  |
| n8n Forms        | Browser-based submission, feedback, and PDF download |

## Workflow Stages

### 1. Input

The workflow begins through an n8n form.

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

If the submission is empty or missing a valid URL, the workflow returns a browser error:

```text
No valid URL found
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

If the check fails, the workflow returns a browser error:

```text
No usable content was received from the job-posting scraper.
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

If validation fails, the validator's explanation is displayed on the form completion page.

## 4. Candidate-data retrieval

The resume agent uses individual Notion tool nodes to retrieve verified information about the candidate.

Each Notion tool node is connected to a specific database, so the agent can only access the data it is explicitly given:

* `GetSkillsDB` -- skills and proficiencies
* `GetExperienceDB` -- employment history
* `GetCredentialsDB` -- certifications and education
* `GetProjectsDB` -- projects and infrastructure work

Scoping each tool to a single database keeps retrieval auditable and makes the workflow easy to reuse with a different Notion workspace.

The Notion workspace acts as the factual source of truth for:

* Employment history
* Exact job titles and dates
* Technical skills
* Projects
* Infrastructure and homelab work
* Education
* Certifications
* Accomplishments and verified metrics

The agent is limited to a small number of targeted Notion tool calls and is instructed to stop retrieving once it has enough verified material to write an accurate two-page resume.

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

* "Used" into "administered"
* "Helped" into "led"
* "Familiar with" into "expert"
* "Tested" into "deployed in production"
* Personal projects into professional employment
* Homelab work into enterprise infrastructure
* Technology exposure into years of experience

Quantified outcomes may only be used when the exact number exists in the retrieved source material.

## 7. Resume structure and length controls

The agent produces a standard single-column Markdown resume with the following sections:

```text
Candidate Full Name
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
  "resume_markdown": "# Candidate Full Name\n..."
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

The completed PDF is returned as a downloadable binary file through the n8n form completion page.

## Error handling

The workflow uses user-facing responses instead of terminating failed branches with generic workflow errors.

### Missing URL

```text
Form submission
-> URL not found
-> Display browser error
```

### Scrape failure

```text
Firecrawl result
-> Content check fails
-> Display browser error
```

### AI validation failure

```text
AI validator
-> status = validation_failed
-> Display validation message
```

This keeps expected input and content failures visible to the user without relying on generic n8n execution errors.

## Required credentials

The workflow requires the following n8n credentials:

* Firecrawl API
* Google Gemini API
* Notion OAuth2
* Google Drive OAuth2

Secret credential values are stored by n8n and are not included in the workflow export.

## Prerequisites

Before importing the workflow, prepare:

1. An n8n instance with community-node support
2. A Firecrawl account and API key
3. A Google AI Studio account with an API key that has access to Gemini Flash 3.6
4. A Notion workspace containing structured professional information
5. A Notion OAuth2 connection inside n8n with access to the required databases
6. A Google Drive folder for generated resumes
7. OAuth credentials for Google Drive inside n8n

## Notion knowledge-base recommendations

The quality and factual accuracy of the generated resume depend heavily on the structure of the Notion workspace.

The workflow connects four individual Notion tool nodes, one per database. Recommended databases:

* Skills -- technical skills and proficiencies
* Experience -- employment history
* Credentials -- certifications and education
* Projects -- projects and infrastructure work

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

## Configuration

After importing the workflow, update the following values.

### Google Drive

Replace:

```text
YOUR_GOOGLE_DRIVE_PARENT_FOLDER_ID
```

with the folder ID where generated company folders should be created.

Reconnect the Google Drive OAuth2 credential.

### Notion

Each Notion tool node is pointed at a specific database. Replace the placeholder database IDs with your own:

* `GetSkillsDB` -> `YOUR_NOTION_SKILLS_DATABASE_ID`
* `GetExperienceDB` -> `YOUR_NOTION_EXPERIENCE_DATABASE_ID`
* `GetCredentialsDB` -> `YOUR_NOTION_CREDENTIALS_DATABASE_ID`
* `GetProjectsDB` -> `YOUR_NOTION_PROJECTS_DATABASE_ID`

Reconnect the Notion OAuth2 credential on each Notion tool node and confirm it can access the intended databases.

### Google Gemini

Reconnect the Google Gemini API credential.

The workflow currently uses:

```text
models/gemini-3.6-flash
```

Another tool-capable model may be substituted, but it should reliably support:

* Structured output
* Tool use
* Long job-description inputs
* Multi-step retrieval
* Instruction following

### Firecrawl

Reconnect the Firecrawl credential and confirm the scrape operation returns Markdown and metadata.

### Resume agent prompt

The ResumeAgent system message contains placeholder values for the candidate's personal details:

* `CANDIDATE_FULL_NAME`
* `CANDIDATE_CITY`, `CANDIDATE_STATE`
* `candidate@example.com`
* `linkedin.com/in/candidate-profile`

Replace these with the candidate's verified contact details, or leave them as placeholders and let the agent fill them from Notion.

The output PDF filename (`YOUR_NAME_Resume.pdf`) can also be customized on the DownloadPDF node.

## Importing the workflow

1. Download or clone this repository.
2. Open the n8n workflow editor.
3. Select **Import from File**.
4. Import the workflow JSON (`resume-tailor/workflow.json`).
5. Reconnect all required credentials.
6. Replace the placeholder Google Drive folder ID.
7. Replace the placeholder Notion database IDs on each Notion tool node.
8. Confirm each Notion tool node can access the intended database.
9. Update the candidate placeholder values in the ResumeAgent prompt if desired.
10. Test a valid URL through the form.
11. Test a blank submission.
12. Test a blocked or inaccessible page.
13. Test a page that is not a job posting.
14. Activate the workflow after all success and error paths work correctly.

## Suggested repository structure

```text
.
+-- README.md
+-- resume-tailor/
|   +-- README.md
|   +-- workflow.json
+-- examples/
|   +-- sample-job-posting.md
|   +-- sample-output-redacted.pdf
+-- docs/
|   +-- workflow-overview.png
|   +-- notion-data-model.md
+-- LICENSE
```

The `examples` and `docs` directories are optional but useful for presenting the project publicly.

## Public-export checklist

Before committing the n8n workflow export, remove or replace:

* Credential IDs
* Personal credential names
* Google Drive folder IDs
* Cached Google Drive URLs
* Webhook IDs
* n8n instance IDs
* Pinned execution data
* Real generated resumes
* Private job URLs
* Private Notion identifiers
* Candidate names, emails, and contact details
* API keys, tokens, or secrets

The public workflow should contain clearly labeled placeholders.

Example:

```json
{
  "credentials": {
    "notionOAuth2Api": {
      "id": "YOUR_NOTION_CREDENTIAL_ID",
      "name": "Notion account"
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

* Restricting each Notion tool node to a single resume-specific database
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
* Expected failures receive useful responses.
* Workflow decisions remain visible inside n8n.
* The final document remains available for human review.

The goal is not to autonomously apply for jobs. The goal is to produce a targeted, factually grounded resume draft faster and more consistently.

## License

Choose a license appropriate for the way others may use the workflow.

For permissive reuse, consider the MIT License.

## Disclaimer

This project generates draft resume content using AI.

Users are responsible for reviewing all generated documents for factual accuracy, suitability, formatting, and compliance with applicable employment and privacy requirements before submitting them to an employer.
