---
sources: [summaries/cli-github.md, summaries/cli-gh.md, summaries/cli-gh-create-repo.md]
brief: CLI tooling for managing GitHub repositories and workflows from the terminal.
---

# GitHub CLI

GitHub CLI refers to command-line tooling for interacting with GitHub directly from a terminal, including tasks such as repository creation, authentication, and routine hosted-repository operations. In this wiki, the concept is grounded by [[summaries/cli-gh-create-repo]], which connects repository creation workflows to broader command-line and Git-hosting practices.

## Overview

GitHub CLI supports a terminal-first workflow for developers who want to manage GitHub resources without leaving the shell. It fits naturally into broader [[concepts/command-line-workflows]] and overlaps with hosted Git operations described by [[concepts/git-hosting-cli-workflows]].

The core idea is that repository setup and management can be handled as part of a repeatable developer workflow rather than as a separate browser-based step.

## Role in the source material

[[summaries/cli-gh-create-repo]] is a short bridge page linking:

- [[summaries/cli-gh-create-repo]]
- related Git-oriented material in another summary page

From that connection, the main contribution to this concept is organizational:

- GitHub CLI is framed as a tool for creating repositories from the command line.
- It sits at the intersection of local Git usage and remote hosting workflows.
- It helps unify local project setup with remote publication and repository initialization.

## Key ideas

### Repository creation from the terminal

A central GitHub CLI use case is creating a new hosted repository without switching to the web UI. This aligns closely with [[concepts/repository-creation]] and makes repository setup part of a seamless shell-based process.

### Part of command-line workflows

GitHub CLI extends terminal-based development practices rather than replacing them. It belongs in a broader ecosystem of [[concepts/command-line-workflows]] and shell tooling where developers prefer automation, scripting, and reproducibility.

### Bridge between local and remote work

The concept connects local Git activity with remote GitHub operations. In practice, this means a repository can move from local development context to hosted collaboration context in one workflow, making GitHub CLI relevant to [[concepts/git-hosting-cli-workflows]].

### Authentication and access

Because hosted repository operations usually require credentials, GitHub CLI often depends on secure login and token handling patterns. This links the concept to [[concepts/cli-authentication-and-secrets]].

## Why this concept matters

GitHub CLI matters because it reduces friction in common development tasks:

- creating repositories
- connecting local projects to remote hosting
- standardizing terminal-first setup habits
- supporting automation-friendly workflows

For a knowledge base, it is a useful cross-document concept because it can tie together notes about shell usage, Git hosting, authentication, and repository lifecycle management.

## Related concepts

- [[concepts/command-line-workflows]]
- [[concepts/git-hosting-cli-workflows]]
- [[concepts/repository-creation]]
- [[concepts/cli-authentication-and-secrets]]

## Related summary

- [[summaries/cli-gh-create-repo]]

See also: [[summaries/cli-gh]]

See also: [[summaries/cli-github]]