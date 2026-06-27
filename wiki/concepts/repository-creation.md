---
sources: [summaries/cli-gh-create-repo.md]
brief: Creating hosted code repositories from local or CLI-based workflows.
---

# Repository Creation

Repository creation is the process of establishing a new code repository in a hosting platform and connecting it to a working development workflow. In this wiki, the concept is framed through [[summaries/cli-gh-create-repo]], which links Git-oriented notes with GitHub CLI–based repository setup.

## Overview

This concept sits at the intersection of local version-control practice and remote hosting setup. A repository is not just a storage location for code; it is also the anchor for collaboration, visibility, access control, and automation. Creating a repository often marks the transition from local work to a shareable, managed project.

In the source note [[summaries/cli-gh-create-repo]], repository creation is treated as a connective workflow topic rather than a standalone tutorial. The note primarily points readers toward related materials about Git and GitHub CLI usage, indicating that repository creation is part of a broader command-line development workflow.

## What the source document contributes

[[summaries/cli-gh-create-repo]] is a lightweight bridge document that:

- connects Git-focused material with GitHub-hosted repository setup
- points directly to related CLI guidance
- emphasizes navigation between local Git usage and remote repository creation

From that, this concept can be understood as involving two coordinated domains:

1. local repository preparation
2. remote repository provisioning

## Core workflow idea

Repository creation commonly includes steps such as:

- preparing a local project directory
- initializing or organizing Git history
- creating a remote repository on a hosting service
- linking the local repository to the remote
- pushing the initial contents

When done through command-line tools, this becomes part of broader [[concepts/command-line-workflows]] and [[concepts/git-hosting-cli-workflows]]. In the GitHub-specific case, it also belongs under [[concepts/github-cli]].

## Why this concept matters

Repository creation is a foundational enabling step for:

- publishing code
- starting new projects cleanly
- standardizing setup across projects
- enabling automation, review, and collaboration

It also connects naturally to [[concepts/repository-hygiene]], since the moment a repository is created is often when naming, structure, visibility, and initial documentation decisions are made.

## Relationship to adjacent concepts

- [[concepts/github-cli]]: Repository creation may be performed directly from the terminal using GitHub tooling.
- [[concepts/git-hosting-cli-workflows]]: Places repository creation within a larger set of hosted Git operations.
- [[concepts/command-line-workflows]]: Highlights terminal-first developer practices.
- [[concepts/repository-hygiene]]: Good repository setup often begins at creation time.

## Related page

- [[summaries/cli-gh-create-repo]]