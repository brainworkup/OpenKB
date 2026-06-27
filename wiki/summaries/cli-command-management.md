---
doc_type: short
full_text: sources/cli-command-management.md
---

# cli-command-management

A hub document that groups many CLI command summaries, primarily focused on day-to-day developer tooling. It acts as a navigation layer over command-specific notes rather than introducing a single standalone workflow.

## Scope

The document aggregates summaries for:
- Git commands and workflows: branching, cloning, fetching, committing, diffing, cherry-picking, pushing, pull request helpers, remote/tag renaming, and branch deletion
- GitHub CLI usage: auth, aliases, API access, and repository creation
- GitLab tooling: `gitlab` and `gitlab-ctl`
- General shell utilities: `ls`, `less`, `grep`, `prettier`, `http`, `gzip`, `gunzip`, `grpcurl`, and `fuck`

## Key idea

This page is best understood as a command-management index for recurring terminal tasks. Its main contribution is organizing operational knowledge across several related areas:
- command-line tooling
- Git workflows
- GitHub CLI
- [[concepts/developer-productivity]]

## Command clusters

### Git workflow cluster
The largest cluster covers common [[concepts/version-control]] operations:
- Repository setup and cloning
- Branch creation and deletion
- Commit and push workflows
- Fetching and synchronizing remotes
- Diff and difftool usage
- Cherry-picking changes
- Pull request related commands
- Remote and tag renaming

Related summaries include:
cli.git, cli.git status, cli.git setup, cli.git clone, cli.git create branch, cli.git delete branch, cli.git commit, cli.git push, cli.git fetch, cli.git diff, cli.git diff files, cli.git difftool, cli.git cherry pick, cli.git pr, cli.git rename remote, cli.git rename tag, cli.git cp.

### GitHub and GitLab CLI cluster
These notes point to hosted-forge automation and administration tasks:
- GitHub authentication and API access
- CLI aliasing and repo creation
- GitLab command-line and service control usage

Related summaries:
cli.gh auth, cli.gh api, cli.gh alias, cli.gh.create repo cli, cli.gitlab, cli.gitlab ctl.

### General utility cluster
These commands support navigation, formatting, inspection, compression, and requests:
- File listing and paging: cli.ls, cli.less
- Searching and formatting: cli.grep, cli.prettier
- HTTP and RPC interaction: cli.http, cli.grpcurl
- Compression: cli.gzip, cli.gunzip
- Command correction helpers: cli.fuck

## See also emphasis

The document explicitly highlights a few utility notes as related entry points:
- cli.gzip
- cli.http
- cli.less
- cli.ls
- cli.prettier

This suggests those tools may serve as especially common or useful anchors within the broader CLI toolkit.

## Practical interpretation

Use this page as a starting point when looking for:
- a remembered command but not its exact syntax
- a specific Git operation inside a larger workflow
- a CLI for interacting with GitHub or GitLab
- a common shell utility for viewing, formatting, querying, or compressing data

## Related concept candidates

Potential cross-document synthesis pages:
- [[concepts/command-line-workflows]]
- git workflows
- [[concepts/version-control]]
- [[concepts/git-hosting-cli-workflows]]
- [[concepts/shell-utility-tooling]]
- [[concepts/developer-productivity]]

## Related Concepts
- [[concepts/cli-authentication-and-secrets]]
- [[concepts/grpc]]
- [[concepts/python-networking]]
- [[concepts/repository-hygiene]]
- [[concepts/shell-environments]]
- [[concepts/terminal-output-formatting]]
