---
sources: [summaries/quarto.md, summaries/SKILL.md]
brief: Preview deployments let teams review documentation or app changes in isolated environments before going live.
---

# Preview Deployments

Preview deployments are temporary, isolated deployment environments created from a specific branch or pull request, allowing teams to review and validate changes before they are merged and published to production.

## Core Idea

Rather than merging changes blindly and hoping they look correct in production, preview deployments spin up a live, accessible version of the site or application reflecting the proposed changes. Reviewers can inspect the result in context — catching layout issues, broken links, or content errors that are hard to spot in raw source files.

## In the Context of Documentation

For documentation platforms like Mintlify, preview deployments are especially valuable. Technical writers and engineers can:

- Review rendered documentation for a pull request before it merges
- Validate navigation structure changes
- Confirm that new pages, images, or code samples render correctly
- Share a live preview URL with stakeholders for approval

The [[summaries/SKILL]] document describes how the Mintlify API exposes programmatic control over preview deployments, enabling teams to create and manage them via API calls rather than only through the dashboard UI.

## Relationship to CI/CD and Automation

Preview deployments are a natural fit for [[concepts/deployment-automation]] workflows. In a typical pipeline:

1. A pull request is opened.
2. A CI system calls the deployment API to create a preview environment.
3. A bot posts the preview URL as a comment on the pull request.
4. Reviewers inspect the preview before approving the merge.
5. On merge, the preview is torn down and production is updated.

This pattern reduces the risk of publishing broken or poorly formatted documentation.

## Relationship to Documentation as Code

Preview deployments reinforce the [[concepts/documentation-as-code]] philosophy: documentation lives in version control, changes go through pull request review, and automated tooling validates the output before it reaches end users. The preview environment is the documentation equivalent of a staging server.

## Key Benefits

- **Risk reduction:** Catch errors before they reach production readers.
- **Collaboration:** Non-technical stakeholders can review rendered output without reading raw Markdown or MDX.
- **Speed:** Automated creation via API (as described in [[summaries/SKILL]]) removes manual steps from the review process.
- **Auditability:** Each preview is tied to a specific commit or branch, making it easy to trace what changed.

## Related Concepts

- [[concepts/deployment-automation]] — automating the creation and teardown of preview environments
- [[concepts/documentation-as-code]] — the broader practice of treating docs like software, of which previews are a key workflow component
- [[concepts/mdx-authoring]] — the source format commonly previewed in documentation platforms like Mintlify


See also: [[summaries/quarto]]