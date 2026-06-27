---
sources: [summaries/README_20260413215204.md, summaries/README_20260413204228.md, summaries/snapshot-2026-04-14T04_21_45_999Z_20260413212342.md, summaries/snapshot-2026-04-14T04_21_45_999Z_20260413212147.md, summaries/File Folder Structure Rebuild.md, summaries/Apply-to-Y-Combinator.md, summaries/2026-06-26-2133-plan.md, summaries/cognition.instructions.md, summaries/0001-voice-record-architecture-decisions.md, summaries/003-modular-template-structure.md, summaries/001-choose-quarto-typst.md, summaries/quarto.md, summaries/brand-yml-spec.md, summaries/README.md, summaries/project-setup-progress.md, summaries/RECOVERY_NOTES.md, summaries/PROJECT_SETUP_COMPLETE.md, summaries/SKILL.md]
brief: The practice of writing, managing, and deploying documentation using the same tools and workflows as software development.
---

# Documentation as Code

Documentation as Code (Docs as Code) is the practice of writing, managing, and deploying documentation using the same tools and workflows as software development: plain text files, version control (Git), code review, and automated build/deployment pipelines.

## Core Principles

- **Plain text source files** — Documentation is written in human-readable markup formats (Markdown, MDX) rather than binary or proprietary formats. This makes content diffable, searchable, and portable.
- **Version control** — All documentation lives in a Git repository alongside (or separate from) application code. History, authorship, and change tracking come for free.
- **Automated deployment** — Pushing changes to a repository triggers a build and publish process without manual intervention.
- **Code review workflows** — Documentation changes go through pull requests, enabling peer review, discussion, and approval before publication.
- **Validation and linting** — Documentation is checked programmatically for broken links, accessibility issues, and build errors before deployment.

## How Mintlify Implements Docs as Code

[[summaries/SKILL]] describes Mintlify as a platform built entirely around Docs as Code principles:

### File-Based Authoring
Every page is an MDX file with YAML frontmatter. The file system reflects the content structure. Naming conventions (kebab-case), directory organization, and file paths directly map to published URLs.

### Central Configuration as Code
The `docs.json` file at the project root is the single source of truth for the entire site: navigation structure, theme, colors, integrations, and API specs. Changing the site means changing a file — not clicking through a GUI.

### Git-Driven Deployment
Mintlify deploys automatically when changes are pushed to the connected Git repository. There is no separate publish step for content authors. Beyond Git push events, the **Mintlify REST API** enables programmatic control over the deployment lifecycle — teams can trigger rebuilds, query deployment status, and manage preview deployments via API calls, integrating documentation pipelines into broader CI/CD workflows.

### CLI-Based Validation
Before merging, authors run CLI tools to validate the documentation:
- `mint broken-links` — Detects dead internal links
- `mint validate` — Confirms the build succeeds
- `mint a11y` — Checks accessibility

This mirrors the role of test suites and linters in software development.

### Structured Navigation via Config
Navigation is defined declaratively in `docs.json` rather than managed through a CMS interface. Adding a new page requires both creating the MDX file and adding an entry to the config — a deliberate, reviewable change.

## Programmatic API Access

Mature Docs as Code setups often extend beyond Git workflows into full API-driven automation. The Mintlify API (detailed in [[summaries/SKILL]]) exposes endpoints for:

- **Triggering deployments** — Rebuild documentation in response to external events, such as a codebase change that doesn't originate from a Git push.
- **Querying site metadata** — Retrieve deployment status, configured domains, and navigation structure programmatically.
- **Managing preview deployments** — Spin up and inspect branch or pull request previews without manual dashboard interaction.

All requests authenticate via a Bearer API key, keeping access control consistent with developer-oriented tooling. This connects to [[concepts/deployment-automation]] and [[concepts/preview-deployments]] as natural extensions of the Docs as Code philosophy.

## Relationship to Related Concepts

Docs as Code depends heavily on [[concepts/plain-text-documentation]]: plain text is what makes documentation tractable by the same tools as code (diff, grep, version control). [[concepts/mdx-authoring]] extends plain Markdown with component-based interactivity while preserving the plain-text, file-based nature that Docs as Code requires.

[[concepts/semantic-linefeeds]] is a complementary practice that makes plain-text documentation even more Git-friendly by aligning line breaks with sentence boundaries, producing cleaner diffs.

[[concepts/deployment-automation]] and [[concepts/preview-deployments]] represent the operational layer of Docs as Code — the infrastructure that makes automated, reviewable publishing possible.

## Benefits

| Benefit | Mechanism |
|---|---|
| Auditability | Full Git history of every change |
| Collaboration | Pull requests and code review |
| Automation | CI/CD pipelines for validation and deployment |
| Portability | Plain text files work with any editor or toolchain |
| Consistency | Linting and validation enforce standards at scale |
| Programmability | REST APIs allow integration with external systems |

## Common Gotchas

- **Config and content must stay in sync.** In Mintlify, creating an MDX file without adding it to `docs.json` results in a hidden page. The configuration file is as important as the content files.
- **Proprietary config formats can create lock-in.** `docs.json` is Mintlify-specific; migrating platforms requires converting both content and configuration.
- **Migrations from non-Docs-as-Code platforms** (like CMS-based tools) require exporting content to plain text files. Mintlify provides `@mintlify/scraping` to assist with migrations from ReadMe and Docusaurus.
- **API key management** — Programmatic access introduces secrets that must be stored securely and rotated, adding operational overhead similar to any other service credential.

## See Also

- [[summaries/SKILL]] — Mintlify best practices skill document
- [[concepts/plain-text-documentation]]
- [[concepts/mdx-authoring]]
- [[concepts/semantic-linefeeds]]
- [[concepts/deployment-automation]]
- [[concepts/preview-deployments]]


See also: [[summaries/PROJECT_SETUP_COMPLETE]]

See also: [[summaries/RECOVERY_NOTES]]

See also: [[summaries/project-setup-progress]]

See also: [[summaries/README]]

See also: [[summaries/brand-yml-spec]]

See also: [[summaries/quarto]]

See also: [[summaries/001-choose-quarto-typst]]

See also: [[summaries/003-modular-template-structure]]

See also: [[summaries/0001-voice-record-architecture-decisions]]

See also: [[summaries/cognition.instructions]]

See also: [[summaries/2026-06-26-2133-plan]]

See also: [[summaries/Apply-to-Y-Combinator]]

See also: [[summaries/File Folder Structure Rebuild]]

See also: [[summaries/snapshot-2026-04-14T04_21_45_999Z_20260413212147]]

See also: [[summaries/snapshot-2026-04-14T04_21_45_999Z_20260413212342]]

See also: [[summaries/README_20260413204228]]

See also: [[summaries/README_20260413215204]]