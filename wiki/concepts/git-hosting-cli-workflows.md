---
sources: [summaries/cli-github.md, summaries/cli-gh.md, summaries/cli-gh-create-repo.md, summaries/cli-ftp.md, summaries/cli-commands.md, summaries/cli-command-management.md]
brief: CLI workflows for managing GitHub and GitLab-hosted repository tasks.
---

# Git Hosting CLI Workflows

Git hosting CLI workflows are command-line patterns for interacting with repository hosting platforms such as GitHub and GitLab. In the context of [[summaries/cli-command-management]], this concept captures how hosted-forge tasks extend core Git usage with platform-specific authentication, API access, repository administration, and pull request or project operations.

## Overview

Core Git handles local and remote version control operations, but hosted platforms add higher-level workflows:
- authenticating to a hosting service
- creating or managing repositories
- interacting with pull requests or merge-related tooling
- calling platform APIs from the terminal
- configuring shortcuts and aliases for repetitive tasks
- administering self-hosted or managed Git platform services

This makes Git hosting CLIs a bridge between [[concepts/version-control]], [[concepts/command-line-workflows]], and [[concepts/developer-productivity]].

## Role in command management

Within [[summaries/cli-command-management]], Git hosting CLI workflows appear as a major cluster alongside core Git commands and general shell utilities. The source document groups these tools as part of a practical terminal toolkit for recurring developer operations.

Compared with plain Git commands, hosting CLIs typically support:
- account authentication and session setup
- hosted repository creation
- pull request-centric workflows
- direct API interaction without writing full client code
- command aliases for repeated operational patterns
- platform administration commands in GitLab environments

These patterns are closely related to [[concepts/git-hosting-cli-workflows]], [[concepts/cli-authentication-and-secrets]], and [[concepts/repository-hygiene]].

## Main workflow categories

### GitHub CLI workflows

The source set behind [[summaries/cli-command-management]] includes GitHub CLI tasks such as:
- authentication setup
- API requests
- alias configuration
- repository creation

These workflows support terminal-first repository management and reduce context switching between browser and shell. They fit well into broader [[concepts/command-line-workflows]] and can streamline day-to-day maintenance tasks.

### GitLab CLI and administration workflows

The source set also includes GitLab-oriented commands and `gitlab-ctl`, indicating both user-facing and administrative workflows. This expands the concept beyond developer convenience into service operations, environment management, and hosted platform maintenance.

This administrative angle overlaps with [[concepts/deployment-automation]], [[concepts/security-policy]], and [[concepts/shell-environments]] when CLIs are used in operational settings.

### Pull request and collaboration workflows

Some workflows sit between Git and hosting platforms, especially commands for pull requests and remote coordination. These are not just version-control operations; they are collaboration workflows layered on top of hosted repositories.

This makes the concept adjacent to [[concepts/version-control]] and useful in maintaining efficient team practices and cleaner repository operations.

## Why this concept matters

Git hosting CLI workflows matter because they:
- centralize repetitive hosting tasks in the terminal
- improve speed for common repository operations
- make automation easier than manual browser-driven steps
- support scriptable, reproducible team workflows
- reduce friction between local development and hosted collaboration

In practice, they are especially valuable for developers who already rely heavily on shell-based workflows and want a cohesive operational toolkit.

## Relationship to nearby concepts

- [[concepts/version-control]]: provides the underlying repository model these workflows build on.
- [[concepts/command-line-workflows]]: provides the broader terminal usage context.
- [[concepts/cli-authentication-and-secrets]]: important for auth, tokens, and secure noninteractive usage.
- [[concepts/repository-hygiene]]: hosting CLIs often support cleaner branch, PR, and repo maintenance.
- [[concepts/developer-productivity]]: these tools reduce overhead and support faster execution.
- [[concepts/shell-utility-tooling]]: complementary utilities often surround hosting CLI usage in real workflows.

## Source grounding

This concept is derived from [[summaries/cli-command-management]], which organizes command summaries into Git workflows, GitHub and GitLab CLI usage, and supporting shell utilities. The hosting-specific portion emphasizes GitHub auth, aliases, API access, repository creation, GitLab commands, and GitLab service control as a coherent family of terminal workflows.

See also: [[summaries/cli-commands]]

See also: [[summaries/cli-ftp]]

See also: [[summaries/cli-gh-create-repo]]

See also: [[summaries/cli-gh]]

See also: [[summaries/cli-github]]